# Test Data

###### 1

1000 * 1000 matrix

* prepare: 开销很小 : 0.0075

  ```
           5,444      dTLB-load-misses                                            
           8,547      iTLB-load-misses                                            
               0      cs                                                          
         352,475      cache-misses                                                
  
     0.007538944 seconds time elapsed
  
     0.007541000 seconds user
     0.000000000 seconds sys
  ```

* base: 一个函数搞定(3 个 for)：delta = 1.470274064 seconds  (173 / 141 = 1.22  1.79 / 1.47 = 1.21)

  ```
      14,330,672,706      dTLB-load-misses                                              (44.44%)
             289,321      iTLB-load-misses                                              (44.45%)
                  18      cs                                                          
      61,424,749,821      L1-dcache-load-misses                                         (44.45%)
          11,483,224      L1-icache-load-misses                                         (44.45%)
       1,733,808,387      LLC-load-misses                                               (44.45%)
           1,241,992      LLC-store-misses                                              (44.45%)
          50,908,563      branch-load-misses                                            (44.44%)
          50,894,817      branch-misses                                                 (44.44%)
       2,827,721,190      cache-misses                                                  (44.44%)
  ```

* base2: 1000 * 1000 func call: FUNC: TIMES 50 delta = 1.798418764 seconds

  ```
      13,584,313,529      dTLB-load-misses                                              (44.44%)
             314,823      iTLB-load-misses                                              (44.45%)
                  22      cs                                                          
      60,959,313,049      L1-dcache-load-misses                                         (44.45%)
          11,664,459      L1-icache-load-misses                                         (44.45%)
       1,408,543,354      LLC-load-misses                                               (44.45%)
           1,101,588      LLC-store-misses                                              (44.44%)
          51,047,308      branch-load-misses                                            (44.44%)
          51,036,621      branch-misses                                                 (44.44%)
       3,064,375,948      cache-misses                                                  (44.44%)
  ```

* base3: 1000 * 1000 * 10 func call  delta =7.471266558 seconds

           1,288,150,113      dTLB-load-misses                                              (44.44%)
                 143,595      iTLB-load-misses                                              (44.44%)
                       9      cs                                                          
           6,126,855,320      L1-dcache-load-misses                                         (44.44%)
               5,975,427      L1-icache-load-misses                                         (44.44%)
             168,677,918      LLC-load-misses                                               (44.45%)
                 333,305      LLC-store-misses                                              (44.45%)
               5,447,178      branch-load-misses                                            (44.45%)
               5,449,549      branch-misses                                                 (44.45%)
             525,474,462      cache-misses                                                  (44.45%)

* 1000 threads / a row => 1.7083 ～ 1.84  / 2002000 == base2

  ```
      16,073,828,538      dTLB-load-misses                                              (44.44%)
             587,748      iTLB-load-misses                                              (44.44%)
                  21      cs                                                          
      62,872,265,286      L1-dcache-load-misses                                         (44.44%)
          13,860,073      L1-icache-load-misses                                         (44.45%)
          55,951,593      LLC-load-misses                                               (44.45%)
          18,591,178      LLC-store-misses                                              (44.45%)
         151,340,305      branch-load-misses                                            (44.45%)
         151,360,667      branch-misses                                                 (44.45%)
       2,908,613,540      cache-misses                                                  (44.44%)
  ```

* 1000 * 1000 threads / a val => 2.04 ~ 2.18 / 2000000

  ```
      15,304,211,801      dTLB-load-misses                                              (44.44%)
             750,869      iTLB-load-misses                                              (44.44%)
                  25      cs                                                          
      63,015,458,252      L1-dcache-load-misses                                         (44.44%)
          14,924,580      L1-icache-load-misses                                         (44.45%)
         914,543,901      LLC-load-misses                                               (44.45%)
         197,409,708      LLC-store-misses                                              (44.45%)
         151,398,371      branch-load-misses                                            (44.45%)
         151,286,250      branch-misses                                                 (44.44%)
       3,346,189,562      cache-misses                                                  (44.44%)
  ```

* 1 coroutine 1000 * 1000 coroutine await: 1.95 ~ 2.14

  ```
      14,066,612,064      dTLB-load-misses                                              (44.44%)
             324,898      iTLB-load-misses                                              (44.44%)
                  26      cs                                                          
      61,837,028,637      L1-dcache-load-misses                                         (44.44%)
          18,807,647      L1-icache-load-misses                                         (44.44%)
         908,656,238      LLC-load-misses                                               (44.45%)
           1,123,789      LLC-store-misses                                              (44.45%)
          51,232,988      branch-load-misses                                            (44.45%)
          51,246,158      branch-misses                                                 (44.45%)
       2,261,484,886      cache-misses                                                  (44.44%)
  ```

* 1000 coroutine:  1.542314480

  ```
      13,725,951,550      dTLB-load-misses                                              (44.44%)
             224,432      iTLB-load-misses                                              (44.44%)
                 433      cs                                                          
      62,144,073,439      L1-dcache-load-misses                                         (44.45%)
          12,597,244      L1-icache-load-misses                                         (44.45%)
          43,333,693      LLC-load-misses                                               (44.45%)
          10,279,469      LLC-store-misses                                              (44.45%)
          50,927,232      branch-load-misses                                            (44.45%)
          50,934,657      branch-misses                                                 (44.44%)
       1,856,133,048      cache-misses                                                  (44.43%)
  ```

  

