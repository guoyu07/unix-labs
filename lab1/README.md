## Unix Lab 1

> 13302010039 童仲毅

### Project structure

This is a Clion project. Make sure you are using `CMakeLists` to build the project.

- `main.c`: Functional tests for five functions.
- `my_fread.h`, `my_fread.c`: Self implemented `fread` function.
- `timer.h`, `timer.c`: Provide `timer_start` and `timer_end` to calculate the execution time of a block of code, format and output.
- `CMakeLists.txt`: CMake configurations.

To test read functions, put `data` under the root of the project. The data below are collected in OS X.

### IO efficiency of UNIX `read()` function

| BUFSIZE |   real    |   user    |  system   |  clocks  | iterations |
| :-----: | :-------: | :-------: | :-------: | :------: | :--------: |
|    2    | 78.480000 | 12.170000 | 65.770000 | 77945482 | 163184640  |
|    4    | 40.230000 | 6.150000  | 33.650000 | 39801693 |  81592320  |
|    8    | 19.960000 | 3.060000  | 16.790000 | 19853185 |  40796160  |
|   16    | 10.170000 | 1.540000  | 8.540000  | 10074817 |  20398080  |
|   32    | 5.250000  | 0.780000  | 4.350000  | 5134962  |  10199040  |
|   64    | 2.690000  | 0.400000  | 2.220000  | 2614420  |  5099520   |
|   128   | 1.310000  | 0.190000  | 1.090000  | 1281768  |  2549760   |
|   256   | 0.700000  | 0.100000  | 0.590000  |  691418  |  1274880   |
|   512   | 0.420000  | 0.050000  | 0.350000  |  401278  |   637440   |
|  1024   | 0.240000  | 0.020000  | 0.200000  |  228241  |   318720   |
|  2048   | 0.230000  | 0.020000  | 0.160000  |  169409  |   159360   |
|  4096   | 0.270000  | 0.010000  | 0.130000  |  142003  |   79680    |
|  8192   | 0.170000  | 0.000000  | 0.140000  |  143774  |   39840    |
|  16384  | 0.170000  | 0.000000  | 0.120000  |  122419  |   19920    |
|  32768  | 0.090000  | 0.000000  | 0.060000  |  65810   |    9960    |
|  65536  | 0.100000  | 0.000000  | 0.070000  |  69254   |    4980    |
| 131072  | 0.110000  | 0.110000  | 0.100000  |  101988  |    2490    |
| 262144  | 0.160000  | 0.000000  | 0.120000  |  117981  |    1245    |
| 524288  | 0.060000  | 0.000000  | 0.060000  |  56166   |    623     |

When we double the `BUFSIZE`, the execution time is cut half down. But when `BUFSIZE` reaches 4096 bytes, it starts to vary slowly. After that, the duration has some thrashing, but shows an overall reduction.

According to diskutil, OS X's `Allocation Block Size` is 4096 bytes, which explains the result. When `BUFSIZE` is not large, there are a lot of time consumption by frequent system calls. Thus the execution time is corresponded to times of iteration. When `BUFSIZE` is large, most of the time it is doing pure IO operations and system call contribution is trival.

### IO efficiency of UNIX `fread()` function

| BUFSIZE |   real   |   user   |  system  | clocks  | iterations |
| :-----: | :------: | :------: | :------: | :-----: | :--------: |
|    2    | 7.840000 | 7.570000 | 0.130000 | 7707408 | 163184640  |
|    4    | 4.000000 | 3.830000 | 0.120000 | 3942748 |  81592320  |
|    8    | 1.950000 | 1.850000 | 0.090000 | 1936403 |  40796160  |
|   16    | 1.050000 | 0.950000 | 0.090000 | 1044974 |  20398080  |
|   32    | 0.570000 | 0.480000 | 0.090000 | 570474  |  10199040  |
|   64    | 0.370000 | 0.260000 | 0.100000 | 364834  |  5099520   |
|   128   | 0.210000 | 0.130000 | 0.080000 | 205984  |  2549760   |
|   256   | 0.210000 | 0.080000 | 0.120000 | 199000  |  1274880   |
|   512   | 0.160000 | 0.050000 | 0.110000 | 155809  |   637440   |
|  1024   | 0.130000 | 0.030000 | 0.100000 | 134266  |   318720   |
|  2048   | 0.120000 | 0.030000 | 0.090000 | 110432  |   159360   |
|  4096   | 0.140000 | 0.020000 | 0.110000 | 139600  |   79680    |
|  8192   | 0.080000 | 0.010000 | 0.070000 |  79474  |   39840    |
|  16384  | 0.090000 | 0.010000 | 0.070000 |  81457  |   19920    |
|  32768  | 0.200000 | 0.010000 | 0.090000 |  92887  |    9960    |
|  65536  | 0.100000 | 0.000000 | 0.060000 |  58818  |    4980    |
| 131072  | 0.130000 | 0.110000 | 0.090000 |  92524  |    2490    |
| 262144  | 0.140000 | 0.000000 | 0.110000 | 112197  |    1245    |
| 524288  | 0.050000 | 0.000000 | 0.050000 |  50746  |    623     |

`fread` is 10 times faster than `read`, becasue it adpots the idea of "read ahead". It reads a large block of data into memory and when we try to read a few bytes at a time, it actually returns cached data from memory. This approach signaficantly reduces consumption of system calls.

### IO efficiency of UNIX `write()` function without `O_SYNC` flag

