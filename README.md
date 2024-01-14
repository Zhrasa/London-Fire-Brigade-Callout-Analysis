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

To ensure that the correct datatypes were allocated to each variable `Easting_rounded`, `Northing_rounded` and `HourOfCall` needed converting to categorical types.


![alt text](image.png)

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
