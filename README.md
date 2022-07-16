
# Traffic Prediction project
The project can help plan ride successfully, and get more accurate information about the duration of the ride . By professionally characterizing the road and providing a correct forecast for each hour of ride and for each section of road.

By: Bini Shaked - Reichman university – February 2022
# introduction 
Road congestion is characterized by the fact that a reduction in the number of vehicles does not reduce travel time linearly (Feitelson, 2017). A particular section of road has a capacity or volume of travel, measured in the number of vehicles per hour, where vehicles travel without congestion – meaning that the time and speed of travel are similar or identical to the time and speed of travel when the road is mostly empty. The road goes into a state of congestion when demand approaches or rises above capacity, then the driving speed decreases greatly and the travel time becomes very long, and the amount of vehicles on the road drops well below capacity when there is no congestion (Forecast, 2019).
Today, an issue that comes up again and again is the need for a solution to the problem of traffic jams, and here there are two sides that try to solve the problem, each on its own side. On the one hand, the state, it is important for it to solve the problem of traffic jams froma book of considerations:
-	Environmental considerations as the heavy congestion on the roads has, throughout history, led to heavy air pollution that is only increasing and causing great damage.
-	Economic considerations – reducing expenditure on infrastructure and harming GDP as people waste long hours in traffic jams, and studies predict that by 2030 we will be able to withstand traffic jams for another 60 minutes.
On the other hand, road users (public transport passengers or private drivers), use navigation apps to plan their trip to the fastest road. The accuracy of road congestion forecasts depends on the number of app users. There are isolated areas where there are not enough users of navigation apps or there is no stable cellular network. In these areas, it is not possible to make a reliable forecast about road congestion only according to real-time measurements,  Therefore, in the project we will focus on forecasting the congestion on the road based on historical information about this road.
# The Research Question
We would like to predict a given congestion on the road for each hour of the day based on data we collected from that road using a sensor, the goal is to predict the duration of the trip in the best possible way, thereby helping the navigation apps to increase the well-being of road users.
In conclusion, the project can lead to successful trip planning, and obtaining more accurate information about the duration of the trip, by professionally characterizing the road and providing a correct forecast for each hour of travel and for each section of road.

# 2. Stages of work

## information 
For the project,Thy used the NYCopendata website. The site contains data sets from various fields, including transportation in New York City. For the purpose of predicting traffic congestion, he searched for datasets with measurements of traffic congestion on the city streets. On the site there are two types of such sets.

Type One: Traffic Volume Countswhich contains measurements of the amount of vehicles on a particular road segment and is not soft for one day in a row at most. According to the data's description, the measurements were made manually and collected with a frequency of one hour, that is, 24 measurements in a row of a particular street. For a two-way road there are 24 measurements for a lane in each direction. Startworking on this data set and it'snot suitable for training a model that can solve the prediction problem. The reason is that the modelwe are  Want to train, is an autoregression model that performs prediction according to continuous measurement in time. Because the data is continuous for only one day, and not several days or weeks in a row, it will not be possible to train such a model because of the lack of continuous measurements. For example, even if we have 24 hours of measurement for a particular street, if we make a prediction of load in the same extension for the next day, it will not be possible for us to verify theprediction, that is, we will not be able to say whether we were right or wrong,  Because we won't have real data the next day.

