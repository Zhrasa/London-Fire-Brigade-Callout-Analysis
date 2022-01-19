# London Fire Brigade Analysis
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
* Firstly, Numerical ID features (UPRN and USRN) were transformed into the object type.
### Null/Missing Values
* Secondly, the partially missing (SpecialServiceType, Postcode_full), duplicate (IncGeo_BoroughName, IncGeo_BoroughCode, IncGeo_WardCode, IncGeo_WardName) or single-value (FRS; ProperCase was renamed to Borough and removed later) features and ID features (IncidentNumber, USRN, UPRN) were removed
The Year feature was dropped, while the DatetimeOfCall was used only for descriptive modelling.

To ensure that the correct datatypes were allocated to each variable `Easting_rounded`, `Northing_rounded` and `HourOfCall` needed converting to categorical types. From the numerical histograms, it can be seen that most of the incidents, including the false alarms and special services, happened between 15:00 and 20:00 hours. The Easting_rounded and Northing_Rounded histograms are slightly skewed to the right. The Attendance Time for the First Pump Arriving was around 6 minutes on average.The DatetimeOfCall histogram demonstrates a small increase in the number of calls during the period observed. The DayOfCall and Month histograms indicate that there were slightly more calls during summer. The rest of the histograms are heavily skewed to the left.

![image](https://github.com/Zhrasa/London-Fire-Brigade-Callout-Analysis/assets/99383300/b2fe713c-6f02-427b-83f3-2542b3a687fa)
Fig. The histograms of the numerical and temporal features. 

![The histograms of the categorical features.](https://github.com/Zhrasa/London-Fire-Brigade-Callout-Analysis/assets/99383300/f3dcb8f6-8124-404d-8351-7b9d1660aee8)
Fig The histograms of the categorical features.


 ![image](https://github.com/Zhrasa/London-Fire-Brigade-Callout-Analysis/assets/99383300/09fe2fd2-2f66-4b52-bb9f-c211d452b0f0)

Fig. The scatter plot of fires and false alarms.

======= 


## Software

Google Colaboratory
Google Spreadsheets
Google Docs

### python libraries 

The Python packages applied were 
Pandas and Numpy for statistical analysis
Matplotlib and Seaborn for exploratory data analysis and visualisation
scikit-learn and XGBoost for pre-processing, modelling, and evaluation
openpyxl to load the dataset.
The machine learning algorithms used include Decision Tree, Logistic Regression, Random Forest, and Extreme Gradient Boosting. 
