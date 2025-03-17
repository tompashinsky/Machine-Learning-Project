### <center> **Introduction to Machine Learning | Final Project | Spring 2024**
    
<center> *This project was submitted by Niv Noyman and Tom Pashinsky*<br>

#### In this project, we will build a model that predicts if a person will get hired for a job or not.
<br>
<br>
<br>
<br>
<br>

# **Part A - Exploration**
### First, we will analyze the "train" and "test" datasets: <br>
### 1. **Size:**
How many rows and columns each dataset has: 
### 2. **"ID" unique values:**
We will check if all the ID's in the "ID" column appear exactly once: 
### 3. **Missing values:**
We will check how many missing values there are in each column (except the "ID" column):
### 4. **Categorical variables:**
### 5. **Statistics and Graphical Representation:**
<br>

### Histograms
Down here, we plotted histograms for the numeric features: 
### Distributions <br>
Here we will plot the distributions of the numeric features using a technique we didn't see in class - **KDE**. <br>
This technique helps to estimate the probability density function of a random variable. It creates a smooth curve from discretely sampled data that reflects the underlying density distribution.
### Statistics for numeric features
Here we will check some statistical values of the numeric features (Mean, std, min / max values etc.):
### Statistics for categorial features
Here we will count how many values there are in each column:
### Correlation
In order to see if there are correlations between the features, we will convert the categorial features to numerical ones (the conversion is only for creating the correlation map, we will convert the categorial features in a different way for building the models later on in the notebook) and then create a correlation map:
## **Here, we will repeat the exploration stage, this time with the "test" dataset**:
<br>
<br>
<br>
<br>
<br>

# **Part B - Preprocessing**

In this section we will detail issues that arose during the exploration stage. First we implemented the solutions on the train file, and at the end of the notebook we applied everything on the test file. **We note that all the detailed information below refers to the data from the train file only:**

### 1. **Outliers:**
As we stated in the exploration stage, some of the features have a small number of extreme values. <br>
Therefore, we built a function that removes rows with values smaller than the 0.01 percentile and greater than the 0.99 percentile. <br>
**This step is not supposed to drastically change the dataset and its behaviors.**
### 2. **Creating new features:**
**Our working assumption is that when considering whether a candidate will be hired, there may be some importance to his knowledge of different technologies and programming languages.** So we will check how many technologies there are in the training dataset, and if there is a correlation between knowing certain programming languages and accepting the candidate for a job:
### 3. **Completing missing values:**
We will implement 2 methods to deal with the problem:
- **Connections between features** - there are several features with connections between them:
   - **"B" and "years_of_experience"** - as we saw in the exploration stage, there is a strong positive correlation between this features. In the vast majority of rows, the "B" column values are less than or equal to the "years_of_experience" column values. **It seems that column "B" represents the number of years of experience in the current job.** Therefore, every time we reach a missing cell in one of the columns, we will complete it by choosing the maximum result from two options: the value of the second cell minus the difference of the averages of the columns, or 0 when the value of the second cell minus the difference of the averages of the columns results in a negative number. 

  - **"is_dev" and "stack_experience"** - fill in missing values in the "is_dev" column according to the following rule: if there is a value in the "stack_experience" row that is greater than 0, the value in the "is_dev" row will be developer, otherwise non-developer. <br>
  **The logic behind this course of action is that a person who has stack experience is most likely a software developer.**

  - **"age_group" and "years_of_experience"** - fill in missing values in the "age_group" column according to the following rule: a person with over 20 years of experience will be defined as old, otherwise he will be defined as young.


  - **"worked_in_the_past" and "years_of_experience"** - fill in missing values in the "worked_in_the_past" column according to the following rule: if the value in the "years_of_experience" column is greater than 0, the value in the "worked_in_the_past" column is True, otherwise it is False. <br>
  It makes sense that if someone has experience in working, he must have worked somewhere. <br><br>
  
- **Proportion** - we decided to fill the missing slots in each column (apart from the columns "is_dev", "age_group", "worked_in_the_past") with values that would preserve the original data ratio. **We chose this way because we did not want arbitrary data filling to affect the distribution of features.**
### 4. **Categorical variables:**
In the columns containing categorical variables, we will convert the categorical values to numerical values. <br>
**Converting categorical values to numeric values may result in loss of information, and we did not want arbitrary numeric values to be obtained.** Therefore, in order to maintain the importance of the order of each categorical value, we built a function that will check for each categorical value the ratio between the times it appears in the same row with label 1 compared to the times it appears with label 0, so that the number we place in place of the categorical value is the same ratio , between 0 and 1. Thus, categorical values with a higher frequency of label '1' will receive a higher "value" in training and test datasets. <br>
**It is important to note that since the test file has no labels, and the function relies on the labels, we simply streamed the numerical values created for the train file into the test file.**
### 5. **Normalization:**
It seems that the features "years_of_experience", "prev_salary", "A", "B", "D", "stack_experience" and "country" are not normalized (Their range of values is large). **We will normalize them so that all the data would be on the same scale, otherwise a bias may arise in the data.** For the purpose of normalization we will use the StandardScaler method.
### 6. **Dimensionality:**
After creating the features of the technology types, we had a total amount of 133 features. **Such a large amount of features may cause overfitting,** so that the model will adapt itself too much to the training set. It means that the variance will be very high, the bias will be low, and the performance of the model on the 'test' set will be less good. Thats is why we will implement two ways to reduce the number of features:
- **Manual download** - in the beginning of the preprocessing stage, we dropped the "ID" and "label" columns, because they are not relevant for training the model. Also, since we created 116 new features of technology types, we selected only 20 technologies with the highest correlation to the label "1" for the purpose of training the models.
<br>
- **PCA** - before implementing it, we were debating which method to choose. We were afraid that using PCA will cost in losing information and geting lower peformance. On the other hand, we noticed that the "Feature Selection" method takes too much time to run the models. Finally, we decided to use the PCA method, with the main reason being due to runtime considerations of model training. In order to lose as little information as possible, we defined in the PCA implementation that the variance will keep 99% of the variance of the original dataset.
<br>
<br>
<br>
<br>
<br>

# **Part C - Running models**
In this part, we will train 4 selected models.
First, we will implement PCA and split the training dataset to "train" and "validation":
## Basic models
## 1. Logistic Regression model:
## 2. Naive Bayes model:
## Advanced models
## 1. Random Forest model:
## 2. Multi-Layer Perceptron model:
### It seems that the MLP is our best model!
So we will analyze its performance and later on run it with the Test dataset.
## Feature Importance
We want to know which features are mostly contribute the model's performance. <br>
Since we learned in class only "Feature Importance" method which is used only for Random Forest model, we will use a technique called **"Permutation Feature Importance"**. This method helps to measure how much the model’s performance **decreases** when the values of a particular feature are randomly shuffled or permuted while keeping other variables unchanged.
<br>
<br>
<br>
<br>
<br>

# **Part D - Model Evaluating**
### Here we can see again the Confusion Matrix of the MLP model:
### Here we will run our MLP model without using K-Cross validation:
## Here you can see all the 4 ROC curves of the model above:
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
# **Part E - Prediction**
### First, we will create a function that units all the functions from the preprocessing stage:
# Pipline
Here, you can run the important things:
1. Loading the datasets and applying the preprocessing stage.
2. Defining our best model and training it.
3. Creating predictions on the data from the "test" dataset, and save it in a file called "result_11.csv"


























