<h1>Introduction to Machine Learning | Final Project</h1>
    
This project was submitted by Niv Noyman and Tom Pashinsky<br>

#### In this project, we will build a model that predicts if a person will get hired for a job or not.
<br>
<br>
<h2>Part A - Exploration :mag_right:</h2>
<b>First, we analyzed the "train" and "test" datasets:</b><br>
&emsp;1. <b>Size:</b> How many rows and columns each dataset has?<br> 
&emsp;2. <b>ID unique values:</b> Do all the ID's in the "ID" column appear exactly once?<br>
&emsp;3. <b>Missing values:</b> How many missing values there are in each column (except the "ID" column)?<br>
&emsp;4. <b>Categorical variables</b><br>
&emsp;5. <b>Statistics and Graphical Representation</b><br><br>
After that, we plotted histograms and distributions, checked statistics (Mean, std, min / max values etc.) and correlation between features.
<br>
<br>
<h2>Part B - Preprocessing :pencil2:</h2>
In this section we will addressed issues that arose during the exploration stage:<br>
1. <b>Outliers:</b> We removed rows with values below the 0.01 percentile or above the 0.99 percentile to eliminate<br>
&emsp;extreme outliers withoutsignificantly altering the dataset.<br>
2. <b>New Features:</b> We created new features representing the candidate’s knowledge of technologies/programming<br>
&emsp;languages and checked theircorrelation with the hiring label.<br>
3. <b>Handling Missing Values:</b><br>
&emsp;- <b>Feature relationships:</b> We used logical rules based on correlations (e.g., "B" vs. "years_of_experience", "is_dev" vs. "stack_experience")<br>
&emsp;&ensp;to fill missing values.<br>
&emsp;- <b>Proportional filling:</b> For other columns, missing values were imputed to preserve the original distribution.<br>
4. <b>Categorical Variables:</b> We converted categorical features to numeric by calculating the ratio of label '1' to label '0' occurrences, assigning<br>
&emsp;higher values to categories more associated with label '1'.<br>
5. <b>Normalization:</b> Features with wide ranges were scaled using StandardScaler to prevent bias from unnormalized data.<br>
6. <b>Dimensionality Reduction:</b><br>
&emsp;- <b>Dropped irrelevant columns</b> ("ID", "label") and selected only the top 20 most relevant technology features.<br>
&emsp;- <b>Applied PCA</b> to retain 99% of the variance while reducing dimensionality for faster model training and to avoid overfitting.
<br>
<br>
<h2>Part C - Creating & training models :hammer:</h2>
In this part, we chose 4 models and trained them:<br>
&emsp; 1. <b>Logistic Regression</b><br>
&emsp; 2. <b>Naive Bayes</b><br>
&emsp; 3. <b>Random Forest</b><br>
&emsp; 4. <b>Multi-Layer Perceptron (MLP)</b><br><br>
After implementing PCA, spliting the training dataset to "train" and "validation", we saw that the MLP model got the best results.<br>
So we analyzed its performance and later on ran it with the Test dataset.<br><br>
<b>Feature Importance</b><br>
We wanted to know which features are mostly contribute the model's performance.<br>
Since we learned in class only "Feature Importance" method which is used only for Random Forest model, we used a technique called <b>"Permutation Feature Importance"</b>. This method helps to measure how much the model’s performance <b>decreases</b> when the values of a particular feature are randomly shuffled or permuted while keeping other variables unchanged.
<br>
<br>
<h2>Part D - Model Evaluating :pencil:</h2>
Here, we cretaed a <b>Confusion Matrix</b> for the MLP's peformance and analyzed the results.<br>
Moreover, we ran the MLP without using K-Cross validation, and also plotted all the ROC curves of the 4 models.
<br>
<br>
<h2>Part E - Prediction :crystal_ball:</h2>
Finally, we created a <b>Pipline</b> that runs the main functions in the notebook:<br>
&emsp;1. Loading the datasets and applying the preprocessing stage.<br>
&emsp;2. Defining our best model and training it.<br>
&emsp;3. Creating predictions on the data from the "test" dataset, and save it in a file called <b>"results_11.csv"</b>


























