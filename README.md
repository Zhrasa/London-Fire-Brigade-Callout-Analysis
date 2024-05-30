<div align="center">
  <h1>London Fire Brigade Analysis</h1>
</div>

This dataset will be employed to answer the business problem questions:
The only location in the UK to be named in Frommer's top ten global destinations for 2012 was Greenwich. This aims to investigate a genuine issue that the London Fire Brigade has.

## Business Problems:
This dataset will be employed to answer the following questions:
* With the assumption that people would not call the LFB falsely for the Special Services, the question is how to accurately predict whether an alarm is false and to determine the risk factors for fires. 
* What is the relationship between property categories and incident groups? Are there specific property categories that are more likely to suffer from fires?
* What are the locations and dates of the fires? Are there more fires during or close to bank holidays?
* Have all the fires in the borough been attended by the local stations? Have the local stations been used for other Fire Service grounds? What are the features of the incidents attended by other stations and is there a difference from the incidents attended by the local stations?
* What does the number of pumps necessary per day depend on? How many pumps should a station have?

## Data Pre-processing
### Data Types
* Firstly, Numerical ID features (UPRN and USRN) were transformed into the object type. To ensure that the correct datatypes were allocated to each variable `Easting_rounded`, `Northing_rounded` and `HourOfCall` needed converting to categorical types. 

### Null/Missing Values
* Secondly, the partially missing (SpecialServiceType, Postcode_full), duplicate (IncGeo_BoroughName, IncGeo_BoroughCode, IncGeo_WardCode, IncGeo_WardName) or single-value (FRS; ProperCase was renamed to Borough and removed later) features and ID features (IncidentNumber, USRN, UPRN) were removed
The Year feature was dropped, while the DatetimeOfCall was used only for descriptive modelling. 
* Thirdly, the null values which logically meant zero (NumStationsWithPumpsAttending, NumPumpsAttending, Notional Cost (£), PumpHoursRoundUp, PumpCount) were assigned the value zero. FirstPumpArriving_AttendanceTime and SecondPumpArriving_AttendanceTime were not filled, because logically they did not mean instant, 0 seconds, arrival, but that the teams had not been dispatched.
* Finally, the outliers were removed. For the numerical features, the Standard score or z-score was applied. The rows with any numerical value having a z-score above 3 were dropped. Among the categorical features the unique values with counts lower than 1% (128, rounding down) were removed. Then the One Hot Encoding method function was created to be used in the following section for the training dataset on the categorical object and numerical datetime features, vastly increasing the total number of them.  


From the numerical histograms, it can be seen that most of the incidents, including the false alarms and special services, happened between 15:00 and 20:00 hours. The Easting_rounded and Northing_Rounded histograms are slightly skewed to the right. The Attendance Time for the First Pump Arriving was around 6 minutes on average.The DatetimeOfCall histogram demonstrates a small increase in the number of calls during the period observed. The DayOfCall and Month histograms indicate that there were slightly more calls during summer. The rest of the histograms are heavily skewed to the left.

