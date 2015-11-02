Displayed rates were computed over 150,000 records, taking the median of 15
runs. Percentages are relative to the fastest throughput for the schema. `-1`
values indicate that the library doesn't support the schema.

All benchmarks were run on a MacBook Air (1.7GHz Intel Core i7). The code is
available in `etc/benchmarks/`.


## Various implementations

As of `2.2.0` (0b47aef).

### Decoding

```
library          java-avro      node-avro-io     node-avsc      node-etp-avro     node-json     node-pson     python-fastavro
                       ops    %          ops   %       ops    %           ops   %       ops   %       ops   %             ops   %
schema
ArrayString.avsc   2027027  100       192593  10   1264461   62        230202  11   1042568  51    448663  22          144721   7
Bytes.avsc         4559271  100       413293   9   1812154   40        592947  13    699662  15    850280  19          568996  12
Cake.avsc           455235  100        22387   5    251962   55            -1  -1    110073  24     47855  11           26412   6
Coupon.avsc        1940492  100        63688   3    773569   40        112527   6    336135  17    116417   6           62572   3
Double.avsc        5976096   89       380770   6   6743172  100        561099   8   1773422  26    809907  12          659664  10
Enum.avsc          4702194   28       508677   3  16668758  100        509914   3   2279500  14    777190   5          556952   3
HistoryItem.avsc   1036628  100        50045   5    994382   96            -1  -1    205770  20    103970  10           45748   4
Human.avsc         2358491  100        96249   4    950407   40        165736   7    387177  16    190994   8          117193   5
Int.avsc           5119454   31       498767   3  16274084  100        597629   4   2520045  15   1013789   6          560738   3
Long.avsc          5357143   42       456571   4  12670971  100        587062   5   2205832  17    931866   7          391230   3
PciEvent.avsc       193673  100           -1  -1    101120   52            -1  -1     41310  21     21227  11              -1  -1
Person.avsc        2767528   27       181177   2  10117772  100        293385   3    869475   9    340546   3          223639   2
String.avsc        4792332  100       362911   8   3788316   79        421641   9   2029504  42    744025  16          486926  10
User.avsc           623701  100        26324   4    569065   91         53566   9    116774  19     54745   9           20061   3
```

### Encoding

```
library          java-avro      node-avro-io    node-avsc      node-etp-avro     node-json      node-pson    python-fastavro
                       ops    %          ops  %       ops    %           ops   %       ops    %       ops  %             ops   %
schema
ArrayString.avsc    817884   90        20072  2    399605   44        236206  26    909724  100     71178  8          125307  14
Bytes.avsc         1460565  100        61486  4   1340199   92        589985  40    198248   14     82836  6          427232  29
Cake.avsc           132638  100           -1 -1    130551   98            -1  -1     57293   43      7520  6           14707  11
Coupon.avsc         612495  100         7019  1    452894   74            -1  -1    144950   24     36252  6           47567   8
Double.avsc        1473477   90       116066  7   1640648  100        656834  40   1584090   97     92954  6          507559  31
Enum.avsc          1521298   66        67208  3   1537230   67        672624  29   2310191  100     94781  4          379254  16
HistoryItem.avsc    327368   65           -1 -1    503480  100            -1  -1    115884   23     11249  2           32272   6
Human.avsc          754907  100        15612  2    559688   74        254789  34    170569   23     33862  4           78273  10
Int.avsc           1500000   62        75397  3   1941959   81        702567  29   2409767  100     98595  4          390308  16
Long.avsc          1533742   83        33722  2   1839793  100            -1  -1   1741366   95     94302  5          275998  15
PciEvent.avsc        63089  100           -1 -1     58533   93            -1  -1     20785   33      2542  4              -1  -1
Person.avsc        1166407   99           -1 -1   1176498  100        421560  36    932495   79     36997  3          152053  13
String.avsc        1445087   76        55757  3    974764   51        478458  25   1901735  100     92493  5          345463  18
User.avsc           147246   37           -1 -1    396437  100            -1  -1     77232   19      6585  2           15410   4
```


## Avro implementations

As of `3.0.0` (7bc3503).

### Decoding

Throughput rates for different schemas (records per second, higher is better):

![Decoding](https://raw.githubusercontent.com/mtth/avsc/3.0/etc/benchmarks/charts/avro-decode-throughput-7bc3503.png)

```
library          java-avro      node-avsc      python-avro    ruby-avro
                       ops    %       ops    %         ops  %       ops  %
schema
ArrayString.avsc   2000000  100   1266530   63       21164  1     58700  3
Bytes.avsc         4601227  100   1801376   39       60665  1    217912  5
Cake.avsc           452216  100    256442   57        2974  1      7683  2
Coupon.avsc        1877347  100    769071   41        8908  0     22245  1
Double.avsc        6198347   95   6532007  100       40843  1    229785  4
Enum.avsc          4792332   29  16794592  100       53190  0    199045  1
HistoryItem.avsc   1023891  100   1001385   98        4952  0     14898  1
Human.avsc         2377179  100    968227   41       14835  1     42146  2
Int.avsc           4901961   30  16257795  100       60006  0    190781  1
Long.avsc          5300353   42  12539456  100       52885  0    153929  1
PciEvent.avsc       196104  100    101511   52        1285  1      3527  2
Person.avsc        2846300   27  10644791  100       21589  0     66213  1
String.avsc        4901961  100   3761433   77       62766  1    207224  4
Union.avsc         3768844   58   6476772  100       37964  1    135492  2
User.avsc           638570  100    570437   89        2488  0      5868  1
```

### Encoding

Throughput rates for different schemas (records per second, higher is better):

![Encoding](https://raw.githubusercontent.com/mtth/avsc/3.0/etc/benchmarks/charts/avro-encode-throughput-7bc3503.png)

```
library          java-avro      node-avsc      python-avro    ruby-avro
                       ops    %       ops    %         ops  %       ops   %
schema
ArrayString.avsc    809935  100    394226   49       23023  3     84439  10
Bytes.avsc         1498501  100   1356937   91       66168  4    145516  10
Cake.avsc           136799  100    132264   97        3732  3      5867   4
Coupon.avsc         669643  100    400925   60       12057  2     13894   2
Double.avsc        1564129   95   1651538  100       54656  3    160231  10
Enum.avsc          1479290  100   1320349   89       67877  5    287977  19
HistoryItem.avsc    349650   90    388593  100        6724  2      8363   2
Human.avsc          746640  100    511139   68       18434  2     34216   5
Int.avsc           1518219   78   1940986  100       83612  4    129737   7
Long.avsc          1559252   85   1824444  100       67836  4     72847   4
PciEvent.avsc        64097  100     59451   93        1482  2      2689   4
Person.avsc        1139818  100   1044261   92       27804  2     61715   5
String.avsc        1477833  100    975808   66       66754  5    271025  18
Union.avsc         1262626  100   1075132   85       44135  3    130373  10
User.avsc           153453   50    305900  100        2886  1      3104   1
```
