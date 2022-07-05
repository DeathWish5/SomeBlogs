## u/rCore-Tutorial-Code-2022S 分章节对比

#### ch1

* 几乎完全一致
  * 修改建议：无

#### ch2

* trampoline / trapframe 不同

  * ```rust
    pub struct TrapContext {
        pub x: [usize; 32],
        pub sstatus: Sstatus,
        pub sepc: usize,
    }
    ```

  * ```c
    struct trapframe {
    	/*   0 */ uint64 kernel_satp; // ucore 为了减少后续修改 trapframe 中直接有 satp
    	/*   8 */ uint64 kernel_sp;   // rCore 的 trapframe 存在内核栈上，kernel_sp = sscratch，uCore 则放在一个写死的位置 
    	/*  16 */ uint64 kernel_trap; // rCore 完成寄存器保存后跳转到固定地址，ucore 则跳转到这个地方
    	/*  24 */ uint64 epc; // 一致
    	/*  32 */ uint64 kernel_hartid; // uCore 假定用户态会修改 tp，rCore 假定不会，其实确实不会
    	/*  40 */ uint64 x[32]; // 一致
    };
    ```

  * 由于 trapframe 不同，trampoline 的细节也有点不一样

  * 修改建议：可以适当简化 uCore，多出的四个量都可以简化，但 satp 后续要加回来。不过其实无伤大雅，可以不改。

* rCore trapframe 在 kenrel stack 上，uCore 在一个写死的静态位置

  * 这导致了部分代码的逻辑不太一样，比如 rCore 初始化进程需要在 kstack 上 push 一个 trapframe 但是 uCore 就不需要。比如 uCore 需要专门储存 kernel_stack/trapframe 位置，rCore 只需要存一个。
  * 修改意见：其实两者都可以，可以 uCore 和 rCore 打一架，谁赢了听谁的。其实 trapframe 储存在栈上是 x86 的经典写法，这是由于硬件会自动在栈上 push 一些东西，但是 riscv 已经不需要了。所以我感觉 uCore 的写法更加容易理解，代码更简洁。建议打架的时候多叫几个人，打赢就行了。

* rCore 的模块化更好

  * ```rust
    // rcore 使用一个结构体来管理 task
    struct AppManager {
        num_app: usize,
        current_app: usize,
        app_start: [usize; MAX_APP_NUM + 1],
    }
    ```

  * ```c
    // uCore 使用一些全局变量来管理进程
    static int app_cur, app_num;
    static uint64 *app_info_ptr;
    extern char _app_num[], userret[], boot_stack_top[], ekernel[];
    ```

  * 修改意见：可以使用一个结构体管理起来

* rCore 的全局变量由于语言要求有 refcell 作包装，uCore 直接访问

  * 在单核 / 无内核中断的情况下，都是安全的
  * 修改意见：不需要修改，rust 语言特色

* rCore 的目录结构更好，uCore 没有目录

  * 修改意见：一个目录确实有点暴力，推荐增加合适的目录结构，可以参考 [uCore-SMP](https://github.com/TianhuaTao/uCore-SMP)，但需要注意目录名和文件名与后续一致，否则将带来实验 merge 难度的大幅度提升（文件名不同会导致 git merge 几乎失效，只能手动进行很难收）。

#### ch3

* 继承 ch2 的所有问题，修改意见不变

  * 尤其是模块化的问题

  * 但是个人感觉 rCore 的模块化太过了，增加了不少代码复杂性，uCore 简单增加一层壳子就好了。（可以看看 [nimbos](https://github.com/rvm-rtos/nimbos), 贾爷的设计更加类似 uCore）


* 重大差异：rCore 引入了堆分配器

  * ```rust
    #[global_allocator]
    /// heap allocator instance
    static HEAP_ALLOCATOR: LockedHeap = LockedHeap::empty();
    ```

  * 这是一个 buddy_system 分配器，然而 uCore 自始至终都没有堆分配器，全靠静态开大数组。这导致了 uCore 进程存在上限等一些问题。

  * 修改建议：增加简单的堆分配器，实现 `kmalloc` 和 `kfree`，可以参考[老ucore](https://github.com/chyyuu/ucore_os_plus/blob/clang-dev/ucore/src/kern-ucore/mm/slab.c#L411)。但不一定需要实现 slab 这种复杂的分配。

* rCore 的函数封装更多，其实这个问题在 ch2 也有

  * 比如初始化 task_context, rCore 在初始化进程的时候使用一个函数完成，uCore 则在初始化进程的时候直接在写了三行代码，两者等价，但 rCore 花里胡哨的而且代码行数很多，而且跨多个文件。

  * ```rust
    pub fn goto_restore(kstack_ptr: usize) -> Self {
        extern "C" {
            fn __restore();
        }
        Self {
            ra: __restore as usize,
            sp: kstack_ptr,
            s: [0; 12],
        }
    }
    
    fn init() {
        t.task_cx = TaskContext::goto_restore(init_app_cx(i));
    }
    ```

  * ```c
    int init() {
        memset(&p->context, 0, sizeof(p->context));
        p->context.ra = (uint64)usertrapret;
        p->context.sp = p->kstack + PAGE_SIZE;
    }
    ```

  * 修改意见：很难说那个好那个坏，rCore 的写法让大家更加清楚自己在干啥，模块化更好，但是也导致了 rCore 后期封装过多而导致代码反而更难读懂的现象。所以我建议，uCore 仍保持简洁的风格，通过增加注释来让同学们看的更懂。更多的函数封装不一定是好事。

* 调度的具体逻辑不同：uCore 有 idle 进程，rCore 没有，但是 rCore 在 lab4 又加回去了
  * uCore 的调度：
    * idle 找一个就绪进程，然后切换过去
    * 进程结束后，切回 idle
    * idle 重复步骤 1
  * rCore 的调度：
    * 无 idle 进程，每个进程结束后，找到下一个进程然后直接切换过去
  * 修改意见：rCore 后来也反悔了，idle 是更加经典的写法，建议不改

#### ch4

* rCore 的空闲页面使用 bitmap 管理，uCore 使用一个链表结构
  * 修改意见：两者都可以，但是还是推荐统一，建议 uCore 作者辛苦一下写个 bitmap_alloctor，这个能力更强。
* rCore 有 memory_set 管理页表，uCore 页表裸奔
  * 修改意见：辛苦一下吧。。没有 memory_set 这个结构无法实现 page_fault 和延迟处理啥的，这个理论是一个正经 os 必须实现的
  * 而相应增加的接口也需要设计者好好思考怎么写

