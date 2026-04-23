#Uncleaned

Text/categorical fields (sym, event_type, order_type, side, order_id, endtag, tradets) are stored as strings.

Numeric fields (price, quantity, client_tag, inception_pnl) are stored as floats.

Interpretation:(numerics/not done the strings)
price
Has negative values (min = -24577.5), which are impossible as real prices.
Has an extremely large max 9,999,999, which is almost certainly a sentinel / error code, not a real market price.
Typical prices (median 158.205, 75% quantile 4995.25) look like realistic FX / XAUUSD / NAS100 values.
quantity
Has negative quantities (min = -49), which don’t make sense as volume, since direction is already in side.
Max quantity is huge (4.98e7), indicating some very large trades or possibly testing/extreme cases.
client_tag
Looks like a code spanning a fixed range (21389 to 73106). Quartiles are neat (40599, 51842, 62471), so this seems fine.
inception_pnl
Reasonable PnL range: roughly -75 to +79. This looks like some PnL metric with symmetric-ish distribution, no obvious errors.
 
 Shape: (30200, 11)

Dtypes:
tradets           object
sym               object
event_type        object
order_type        object
side              object
price            float64
quantity         float64
order_id          object
client_tag       float64
endtag            object
inception_pnl    float64
dtype: object

Missing values:
tradets          176
sym                0
event_type         0
order_type         0
side              42
price            300
quantity           0
order_id           0
client_tag       273
endtag           206
inception_pnl      0
dtype: int64

Numeric summary:
              price      quantity    client_tag  inception_pnl
count  2.990000e+04  3.020000e+04  29927.000000   30200.000000
mean   5.607063e+03  8.809990e+04  48959.882681       8.245348
std    1.160018e+05  1.690701e+06  18124.717451      16.927948
min   -2.457750e+04 -4.900000e+01  21389.000000     -74.577400
25%    1.150600e+00  2.100000e+01  40599.000000      -2.721525
50%    1.582050e+02  3.700000e+01  51842.000000       8.689400
75%    4.995250e+03  1.000000e+02  62471.000000      19.467550
max    9.999999e+06  4.985220e+07  73106.000000      78.579400
 
 #Cleaned
Rows removed by symbol-specific bounds: 81
Dropped exact duplicates: 197

Numeric summary:
              price      quantity    client_tag  inception_pnl    ts_seconds
count  29330.000000  2.933000e+04  29330.000000   29330.000000  29330.000000
mean    4221.293237  9.048124e+04  48536.825060       8.271214   1797.496533
std     8102.372709  1.715088e+06  18624.565853      16.914777   1039.091723
min        1.147500  0.000000e+00     -1.000000     -74.577400      0.000000
25%        1.150650  2.100000e+01  40599.000000      -2.663100    903.325000
50%      158.225000  3.700000e+01  51842.000000       8.720100   1801.750000
75%     4995.300000  1.000000e+02  62471.000000      19.487200   2700.000000
max    24623.500000  4.985220e+07  73106.000000      78.579400   3599.700000
               min         max
sym                           
EURUSD      1.1475      1.1514
GBPUSD      1.3270      1.3304
NAS100  24456.0000  24623.5000
USDJPY    158.1850    159.1450
XAUUSD   4987.6000   5010.7000

 --- Dataset Segmentation Results ---
Total Cleaned Events (df_clean): 29330
Directional Trades Only (df_dir): 29248
Non-directional / Market Making excluded: 82