| BUFSIZE |    real    |   user    |  system   |  clocks   | iterations |
| :-----: | :--------: | :-------: | :-------: | :-------: | :--------: |
|    2    | 111.410000 | 13.900000 | 96.490000 | 110392938 | 163315712  |
|    4    | 55.320000  | 6.860000  | 48.070000 | 54929186  |  81657856  |
|    8    | 27.570000  | 3.470000  | 23.920000 | 27384844  |  40828928  |
|   16    | 13.470000  | 1.720000  | 11.710000 | 13431724  |  20414464  |
|   32    |  6.930000  | 0.850000  | 5.960000  |  6820762  |  10207232  |
|   64    |  3.340000  | 0.430000  | 2.920000  |  3337348  |  5103616   |
|   128   |  1.700000  | 0.220000  | 1.470000  |  1688509  |  2551808   |
|   256   |  0.850000  | 0.100000  | 0.740000  |  851720   |  1275904   |
|   512   |  0.460000  | 0.060000  | 0.400000  |  454129   |   637952   |
|  1024   |  0.260000  | 0.020000  | 0.230000  |  249993   |   318976   |
|  2048   |  0.120000  | 0.020000  | 0.110000  |  126332   |   159488   |
|  4096   |  0.080000  | 0.000000  | 0.060000  |   70458   |   79744    |
|  8192   |  0.060000  | 0.010000  | 0.060000  |   60456   |   39872    |
|  16384  |  0.040000  | 0.000000  | 0.040000  |   40732   |   19936    |
|  32768  |  0.050000  | 0.000000  | 0.050000  |   53560   |    9968    |
|  65536  |  0.040000  | 0.000000  | 0.040000  |   37433   |    4984    |
| 131072  |  0.040000  | 0.000000  | 0.030000  |   36842   |    2492    |
| 262144  |  0.040000  | 0.000000  | 0.050000  |   41872   |    1246    |
| 524288  |  0.050000  | 0.000000  | 0.040000  |   47650   |    623     |

In this case, I write a malloced memory space to the disk. Similar to `read`, execution time is cut half down when`BUFSIZE` doubles. The reason is also similiar and so abbreviated.

### IO efficiency of UNIX `write()` function with `O_SYNC` flag

| BUFSIZE |   real    |   user   |  system  | clocks  | iterations |
| :-----: | :-------: | :------: | :------: | :-----: | :--------: |
|  1024   | 17.730000 | 0.050000 | 3.240000 | 3291246 |   318976   |
|  2048   | 8.650000  | 0.020000 | 1.620000 | 1643106 |   159488   |
|  4096   | 4.150000  | 0.010000 | 0.790000 | 796504  |   79744    |
|  8192   | 2.570000  | 0.010000 | 0.430000 | 434440  |   39872    |
|  16384  | 1.700000  | 0.000000 | 0.260000 | 268467  |   19936    |
|  32768  | 1.240000  | 0.000000 | 0.170000 | 172203  |    9968    |
|  65536  | 1.020000  | 0.010000 | 0.130000 | 130718  |    4984    |
| 131072  | 1.140000  | 0.000000 | 0.170000 | 168924  |    2492    |
| 262144  | 0.820000  | 0.000000 | 0.150000 | 151838  |    1246    |
| 524288  | 0.620000  | 0.000000 | 0.090000 |  94931  |    623     |

With `O_SYNC` flag, the execution is extremely slow. So I start with `BUFSIZE` of 1024 than 2, however the we can also see the general trend. `O_SYNC` guarantees that the call will not return before all data has been transferred to the disk.

### IO efficiency of my own `my_fread` function

| BUFSIZE |   real   |   user   |  system  | clocks  | iterations |
| :-----: | :------: | :------: | :------: | :-----: | :--------: |
|    2    | 2.130000 | 1.980000 | 0.080000 | 2058709 | 163184640  |
|    4    | 1.050000 | 0.980000 | 0.050000 | 1035738 |  81592320  |
|    8    | 0.500000 | 0.450000 | 0.050000 | 496630  |  40796160  |
|   16    | 0.280000 | 0.220000 | 0.060000 | 274174  |  20398080  |
|   32    | 0.170000 | 0.120000 | 0.040000 | 165773  |  10199040  |
|   64    | 0.120000 | 0.070000 | 0.050000 | 119146  |  5099520   |
|   128   | 0.080000 | 0.030000 | 0.050000 |  78587  |  2549760   |
|   256   | 0.060000 | 0.020000 | 0.040000 |  63146  |  1274880   |
|   512   | 0.080000 | 0.020000 | 0.060000 |  79329  |   637440   |
|  1024   | 0.070000 | 0.010000 | 0.050000 |  62022  |   318720   |
|  2048   | 0.060000 | 0.010000 | 0.060000 |  62891  |   159360   |
|  4096   | 0.060000 | 0.010000 | 0.040000 |  52841  |   79680    |
|  8192   | 0.070000 | 0.010000 | 0.060000 |  65990  |   39840    |
|  16384  | 0.060000 | 0.010000 | 0.050000 |  61280  |   19920    |
|  32768  | 0.070000 | 0.020000 | 0.050000 |  66841  |    9960    |
|  65536  | 0.080000 | 0.010000 | 0.060000 |  79382  |    4980    |
| 131072  | 0.080000 | 0.020000 | 0.060000 |  77062  |    2490    |
| 262144  | 0.070000 | 0.010000 | 0.060000 |  69200  |    1245    |
| 524288  | 0.080000 | 0.020000 | 0.060000 |  76453  |    622     |

My implementation is much faster than the `fread` implementation. I think it's because I use a much larger `BUFSIZE`. The idea is to adopt the `read ahead` approach, and cache a large block data in memory and prepare for future use.