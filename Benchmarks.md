# Benchmarks

Displayed rates were computed over 100,000 records, taking the median of 10
runs. Percentages are relative to the fastest throughput for the schema. `-1`
values indicate that the library doesn't support the schema.

All benchmarks were run on a MacBook Air (1.7GHz Intel Core i7). The code is
available in the `benchmarks` folder of the main `avsc` repository.


## Decoding

```
library          java-avro      node-avro-io     node-avsc      node-json     python-fastavro
                       ops    %          ops   %       ops    %       ops   %             ops   %
schema
ArrayString.avsc   1444170  100       180434  12   1116972   77   1015760  70          137981   9
Bytes.avsc         3690489   99       400372  10   1455208   39    693466  18          554559  15
Cake.avsc           469595  100           -1  -1    137547   29    112013  23           26348   5
Coupon.avsc        1444104  100        71385   4    345052   23    337949  23           62860   4
Double.avsc        5509683  100       369948   6   5162606   93   1653687  30          679017  12
Enum.avsc          3899835   59       486224   7   6563342  100   2203458  33          569725   8
Human.avsc         1748257   99       100706   5    619877   35    392293  22          118434   6
Int.avsc           4464374   98       489672  10   4535021  100   2299179  50          549700  12
Long.avsc          4291924  100       419211   9   2278246   53   2024138  47          358293   8
Person.avsc        2275669  100           -1  -1   1766738   77    827108  36          194667   8
String.avsc        3335149   99       277700   8   2913463   87   1669694  50          411436  12
User.avsc           571625  100           -1  -1    119607   20    107774  18           18427   3
```

## Encoding

```
library          java-avro      node-avro-io    node-avsc      node-json      python-fastavro
                       ops    %          ops  %       ops    %       ops    %             ops   %
schema
ArrayString.avsc    750492   77        18392  1    355901   36    963256  100          120425  12
Bytes.avsc         1287214   97        85220  6   1320270  100    201507   15          497583  37
Cake.avsc           186441  100           -1 -1    127475   68     59808   32           15884   8
Coupon.avsc         788386  100         7121  0    442838   56    153475   19           49827   6
Double.avsc        1438851   89           -1 -1   1597551   99   1600710  100          617162  38
Enum.avsc               -1   -1        67312  2   1519959   67   2260376  100          468035  20
Human.avsc          932401  100        15996  1    521640   55    179512   19           91596   9
Int.avsc           1366161   57        76585  3   1817656   76   2384937  100          483512  20
Long.avsc          1349881   76        38350  2   1755053  100   1628519   92          321225  18
Person.avsc         915711   86           -1 -1   1056454  100    906174   85          149767  14
String.avsc        1210347   74        68354  4    756733   46   1630107   99          322277  19
User.avsc           253374   78           -1 -1    323148  100     74405   23           14354   4
```
