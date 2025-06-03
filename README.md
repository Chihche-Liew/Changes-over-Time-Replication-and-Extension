# Replication Process and Results of Donelson et al., 2011

This project replicates and extends the empirical results of Donelson et al. (2011) using Compustat data from WRDS. It covers the full replication pipeline from data extraction, preprocessing, and variable construction, to producing key empirical tables on expense volatility and earnings persistence. The replication not only validates the original study (1967-2005) but also extends the analysis to more recent periods (up to 2023) to test the robustness of the findings.

## 00. Downloading Data from WRDS

**NOTE:** The private access to WRDS of user is required.

```
SELECT gvkey, datadate, fyear, at, sale, ib, cogs, xsga, txt, dp, oiadp, spi
FROM comp.funda
WHERE indfmt = 'INDL'
AND datafmt = 'STD'
AND popsrc = 'D'
AND consol = 'C'
AND fyear >= 1950
AND fyear <= 2023
```

I used the query above to get data from Compustat Funda dataset, the definition of variables are listed below:

- `at`: Total Assets
- `sale`: Gross Sales
- `ib`: Income Before Extraordinary Items
- `cogs`: Cost of Goods Sold
- `xsga`: Selling, General and Administrative Expense
- `txt`: Income Taxes - Total
- `dp`: Depreciation and Amortization
- `oiadp`: Operating Income After Depreciation
- `spi`: Special Items

At the same time, I controlled the `indfmt` as ‘INDL’ which follows Donelson et al. to exclude the financial firms.

> We exclude financial firms, due to the difficulty in interpreting conventional income statement components for these firms.

Plus, I controlled the `popsrc` and `consol` to confirm the company was U.S. domestic.

The raw data from Compustat has 572126 firm-year observations.

## 01. Preprocessing the Master Data

Following Donelson et al. I have set some **filters** to construct our replication sample:

- Drop observations whose `cogs`, `xsga`, `txt` and `oiadp` are missing
- Save the top 1000 companies for each year by total assets
- Keep observations that have 2-year average earnings, 5-year and 10-year earnings volatility

Following the Appendix, I generated the variables below:

- exp = (sale - ib) / at
- rev = sale / at
- earn = rev - exp
- cogs, sga, depr, tax, oiadp, si = ~.x/at
- oth = exp - cogs - sga - depr - tax - si
- depr equals 0 if missing
- si_indicator = 1 if si >= 0.01, otherwise 0

Then, I generated the rolling variables like average 2-year revenue, expense and earnings and their volatility. Plus, I generated the $Earn_{t-1}$, $Exp_{t+1}$ and $Exp_{t-1}$ in the panel.

Finally, I performed the winsorization at the 1% and 99% levels with the variables which Donelson et al. mentioned at the bottom of page 947 and in the footnote 3:

> Finally, we eliminate observations in the extreme 1 percent of the distribution each year of two-year earnings, two-year revenue and expense, two-year earnings volatility, current revenue, current expense, last-year expense, and next-year expense;
>
> To mitigate the influence of extreme observations in our volatility and persistence tests, we eliminate observations in the extreme 1 percent of the distribution each year of earnings volatility and current- and prior-year earnings.

After all the processes above, we have the master data which has 36385 observations from 1960 to 2023. (If we omitted the filter 3: Keep observations with 2-year average earnings, 5-year and 10-year earnings volatility, we shall have 69537 observations from 1950 to 2023.)

## 02. Replication Results of Table 1

### 02.1 Original Table 1: 1967 - 2005

I closely followed the original process and calculation, the replication result of table 1 is as followed:

| PANEL A     | $Exp_{t-1}$ | $Exp_t$   | $Exp_{t+1}$ |
| :---------- | :---------- | :-------- | :---------- |
| 1967 - 1985 | 0.016106    | 1.008213  | -0.015138   |
| 1986 - 2005 | 0.101731    | 0.880352  | 0.032657    |
| difference  | 0.085624    | -0.127861 | 0.047796    |
| t-statistic | -5.476643   | 4.763140  | -3.378902   |
| p-value     | 0.000003    | 0.000029  | 0.001726    |

| PANEL B     | Volatility | Persistence |
| :---------- | :--------- | :---------- |
| 1967 - 1985 | 0.015823   | 0.854779    |
| 1986 - 2005 | 0.025552   | 0.669035    |
| difference  | 0.00973    | -0.185745   |
| t-statistic | -10.139186 | 5.424967    |
| p-value     | 0.0        | 0.000004    |

### 02.2 Extended Table 1: 1967 - 2023

