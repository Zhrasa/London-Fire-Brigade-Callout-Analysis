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
| SelectKBest, f_classif  |  178  |   |   |
|  SelectKBest, mutual_info_regression |  178  |   |   |
|  RFE | 178  |   |   |
| RFE  |  18  |   |   |

</div>

		
SelectKBest, f_classif	178	0.7417548226509023	0.6153846153846154
SelectKBest, mutual_info_regression	178	0.6969508400746733	0.5616561656165616
RFE	178	0.7367766023646546	0.5774225774225774
RFE	18	0.7392657125077785	0.5814185814185814

The hyperparameter tuning is described in the following subsection, Model Parameters. 

3.1.2. PropertyCategory and IncidentGroup Association Analysis
Tables XIX and XX demonstrate the association between the IncidentGroup and PropertyCategory features. It was assumed that the data had been entered correctly and the outliers were natural, as opposed to data entry errors.




TABLE XIX
Association Between IncidentGroup and PropertyCategory (with Outliers)
PropertyCategory	IncidentGroup	Fr_PC	Fr_IG	Fr_Itemset	FrP_PC	FrP_IG	Support	Conf_PC->IG	Conf_IG->PC
Aircraft	Special Service	1	4886	1	100.00	0.02	0.01	100.00	0.02
Outdoor Structure	Fire	707	2777	574	81.19	20.67	4.46	81.19	20.67
Other Residential	False Alarm	544	5198	430	79.04	8.27	3.34	79.04	8.27
Non Residential	False Alarm	2313	5198	1710	73.93	32.90	13.30	73.93	32.90
Road Vehicle	Special Service	1152	4886	747	64.84	15.29	5.81	64.84	15.29
Outdoor	Fire	1476	2777	892	60.43	32.12	6.94	60.43	32.12
Boat	Special Service	5	4886	3	60.00	0.06	0.02	60.00	0.06
Rail Vehicle	Special Service	7	4886	4	57.14	0.08	0.03	57.14	0.08
Dwelling	Special Service	6656	4886	3290	49.43	67.34	25.58	49.43	67.34
Dwelling	False Alarm	6656	5198	2642	39.69	50.83	20.54	39.69	50.83
Rail Vehicle	Fire	7	2777	2	28.57	0.07	0.02	28.57	0.07
Road Vehicle	Fire	1152	2777	317	27.52	11.42	2.46	27.52	11.42
Outdoor	Special Service	1476	4886	336	22.76	6.88	2.61	22.76	6.88
Boat	False Alarm	5	5198	1	20.00	0.02	0.01	20.00	0.02
Boat	Fire	5	2777	1	20.00	0.04	0.01	20.00	0.04
Outdoor	False Alarm	1476	5198	248	16.80	4.77	1.93	16.80	4.77
Non Residential	Special Service	2313	4886	377	16.30	7.72	2.93	16.30	7.72
Rail Vehicle	False Alarm	7	5198	1	14.29	0.02	0.01	14.29	0.02
Other Residential	Special Service	544	4886	73	13.42	1.49	0.57	13.42	1.49
Outdoor Structure	False Alarm	707	5198	78	11.03	1.50	0.61	11.03	1.50
Dwelling	Fire	6656	2777	724	10.88	26.07	5.63	10.88	26.07
Non Residential	Fire	2313	2777	226	9.77	8.14	1.76	9.77	8.14
Outdoor Structure	Special Service	707	4886	55	7.78	1.13	0.43	7.78	1.13
Road Vehicle	False Alarm	1152	5198	88	7.64	1.69	0.68	7.64	1.69
Other Residential	Fire	544	2777	41	7.54	1.48	0.32	7.54	1.48
  