![image](https://github.com/Zhrasa/London-Fire-Brigade-Callout-Analysis/assets/99383300/ac9876ae-f668-4278-801e-169c9c5f4221)
<div align="center">   Fig. The histograms of the numerical and temporal features. </div>


![image](https://github.com/Zhrasa/London-Fire-Brigade-Callout-Analysis/assets/99383300/f80c573a-004b-4bb8-bd19-09d1c669cffe)

<div align="center"> Fig The histograms of the categorical features.</div>


![image](https://github.com/Zhrasa/London-Fire-Brigade-Callout-Analysis/assets/99383300/7b134ecb-3086-4c50-a5c3-595c7107d90d)

<div align="center"> Fig. The scatter plot of fires and false alarms.</div>


![image](https://github.com/user-attachments/assets/00a05f8c-5127-47b0-bf94-3fd087949ee1)

 <div align="center"> Fig. The box plots </div>


 

### Model Construction

#### IncidentGroup Fire/False Alarm Prediction

* For the predictive modelling of the categorical features, a classification task, Decision Tree Classifier, Random Forest Classifier, and Extreme Gradient Boosting Classifier were used. The former provided a clear explanation and visualisation, the other two performed better with the evaluation metrics used for the targets. 
The preprocessing commands described in the previous section, Data Preparation, were defined as functions to be applied only to the training set, leaving the testing set more appropriate and realistic.
The One Hot Encoding method was applied to the dataset and it was split into the training and testing sets. The split was not random, but based on the order of the entries - the last eighth, consisting of 1,607 rows, was set for testing, providing reasonable variability and size. The rationale behind using the newest records was that a prediction model would be used to predict the unseen data, less affected by the older trends existing in the training set. The training set consisted of 7,000 entries after the outliers were removed. 
The features were separated into the known and unknown at the time of dispatching a team, as described in Table I. 
Several functions from the scikit-learn Python package were used to select the features which would provide the most accurate prediction - SelectKBest and Recursive Feature Elimination (RFE) with feature ranking. The former required to specify the scoring metrics. F_classif (computing the ANOVA F-value) and mutual_info_classif (measuring the dependency between the features) were used with SelectKBest. The number of features was set as one-fourth of the total number (710), 178 (rounding up). 
The Decision Tree Classifier was chosen as the estimator for the RFE function. Additionally, a set of 18 most important features was generated with RFE. Tables XI and XII describe the summarised feature importance values for the Fire and False Alarm Incident Groups. It can be seen that the temporal and property-related features had the highest importance for both target features, with DayOfCall and PropertyCategory reaching 0.28 and 0.25 for Fire and DayOfCall, PropertyType, and HourOfCall 0.31, 0.13, and 0.12 for False Alarm.

<div align="center"> TABLE I  
  
RFE Feature Importance for the Fire IncidentGroup

|**Feature** | **Importance (Sum)** |
|--------------|--------------|
| DayOfCall | 0.28 |
| PropertyCategory | 0.25 |
| PropertyType | 0.09 |
| HourOfCall | 0.09 | 
| Ward | 0.07 |
| Easting_rounded | 0.05 |
| NumCalls | 0.05 |
| Northing_rounded | 0.04 |
| IncidentStationGround | 0.03 |
| Postcode_district | 0.03 |
| Month | 0.02 |
| AddressQualifier | 0.01 |
</div>


<div align="center"> TABLE II  
  
RFE Feature Importance for the False Alarm IncidentGroup 


|**Feature** | **Importance (Sum)** |
|--------------|--------------|
| DayOfCall | 0.31 |
| PropertyCategory | 0.12 |
| PropertyType | 0.13 |
| HourOfCall | 0.12 | 
| Ward | 0.05 |
| Easting_rounded | 0.08 |
| NumCalls | 0.01 |
| Northing_rounded | 0.08 |
| IncidentStationGround | 0.01 |
| Postcode_district | 0.02 |
| Month | 0.8 |
| AddressQualifier | 0.02 |
</div>


Due to the application of the One Hot Encoding method, the feature selection functions selected the encoded features, i.e. the specific values of the categorical features. 
The Decision Tree algorithm was run with each selection of features. F1 score (binary) and accuracy were chosen as the evaluation metrics. The output of the algorithm is present in Tables III and IV. The SelectKBest (f_classif) function provided the highest F1 score for the Fire IncidentGroup and RFE with 18 selected features for the False Alarm IncidentGroup.


<div align="center"> TABLE III  
  
Feature Sets’ Performance for the Decision Tree Algorithm for the Fire IncidentGroup


|**Feature selection function** | **Number of features** |**F1 score** |
|--------------|--------------|--------------|
| All features | 710 | 	0.6195190948  |
| SelectKBest (f_classif)  | 178 | 	0.6701986755  |
| SelectKBest (mutual_info_regression) | 178 | 	0.5046728972  |
|  RFE | 178 |	0.6614173228  |
|  RFE | 18 |  0.6563706564 |


</div>
		
<div align="center"> TABLE IV  
  
Feature Sets’ Performance for the Decision Tree Algorithm for the False Alarm IncidentGroup

|**Feature selection function** | **Number of features** |**F1 score** |
|--------------|--------------|--------------|
| All features | 710 | 	0.5501672241  |
| SelectKBest (f_classif)  | 178 | 	0.5158227848 |
| SelectKBest (mutual_info_regression) | 178 | 	0.5090218424 |
|  RFE | 178 |	0.5389121339 |
|  RFE | 18 |  0.5520134228 |


</div>

 

Several other machine learning algorithms were compared on all features of the dataset to determine the ones to be used. The results are presented in Tables V and VI. For the Fire IncidentGroup, the XGBClassifier was chosen as the second algorithm, while for the False Alarm IncidentGroup, the RandomForestClassifier provided the best overall result before hyperparameter tuning.

<div align="center"> TABLE V  
  
Machine Learning Algorithms’ Performance for the Fire IncidentGroup

|**Algorithm** | **Accuracy** |**F1 score** |
|--------------|--------------|--------------|
| LogisticRegression  | 0.8786558805227132 | 0.7022900763358778  |
|  RandomForestClassifier		 | 0.8686994399502178 | 0.6602254428341385  |
| GradientBoostingClassifier  | 0.8742999377722465  |  0.6833855799373041 |
|  XGBClassifier |0.8755444928438083 | 0.6978851963746224  |
</div>

		
<div align="center"> TABLE VI  
  
Machine Learning Algorithms’ Performance for the False Alarm IncidentGroup

|**Algorithm** | **Accuracy** |**F1 score** |
|--------------|--------------|--------------|
| LogisticRegression  | 0.7299315494710641|  	0.5905660377358491 |
|  RandomForestClassifier		 |0.7454884878655881 | 	0.5930348258706468 |
| GradientBoostingClassifier  | 0.7479775980087119| 	0.5829042224510814 |
|  XGBClassifier  |0.7280647168637212 | 0.5563451776649746  |
</div>
		
		

		

The feature sets were evaluated with the untuned secondary models chosen (Tables XVII and XVIII). The RFE-selected features provided higher performance for XGBClassifier and the SelectKBest with f_classif for RandomForestClassifier. 


<div align="center"> TABLE VII  
  
Feature Sets’ Performance for the Extreme Gradient Boosting Algorithm for the Fire IncidentGroup

|**Feature selection function**| **Number of features** | **Accuracy** |**F1 score** |
|--------------|--------------|--------------|--------------|
|  All features | 710  |  0.8755444928438083 | 0.6978851963746224  |
| SelectKBest, f_classif  |  178 |  0.8724331051649036 | 0.6870229007633588  |
|  SelectKBest, mutual_info_regression |  178 | 0.8630989421281892  | 0.6283783783783783  |
|  RFE | 178  | 0.8855009334163036  | 0.7378917378917379  |
| RFE  |  18 | 0.8830118232731798  |  0.7329545454545454 |

</div>
		

<div align="center"> TABLE VIII  
  
Feature Sets’ Performance for the Random Forest Algorithm for the False Alarm IncidentGroup

|**Feature selection function**| **Number of features** | **Accuracy** |**F1 score** |
|--------------|--------------|--------------|--------------|
|  All features | 710  | 0.7492221530802738  | 0.5957873620862588  |
| SelectKBest, f_classif  |  178  | 0.7417548226509023  | 0.6153846153846154  |
|  SelectKBest, mutual_info_regression |  178  | 	0.6969508400746733	  | 0.5616561656165616  |
|  RFE | 178  |  0.7367766023646546 |  0.5774225774225774 |
| RFE  |  18  | 0.7392657125077785	  |  0.5814185814185814 |

</div>



The hyperparameter tuning is described in the following subsection, Model Parameters. 

#### PropertyCategory and IncidentGroup Association Analysis
Tables XIX and XX demonstrate the association between the IncidentGroup and PropertyCategory features. It was assumed that the data had been entered correctly and the outliers were natural, as opposed to data entry errors.

<div align="center"> TABLE IX  

Association Between IncidentGroup and PropertyCategory (with Outliers)

| **PropertyCategory**  | **IncidentGroup** | **Fr_PC** | **Fr_IG** | **Fr_Itemset** | **FrP_PC** | **FrP_IG** | **Support** | **Conf_PC->IG** | **Conf_IG->PC** |
|-----------------------|------------------|-----------|-----------|----------------|------------|------------|-------------|-----------------|-----------------|
| Aircraft              | Special Service  | 1         | 4886      | 1              | 100.00     | 0.02       | 0.01        | 100.00          | 0.02            |
| Outdoor Structure     | Fire             | 707       | 2777      | 574            | 81.19      | 20.67      | 4.46        | 81.19           | 20.67           |
| Other Residential     | False Alarm      | 544       | 5198      | 430            | 79.04      | 8.27       | 3.34        | 79.04           | 8.27            |
| Non Residential       | False Alarm      | 2313      | 5198      | 1710           | 73.93      | 32.90      | 13.30       | 73.93           | 32.90           |
| Road Vehicle          | Special Service  | 1152      | 4886      | 747            | 64.84      | 15.29      | 5.81        | 64.84           | 15.29           |
| Outdoor               | Fire             | 1476      | 2777      | 892            | 60.43      | 32.12      | 6.94        | 60.43           | 32.12           |
| Boat                  | Special Service  | 5         | 4886      | 3              | 60.00      | 0.06       | 0.02        | 60.00           | 0.06            |
| Rail Vehicle          | Special Service  | 7         | 4886      | 4              | 57.14      | 0.08       | 0.03        | 57.14           | 0.08            |
| Dwelling              | Special Service  | 6656      | 4886      | 3290           | 49.43      | 67.34      | 25.58       | 49.43           | 67.34           |
| Dwelling              | False Alarm      | 6656      | 5198      | 2642           | 39.69      | 50.83      | 20.54       | 39.69           | 50.83           |
| Rail Vehicle          | Fire             | 7         | 2777      | 2              | 28.57      | 0.07       | 0.02        | 28.57           | 0.07            |
| Road Vehicle          | Fire             | 1152      | 2777      | 317            | 27.52      | 11.42      | 2.46        | 27.52           | 11.42           |
| Outdoor               | Special Service  | 1476      | 4886      | 336            | 22.76      | 6.88       | 2.61        | 22.76           | 6.88            |
| Boat                  | False Alarm      | 5         | 5198      | 1              | 20.00      | 0.02       | 0.01        | 20.00           | 0.02            |
| Boat                  | Fire             | 5         | 2777      | 1              | 20.00      | 0.04       | 0.01        | 20.00           | 0.04            |
| Outdoor               | False Alarm      | 1476      | 5198      | 248            | 16.80      | 4.77       | 1.93        | 16.80           | 4.77            |
| Non Residential       | Special Service  | 2313      | 4886      | 377            | 16.30      | 7.72       | 2.93        | 16.30           | 7.72            |
| Rail Vehicle          | False Alarm      | 7         | 5198      | 1              | 14.29      | 0.02       | 0.01        | 14.29           | 0.02            |
| Other Residential     | Special Service  | 544       | 4886      | 73             | 13.42      | 1.49       | 0.57        | 13.42           | 1.49            |
| Outdoor Structure     | False Alarm      | 707       | 5198      | 78             | 11.03      | 1.50       | 0.61        | 11.03           | 1.50            |
| Dwelling              | Fire             | 6656      | 2777      | 724            | 10.88      | 26.07      | 5.63        | 10.88           | 26.07           |
| Non Residential       | Fire             | 2313      | 2777      | 226            | 9.77       | 8.14       | 1.76        | 9.77            | 8.14            |
| Outdoor Structure     | Special Service  | 707       | 4886      | 55             | 7.78       | 1.13       | 0.43        | 7.78            | 1.13            |
| Road Vehicle          | False Alarm      | 1152      | 5198      | 88             | 7.64       | 1.69       | 0.68        | 7.64            | 1.69            |
| Other Residential     | Fire             | 544       | 2777      | 41             | 7.54       | 1.48       | 0.32        | 7.54            | 1.48            |
</div>
  <div align="center"> TABLE X

Association Between IncidentGroup and PropertyCategory (without Outliers)

| **PropertyCategory**  | **IncidentGroup** | **Fr_PC** | **Fr_IG** | **Fr_Itemset** | **FrP_PC** | **FrP_IG** | **Support** | **Conf_PC->IG** | **Conf_IG->PC** |
|-----------------------|------------------|-----------|-----------|----------------|------------|------------|-------------|-----------------|-----------------|
| Outdoor Structure     | Fire             | 230       | 1273      | 209            | 90.87      | 16.42      | 2.88        | 90.87           | 16.42           |
| Other Residential     | False Alarm      | 229       | 2585      | 195            | 85.15      | 7.54       | 2.69        | 85.15           | 7.54            |
| Non Residential       | False Alarm      | 472       | 2585      | 401            | 84.96      | 15.51      | 5.53        | 84.96           | 15.51           |
| Road Vehicle          | Special Service  | 698       | 3389      | 500            | 71.63      | 14.75      | 6.90        | 71.63           | 14.75           |
| Outdoor               | Fire             | 680       | 1273      | 379            | 55.74      | 29.77      | 5.23        | 55.74           | 29.77           |
| Dwelling              | Special Service  | 4938      | 3389      | 2627           | 53.20      | 77.52      | 36.25       | 53.20           | 77.52           |
| Dwelling              | False Alarm      | 4938      | 2585      | 1821           | 36.88      | 70.44      | 25.13       | 36.88           | 70.44           |
| Outdoor               | Special Service  | 680       | 3389      | 192            | 28.24      | 5.67       | 2.65        | 28.24           | 5.67            |
| Road Vehicle          | Fire             | 698       | 1273      | 158            | 22.64      | 12.41      | 2.18        | 22.64           | 12.41           |
| Outdoor               | False Alarm      | 680       | 2585      | 109            | 16.03      | 4.22       | 1.50        | 16.03           | 4.22            |
| Non Residential       | Special Service  | 472       | 3389      | 51             | 10.81      | 1.50       | 0.70        | 10.81           | 1.50            |
| Dwelling              | Fire             | 4938      | 1273      | 490            | 9.92       | 38.49      | 6.76        | 9.92            | 38.49           |
| Outdoor Structure     | False Alarm      | 230       | 2585      | 19             | 8.26       | 0.74       | 0.26        | 8.26            | 0.74            |
| Other Residential     | Fire             | 229       | 1273      | 17             | 7.42       | 1.34       | 0.23        | 7.42            | 1.34            |
| Other Residential     | Special Service  | 229       | 3389      | 17             | 7.42       | 0.50       | 0.23        | 7.42            | 0.50            |
| Road Vehicle          | False Alarm      | 698       | 2585      | 40             | 5.73       | 1.55       | 0.55        | 5.73            | 1.55            |
| Non Residential       | Fire             | 472       | 1273      | 20             | 4.24       | 1.57       | 0.28        | 4.24            | 1.57            |
| Outdoor Structure     | Special Service  | 230       | 3389      | 2              | 0.87       | 0.06       | 0.03        | 0.87            | 0.06            |


</div>
The itemsets where the support of the IncidentGroup and PropertyCategory itemsets was greater than 1 and Confidence_PC was greater than 50 are present in Tables XXI and XXII. It can be said that in the dataset without outliers and with support above the threshold, there was a clear connection between Dwellings and Road Vehicles requiring Special Services, Outdoor and Outdoor Structures and Fires, and Non-Residential and Other Residential and False Alarms. 



  <div align="center"> TABLE XXI

Association Between IncidentGroup and PropertyCategory 
(with Outliers, Support > 1 & Conf_PC->IG > 50) 

| **PropertyCategory**  | **IncidentGroup** | **Fr_PC** | **Fr_IG** | **Fr_Itemset** | **FrP_PC** | **FrP_IG** | **Support** | **Conf_PC->IG** | **Conf_IG->PC** |
|-----------------------|------------------|-----------|-----------|----------------|------------|------------|-------------|-----------------|-----------------|
| Non Residential       | False Alarm      | 2313      | 5198      | 1710           | 73.93      | 32.90      | 13.30       | 73.93           | 32.90           |
| Outdoor               | Fire             | 1476      | 2777      | 892            | 60.43      | 32.12      | 6.94        | 60.43           | 32.12           |
| Road Vehicle          | Special Service  | 1152      | 4886      | 747            | 64.84      | 15.29      | 5.81        | 64.84           | 15.29           |
| Outdoor Structure     | Fire             | 707       | 2777      | 574            | 81.19      | 20.67      | 4.46        | 81.19           | 20.67           |
| Other Residential     | False Alarm      | 544       | 5198      | 430            | 79.04      | 8.27       | 3.34        | 79.04           | 8.27            |

</div>

  <div align="center"> TABLE XII

Association Between IncidentGroup and PropertyCategory 
(without Outliers, Support > 1 & Conf_PC-IG > 50) 
| **PropertyCategory**  | **IncidentGroup** | **Fr_PC** | **Fr_IG** | **Fr_Itemset** | **FrP_PC** | **FrP_IG** | **Support** | **Conf_PC->IG** | **Conf_IG->PC** |
|-----------------------|------------------|-----------|-----------|----------------|------------|------------|-------------|-----------------|-----------------|
| Dwelling              | Special Service  | 4938      | 3389      | 2627           | 53.20      | 77.52      | 36.25       | 53.20           | 77.52           |
| Road Vehicle          | Special Service  | 698       | 3389      | 500            | 71.63      | 14.75      | 6.90        | 71.63           | 14.75           |
| Non Residential       | False Alarm      | 472       | 2585      | 401            | 84.96      | 15.51      | 5.53        | 84.96           | 15.51           |
| Outdoor               | Fire             | 680       | 1273      | 379            | 55.74      | 29.77      | 5.23        | 55.74           | 29.77           |
| Outdoor Structure     | Fire             | 230       | 1273      | 209            | 90.87      | 16.42      | 2.88        | 90.87           | 16.42           |
| Other Residential     | False Alarm      | 229       | 2585      | 195            | 85.15      | 7.54       | 2.69        | 85.15           | 7.54            |


</div>


#### Geographical and Temporal Feature Analysis
In order to determine whether there was a connection between Bank Holidays, Fires, and False Alarms, a scatter plot was used . Regardless of the outliers, there was no noticeable difference in the number of incidents of any type.

![image](https://github.com/user-attachments/assets/787d48ce-dd26-462c-8aee-1a0a29c087ca)

<div align="center">   Fig. The scatter plot with outliers. </div>


![image](https://github.com/user-attachments/assets/7b655694-aac7-45e3-8723-3c63670d2a7c)

<div align="center"> The scatter plot with outliers.  </div>

As for the geographical features, it was determined that there were fewer fires and false alarms at parks.

#### IncidentStationGround and FirstPumpArriving_DeployedFromStation Analysis
The entries where FirstPumpArriving_DeployedFromStation was not null, including the outliers, were chosen for the analysis and the dataset was split into two sets - one with the IncidentStationGround and FirstPumpArriving_DeployedFromStation matching and the other with them being different. The scatter plots with the locations of the incidents were based on the IncidentStationGround and FirstPumpArriving_DeployedFromStation (Fig. above). A cluster of incidents is visible on the northern part of the border between East Greenwich and Plumstead station grounds. 

 ![image](https://github.com/user-attachments/assets/14f9d35f-b232-4b6b-b81c-81de98408316)

<div align="center"> The scatter plot of the incident locations with the attending stations marked by the Station Ground.  </div>


![image](https://github.com/user-attachments/assets/8fb572b9-c775-4c5a-9d17-ddd7b684e1ce)

<div align="center"> The scatter plot of the incident locations with the attending stations marked. </div>

The description of the numerical features of the datasets did not show a significant difference in most features, except FirstPumpArriving_AttendanceTime where the parameters were from 27 to 100% higher (Table XIII). For the categorical features, only Ward, IncidentStationGround, and FirstPumpArriving_DeployedFromStation were different between the datasets (Table XIV).

<div align="center"> TABLE XIII

Dataset Descriptions (Numeric) 

| **df_same**                                  | **count**  | **mean**    | **std**    | **min**  | **25%**   | **50%**   | **75%**   | **max**     |
|----------------------------------------------|------------|-------------|------------|----------|-----------|-----------|-----------|-------------|
| HourOfCall                                   | 10,120.00  | 13.37       | 6.47       | 0.00     | 9.00      | 14.00     | 19.00     | 23.00       |
| Easting_rounded                              | 10,120.00  | 542,756.04  | 2,569.49   | 537,150  | 540,750   | 543,050   | 544,750   | 547,650     |
| Northing_rounded                             | 10,120.00  | 177,611.96  | 1,839.38   | 171,550  | 176,950   | 178,050   | 178,750   | 181,250     |
| FirstPumpArriving_AttendanceTime             | 10,120.00  | 290.90      | 120.39     | 1.00     | 218.00    | 283.00    | 346.00    | 1,181.00    |
| NumStationsWithPumpsAttending                | 10,120.00  | 1.28        | 0.68       | 1.00     | 1.00      | 1.00      | 1.00      | 7.00        |
| NumPumpsAttending                            | 10,120.00  | 1.50        | 0.88       | 1.00     | 1.00      | 1.00      | 2.00      | 8.00        |
| PumpCount                                    | 10,120.00  | 1.57        | 1.49       | 1.00     | 1.00      | 1.00      | 2.00      | 105.00      |
| PumpHoursRoundUp                             | 10,120.00  | 1.40        | 5.33       | 1.00     | 1.00      | 1.00      | 1.00      | 484.00      |
| NotionalCost                                 | 10,120.00  | 489.91      | 1,934.76   | 333.00   | 339.00    | 352.00    | 364.00    | 176,176.00  |
| NumCalls                                     | 10,120.00  | 1.36        | 1.56       | 1.00     | 1.00      | 1.00      | 1.00      | 48.00       |
| DayOfCall                                    | 10,120.00  | 183.47      | 100.55     | 1.00     | 99.00     | 187.00    | 268.00    | 366.00      |
| Month                                        | 10,120.00  | 6.54        | 3.29       | 1.00     | 4.00      | 7.00      | 9.00      | 12.00       |

| **df_diff**                                  | **count**  | **mean**    | **std**    | **min**  | **25%**   | **50%**   | **75%**   | **max**     |
|----------------------------------------------|------------|-------------|------------|----------|-----------|-----------|-----------|-------------|
| HourOfCall                                   | 2,076.00   | 13.90       | 5.44       | 0.00     | 11.00     | 14.00     | 18.00     | 23.00       |
| Easting_rounded                              | 2,076.00   | 542,116.86  | 2,478.70   | 537,250  | 540,250   | 542,450   | 543,750   | 547,950     |
| Northing_rounded                             | 2,076.00   | 177,095.81  | 1,917.75   | 171,550  | 176,150   | 177,550   | 178,450   | 181,150     |
| FirstPumpArriving_AttendanceTime             | 2,076.00   | 420.78      | 153.02     | 2.00     | 324.00    | 399.50    | 490.00    | 1,181.00    |
| NumStationsWithPumpsAttending                | 2,076.00   | 1.59        | 0.94       | 1.00     | 1.00      | 1.00      | 2.00      | 8.00        |
| NumPumpsAttending                            | 2,076.00   | 1.74        | 1.08       | 1.00     | 1.00      | 1.00      | 2.00      | 8.00        |
| PumpCount                                    | 2,076.00   | 1.80        | 1.25       | 1.00     | 1.00      | 1.00      | 2.00      | 17.00       |
| PumpHoursRoundUp                             | 2,076.00   | 1.46        | 2.15       | 1.00     | 1.00      | 1.00      | 1.00      | 45.00       |
| NotionalCost                                 | 2,076.00   | 511.70      | 751.29     | 333.00   | 346.00    | 352.00    | 364.00    | 16,380.00   |
| NumCalls                                     | 2,076.00   | 1.44        | 1.54       | 1.00     | 1.00      | 1.00      | 1.00      | 28.00       |
| DayOfCall                                    | 2,076.00   | 186.70      | 97.73      | 1.00     | 107.00    | 192.00    | 266.00    | 366.00      |
| Month                                        | 2,076.00   | 6.64        | 3.20       | 1.00     | 4.00      | 7.00      | 9.00      | 12.00       |


 </div>



<div align="center">

TABLE XIV
Dataset Descriptions (Categorical)

| **df_same**                                  | **count**  | **mean**    | **std**    | **min**  | **25%**   | **50%**   | **75%**   | **max**     |
|----------------------------------------------|------------|-------------|------------|----------|-----------|-----------|-----------|-------------|
| HourOfCall                                   | 10,120.00  | 13.37       | 6.47       | 0.00     | 9.00      | 14.00     | 19.00     | 23.00       |
| Easting_rounded                              | 10,120.00  | 542,756.04  | 2,569.49   | 537,150  | 540,750   | 543,050   | 544,750   | 547,650     |
| Northing_rounded                             | 10,120.00  | 177,611.96  | 1,839.38   | 171,550  | 176,950   | 178,050   | 178,750   | 181,250     |
| FirstPumpArriving_AttendanceTime             | 10,120.00  | 290.90      | 120.39     | 1.00     | 218.00    | 283.00    | 346.00    | 1,181.00    |
| NumStationsWithPumpsAttending                | 10,120.00  | 1.28        | 0.68       | 1.00     | 1.00      | 1.00      | 1.00      | 7.00        |
| NumPumpsAttending                            | 10,120.00  | 1.50        | 0.88       | 1.00     | 1.00      | 1.00      | 2.00      | 8.00        |
| PumpCount                                    | 10,120.00  | 1.57        | 1.49       | 1.00     | 1.00      | 1.00      | 2.00      | 105.00      |
| PumpHoursRoundUp                             | 10,120.00  | 1.40        | 5.33       | 1.00     | 1.00      | 1.00      | 1.00      | 484.00      |
| NotionalCost                                 | 10,120.00  | 489.91      | 1,934.76   | 333.00   | 339.00    | 352.00    | 364.00    | 176,176.00  |
| NumCalls                                     | 10,120.00  | 1.36        | 1.56       | 1.00     | 1.00      | 1.00      | 1.00      | 48.00       |
| DayOfCall                                    | 10,120.00  | 183.47      | 100.55     | 1.00     | 99.00     | 187.00    | 268.00    | 366.00      |
| Month                                        | 10,120.00  | 6.54        | 3.29       | 1.00     | 4.00      | 7.00      | 9.00      | 12.00       |

| **df_diff**                                  | **count**  | **mean**    | **std**    | **min**  | **25%**   | **50%**   | **75%**   | **max**     |
|----------------------------------------------|------------|-------------|------------|----------|-----------|-----------|-----------|-------------|
| HourOfCall                                   | 2,076.00   | 13.90       | 5.44       | 0.00     | 11.00     | 14.00     | 18.00     | 23.00       |
| Easting_rounded                              | 2,076.00   | 542,116.86  | 2,478.70   | 537,250  | 540,250   | 542,450   | 543,750   | 547,950     |
| Northing_rounded                             | 2,076.00   | 177,095.81  | 1,917.75   | 171,550  | 176,150   | 177,550   | 178,450   | 181,150     |
| FirstPumpArriving_AttendanceTime             | 2,076.00   | 420.78      | 153.02     | 2.00     | 324.00    | 399.50    | 490.00    | 1,181.00    |
| NumStationsWithPumpsAttending                | 2,076.00   | 1.59        | 0.94       | 1.00     | 1.00      | 1.00      | 2.00      | 8.00        |
| NumPumpsAttending                            | 2,076.00   | 1.74        | 1.08       | 1.00     | 1.00      | 1.00      | 2.00      | 8.00        |
| PumpCount                                    | 2,076.00   | 1.80        | 1.25       | 1.00     | 1.00      | 1.00      | 2.00      | 17.00       |
| PumpHoursRoundUp                             | 2,076.00   | 1.46        | 2.15       | 1.00     | 1.00      | 1.00      | 1.00      | 45.00       |
| NotionalCost                                 | 2,076.00   | 511.70      | 751.29     | 333.00   | 346.00    | 352.00    | 364.00    | 16,380.00   |
| NumCalls                                     | 2,076.00   | 1.44        | 1.54       | 1.00     | 1.00      | 1.00      | 1.00      | 28.00       |
| DayOfCall                                    | 2,076.00   | 186.70      | 97.73      | 1.00     | 107.00    | 192.00    | 266.00    | 366.00      |
| Month                                        | 2,076.00   | 6.64        | 3.20       | 1.00     | 4.00      | 7.00      | 9.00      | 12.00       |

 </div>

The distribution of the categorical variables was similar between the datasets, not counting the outliers 

![image](https://github.com/user-attachments/assets/65c49751-824e-431d-be98-461931973379)

 <div align="center"> The histograms of the categorical features. </div>

No significant connection between the dates of the incidents and the target features was discovered (Fig. 3.6).

![image](https://github.com/user-attachments/assets/0966bbf3-5c90-48ee-86ed-76b22fc5cc0b)

<div align="center">The scatter plot of incident dates and false alarms. </div>

The association analysis between IncidentStationGround and FirstPumpArriving_DeployedFromStation showed that while the conditional probability for the specific values of FirstPumpArriving_DeployedFromStation given certain values of IncidentStationGround reached 100%, the support remained low. Overall, the IncidentStationGround -> FirstPumpArriving_DeployedFromStation confidence was mostly below 1%, while the opposite was mostly related to the Station Grounds outside of the borough, thus not represented adequately in the dataset.
<div align="center">
TABLE XV
Association Between IncidentStationGround and FirstPumpArriving_DeployedFromStation 
(with Outliers, Conf_ISG->FP > 50 or Conf_FP->ISG > 50) 

| **IncidentStation Ground** | **FirstPumpArriving_ DeployedFromStation** | **Fr_ISG** | **Fr_FP** | **Fr_Itemset** | **FrP_ISG** | **FrP_FP** | **Support** | **Conf_ISG->FP** | **Conf_FP->ISG** |
|----------------------------|--------------------------------------------|------------|-----------|----------------|-------------|------------|-------------|-----------------|-----------------|
| Plumstead                  | Erith                                       | 4,911      | 174       | 173            | 3.52        | 99.43      | 1.42        | 3.52            | 99.43           |
| Eltham                     | Sidcup                                      | 2,045      | 180       | 163            | 7.97        | 90.56      | 1.34        | 7.97            | 90.56           |
| Greenwich                  | Lewisham                                    | 1,032      | 103       | 55             | 5.33        | 53.40      | 0.45        | 5.33            | 53.40           |
| Plumstead                  | Bexley                                      | 4,911      | 36        | 31             | 0.63        | 86.11      | 0.25        | 0.63            | 86.11           |
| East Greenwich             | Poplar                                      | 3,232      | 28        | 27             | 0.84        | 96.43      | 0.22        | 0.84            | 96.43           |
| Greenwich                  | New Cross                                   | 1,032      | 21        | 15             | 1.45        | 71.43      | 0.12        | 1.45            | 71.43           |
| Erith                      | Plumstead                                   | 4          | 4,681     | 4              | 100.00      | 0.09       | 0.03        | 100.00          | 0.09            |
| East Greenwich             | Millwall                                    | 3,232      | 5         | 3              | 0.09        | 60.00      | 0.02        | 0.09            | 60.00           |
| Eltham                     | Bromley                                     | 2,045      | 3         | 3              | 0.15        | 100.00     | 0.02        | 0.15            | 100.00          |
| East Greenwich             | Plaistow                                    | 3,232      | 2         | 2              | 0.06        | 100.00     | 0.02        | 0.06            | 100.00          |
| East Greenwich             | Dockhead                                    | 3,232      | 1         | 1              | 0.03        | 100.00     | 0.01        | 0.03            | 100.00          |
| Plumstead                  | Woodside                                    | 4,911      | 1         | 1              | 0.02        | 100.00     | 0.01        | 0.02            | 100.00          |
| East Greenwich             | Northolt                                    | 3,232      | 1         | 1              | 0.03        | 100.00     | 0.01        | 0.03            | 100.00          |
| East Greenwich             | Bethnal Green                               | 3,232      | 1         | 1              | 0.03        | 100.00     | 0.01        | 0.03            | 100.00          |
| East Greenwich             | Homerton                                    | 3,232      | 1         | 1              | 0.03        | 100.00     | 0.01        | 0.03            | 100.00          |
| Greenwich                  | Old Kent Road                               | 1,032      | 1         | 1              | 0.10        | 100.00     | 0.01        | 0.10            | 100.00          |

</div>


#### PumpCount Statistical Analysis
The objective was to determine which features affect PumpCount and how many pumps were necessary for a station per day, thus the data was grouped by DayOfYear, Year, and IncidentStationGround. The resulting information is present in Tables XVI and XVII, with and without the outliers. The means were 4 and 3, while the medians were 3 and 2, which allowed to conclude that the minimal number of pumps per station should be 4. 

 <div align="center"> TABLE XVI

PumpCount Description with Outliers

|                | **DayOfCall** | **Year** | **PumpCount** |
|----------------|---------------|----------|---------------|
| **Count**      | 5,282         | 5,282    | 5,282         |
| **Mean**       | 181.71        | 2020.49  | 3.82          |
| **Std**        | 102.38        | 1.13     | 3.49          |
| **Min**        | 1             | 2019     | 0             |
| **25%**        | 94            | 2019     | 2             |
| **50% (Median)**| 183          | 2021     | 3             |
| **75%**        | 269           | 2021     | 5             |
| **Max**        | 366           | 2022     | 105           |

</div>

<div align="center"> TABLE XVII 
PumpCount Description without Outliers

|                | **DayOfCall** | **Year** | **PumpCount** |
|----------------|---------------|----------|---------------|
| **Count**      | 3,850         | 3,850    | 3,850         |
| **Mean**       | 175.36        | 2020.33  | 2.66          |
| **Std**        | 104.93        | 1.06     | 1.88          |
| **Min**        | 1             | 2019     | 0             |
| **25%**        | 85            | 2019     | 1             |
| **50% (Median)**| 171          | 2020     | 2             |
| **75%**        | 266           | 2021     | 3             |
| **Max**        | 366           | 2022     | 13            |


 </div>

 The features, after One Hot Encoding had been applied to them, listed in Table XVIII had the strongest connection to PumpCount. It could be assumed that the automatic false alarms affected the number of pumps, while the NumPumpsAttending, NumStationsWithPumpsAttending, and PumpHoursRoundUp had a direct connection to PumpCount, but no causation. Special Service, on the other hand, correlated negatively with PumpCount. Second pumps more often deployed from Plumstead was an interesting observation. 

<div align="center"> TABLE XVIII
PumpCount Correlation after One Hot Encoding

| **Feature**                                                    | **Correlation** |
|--------------------------------------------------------------- |-----------------|
| StopCodeDescription_Special Service                            | -0.456083       |
| IncidentGroup_Special Service                                  | -0.456083       |
| PropertyCategory_Non Residential                               | -0.129292       |
| StopCodeDescription_Secondary Fire                             | -0.122753       |
| PropertyCategory_Outdoor                                       | -0.115533       |
| AddressQualifier_Open land/water - nearest gazetteer location   | -0.093454       |
| PropertyType_Infant/Primary school                             | -0.092524       |
| PumpHoursRoundUp                                               | 0.360011        |
| StopCodeDescription_AFA                                        | 0.388027        |
| IncidentGroup_False Alarm                                      | 0.413127        |
| SecondPumpArriving_DeployedFromStation_Plumstead               | 0.49605         |
| NumStationsWithPumpsAttending                                  | 0.692839        |
| NumPumpsAttending                                              | 0.956851        |
| PumpCount                                                      | 1               |

</div>

### Model Parameters
The reasoning for the data selection for the descriptive modelling was explained in the previous subsection. It could be added that the colour scheme for the figures was based on the Tableau default colours for clearer visibility and consistency between the applications. 
Only the first business problem applied predictive modelling, thus this subsection describes the hyperparameters of the models used for each target value. 
The GridCV function of the scikit-learn was considered for hyperparameter tuning but decided against due to the computational costs. Instead, the for loops were used. For the Decision Tree Classifiers, the values of the criterion, splitter, maximum depth, and maximum leaf nodes were adjusted. The minimum number of samples per leaf for the Fire model was set to 10% of the testing set, 161. The resulting performance of the models is shown in Table XIX. F1 score was prioritised over the accuracy, thus the Gini impurity criterion, the best splitter, and the maximum number of leaf nodes equal to 4 were chosen.

<div align="center">
TABLE XIX
The Fire IncidentGroup Decision Tree Classifier Hyperparameter Tuning

| **Evaluation Metric** | **Value** | **Criterion** | **Splitter** | **Max Leaf Nodes** | **Evaluation Metric** | **Value** | **Criterion** | **Splitter** | **Max Leaf Nodes** |
|-----------------------|-----------|---------------|--------------|--------------------|-----------------------|-----------|---------------|--------------|--------------------|
| Accuracy              | 0.7922    | log_loss      | random       | 8                  | F1 score              | 0.2983    | log_loss      | random       | 8                  |
| Accuracy              | 0.8027    | entropy       | random       | 8                  | F1 score              | 0.4008    | entropy       | random       | 8                  |
| Accuracy              | 0.8345    | gini          | best         | None               | F1 score              | 0.6044    | entropy       | random       | 16                 |
| Accuracy              | 0.8444    | entropy       | random       | 16                 | F1 score              | 0.6210    | gini          | random       | None               |
| Accuracy              | 0.8463    | gini          | random       | 4                  | F1 score              | 0.6210    | gini          | random       | 8                  |
| Accuracy              | 0.8463    | entropy       | random       | 4                  | F1 score              | 0.6210    | gini          | random       | 16                 |
| Accuracy              | 0.8519    | gini          | random       | None               | F1 score              | 0.6210    | entropy       | random       | None               |
| Accuracy              | 0.8519    | gini          | random       | 8                  | F1 score              | 0.6210    | log_loss      | random       | 16                 |
| Accuracy              | 0.8519    | gini          | random       | 16                 | F1 score              | 0.6243    | gini          | best         | None               |
| Accuracy              | 0.8519    | entropy       | random       | None               | F1 score              | 0.6263    | gini          | random       | 4                  |
| Accuracy              | 0.8519    | log_loss      | random       | 16                 | F1 score              | 0.6263    | entropy       | random       | 4                  |
| Accuracy              | 0.8575    | gini          | best         | 8                  | F1 score              | 0.6577    | gini          | best         | 8                  |
| Accuracy              | 0.8575    | entropy       | best         | None               | F1 score              | 0.6577    | entropy       | best         | None               |
| Accuracy              | 0.8575    | entropy       | best         | 8                  | F1 score              | 0.6577    | entropy       | best         | 8                  |
| Accuracy              | 0.8575    | entropy       | best         | 16                 | F1 score              | 0.6577    | entropy       | best         | 16                 |
| Accuracy              | 0.8575    | log_loss      | best         | None               | F1 score              | 0.6577    | log_loss      | best         | None               |
| Accuracy              | 0.8575    | log_loss      | best         | 8                  | F1 score              | 0.6577    | log_loss      | best         | 8                  |
| Accuracy              | 0.8575    | log_loss      | best         | 16                 | F1 score              | 0.6577    | log_loss      | best         | 16                 |
| Accuracy              | 0.8587    | log_loss      | random       | None               | F1 score              | 0.6637    | log_loss      | random       | None               |
| Accuracy              | 0.8643    | log_loss      | random       | 4                  | F1 score              | 0.6930    | log_loss      | random       | 4                  |
| Accuracy              | 0.8712    | gini          | best         | 4                  | F1 score              | 0.7107    | gini          | best         | None               |
| Accuracy              | 0.8712    | entropy       | best         | 4                  | F1 score              | 0.7107    | gini          | best         | 16                 |
| Accuracy              | 0.8712    | log_loss      | best         | 4                  | F1 score              | 0.7266    | gini          | best         | 4                  |
| Accuracy              | 0.8718    | gini          | best         | None               | F1 score              | 0.7266    | entropy       | best         | 4                  |
| Accuracy              | 0.8718    | gini          | best         | 16                 | F1 score              | 0.7266    | log_loss      | best         | 4                  |

</div>
The model’s performance with a range of maximum depths was visualised, demonstrating the lowest and most reliable maximum depth of the tree, 3 (Fig.below).

![image](https://github.com/user-attachments/assets/4f572575-5d08-49f5-8818-ec05cc10d1b1)

<div align="center">
Fig. The change in the accuracy (blue) and the f1 score (orange) with the maximum depth (x) of the Fire IncidentGroup Decision Tree.</div>

The resulting model used the DayOfCall (April 11th and April 12th) and the NumCalls (less than 3) features to determine whether an incident was an actual fire (Fig. below). No attempt was made to retrain the model while preserving the temporal features as integers instead of one-hot-encoded categorical due to the time constraints.


![image](https://github.com/user-attachments/assets/2dc202d1-4bba-4b92-a461-5071cfef900a)
<div align="center">
Fig.  The Fire IncidentGroup Decision Tree visualisation
</div>

For the second model for the Fire IncidentGroup target feature, Extreme Gradient Boosting Classifier, the values of the booster, objective, and target evaluation metric were adjusted (Table XX). The linear booster, the objective of hinge loss for binary classification, and the area under the PR curve evaluation metric were chosen. 
<div align="center">
TABLE XX
The Fire IncidentGroup Extreme Gradient Boosting Classifier Hyperparameter Tuning

| **Evaluation Metric** | **Value** | **Booster** | **Objective** | **Evaluation Metric** | **Evaluation Metric** | **Value** | **Booster** | **Objective** | **Evaluation Metric** |
|-----------------------|-----------|-------------|---------------|-----------------------|-----------------------|-----------|-------------|---------------|-----------------------|
| Accuracy              | 0.8805    | gbtree      | binary:hinge  | error                 | F1-Score              | 0.7181    | gblinear    | binary:logitraw | error                 |
| Accuracy              | 0.8805    | gbtree      | binary:hinge  | auc                   | F1-Score              | 0.7181    | gblinear    | binary:logitraw | auc                   |
| Accuracy              | 0.8805    | gbtree      | binary:hinge  | aucpr                 | F1-Score              | 0.7181    | gblinear    | binary:logitraw | aucpr                 |
| Accuracy              | 0.8805    | dart        | binary:hinge  | error                 | F1-Score              | 0.7212    | gbtree      | binary:logitraw | error                 |
| Accuracy              | 0.8805    | dart        | binary:hinge  | auc                   | F1-Score              | 0.7212    | gbtree      | binary:logitraw | auc                   |
| Accuracy              | 0.8805    | dart        | binary:hinge  | aucpr                 | F1-Score              | 0.7212    | gbtree      | binary:logitraw | aucpr                 |
| Accuracy              | 0.8811    | gbtree      | binary:logitraw | error               | F1-Score              | 0.7212    | dart        | binary:logitraw | error                 |
| Accuracy              | 0.8811    | gbtree      | binary:logitraw | auc                 | F1-Score              | 0.7212    | dart        | binary:logitraw | auc                   |
| Accuracy              | 0.8811    | gbtree      | binary:logitraw | aucpr               | F1-Score              | 0.7212    | dart        | binary:logitraw | aucpr                 |
| Accuracy              | 0.8811    | dart        | binary:logitraw | error               | F1-Score              | 0.7265    | gbtree      | binary:hinge    | error                 |
| Accuracy              | 0.8811    | dart        | binary:logitraw | auc                 | F1-Score              | 0.7265    | gbtree      | binary:hinge    | auc                   |
| Accuracy              | 0.8811    | dart        | binary:logitraw | aucpr               | F1-Score              | 0.7265    | gbtree      | binary:hinge    | aucpr                 |
| Accuracy              | 0.8818    | gblinear    | binary:logitraw | error               | F1-Score              | 0.7265    | dart        | binary:hinge    | error                 |
| Accuracy              | 0.8818    | gblinear    | binary:logitraw | auc                 | F1-Score              | 0.7265    | dart        | binary:hinge    | auc                   |
| Accuracy              | 0.8818    | gblinear    | binary:logitraw | aucpr               | F1-Score              | 0.7265    | dart        | binary:hinge    | aucpr                 |
| Accuracy              | 0.8855    | gbtree      |               | F1-Score              | 0.7379    | gbtree      |               |                 |
| Accuracy              | 0.8855    | gbtree      | binary:logistic | error               | F1-Score              | 0.7379    | gbtree      | binary:logistic | error                 |
| Accuracy              | 0.8855    | gbtree      | binary:logistic | auc                 | F1-Score              | 0.7379    | gbtree      | binary:logistic | auc                   |
| Accuracy              | 0.8855    | gbtree      | binary:logistic | aucpr               | F1-Score              | 0.7379    | gbtree      | binary:logistic | aucpr                 |
| Accuracy              | 0.8855    | dart        | binary:logistic | error               | F1-Score              | 0.7379    | dart        | binary:logistic | error                 |
| Accuracy              | 0.8855    | dart        | binary:logistic | auc                 | F1-Score              | 0.7379    | dart        | binary:logistic | auc                   |
| Accuracy              | 0.8855    | dart        | binary:logistic | aucpr               | F1-Score              | 0.7379    | dart        | binary:logistic | aucpr                 |
| Accuracy              | 0.8880    | gblinear    | binary:hinge    | auc                 | F1-Score              | 0.7421    | gblinear    | binary:hinge    | auc                   |
| Accuracy              | 0.8886    | gblinear    | binary:logistic | error               | F1-Score              | 0.7424    | gblinear    | binary:logistic | error                 |
| Accuracy              | 0.8886    | gblinear    | binary:logistic | auc                 | F1-Score              | 0.7424    | gblinear    | binary:logistic | auc                   |
| Accuracy              | 0.8886    | gblinear    | binary:logistic | aucpr               | F1-Score              | 0.7424    | gblinear    | binary:logistic | aucpr                 |
| Accuracy              | 0.8892    | gblinear    | binary:hinge    | error               | F1-Score              | 0.7450    | gblinear    | binary:hinge    | error                 |
| Accuracy              | 0.8899    | gblinear    | binary:hinge    | aucpr               | F1-Score              | 0.7461    | gblinear    | binary:hinge    | aucpr                 |
</div>

For the other target feature, the False Alarm IncidentGroup, the Gini impurity criterion, the best splitter, and the unlimited maximum number of leaf nodes were chosen (Table XXI).
<div align="center">
TABLE XXI
 The False Alarm IncidentGroup Decision Tree Classifier Hyperparameter Tuning

| **Evaluation Metric** | **Value** | **Criterion** | **Splitter** | **Max Leaf Nodes** | **Evaluation Metric** | **Value** | **Criterion** | **Splitter** | **Max Leaf Nodes** |
|-----------------------|-----------|---------------|--------------|--------------------|-----------------------|-----------|---------------|--------------|--------------------|
| Accuracy              | 0.6503    | gini          | best         | None               | F1-Score              | 0.5301    | gini          | best         | None               |
| Accuracy              | 0.7293    | gini          | best         | None               | F1-Score              | 0.5484    | gini          | best         | 4                  |
| Accuracy              | 0.7293    | entropy       | best         | None               | F1-Score              | 0.5484    | gini          | best         | 8                  |
| Accuracy              | 0.7293    | log_loss      | best         | None               | F1-Score              | 0.5484    | gini          | best         | 16                 |
| Accuracy              | 0.7386    | gini          | best         | 4                  | F1-Score              | 0.5484    | gini          | random       | 4                  |
| Accuracy              | 0.7386    | gini          | best         | 8                  | F1-Score              | 0.5484    | gini          | random       | 8                  |
| Accuracy              | 0.7386    | gini          | best         | 16                 | F1-Score              | 0.5484    | gini          | random       | 16                 |
| Accuracy              | 0.7386    | gini          | random       | None               | F1-Score              | 0.5484    | entropy       | best         | 4                  |
| Accuracy              | 0.7386    | gini          | random       | 4                  | F1-Score              | 0.5484    | entropy       | best         | 8                  |
| Accuracy              | 0.7386    | gini          | random       | 8                  | F1-Score              | 0.5484    | entropy       | best         | 16                 |
| Accuracy              | 0.7386    | gini          | random       | 16                 | F1-Score              | 0.5484    | entropy       | random       | None               |
| Accuracy              | 0.7386    | entropy       | best         | 4                  | F1-Score              | 0.5484    | entropy       | random       | 4                  |
| Accuracy              | 0.7386    | entropy       | best         | 8                  | F1-Score              | 0.5484    | entropy       | random       | 8                  |
| Accuracy              | 0.7386    | entropy       | best         | 16                 | F1-Score              | 0.5484    | entropy       | random       | 16                 |
| Accuracy              | 0.7386    | entropy       | random       | None               | F1-Score              | 0.5484    | log_loss      | best         | 4                  |
| Accuracy              | 0.7386    | entropy       | random       | 4                  | F1-Score              | 0.5484    | log_loss      | best         | 8                  |
| Accuracy              | 0.7386    | entropy       | random       | 8                  | F1-Score              | 0.5484    | log_loss      | best         | 16                 |
| Accuracy              | 0.7386    | entropy       | random       | 16                 | F1-Score              | 0.5484    | log_loss      | random       | None               |
| Accuracy              | 0.7386    | log_loss      | best         | 4                  | F1-Score              | 0.5484    | log_loss      | random       | 4                  |
| Accuracy              | 0.7386    | log_loss      | best         | 8                  | F1-Score              | 0.5484    | log_loss      | random       | 8                  |
| Accuracy              | 0.7386    | log_loss      | best         | 16                 | F1-Score              | 0.5484    | log_loss      | random       | 16                 |
| Accuracy              | 0.7386    | log_loss      | random       | None               | F1-Score              | 0.5588    | gini          | random       | None               |
| Accuracy              | 0.7386    | log_loss      | random       | 4                  | F1-Score              | 0.5637    | gini          | best         | None               |
| Accuracy              | 0.7386    | log_loss      | random       | 8                  | F1-Score              | 0.5637    | entropy       | best         | None               |
| Accuracy              | 0.7386    | log_loss      | random       | 16                 | F1-Score              | 0.5637    | log_loss      | best         | None               |


</div>
The model’s performance with a range of maximum depths was visualised, demonstrating that the f1 score reaches the highest at the maximum depth of the tree 10 (Fig. below).
<div align="center">

![image](https://github.com/user-attachments/assets/7f10cf25-9957-40fb-8568-9c31fa258e6b)
Fig.  The change in the accuracy (blue) and the f1 score (orange) with the maximum depth (x) of the False Alarm IncidentGroup Decision Tree.
</div>

The resulting model used the DayOfCall, the NumCalls, and the geographical (Easting_rounded and Northing_rounded) features to determine whether an incident was an actual fire (Fig. below).

![image](https://github.com/user-attachments/assets/59c9b376-a89e-499c-a2df-c475433fd375)
<div align="center">
Fig. The False Alarm IncidentGroup Decision Tree visualisation.
</div>

======= 


## Software

Google Colaboratory

Google Spreadsheets

Google Docs

### python libraries 

The Python packages applied were Pandas and Numpy for statistical analysis 

Matplotlib and Seaborn for exploratory data analysis and visualisation

scikit-learn and XGBoost for pre-processing, modelling, and evaluation 

openpyxl to load the dataset.

The machine learning algorithms used include Decision Tree, Logistic Regression, Random Forest, and Extreme Gradient Boosting. 
