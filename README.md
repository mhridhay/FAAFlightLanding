# FAAFlightLanding

# FAA Dataset

**Background**: Flight landing  
**Motivation**: To reduce the risk of landing overrun  
**Goal**: To study what factors and how they would impact the landing distance of a commercial flight    
**Data**: FAA  

## Exploratory Data Analysis

### 1. Variable Dictionary

|Name|Description|
|:-:|:----|
|Aircraft| The make of an aircraft (Boeing or Airbus). |
|Duration (in minutes)| Flight duration between taking off and landing. The duration of a normal flight should always be greater than 40min.  |
|No_pasg| The number of passengers in a flight.  |
|Speed_ground (in miles per hour)| The ground speed of an aircraft when passing over the threshold of the runway. If its value is less than 30MPH or greater than 140MPH, then the landing would be considered as abnormal.  |
|Speed_air (in miles per hour)| The air speed of an aircraft when passing over the threshold of the runway. If its value is less than 30MPH or greater than 140MPH, then the landing would be considered as abnormal.  |
|Height (in meters)| The height of an aircraft when it is passing over the threshold of the runway. The landing aircraft is required to be at least 6 meters high at the threshold of the runway.  |
|Pitch (in degrees)| Pitch angle of an aircraft when it is passing over the threshold of the runway.  |
|Distance (in feet)| The landing distance of an aircraft. More specifically, it refers to the distance between the threshold of the runway and the point where the aircraft can be fully stopped. The length of the airport runway is typically less than 6000 feet. |

### 2. Initial exploration of the data
 
  1. The *duration* variable was missing in dataset **faa2**  
  2. We have *950 observations and 8 variables* in merged dataset  
  3. No duplicates were found in the merged dataset, so we dont need to drop any observation  
  4. Minimum value for *height* is negative  
  5. There are a lot of missing values for *speed_air* and *duration*  
  
### 3. Data Cleaning and further exploration  

  1. Found abnormal values for *duration*, *speed_ground*,*height* and *distance*  
  2. *23 abnormal* observations were filtered out  
  3. Histograms show all variables to be normally distributed except *distance* and *speed_air*  
  4. Mean and median values for *distance* significantly differ 
  5. We have nearly equal number of observations for both type of aircrafts  
  
## Linear Regression  

### 1. Factors impacting “landing distance”  

Combined scatter plot and correlation data is seen to match.

### 2. Single-factor Regression  

  1. Rank regression results for landing distance on each of the variables
  2. Rank regression results for landing distance on each of the normalized variables  
  3. Comparing the results:  
    1. Correlation for *speed_ground* and *speed_air* are very high (model1)  
    2. p_values indicate *speed ground* to be more important than *speed_air* (model2) 
    3. Normalized coefficients show that *speed_ground* has higher importance (model3)  
    Select 2. above as the ranking order of significant factors based on p_values 

### 3. Check collinearity  

  1. *model1* has a lower R<sup>2</sup><sub>adj</sub> and higher residual standard error than *model2* and *model3* 
  2. In *model3*, *speed_ground*'s coefficient sign changes and is statistically insignificant  
  3. *model2* and *model3* both drop 732 observations 
  4. Collinearity exists between *speed_ground* and *speed_air*  
     Picking *speed_ground* as it has no null values while *speed_air* has 711 NA's  

  |Models|Intercept|speed_ground|speed_air|Adjusted R-square|
  |:----:|:-------:|:----------:|:-------:|:---------------:|
  |model1| -1766.8 |     41.5   |    -    |       0.75      |
  |model2| -5417.6 |      -     |   79.2  |       0.89      |
  |model3| -5425.5 |    -12.3   |   91.6  |       0.89      |  
  
### 4. Variable selection based on p-value ranking

  1. R<sup>2</sup> values increase with addition of variables and level off after 3
  2. R<sup>2</sup><sub>adj</sub> values increase with addition of variables and level off after 3
  3. AIC values decrease with addition of variables and level off after 2
  4. Select model with 3 variables - *speed_ground*, *aircraft* and *height* since it has a low AIC value and higher R<sup>2</sup><sub>adj</sub> with the lowest complexity  
  5. Model based on *aircraft*:  
    1. Boeing: ``` distance = -2031.4 + 42.5*speed\_ground + 14.3*height ```  
    2. Airbus ``` distance = -2528 + 42.5*speed\_ground + 14.3*height ```  

### 5. Variable selection based on automated algorithm  
StepAIC function gives the same model as compared to the p-value table.  

## Logistic regression

### 1 Create binary responses 

#### 1.1 Import FAA datasets
FAA1 and FAA2 datasets have 800 and 150 observations each with 8 variables.

#### 1.2 Clean dataset to create faa_clean
100 observations were duplicates in combined dataset.  
19 observations were filtered out as they had abnormal values.

#### 1.3 Add binary variables
**long.landing** is when distance is greater than 2500. There are 103 such observations.  
**risky.landing** is distance is greater than 3000. There are 61 such observarions.  
Remove continuous variable **distance**.

### 2 Factors affecting "long.landing"