TABLE XX
Association Between IncidentGroup and PropertyCategory (without Outliers)
PropertyCategory	IncidentGroup	Fr_PC	Fr_IG	Fr_Itemset	FrP_PC	FrP_IG	Support	Conf_PC->IG	Conf_IG->PC
Outdoor Structure	Fire	230	1273	209	90.87	16.42	2.88	90.87	16.42
Other Residential	False Alarm	229	2585	195	85.15	7.54	2.69	85.15	7.54
Non Residential	False Alarm	472	2585	401	84.96	15.51	5.53	84.96	15.51
Road Vehicle	Special Service	698	3389	500	71.63	14.75	6.90	71.63	14.75
Outdoor	Fire	680	1273	379	55.74	29.77	5.23	55.74	29.77
Dwelling	Special Service	4938	3389	2627	53.20	77.52	36.25	53.20	77.52
Dwelling	False Alarm	4938	2585	1821	36.88	70.44	25.13	36.88	70.44
Outdoor	Special Service	680	3389	192	28.24	5.67	2.65	28.24	5.67
Road Vehicle	Fire	698	1273	158	22.64	12.41	2.18	22.64	12.41
Outdoor	False Alarm	680	2585	109	16.03	4.22	1.50	16.03	4.22
Non Residential	Special Service	472	3389	51	10.81	1.50	0.70	10.81	1.50
Dwelling	Fire	4938	1273	490	9.92	38.49	6.76	9.92	38.49
Outdoor Structure	False Alarm	230	2585	19	8.26	0.74	0.26	8.26	0.74
Other Residential	Fire	229	1273	17	7.42	1.34	0.23	7.42	1.34
Other Residential	Special Service	229	3389	17	7.42	0.50	0.23	7.42	0.50
Road Vehicle	False Alarm	698	2585	40	5.73	1.55	0.55	5.73	1.55
Non Residential	Fire	472	1273	20	4.24	1.57	0.28	4.24	1.57
Outdoor Structure	Special Service	230	3389	2	0.87	0.06	0.03	0.87	0.06

The itemsets where the support of the IncidentGroup and PropertyCategory itemsets was greater than 1 and Confidence_PC was greater than 50 are present in Tables XXI and XXII. It can be said that in the dataset without outliers and with support above the threshold, there was a clear connection between Dwellings and Road Vehicles requiring Special Services, Outdoor and Outdoor Structures and Fires, and Non-Residential and Other Residential and False Alarms. 

TABLE XXI
Association Between IncidentGroup and PropertyCategory 
(with Outliers, Support > 1 & Conf_PC->IG > 50) 
PropertyCategory	IncidentGroup	Fr_PC	Fr_IG	Fr_Itemset	FrP_PC	FrP_IG	Support	Conf_PC->IG	Conf_IG->PC
Non Residential	False Alarm	2313	5198	1710	73.93	32.90	13.30	73.93	32.90
Outdoor	Fire	1476	2777	892	60.43	32.12	6.94	60.43	32.12
Road Vehicle	Special Service	1152	4886	747	64.84	15.29	5.81	64.84	15.29
Outdoor Structure	Fire	707	2777	574	81.19	20.67	4.46	81.19	20.67
Other Residential	False Alarm	544	5198	430	79.04	8.27	3.34	79.04	8.27

TABLE XXII
Association Between IncidentGroup and PropertyCategory 
(without Outliers, Support > 1 & Conf_PC-IG > 50) 
PropertyCategory	IncidentGroup	Fr_PC	Fr_IG	Fr_Itemset	FrP_PC	FrP_IG	Support	Conf_PC->IG	Conf_IG->PC
Dwelling	Special Service	4938	3389	2627	53.20	77.52	36.25	53.20	77.52
Road Vehicle	Special Service	698	3389	500	71.63	14.75	6.90	71.63	14.75
Non Residential	False Alarm	472	2585	401	84.96	15.51	5.53	84.96	15.51
Outdoor	Fire	680	1273	379	55.74	29.77	5.23	55.74	29.77
Outdoor Structure	Fire	230	1273	209	90.87	16.42	2.88	90.87	16.42
Other Residential	False Alarm	229	2585	195	85.15	7.54	2.69	85.15	7.54

3.1.3. Geographical and Temporal Feature Analysis
In order to determine whether there was a connection between Bank Holidays, Fires, and False Alarms, a scatter plot was used (Fig. 3.1, 3.2). Regardless of the outliers, there was no noticeable difference in the number of incidents of any type.

 
Fig. 3.1. The scatter plot with outliers.










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