We used this data to get intuition about the traffic congestion throughout the day on average across all dates and all sections of the city. You can see an expected result of low load during night hours and maximum load during the afternoon. Later we will see this pattern repeated.
 ![image](https://user-images.githubusercontent.com/106141785/179346156-bdeb4b9f-66f5-42e0-abcf-4c42ced76fba.png)

The second type: Automated Traffic Volume Counts, which contains measurements of the quantity of vehicles, in the same structure as the previous dataset. This data is collected automatically, over long periods of time of more than a week, at a frequency of fifteen minutes. This data is much more suitable for our task, because of the high resolution of the measurements (every 15 minutes) and the continuity of the measurements over several days. If one lane on a particular street is selected , and faithful to an autoregression model on the measurements of traffic congestion, we can predict the traffic congestion throughout the day after (or number of days). This data set contains more than 27 million rows but not all of them are relevant to the task. Now we will describe whether we have prepared the data relevant to the training:
1.	Each section of road has a unique identifier. In advance we chose only identifiers that have a great deal of measurements (not necessarily continuity).
2.	After focusing on a particular segment, we checked whether its measurements were continuous. In most cases, the measurements are continuous for a week or two with a break of several months before and after. To find the longest sequences, we introduced a vehicle count graph (y-axis) as a function of a date (x-axis).
 ![image](https://user-images.githubusercontent.com/106141785/179346198-732061cd-5b85-49b7-80cc-5f94893f291f.png)

Each set of points in the graph represents a continuous series of Amos on the road, and if we zoom in into one of the groups, we get the following graph:
 ![image](https://user-images.githubusercontent.com/106141785/179346218-509bd096-043d-4fc1-8668-ebe08298e1e0.png)

You can see that the graph contains measurements over a 22-day period, with one measurement every 15 minutes. We will use this data to train the model.

We assume that the latest measurements made on a particular road represent the congestion on the road in the best possible way, and according to these measurements it is possible to predict the congestion in the future. According to this assumption, the autocorrelation model is faithful by various methods.

Note: In some cases, the street names for the same ID were written in the data differently, for example: N/B AMSTERDAM AVE and AMSTERDAM AVENUE. The only cleaning we've done with data is to change the street names to a uniform name.

## model
The problem of predicting traffic congestion based on traffic congestion in the past is an auto-regression problem ("auto" - because basically the value we want to predict is also the characteristic by which the prediction was made, only at a different time), so we used an auto-regression model fromthe Statsmodels library. The first step is to test autocorrelation of traffic congestion with its lags, that is, with traffic load measurements at past points in time. Here is the graph:
 ![image](https://user-images.githubusercontent.com/106141785/179346252-619b8772-1f1b-4c29-8526-5a054a9b0355.png)

We can see that there is a strong positive correlation between the measurement and its first 25 lags. We will remember that the whole stay is 15 minutes. That is, the 25 closest delays are 6 and 15 hours. There is also a negative correlation between the measurement and its first 50 to 125 delays, that is, 12.5 hours to 31 hours (which makes sense because, for example, if at night there was a low load,  after 12 hours, a day, there will be a high load). This graph indicates that it is probably possible to trainan autoregression model.

## Model types
#### Static model
A static model is anautoregression model that makes a prediction of traffic congestion for a certain period starting from a certain day. Logically, as the forecast date moves away from the last date for which a measurement exists, the prediction is less accurate. You can see the effect in the graph:
![image](https://user-images.githubusercontent.com/106141785/179346270-2dfaf8a6-9a80-43a9-99d2-fa3be2b3cdd4.png)
 
The orange measurement is the long-lasting measurement that we want to predict, and is not exposed to the model. The model performs the prediction according to the blue signal only. The green curve represents the prediction of the model according to the blue signal. It can be seen that as the green signal progresses in time, it loses more and more details, representing only the general "trend". The nature of the predication can be measured using the MSE index.

In order to improve the performance of the model, we will perform a hyperparameter search. In our case, the only parameter is the number of delays on which the model is based. Iteratively faithful models with increasing latency numbers and we will measure the prediction MSE. We will get the following graph:

 ![image](https://user-images.githubusercontent.com/106141785/179346277-71c7314a-8023-41ec-ba90-3b911c506a78.png)

From the graph you can see that for 225 delays we get a minimum MSE -2860. This is what the predication of the static model looks like with an optimal amount of delays:
 
![image](https://user-images.githubusercontent.com/106141785/179346286-941a5151-37fa-4785-a8b9-d7a2cde54da5.png) 

## Dynamic model
We will now present the motivation for a dynamic model. When it comes to the congestion of vehicles on the road, probably, the most relevant prediction for us is a prediction of the near future. That is, in most cases we want to know what the load will be tomorrow and not in a week, what the load will be in the next few hours, and not in another day. Also, if we wanted to know what the load would be in two days, we would prefer to wait another day, in order to make predictions about more up-to-date data. Therefore, we propose another model - "dynamic" that works according to theabove principles.

In our experiment, the model makes a prediction for only the next 15 minutes, and then, after the real information for that quarter of an hour becomes available, we retrain the model on updated data and perform a predication for the next fifteen minutes. And so the operation is repeatedin an iterative way. Under the assumption that the user must receive a prediction "from now on", the performance of the dynamic model exceeds that of the static model:

![image](https://user-images.githubusercontent.com/106141785/179347651-8cfaffd8-7863-4b68-950b-bccbde78a7fd.png)



We will present the graph of loads and forecasts of the model:
 
 ![image](https://user-images.githubusercontent.com/106141785/179347677-2904583a-3963-41fb-b981-f444fbbbe449.png)



# 3.	 Project summary: 




A. The performance of the dynamic model trained during the experiments makes it possible to predict road congestion with great accuracy, and to provide reliable forecasts for end users of the navigation applications. In addition, the static model can give an estimate based on rough historical data for traffic congestion within a few days, in a scenario where no updated information has been received from the sensors. In the context of the research question in the Middle Report, it can be argued that the goal was achieved under the assumptions of a comprehensive sensor system that counts the number of vehicles on the roads. While this assumption may sound too strong, with the improvement of technology and the expansion of network deployment, the price of performing these measurements has dropped significantly.

It is worth noting that the goals specified in the midway report (correct pricing of congestion charges) have changed to the prediction of traffic congestion on a single road. It can be clearly seen that the project has achieved these goals.

2. The stage that significantly helped in the success of the project was the correct identification of the type of information (time series) and type of predictive problem. At this stage, the choice of models was obvious, so the first experiment was carried out with a regular autoregression model. This model brought reasonable results and proved that it is possible to predict road congestion based on past data. The significant improvement for this model was its transformation into a dynamic model. A dynamic model is retrained with the arrival of each new sample. This training method allows accuracy only in forecasting and improves the Model performance. Information that could have improved the performance of the model is continuous historical data for longer periods, which could have improved the performance of the static model. Another information that could have been useful is the average speed of the cars in the segment, which would have made it possible to solve the additionalpredicated problem of speed prediction. Combined with predicting the car count on the road, we could conclude whether the road is jammed or not.

It is possible to note that for each section of road it is necessary to train a different model that performs forecasting. The realization of the project code authorizes this to be done automatically assuming that the code is embedded in a real software system. We will describe the process of training the possible automated model:
-	First, we will get the measurements of the car count on the road
-	We will run a training of anautoregression model with a different amount oflagsinan iterative way
-	A model was selected for which we received a very low MSE (the process can be done once in a long period)
-	The selected model needs to be retrained with the arrival of each new measurement
-	If, due to a malfunction in the sensor system, no new samples arrive, the static model can be used in order to obtain a more rough estimate of the load on the road.


C. During the work on the project we were unable to combine the available information of the Traffic Volume Counts with the Automated Traffic Volume Counts for training the model we focused on automatic measurements only.

We also carried out the prediction of the car count for a single road segment selected from a collection of roads with continuous measurements over a period of about 3 weeks. That is, a road for which there was not a sufficiently long sequence of measurements, we would not be able to train anauto-regressive model for this road.

The assumptions of the research question are very strong assumptions, which ensure the proper work of the model only in the event that there are available and up-to-date measurements of the vehicle count on the roads. These assumptions do not exist in today's reality, and therefore the model will work only as part of a "thought experiment" of a road system with a sophisticated sensor network.

The dynamic model makes accurate predictionsbut does so only for the next 15 minutes, which means that predictions of this model cannot be relied upon for a longer time span as much as forecasting for a very near future. For example, forecasting road congestion for a future hour 16:15 done at 12:00 would not be as accurate as the prediction that the dynamic model would make at 16:00. I mean the disadvantage of the dynamic model is that the most reliable forecasting,  Is done at a time, when the prediction may no longer be relevant.

Bibliography:

Agmon Tamir and Chertoff Yaakov - Taxation Methods for Limiting Road Congestion, June 2011

Assaf façade - collapse of the ability to transmit traffic in traffic jam conditions, 2019

Feitelson Dror – The Dynamics of Congestion, June 2017

Small, Kenneth A., Verhoef, Eric T., The economics of urban transportation,2007

Relevant articles and additional sources for the report:

"Artificial Intelligence will Try to Shorten the Traffic Jams on the Ayalon Highway"- Udi Etzion-Calcalist
https://www.calcalist.co.il/local/articles/0,7340,L-3876932,00.html
"If there are no congestion charges, the traffic jams will continue" – Prof. Omer Moav – Globes
https://www.globes.co.il/news/article.aspx?did=1001368846
The Failure of Traffic Jams – Dubi Ben Gedaliah – Globes
https://www.globes.co.il/news/article.aspx?did=1001369007
OECD Economic Surveys - Israel
https://www.oecd.org/economy/surveys/Israel-2020-OECD-economic-survey-overview.pdf
Israel Economic Snapshot-Economic Forecast Summary (December 2021)- OECD
https://www.oecd.org/economy/israel-economic-snapshot/
 

