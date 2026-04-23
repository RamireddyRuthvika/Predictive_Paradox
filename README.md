# Predictive_Paradox
## Data Loading and Merging
1. Loaded all three datasets using read_csv and read_excel
2. Converted datetime column to datetime format using pd.to_datetime 
3. Merged PGCB_date_power_demand.xlsx and weather_data.xlsx(read from row three and changed time column to datetime) on datetime
4. Transformed economic data into yearly format and merged it with previous merged dataset on year
   
------

## Converting to Hourly Data and Resampling
Timestamps were put in order by date and time.
Weighted averaging based on time differences was used to deal with irregular timestamps, such as half-hour intervals. By this timestamps were accurately converted to hourly time series with accurate data(Only convertable and essential data was redistributed hourly).

------

## Missing Data Handeling and Spike Handeling
Missing data in hourly data was handeled by two methods
1. Interpolation for demand_mw (to ensure smooth transition of data rather than copying previous ones like fill())
2. ffill (forward fill) for remaining features in hourly data(hourly data intially avoids economic data for these steps later merged)
Spikes (extreme 2 percentile values) in demand were clipped by quantile method to reduce impact of sudden raises in demand

------

## Feature Engineering
### Temporal Features
Hour, day of week, month and year were extracted to capture periodic demands 
### Past Demand
#### Lag Features
lag_1, lag_4, lag_168 etc.. were created to track past values and dependencies.
#### Rolling features
mean and rolling features were set to capture local trend of demand
### External Features
Relevant weather factors such as (temperature, humidity) were included to track environmental effects on energy demand and total population(female + male), GDP were included from yearly economics data.

As far as macroeconomic features are concerned, both GDP and population were considered. However, those two features are highly correlated and when tested indepently and together included model gave different errors representing less importance of GDP compared to population.

Furthermore, short-term trends in demand were accurately captured by demand features from history. Therefore, GDP could not offer an additional insight regarding future sales.

Ultimately, population was chosen as a macroeconomic feature since it is more stable and shows superior performance.

------

## Target Definition and Data Leakage Handeling
The target was defined as the next hour demand, created by shifting data of demand by one step ahead
Coloumns which may lead to data leakage or psuedo data leakage (such as energy generation, generation by each power source these match the present demand directly or indirectly and generation depends on demand but not the otherway) were dropped in train_X data and test_X data and total population, GDP were merged with hourly data after lagging by 1 year. Above steps ensures that there is no data leakage.  

-------

## Train and Test Split
Training Data : up to 2023
Test Data : 2024
This ensured that only past information was used to predict, preventing any type of leakage from future data.

------

## Model and Evaluation
Random Forest Regressor was trained using the engineered features and its performance was evaluated by mean absolute percentage error.
final mape ~0.00335(3.4%)

------

## Key Insights from Feature Importance
lag_1(previous hour demand) was the most important feature contributing almost 67% in the prediction that says the strong dependency of next hours demand on current hour.
rolling_mean also have significant effect on predicting future demand by capturing recent trends.
Daily lag(lag_24) helped model to predict daily patterns.
Some of the weather features like temparature also have significant effects on energy demands while others like wind direction does not effect on large scale.
Macroeconomic features like population has very limited impact on hourly energy demand confirming that short term demand like horly demand depends mainly on recent history of the demand rather than long term economic factors.