| PANEL A     | $Exp_{t-1}$ | $Exp_t$   | $Exp_{t+1}$ |
| :---------- | :---------- | :-------- | :---------- |
| 1967 - 2005 | 0.060016    | 0.942644  | 0.009372    |
| 2006 - 2023 | 0.106651    | 0.888941  | 0.025542    |
| difference  | 0.046635    | -0.053703 | 0.016169    |
| t-statistic | -2.273698   | 1.667631  | -0.925695   |
| p-value     | 0.026984    | 0.101179  | 0.358723    |

| PANEL B     | Volatility | Persistence |
| :---------- | :--------- | :---------- |
| 1967 - 2005 | 0.020812   | 0.759526    |
| 2006 - 2023 | 0.028161   | 0.691613    |
| difference  | 0.007349   | -0.067913   |
| t-statistic | -4.693045  | 1.601328    |
| p-value     | 0.000019   | 0.115139    |

The extended results show that the $Exp_{t - 1}$ keeps its coefficient trend of significantly going down but the coefficients of $Exp_{t}$ and $Exp_{t+1}$ represent no significant difference before and after our split year 2005. The Panel B of the extended table shows that the persistence of earnings has still declined significantly since 2005, but not to the same extent as after 1985 (as shown in the original table), and the volatility shows the same significant trend. 

And then I tried to switch the split year from 2005 to 1995, the middle of the whole sample period, the results are as followed:

| PANEL A     | $Exp_{t-1}$ | $Exp_t$   | $Exp_{t+1}$ |
| :---------- | :---------- | :-------- | :---------- |
| 1967 - 1995 | 0.048753    | 0.958363  | 0.004010    |
| 1996 - 2023 | 0.101477    | 0.891947  | 0.025313    |
| difference  | 0.052725    | -0.066416 | 0.021303    |
| t-statistic | -2.864309   | 2.289344  | -1.336533   |
| p-value     | 0.005941    | 0.025995  | 0.186979    |

| PANEL B     | Volatility | Persistence |
| :---------- | :--------- | :---------- |
| 1967 - 1995 | 0.018895   | 0.802339    |
| 1996 - 2023 | 0.027499   | 0.670781    |
| difference  | 0.008604   | -0.131558   |
| t-statistic | -6.905864  | 3.684675    |
| p-value     | 0.0        | 0.000532    |

The table shows more significant results than the previous one, where the $Exp_{t+1}$ maintains its difference direction but no-longer significant even on the 0.05 level. And the Panel B seems similar with the previous ones but only with less degree of difference.

### 02.3 Extended Table 1: 2012 - 2023

Unfortunately, we found no significant difference between the two period of 2012 - 2023. The replication table is as followed:

| PANEL A     | $Exp_{t-1}$ | $Exp_t$   | $Exp_{t+1}$ |
| :---------- | :---------- | :-------- | :---------- |
| 2012 - 2017 | 0.079218    | 0.876390  | 0.065090    |
| 2018 - 2023 | 0.109053    | 0.944703  | -0.028368   |
| difference  | 0.029835    | 0.068313  | -0.093458   |
| t-statistic | -0.664919   | -0.802450 | 1.818835    |
| p-value     | 0.522774    | 0.442966  | 0.102292    |

| PANEL B     | Volatility | Persistence |
| :---------- | :--------- | :---------- |
| 2012 - 2017 | 0.025193   | 0.700139    |
| 2018 - 2023 | 0.028477   | 0.684163    |
| difference  | 0.003284   | -0.015976   |
| t-statistic | -1.55093   | 0.28762     |
| p-value     | 0.155332   | 0.780151    |

In order to eliminate the test bias and significance problems caused by too few samples, I tried to change the sample time of this part to 2006-2023 and the split time to 2014. The following is the result after modifying the time parameters.

| PANEL A     | $Exp_{t-1}$ | $Exp_t$   | $Exp_{t+1}$ |
| :---------- | :---------- | :-------- | :---------- |
| 2006 - 2014 | 0.110974    | 0.880442  | 0.026881    |
| 2015 - 2023 | 0.101789    | 0.898502  | 0.024036    |
| difference  | -0.009185   | 0.018059  | -0.002845   |
| t-statistic | 0.221707    | -0.292275 | 0.071483    |
| p-value     | 0.827532    | 0.774080  | 0.943957    |

| PANEL B     | Volatility | Persistence |
| :---------- | :--------- | :---------- |
| 2006 - 2014 | 0.029342   | 0.698925    |
| 2015 - 2023 | 0.026832   | 0.683387    |
| difference  | -0.00251   | -0.015538   |
| t-statistic | 1.181718   | 0.198287    |
| p-value     | 0.255714   | 0.845483    |

