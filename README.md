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

<div align="center"> TABLE I </div>
<div align="center">RFE Feature Importance for the Fire IncidentGroup</div>

DayOfCall	0.28
PropertyCategory	0.25
PropertyType	0.09
HourOfCall	0.09
Ward	0.07
Easting_rounded	0.05
NumCalls	0.05
Northing_rounded	0.04
IncidentStationGround	0.03
Postcode_district	0.03
Month	0.02
AddressQualifier	0.01


|** Feature ** | **Importance (Sum)** |
|--------------|--------------|
| Row 1, Col 1 | Row 1, Col 2 |
| Row 2, Col 1 | Row 2, Col 2 |
| Row 3, Col 1 | Row 3, Col 2 |
| Row 4, Col 1 | Row 4, Col 2 |
| Row 5, Col 1 | Row 5, Col 2 |
| Row 6, Col 1 | Row 6, Col 2 |
| Row 7, Col 1 | Row 7, Col 2 |
| Row 8, Col 1 | Row 8, Col 2 |
| Row 9, Col 1 | Row 9, Col 2 |
| Row 10, Col 1| Row 10, Col 2|


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
