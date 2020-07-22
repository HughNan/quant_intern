### Weight

####介绍

用来对ETF复制的沪深300指数进行权重更新，基本的原理是：用每个ETF覆盖的行业对应的股票相加，即为该ETF的权重，对应的股票从组合中删除。

在每年的6月与12月，沪深300会调整成分股，在某些特殊情况发生时，如退市、重组等，成分股也会进行调整。在调整成分股当天更新权重，分为两步，首先，更新成分股的权重；其次，使用ETF代替其覆盖行业的股票。

成分股权重更新有两种方法：

1. 仅更新调入股票的权重，使调入股票权重之和等于调出股票权重之和（部分调整）。
2. 对调整后的所有成分股权重全部更新（全部调整）。

不论是哪种方法，根据沪深300的计算规则，更新时均使用股本自由流比例分级靠挡计算其比例。故计算时需要使用这些股票参数：上一交易日的自由流通股本，A股总股本，收盘价。区别在于，仅更新调入股票权重时，仅需要调入股票的这些数据；而全部更新时，需要全部股票的数据。

在非成分股调整日期时，计算权重仅进行第二步，即使用ETF代替其覆盖行业的股票。

**对于成分股的定期调整，建议全部调整，因为部分调整时调入股票的权重与真实相差较大。对于特殊情况的成分股调整，因为调整比例很小，通常仅有1～2支股票调整，建议部分调整。**



#### 数据

更新权重需要以下数据，未特殊说明均由.csv文件提供。

* **基础权重**：在此基础上进行权重更新，一般为上一交易日权重。
* **股票行业**：股票所属的行业 。
* **基金行业覆盖**：基金覆盖的行业及起始交易日期。
* **股票股本及行情信息**：更新日上一交易日的股票自由流通股本，A股总股本及收盘价。若部分更新则仅包含调入股票；若全部更新则为全部股票。成分股未发生改变时，不需要此数据。
* **调出股票代码**：list，包含调出股票的Wind代码。成分股未发生改变，或全部调整时，不需要此数据。



#### 使用方法

##### 函数

$\text{parse_coverage(), parse_stock_industry(), parse_weight(), parse_in_stocks{}}$是分别用来预处理基金行业覆盖、股票行业、基础权重和股票股本行情信息的函数，将.csv文件读入，并将字段名转为标准字段名。

| 字段              | 标准字段名   | 备注                      |
| ----------------- | ------------ | ------------------------- |
| 股票/ETF Wind代码 | code         |                           |
| ETF行业覆盖       | coverage     | 不同行业用","分开，无空格 |
| ETF开始交易日期   | start_date   |                           |
| 股票行业          | industry     |                           |
| 权重              | weight       |                           |
| 自由流通股本      | free_shares  |                           |
| A股总股本         | total_shares |                           |
| 收盘价            | close        |                           |

$\text{proc_in_stocks(), update_constitution()}$用于计算股票权重及更新，$\text{get_weight()}$用ETF替换覆盖行业的股票。$\text{update_weight()}$则将所有函数集成，**使用时直接调用$\text{update_weight()}$即可**。

##### 参数

$\text{update_weight()}$包含以下参数

| 参数                | 类型    | 默认值         | 说明                                                         |
| ------------------- | ------- | -------------- | ------------------------------------------------------------ |
| date                | Int/Str |                | 更新日，%Y%m%d格式，如20170101                               |
|                     |         |                |                                                              |
| fund_cov_path       | Str     |                | .csv文件路径，ETF的行业覆盖及开始日期                        |
| fund_code_col       | Str     | 'code'         | ETF Wind代码字段名                                           |
| fund_cov_col        | Str     | 'coverage'     | ETF 行业覆盖字段名                                           |
| fund_cov_separator  | Str     | ','            | ETF 行业覆盖中不同行业的分隔符                               |
| fund_start_date_col | Str     | 'start_date'   | ETF 开始交易日期                                             |
|                     |         |                |                                                              |
| stock_ind_path      | Str     |                | .csv文件路径，股票行业                                       |
| stock_code_col      | str     | 'code'         | 股票Wind代码字段名                                           |
| stock_ind_col       | Str     | 'industry'     | 股票行业字段名                                               |
|                     |         |                |                                                              |
| base_weight_path    | str     | None           | .csv文件路径，基础权重，基于此进行权重调整。在成分股调整日且全部调整时，不需要此数据。 |
| weight_code_col     | Str     | 'code'         | 股票Wind代码字段名                                           |
| weight_col          | Str     | 'weight'       | 股票权重字段名                                               |
|                     |         |                |                                                              |
| in_stocks_path      | str     | None           | .csv文件路径，更新日前一交易日的股票股本及行情信息。非权重调整日不需要此数据。当部分调整时，仅包含调入股票信息，当全部调整时，包含全部300支股票信息。 |
| in_stocks_code_col  | Str     | 'code'         | 股票Wind代码字段名                                           |
| free_shares_col     | Str     | 'free_shares'  | 股票自由流动股本字段名                                       |
| total_shares_col    | Str     | 'tatal_shares' | 股票A股总股本字段名                                          |
| close_col           | Str     | 'close'        | 股票收盘价字段名                                             |
|                     |         |                |                                                              |
| update_all          | Bool    | True           | 是否全部更新，在非成分股调整日，此参数不产生影响。           |
| out_list            | List    | None           | 调出股票Wind代码列表，仅在成分股调整日且部分调整时需要。     |

**由于参数较多，可以提前在函数的默认参数中将字段名设置为合适值，之后使用无需再传入这些参数。**

对于参数较多造成的调用麻烦，建议先创建参数字段，再传入，如下例所示，

```python
attr_dict = {
    'date': '20200701',
    'fund_cov_path': 'data/fund_industry.csv',
    'stock_ind_path': 'data/stock_industry.csv',
    'base_weight_path': 'data/000300.SH/20200630.csv',
    'in_stocks_path': None,

    'fund_code_col': 'code',
    'fund_cov_col': 'coverage',
    'fund_cov_separator': ',',
    'fund_start_date_col': 'start_date',

    'stock_code_col': 'code',
    'stock_ind_col': 'industry',

    'weight_code_col': 'TICKER',
    'weight_col': 'WEIGHT',

    'in_stocks_code_col': 0,
    'free_shares_col': 'FREE_FLOAT_SHARES',
    'total_shares_col': 'SHARE_TOTALTRADABLE',
    'close_col': 'CLOSE',

    'update_all': True,
    'out_list': None
}

update_weight(**attr_dict)
```

