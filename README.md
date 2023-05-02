*Credits to [Jie Li](https://github.com/Leejere) for developing the front end and much of the initial google cloud funtions. I made improvements on the model and variable selection in accordance with our project's R-markdown*

# SEPTA, Debunched

[Visit the app](https://leejere.github.io/debunch-septa-app/)

## Why this project?

Knowing when buses are coming, riders plan their trips without stress. Unfortunately, disruptions occur and affect some buses more than others, leading to bunching, where two buses arrive at a stop consecutively, accompanied by subsequent gapping, where riders have to wait a long time after the bunched buses left. This should not happen too often to a reliable bus service.

This app is a proof-of-concept of the operational approach, whereby we predict bunching on the fly (up to 20 stops beforehand) to fine-tune the speeds of the buses to prevent before it happens. You can learn more about the approaches, the data, the methodology, and the models in our [project markdown](https://leejere.github.io/otis-corridor/).

## Workflow

To learn more about the back-end development of the app, please see the [workflow](https://github.com/Bamboo-Forest-Rain/SEPTA-Debunched-App/blob/9b0e72d484fac561fa6b768782bb744b762815a8/notes/model-workflow.md) in the "notes" folder

## How the app works

You can select a route and direction from the top panels to view all the active bus trips in the panel and on the map. After you select the bus of interest, a series of predictions will show in the bottom panel regarding whether this bus is likely to initiate bunching 11 to 20 stops ahead.

This app has two modes, Demo and Real-time. In the "Demo" mode, the app's functionalities are showcased using historical data. In the "Real-time" mode, the app uses real-time bus locations to make real-time predictions. However, as a proof-of-concept, the app currently cannot maintain a continuously active backend, and the user needs to manually initiate observation by hitting the Start Observing button then wait for 10 to 20 minutes before real-time predictions are available.


## Caveat

This app is a proof-of-concept and is not intended for operational use. Due to limitations regarding the training data and lack of granularity of real-time bus-location data, the app may not be able to make complete or accurate predictions.

## Some more technical details

The front end of this app is built with React. Real-time functionalities are built using Google Cloud Platforms.

The `main` branch of this repo contains:

- Source files for the app's website.
- Scripts for data processing and source files for Google Cloud Functions.

