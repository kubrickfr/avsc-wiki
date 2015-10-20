Displayed rates were computed over 100,000 records, taking the median of 10
runs. Percentages are relative to the fastest throughput for the schema. `-1`
values indicate that the library doesn't support the schema.

All benchmarks were run on a MacBook Air (1.7GHz Intel Core i7). The code is
available in `etc/benchmarks/`.


## Decoding

```
library          java-avro      node-avro-io     node-avsc      node-json     node-pson     python-fastavro
                       ops    %          ops   %       ops    %       ops   %       ops   %             ops   %
schema
ArrayString.avsc   1572343  100       185154  12   1157871   74   1069920  68    440246  28          142931   9
Bytes.avsc         3839125  100       402724  10   1739157   45    703637  18    856868  22          570367  15
Cake.avsc           455465  100        24184   5    254291   56    109935  24     47619  10           26006   6
Coupon.avsc        1539691  100        70007   5    735162   48    329162  21    116331   8           62949   4
Double.avsc        5571420  100       362415   6   5577026  100   1637616  29    854506  15          657486  12
Enum.avsc          3937008   59       487709   7   6685554  100   2166884  32    759262  11          563819   8
HistoryItem.avsc    843670   83        49520   5   1017480  100    201755  20    103391  10           45733   4
Human.avsc         1736158  100        95576   6    992677   57    382744  22    189637  11          117828   7
Int.avsc           4385965   34       481491   4  12713052  100   2333666  18    969277   8          549939   4
Long.avsc          4464642   42       471455   4  10590688  100   1994525  19    928425   9          392910   4
PciEvent.avsc       192108  100           -1  -1    102776   53     41157  21     21347  11              -1  -1
Person.avsc        2278200   28       178114   2   8161627  100    864446  11    329036   4          225475   3
String.avsc        3992159  100       349059   9   3648425   91   1880001  47    733841  18          494464  12
User.avsc           651297  100        26649   4    586422   90    118480  18     54455   8           20145   3
```

## Encoding

```
library          java-avro      node-avro-io    node-avsc      node-json      node-pson    python-fastavro
                       ops    %          ops  %       ops    %       ops    %       ops  %             ops   %
schema
ArrayString.avsc   1037346  100        19035  2    386042   37    961554   93     67991  7          126910  12
Bytes.avsc         1307210   96        89385  7   1364444  100    204150   15     83740  6          525417  39
Cake.avsc           217889  100           -1 -1    131827   61     57377   26      7310  3           15930   7
Coupon.avsc         636996  100         7130  1    435047   68    148491   23     35396  6           48941   8
Double.avsc        1427558   88       115952  7   1616590  100   1585870   98     92467  6          622871  39
Enum.avsc          1287856   57        66974  3   1472207   65   2249932  100     94179  4          466423  21
HistoryItem.avsc    400833   82           -1 -1    485907  100    120653   25     11049  2           34978   7
Human.avsc          860963  100        15258  2    525516   61    174984   20     33683  4           90949  11
Int.avsc           1336901   56        76182  3   1901947   80   2386869  100     97495  4          480061  20
Long.avsc          1397641   78        42585  2   1780712  100   1684788   95     91843  5          320403  18
PciEvent.avsc        77049  100           -1 -1     59505   77     20828   27      2588  3              -1  -1
Person.avsc         967637   85           -1 -1   1141252  100    969205   85     37005  3          164545  14
String.avsc        1306350   67        79841  4    946847   49   1947281  100     91662  5          397303  20
User.avsc           279803   70           -1 -1    399404  100     79595   20      6543  2           15835   4
```