#### 2.1 Visualize "long.landing"
We see that 12% of the observations have long landing.

#### 2.2 Single-factor Regression Analysis
**speed_ground** is found to have the smallest p-value followed by **speed_air**, **aircraft**, **pitch** and **height**.
**no_pasg** and **duration** may be insignificant and have a small negative effect.

#### 2.3 Visualizing association
The plots indicate that as speed_ground and speed_air increase, we see more long.landing observations.  
Aircraft and pitch are seen to be significant and will be considered in the model.  
Height, no_pasg and duration have very high p-values and so can be conisdered to be insignificant.  
There is very high correlation between speed_ground and speed_air. We drop speed_air as it has missing values and its p-value is lower than speed_ground.

#### 2.4 Logistic Models with AIC and BIC

  1. From table1 we fit a model using the significant factors
    1. Factors used: speed_ground, aircraft and pitch  
    2. AIC = 89.31  
    3. BIC = 108.20
  2. Using the step function with AIC as criteria:  
    1. Factors used: speed_ground, aircraft, height and pitch  
    2. AIC = 63.20  
    We have an extra factor height which gets considered as compared to our original table.
  3. Using the step function with BIC as criteria:  
    1. Factors used: speed_ground, aircraft and height  
    2. BIC = 83.94  
    Compared to the AIC model the pitch factor is dropped.
    
#### 2.5 Model Selection
  1. BIC as the criteria was used to select a parsimonous model
  2. When speed_ground increases by 1 mph, the chance of long.landing increases by 152%
  3. When aircraft is boeing there is a 15573% chance of long.landing
  4. When height increases by 1 meter, there is a 26% higher chance of long.landing

### 3 Factors affecting “risky.landing”

#### 3.1 Visualize "risky.landing"
We see that 7% of the observations have risky landing.

#### 3.2 Single-factor Regression Analysis
**speed_ground** is found to have the smallest p-value followed by **speed_air**, and **aircraft**.  
**pitch** appears to be insignificant but has a positive impact.  
**no_pasg**, **duration** and **height** might be insignificant and have a small negative effect.

#### 3.3 Visualizing association
The plots indicate that as **speed_ground** and **speed_air** increase, we see more risky.landing observations.  
**Aircraft** is seen to be significant and will be considered in the model.  
Pitch, height, no_pasg and duration have high p-values and so can be conisdered to be insignificant.  
There is very high correlation between speed_ground and speed_air. We drop **speed_air** as it has missing values and its p-value is lower than speed_ground.

#### 3.4 Logistic Models with AIC and BIC
  1. From table3 we fit a model using the significant factors  
    1. Factors used: speed_ground, aircraft  
    2. AIC = 46.10  
    3. BIC = 60.26
  2. Using the step function with AIC as criteria:  
    1. Factors used: speed_ground, aircraft and no_pasg  
    2. AIC = 45.71  
    We have an extra factor no_pasg which gets considered as compared to our original table.
  3. Using the step function with BIC as criteria:  
    1. Factors used: speed_ground and aircraft  
    2. BIC = 60.26  
    Compared to the AIC model the no_pasg factor is dropped.

#### 3.5 Model Selection
  1. BIC as the criteria was used to select a parsimonous model
  2. When speed_ground increases by 1 mph, the chance of long.landing increases by 152%
  3. When aircraft is boeing there is a 5565% chance of long.landing
  4. The AIC model includes no_pasg which has a negative coefficient
  5. Since we are interested in factors causing risky landing the AIC model is not considered

### 4 “long.landing” v/s “risky.landing”

#### 4.1 Summary of models
  1. **long.landing** considers height whereas **risky.landing** doesn't
  2. Both types of landing have same effect with increase in speed_ground
  3. The aircraft type is a bigger factor in long.landing

#### 4.2 ROC Curve
Both the models show have very area under the ROC curve indicating good diagnostic abilities for classification of response.  
long.landing AUC = 0.9983  
risky.landing AUC = 0.9986

#### 4.3 Predict probability for an example
Example information: Boeing, duration=200,  no_pasg=80, speed_ground=115, speed_air=120,  height=40, pitch=4
Using the models selected we have: 
  1. Probability of long.landing = 1 with 95% CI = { 0.999985 1.000000 }  
  2. Probability of risky.landing = 0.999789 with 95% CI = { 0.9874843 0.9999965 }  

### 5 Compare models with different link functions

#### 5.1 Logit vs Probit vs Complementary Log-Log
  1. Logit coefficients are seen to be twice the magnitude of Probit link
  2. Complementary Log-Log is slightly higher in magnitude compared to Probit link
  3. All the link functions give the same sign of coefficients

#### 5.2 ROC curve for different links
All three link functions give pretty close ROC curves.

#### 5.3 Comparing Top 5 risky.landing
The top 5 risky.landing probabilities are not the same for the three model variations.

#### 5.4 Predict probability for an example with different link functions
Logit link has a lower predicted probability compared to probit and complementary log-log.
The upper bound in a 95% confidence interval is 1 for all three link functions.
