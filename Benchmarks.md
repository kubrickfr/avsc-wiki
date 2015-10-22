Displayed rates were computed over 150,000 records, taking the median of 15
runs. Percentages are relative to the fastest throughput for the schema. `-1`
values indicate that the library doesn't support the schema.

All benchmarks were run on a MacBook Air (1.7GHz Intel Core i7). The code is
available in `etc/benchmarks/`.


## Decoding

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

## Encoding

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