After we extended the period, there’s still no significant difference between the first and second half. Even though, we can try some other test methods or make other changes to the time parameter, which can be realize in the second code chunk of my jupyter notebook files.

```
start_y = 
end_y = 
split_y = 
```

## 03. Replication Results of Table 2

The same as the Table 1, I made 3 versions of tables with difference time period as followed.

In conclusion, the panels of weight are similar to the original one but the panels of coefficients are not. I guess they maybe used some scale methods inside but without statement. We can have a discussion in the meeting next week maybe.

### 03.1 Original Table 2: 1967 - 2005

|      |             | cogs      | sga      | depr      | tax      | si        | oth      |
| ---- | :---------- | :-------- | :------- | :-------- | :------- | :-------- | :------- |
| Coef | 1967 - 1985 | 0.324612  | 0.444228 | 0.556301  | 1.289574 | -0.584894 | 0.330598 |
|      | 1986 - 2005 | 0.531894  | 0.623009 | 0.698164  | 2.071183 | -0.399195 | 0.395580 |
|      | difference  | 0.207282  | 0.178782 | 0.141863  | 0.781609 | 0.185699  | 0.064982 |
| W    | 1967 - 1985 | 0.860028  | 0.134761 | -0.000958 | 0.005311 | 0.000700  | 0.000158 |
|      | 1986 - 2005 | 0.828359  | 0.147279 | 0.008029  | 0.007647 | 0.006741  | 0.001945 |
|      | difference  | -0.031669 | 0.012518 | 0.008987  | 0.002336 | 0.006041  | 0.001787 |

### 03.2 Extended Table 2: 1967 - 2023

|      |             | cogs     | sga       | depr     | tax      | si        | oth       |
| ---- | :---------- | :------- | :-------- | :------- | :------- | :-------- | :-------- |
| Coef | 1967 - 2005 | 0.430910 | 0.535911  | 0.629051 | 1.690399 | 0.238180  | 0.363922  |
|      | 2006 - 2023 | 0.469084 | 0.687522  | 0.750564 | 1.907331 | 0.133911  | 0.127035  |
|      | difference  | 0.038174 | 0.151611  | 0.121513 | 0.216932 | -0.104270 | -0.236887 |
| W    | 1967 - 2005 | 0.843787 | 0.141180  | 0.003651 | 0.006509 | 0.003798  | 0.001074  |
|      | 2006 - 2023 | 0.858866 | 0.113168  | 0.009831 | 0.009944 | 0.007390  | 0.000802  |
|      | difference  | 0.015078 | -0.028012 | 0.006180 | 0.003435 | 0.003592  | -0.000273 |

|      |             | cogs      | sga       | depr     | tax      | si       | oth       |
| ---- | :---------- | :-------- | :-------- | :------- | :------- | :------- | :-------- |
| Coef | 1967 - 1995 | 0.337226  | 0.451699  | 0.522809 | 1.519892 | 0.191457 | 0.324713  |
|      | 1996 - 2023 | 0.555570  | 0.721819  | 0.819671 | 2.010124 | 0.222713 | 0.256884  |
|      | difference  | 0.218344  | 0.270120  | 0.296862 | 0.490232 | 0.031257 | -0.067829 |
| W    | 1967 - 1995 | 0.854125  | 0.136216  | 0.000701 | 0.005341 | 0.002854 | 0.000764  |
|      | 1996 - 2023 | 0.842178  | 0.128876  | 0.010711 | 0.009926 | 0.007074 | 0.001235  |
|      | difference  | -0.011947 | -0.007340 | 0.010010 | 0.004585 | 0.004220 | 0.000471  |

### 03.3 Extended Table 2: 2012 - 2023

|      |             | cogs      | sga       | depr      | tax       | si        | oth       |
| ---- | :---------- | :-------- | :-------- | :-------- | :-------- | :-------- | :-------- |
| Coef | 2012 - 2017 | 0.440908  | 0.659121  | 0.645848  | 1.547493  | 0.187052  | 0.152747  |
|      | 2018 - 2023 | 0.401987  | 0.656774  | 0.532957  | 2.335698  | -0.000661 | 0.196790  |
|      | difference  | -0.038921 | -0.002347 | -0.112891 | 0.788205  | -0.187714 | 0.044043  |
| W    | 2012 - 2017 | 0.856555  | 0.116833  | 0.010248  | 0.011160  | 0.004461  | 0.000743  |
|      | 2018 - 2023 | 0.861917  | 0.110135  | 0.010984  | 0.007924  | 0.008628  | 0.000412  |
|      | difference  | 0.005362  | -0.006698 | 0.000736  | -0.003235 | 0.004168  | -0.000332 |
