# Doordash Delivery Duration Prediction

## Objective
Build a model from historical data to predict the estimated time for a delivery.

## Tools
- Python

## Data Exploration
- Predict the total delivery duration
- Dollar values are in cents
- Time values are in seconds and timestamp in UTC
- CSV comes with store id's and categories
- Driver information is provided
- Doordash provides data from other prediction models
    - `estimated_order_place_duration`
    - `estimated_store_to_consumer_driving_duration`

## Cleanup
Looking at the dataframe info the `created_at` and `actual_delivery_time` columns are not a datime.
```
doordash_raw_df[['created_at','actual_delivery_time']] = doordash_raw_df[['created_at','actual_delivery_time']].apply(pd.to_datetime)
```
This is the only cleaning I need to do to the data.

## Feature Engineering
I'm first setting up the target which is provided above:  
`actual_delivery_time - created_at`
```
doordash_clean_df['actual_total_delivery_duration'] = (doordash_clean_df['actual_delivery_time'] - doordash_clean_df['created_at']) 
```
The data contains available dashers as well as busy dasher, but since availability is a changing feature I need to engineer new data that will be more useful. So instead I'll be creating a ratio of available dashers:
`total_busy_dashers / total_onshift_dashers`
```
doordash_clean_df['busy_dashers_ratio'] = doordash_clean_df['total_busy_dashers'] / doordash_clean_df['total_onshift_dashers']
```
Considering the following factors in delivery time:  
Order Placed Duraton + Restaurants Prep Time + Estimated Store to Consumer Time  
Doordash provides 
`estimated_order_place_duration` & `estimated_store_to_consumer_driving_duration` but not the preperation time predictions.
I'll start by combining these two columnns:
```
doordash_clean_df['estimated_non_prep_duration'] = doordash_clean_df['estimated_order_place_duration'] + doordash_clean_df['estimated_store_to_consumer_driving_duration']
```

## Data Preperation
