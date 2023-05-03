# Workflow

## Step 1: Workspace Setup

### Steps

- **Repo creation**
  - Create a repository on GitHub for the cloud computing part of the assignment
  - Tools: git

- **Google Cloud project**
  - Create a project on Google Cloud and create service accounts
  - Tools: Google Cloud Platform

## Step 2: Model Creation

### Steps

- **Model selection**
  - Select a model and associated variables based on Yihong's work
  - Tools: R

- **Training data preparation**
  - Based on requirements of the selected model, prepare clean training dataset and save it as training-deploy.gzip in the repo
  - Tools: Python

- **Model training**
  - Train different models N stops ahead
  - Tools: Python

- **Model serialization**
  - Serialize the models and store it in Google Cloud Storage
  - Tools: Python, Google Cloud Storage

## Step 3: Caching Real-Time Data from Transit View API

### Steps

- **Google Cloud Storage structure**
  - Invent a Google Cloud Storage folder structure that is able to cache real time data from the transit-view API
  - Tools: Google Cloud Storage

- **Functionality to write to the storage**
  - Create a Google Cloud Function that grabs data from the Transit View API every 10 seconds and caches it into Google Cloud Storage
  - Tools: Google Cloud Functions

### Information needed from the Transit View API

```json
{
  "bus": [
    {
      "lat": "39.953476000000002",
      "lng": "-75.196358000000004",
      "label": "3054",
      "route_id": "21",
      "trip": "202967",
      "VehicleID": "3054",
      "BlockID": "9105",
      "Direction": "WestBound",
      "destination": "69th Street Transportation Center",
      "heading": 270,
      "late": 3,
      "next_stop_id": "21363",
      "next_stop_name": "Walnut St & 38th St",
      "next_stop_sequence": 36,
      "estimated_seat_availability": "STANDING_ROOM_ONLY",
      "Offset": 0,
      "Offset_sec": "23",
      "timestamp": 1681939647
    },
}
```

### Outcome:

- Record every time a bus changes its next stop
- Outcome is a table like this:

| Route | Trip | Direction | Stop  | Timestamp  |
|-------|------|-----------|-------|------------|
| 21    | 343  | WestBound | 21363 | 1681939647 |
| 21    | 322  | WestBound | 21363 | 1681939647 |

where each row represents a stop arrival instance. One row is added when detecting that a bus's next stop has changed.
When making the prediction, get this table and use data from the last 3 hours, for example.

## Step 4: Upload dictionaries to join with the real-time data

### Steps: 

- **Create and upload a dictionary that contains stop information**
  - Each stop will be marked with a unique Id: "route"-"directionId"-"stopId"
  - The stop-level variable includes a stop's daily average ridership in 2019, the population of the block which the stop is in, the number of signals ahead...etc. These are all preprocessed and ready to be used for predictions.
  - Tools: Python, Google Cloud Storage

- **Create and upload a dictionary containing the unique IDs of the next 11 to 21 stops for each stop**
  - Format:          
  {"21_0_21351": {"next_11_unique_id": "21_0_19077", "next_12_unique_id": "21_0_19078", "next_13_unique_id": "21_0_19079"..."next_21_unique_id": "21_0_30194"}
  - This is to retrieve the next "X" stop information as we predict initiation of bunching at "X" number of stops ahead.
  - Tools: Python, Google Cloud Storage

- **Create and upload a dictionary containing trip schedules**
  - Format:          
  {"60622": {"expectedHeadway": 1098.8, "start_time": "20:29:00"} 
  - SEPTA buses are dispatched based on their schedule everyday. Each bus trip has a unique trip ID. 
  - Tools: Python, Google Cloud Storage

## Step 5: Make Predictions on Request from Front-End

### Steps

- **Data engineering**
  - On user request:
    - Grab data from the cache
    - Grab next X stops info from dictionaries based on the current bus arrivals
    - Grab trip schedules
    - Make joins and calculate headways, speeds, lateness, as well as associated lag variables
    - Preprocess the data so it's ready to be predicted on

  - Tools: Google Cloud Functions

- **Prediction making**
  - Use the serialized data to produce a json containing production results
  - Return results in an array of "TRUE" and "FALSE" based on probability and a set threshold.
  - For example:
    [FALSE, FALSE, FALSE, TRUE, TRUE, TRUE, FALSE, FALSE, FALSE]
  
  - Tools: Google Cloud Functions

### How to engineer data

 - First, from the above table, calculate the headway, speed, latnesses.

 - Then, find out what is the stop 11-20 stops away.

 - Next, join stop-specific data on the 11-to-20-stops-away stops.


 - Call the associated model from the storage bucket.

 - Finally, predict.



| Route | Trip | Direction | Stop  | Timestamp  | Previous bus trip ID | Headway | Speed | Lateness | Prev_Headway | Prev_Speed | Prev_Lateness |
|-------|------|-----------|-------|------------|----------------------|---------|-------|----------|--------------|------------|---------------|
| 21    | 343  | WestBound | 21363 | 1681939647 | 322                  | 10      | 10    | 0        | 10           | 10         | 0             |
| 21    | 322  | WestBound | 21363 | 1681939647 | 343                  | 10      | 10    | 0        | 10           | 10         | 0             |


|Future Xth Stop ID  | Stop level variables |
|--------------------|----------------------|
|21567               | 10                   | 
|21567               | 10                   | 
