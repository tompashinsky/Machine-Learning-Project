# Machine-Learning-Project
{
 "cells": [
  {
   "cell_type": "markdown",
   "id": "37331552",
   "metadata": {},
   "source": [
    "### <center> Tel Aviv University\n",
    "## <center>**Digital Sciences for High-Tech**\n",
    "# <center> **Introduction to Machine Learning - Final Project**\n",
    "#### <center> <i> Spring 2024\n",
    "    \n",
    "### <center> Group 11\n",
    "### <center> Niv Noyman \n",
    "### <center> Tom Pashinsky <br>\n",
    "<br>\n",
    "\n",
    "#### In this project, we will build a model that predicts if a person will get hired for a job or not."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "7d641b82-b195-47a9-843c-34f335e19e5c",
   "metadata": {},
   "source": [
    "### **Import libraries**"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "3ad2205d-1a9b-483d-a28f-4c7f5d7ff6c7",
   "metadata": {},
   "outputs": [],
   "source": [
    "import warnings\n",
    "warnings.filterwarnings('ignore')\n",
    "import numpy as np\n",
    "import pandas as pd\n",
    "import matplotlib.pyplot as plt\n",
    "import sklearn\n",
    "import seaborn as sns\n",
    "from sklearn.decomposition import PCA\n",
    "from sklearn.preprocessing import StandardScaler,FunctionTransformer, LabelEncoder\n",
    "from sklearn.ensemble import RandomForestClassifier\n",
    "from sklearn.linear_model import LogisticRegression\n",
    "from sklearn.metrics import auc, roc_curve, RocCurveDisplay, accuracy_score, roc_auc_score, confusion_matrix, classification_report,ConfusionMatrixDisplay\n",
    "from sklearn.model_selection import StratifiedKFold, GridSearchCV, train_test_split\n",
    "from sklearn.compose import ColumnTransformer\n",
    "from sklearn.naive_bayes import GaussianNB\n",
    "from sklearn.neural_network import MLPClassifier\n",
    "from collections import Counter\n",
    "from sklearn.inspection import permutation_importance\n",
    "import os"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "a7b6a876-5e3b-4d5e-8569-8c64d02f56ee",
   "metadata": {},
   "source": [
    "### Loading the data"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "28654ef3-b585-4954-a64d-8844c223fac1",
   "metadata": {
    "scrolled": true
   },
   "outputs": [],
   "source": [
    "pd.set_option('display.max_columns',None)\n",
    "train_df = pd.read_csv('train.csv')\n",
    "test_df = pd.read_csv('test.csv')\n",
    "train_df_copy = train_df.copy()\n",
    "test_df_copy = test_df.copy()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "fa767203",
   "metadata": {},
   "source": [
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "\n",
    "# **Part A - Exploration**\n",
    "### First, we will analyze the \"train\" and \"test\" datasets: <br>\n",
    "### 1. **Size:**\n",
    "How many rows and columns each dataset has: "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "9a002acf",
   "metadata": {},
   "outputs": [],
   "source": [
    "print(f'The size of the training dataset is: {train_df_copy.shape}')\n",
    "print(f'The size of the test dataset is: {test_df_copy.shape}')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "f0f3c514",
   "metadata": {},
   "source": [
    "It seems that the \"train\" dataset is bigger than the \"test\" dataset."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "ec3c0d03",
   "metadata": {},
   "source": [
    "### 2. **\"ID\" unique values:**\n",
    "We will check if all the ID's in the \"ID\" column appear exactly once: "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "c840c08c-69e8-4235-9729-eb088228cf35",
   "metadata": {},
   "outputs": [],
   "source": [
    "train_df_copy.ID.is_unique"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "69bd4bc9",
   "metadata": {},
   "outputs": [],
   "source": [
    "test_df_copy.ID.is_unique"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "de9a2893-7b70-4ff1-a9af-8dcbaad719dd",
   "metadata": {},
   "source": [
    "It seems that all the ID's appear exactly once. Now we can be sure that information doesn't appear multiple times in the datasets."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "130959f9-6927-484f-9cbc-d5caff35f509",
   "metadata": {},
   "source": [
    "### 3. **Missing values:**\n",
    "We will check how many missing values there are in each column (except the \"ID\" column):"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "e10d8587-61b5-4829-bff5-359363e07a9f",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Dropping the 'ID' column\n",
    "\n",
    "train_df_no_id = train_df_copy.drop(columns=['ID'])\n",
    "test_df_no_id = test_df_copy.drop(columns=['ID'])\n",
    "\n",
    "# Count the number of NaN values in each column of the dataset.\n",
    "\n",
    "def count_nans(df, dataset_name):\n",
    "    nan_counts = df.isna().sum()\n",
    "    print(f'NaN Counts in {dataset_name}:\\n{nan_counts}\\n')\n",
    "\n",
    "count_nans(train_df_no_id, 'Training Dataset')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "c53fd4c4",
   "metadata": {},
   "source": [
    "We can see that the number of missing values in **\"stack_experience\"** is significantly higher than the other columns. **It doesn't necessarily mean that there 14,042 NaNs, but it is possible that some of the cells are actually 0, which means that a person doesn't know any technologies at all.** <br>\n",
    "In addition, there are no missing values in the \"label\" column, so we can use this column for training the models. <br> This problem will be resolved in the preprocessing stage."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "affdaacf-6e25-4b57-a4fb-25b420947187",
   "metadata": {},
   "source": [
    "### 4. **Categorical variables:**"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "25f236f3-65ef-407e-a8cd-bad1996b11af",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Identify categorical columns by checking their data types\n",
    "\n",
    "categorical_columns = train_df_copy.select_dtypes(include=['object', 'category']).columns\n",
    "\n",
    "# Count the number of categorical columns\n",
    "\n",
    "num_categorical_columns = len(categorical_columns)\n",
    "\n",
    "print(f\"Number of categorical columns: {num_categorical_columns}\")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "c1888e15",
   "metadata": {},
   "source": [
    "The \"train\" dataset has 17 columns, 10 of them do not contain numerical values.<br> In some cases each cell contains one of 2 values (for example the \"disability\" column), in some cases each cell contains one of more than 2 values (for example the \"education\" column), and there is even a column containing several words in each cell (the \"stack_experience\" column). <br> We will expand on the way to deal with the problem in the preprocessing stage."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "01941188-4746-45b7-8e8d-ade45719fa33",
   "metadata": {},
   "source": [
    "### 5. **Statistics and Graphical Representation:**\n",
    "<br>\n",
    "\n",
    "### Histograms\n",
    "Down here, we plotted histograms for the numeric features: "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "00c6b0c8-93c2-4373-af66-92cd767d7ee0",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Plot histograms for numeric features in the dataset.\n",
    "\n",
    "def plot_histograms(df, dataset_name):\n",
    "    numeric_cols = df.select_dtypes(include=np.number).columns\n",
    "    df[numeric_cols].hist(bins=30, figsize=(15, 10))\n",
    "    plt.suptitle(f'Histograms of Numeric Features - {dataset_name}')\n",
    "    plt.show()\n",
    "\n",
    "plot_histograms(train_df_no_id, 'Training Dataset')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "a2c015e1",
   "metadata": {},
   "source": [
    "- **\"years_of_experience\" -** it seems that most of the people in the \"train\" dataset (around 6500) have a little less than 10 years of experience, and as the years of experience increase, the amount of people decreases. In general, we can see in the plot that there is a pattern - the graph increases and than drops, and right after it drops it increases again but the peak point is lower than the previous one.\n",
    "- **\"A\" -** it seems that unlike other features, the range of values in feature \"A\" is very small.\n",
    "- **\"D\" -** like the feature \"A\", it seems that the range of values is very small.\n",
    "- **\"prev_salary\" -** it seems that most of the people in the \"train\" dataset (around 3800) have earned on there previous job a salary of approximately 12500, and as the salary increase, the amount of people decreases. The pattern reminds the one in the \"years_of_experience\", and this connection makes sense: the higher of experience you have, the higher the salary you earn.\n",
    "- **\"B\" -** the meaning of this column is not clear yet, but its graph acts similar like the graphs of \"years_of_experience\" and \"prev_salary\". It might indicate a connection between these features, but more exploration is needed.\n",
    "- **\"label\" -** there are more people who got hired to the job (label '1', close to 30000) and than those who didn't (label '0', a little higher than 25000). <br>"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "807d44dc",
   "metadata": {},
   "source": [
    "### Distributions <br>\n",
    "Here we will plot the distributions of the numeric features using a technique we didn't see in class - **KDE**. <br>\n",
    "This technique helps to estimate the probability density function of a random variable. It creates a smooth curve from discretely sampled data that reflects the underlying density distribution."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "b5bdf7af-a65c-4588-b436-344a04066348",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Plot distribution plots for numeric features in the dataset.\n",
    "\n",
    "def plot_distributions(df, dataset_name):\n",
    "    numeric_cols = df.select_dtypes(include=np.number).columns\n",
    "    plt.figure(figsize=(15, 10))\n",
    "    for i, col in enumerate(numeric_cols, 1):\n",
    "        plt.subplot(len(numeric_cols) // 3 + 1, 3, i)\n",
    "        sns.histplot(df[col], kde=True)\n",
    "        plt.title(f'Distribution of Feature \"{col}\" {dataset_name}')\n",
    "    plt.tight_layout()\n",
    "    plt.show()\n",
    "\n",
    "plot_distributions(train_df_no_id, 'Training Dataset')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "e0c24728",
   "metadata": {},
   "source": [
    "According to the plots, the features \"A\" and \"D\" are distributing normally. <br>\n",
    "The distribution of \"years_of_experience\", \"B\" and \"prev_salary\" looks similar but not exactly the same. **In particular, the distribution of \"B\" is more similar to \"years_of_experience\" than \"prev_salary\".** <br>"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "9761783d",
   "metadata": {},
   "source": [
    "### Statistics for numeric features\n",
    "Here we will check some statistical values of the numeric features (Mean, std, min / max values etc.):"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "832719eb-fdf6-42d5-8d61-564a266aa5eb",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Display statistical summary of numeric features in the dataset.\n",
    "\n",
    "def display_numeric_stats(df, dataset_name):\n",
    "    numeric_cols = df.select_dtypes(include=np.number).columns\n",
    "    print(f'Numeric Features Statistics for {dataset_name}:\\n')\n",
    "    return df[numeric_cols].describe().transpose()\n",
    "\n",
    "display_numeric_stats(train_df_no_id, 'Training Dataset')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "b6c634a9",
   "metadata": {},
   "source": [
    "- **Count -** the numeric features (expect for 'label') have less than 55462 values due to empty cells (NaNs). <br>\n",
    "\n",
    "- **Min / Max -** if we compare this information with the histograms above, we can see that there are outliers in some of the features: <br>\n",
    "    - \"A\" - its minimum value is -21.63 and its maximum is at 55.495, while according to the histogram the minimum is approximately -13 and the maximum reaches to 30.\n",
    "    - \"B\" - its maximum value is 50, while according to the histogram the maximum is approximately 42.\n",
    "    - \"D\" - its maximum value is 184.15, while according to the histogram the maximum is approximately 183. <br>\n",
    "    \n",
    "- Another interesting point is that **all the values statistics of \"B\" are smaller or equal to \"years_of_experience\"**. This also might indicate that there is a connection these features. <br> Soon we will create a correlation map so we can check if there is a real connection between them.  "
   ]
  },
  {
   "cell_type": "markdown",
   "id": "dcdbdab1",
   "metadata": {},
   "source": [
    "### Statistics for categorial features\n",
    "Here we will count how many values there are in each column:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "9f28bc05-62a7-433b-8396-8f7548cdef68",
   "metadata": {
    "scrolled": true
   },
   "outputs": [],
   "source": [
    "# Display value counts for categorial features in the DataFrame.\n",
    "    \n",
    "def display_categorial_stats(df, dataset_name):\n",
    "    print(f'Categorial Features Statistics for {dataset_name}:\\n')\n",
    "    categorial_cols = df.select_dtypes(include='object').columns\n",
    "    for col in categorial_cols:\n",
    "        print(f'Column: {col}')\n",
    "        print()\n",
    "        print(df[col].value_counts())\n",
    "        print()\n",
    "\n",
    "display_categorial_stats(train_df_no_id, 'Training Dataset')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "c2e81529-7c1c-4822-beee-0364f2f31334",
   "metadata": {},
   "source": [
    "It looks like the majority of the people in the \"train\" dataset have worked at least once job. <br> Moreover, most of them are young, men, developers without disabilities or mental issues. Most of them have a BA / Bsc degree, live in western countries (such as the USA, Germany, the United Kingdom etc.) and top 3 popular programming languages are Python, C++ and Git."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "683793c3-2a6e-49fc-8d0a-01de933ea654",
   "metadata": {},
   "source": [
    "### Correlation\n",
    "In order to see if there are correlations between the features, we will convert the categorial features to numerical ones (the conversion is only for creating the correlation map, we will convert the categorial features in a different way for building the models later on in the notebook) and then create a correlation map:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "a7650add-db3d-488d-9315-abb6dc9cbcc4",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Encode non-numeric columns in the dataset, in order to be able to create a correlation matrix for the analysis.\n",
    "\n",
    "def replace_words_with_count(df, column_name):\n",
    "    df[column_name] = df[column_name].apply(lambda cell: len(cell.split(';')) if pd.notna(cell) else 0)\n",
    "    return df        \n",
    "def encode_non_numeric_columns(df):\n",
    "    non_numeric_columns = df.select_dtypes(include=['object']).columns\n",
    "    label_encoders = {col: LabelEncoder() for col in non_numeric_columns}\n",
    "    for col in non_numeric_columns:\n",
    "        df[col] = label_encoders[col].fit_transform(df[col].astype(str))\n",
    "    return df\n",
    "train_df_no_id = replace_words_with_count(train_df_no_id,'stack_experience')\n",
    "train_encoded = encode_non_numeric_columns(train_df_no_id)\n",
    "\n",
    "# Plot the correlation matrix for numeric features in the dataset.\n",
    "\n",
    "def plot_correlation_matrix(df, dataset_name):\n",
    "    numeric_cols = df.select_dtypes(include=np.number).columns\n",
    "    corr_matrix = df[numeric_cols].corr()\n",
    "    plt.figure(figsize=(12, 8))\n",
    "    sns.heatmap(corr_matrix, annot=True, cmap='coolwarm', fmt='.2f')\n",
    "    plt.title(f'Correlation Matrix - {dataset_name}')\n",
    "    plt.show()\n",
    "    \n",
    "plot_correlation_matrix(train_encoded, 'Train Dataset')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "102b6778-e68f-4073-bc45-60f2856ed492",
   "metadata": {},
   "source": [
    "According to the map: <br>\n",
    "- **Most of the features don't have correlations with other features.**\n",
    "<br>\n",
    "- **There are few low-medium negative correlations, for example:**\n",
    "    - \"prev_salary\" and \"age_group\" have a negative correlation of -0.26 . It makes sense, because when we converted the categorial features to numeric, it classified \"old\" as '0' and \"young\" as '1'. So the more money a person earns, the more likely he will be classified as '0' (old). <br>\n",
    "<br>    \n",
    "- **There are few medium-strong negative correlations, for example:**\n",
    "    - \"years_of_experience\" and \"age_group\" have a negative correlation of -0.53 . It makes sense that the more experience a person has, the more likely he will be classified as '0' (old).\n",
    "    - \"B\" and \"age_group\" have a negative correlation of -0.54 . The number is very close to the one mentioned above. <br>\n",
    "    <br>\n",
    "- **There are few low-medium positive correlations, for example:**\n",
    "    - \"prev_salary\" and \"years_of_experience\" have a positive correlation of 0.39 . It makes sense that the higher the previous salary of a person is, the more experience he has.\n",
    "    - \"prev_salary\" and \"B\" have a positive correlation of 0.39 . The number is very close to the one mentioned above.\n",
    "    - \"D\" and \"label\" have a positive correlation of 0.41 . We are not sure what \"D\" means, maybe it's kind of a performance measure that affects the chances of being hired.\n",
    "    - \"stack_experience\" and \"label\" have a positive correlation of 0.36 . It seems that knowing a lot of different technologies increases the chances to get hired for a job. <br>\n",
    "    <br>\n",
    "- **There is a strong positive correlation of 0.9 between the features \"B\" and \"years_of_experience\"**. It fits the conclusions we made from the histograms, distributions and statistics above. We will address this relationship in the preprocessing step."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "1039c7da",
   "metadata": {},
   "source": [
    "## **Here, we will repeat the exploration stage, this time with the \"test\" dataset**:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "808c22a9",
   "metadata": {},
   "outputs": [],
   "source": [
    "count_nans(test_df_no_id, 'Test Dataset')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "b5749b34",
   "metadata": {},
   "outputs": [],
   "source": [
    "plot_histograms(test_df_no_id, 'Test Dataset')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "602a89c8",
   "metadata": {},
   "outputs": [],
   "source": [
    "plot_distributions(test_df_no_id, 'Test Dataset')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "b66d9e17",
   "metadata": {},
   "outputs": [],
   "source": [
    "display_numeric_stats(test_df_no_id, 'Test Dataset')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "fb5d68e8",
   "metadata": {},
   "outputs": [],
   "source": [
    "display_categorial_stats(test_df_no_id, 'Test Dataset')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "03b0dcd8",
   "metadata": {},
   "outputs": [],
   "source": [
    "test_encoded = encode_non_numeric_columns(test_df_no_id.copy())\n",
    "plot_correlation_matrix(test_encoded, 'Test Dataset')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "a0fc14c4-c45f-4688-8c3c-fe782090c33a",
   "metadata": {},
   "source": [
    "We can see that although the \"test\" dataset is smaller, its behavior is very similar to the behavior of the \"train\" dataset."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "87d74777-e20e-44f8-abbb-8793de03ff76",
   "metadata": {},
   "source": [
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "\n",
    "# **Part B - Preprocessing**\n",
    "\n",
    "In this section we will detail issues that arose during the exploration stage. First we implemented the solutions on the train file, and at the end of the notebook we applied everything on the test file. **We note that all the detailed information below refers to the data from the train file only:**\n",
    "\n",
    "### 1. **Outliers:**\n",
    "As we stated in the exploration stage, some of the features have a small number of extreme values. <br>\n",
    "Therefore, we built a function that removes rows with values smaller than the 0.01 percentile and greater than the 0.99 percentile. <br>\n",
    "**This step is not supposed to drastically change the dataset and its behaviors.**"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "bffc3b00-e9a8-4fdb-a45e-898a55e4c490",
   "metadata": {
    "scrolled": true
   },
   "outputs": [],
   "source": [
    "def cap_outliers_percentiles_multi(df, columns, lower_percentile=0.01, upper_percentile=0.99):\n",
    "    # Iterate over each specified column in the DataFrame\n",
    "    for column in columns:\n",
    "        # Calculate the lower and upper bounds based on the specified percentiles\n",
    "        lower_bound = df[column].quantile(lower_percentile)\n",
    "        upper_bound = df[column].quantile(upper_percentile)\n",
    "        \n",
    "        # Apply the capping: replace values below the lower bound with the lower bound,\n",
    "        # and values above the upper bound with the upper bound\n",
    "        df[column] = df[column].apply(lambda x: upper_bound if x > upper_bound else (lower_bound if x < lower_bound else x))\n",
    "    \n",
    "    # Return the modified DataFrame with capped outliers\n",
    "    return df"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "44998956",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Apply the function to relevant columns\n",
    "numeric_columns = ['A', 'B', 'D']\n",
    "\n",
    "train_df_copy = cap_outliers_percentiles_multi(train_df_copy, numeric_columns)\n",
    "train_df_copy"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "706d606f",
   "metadata": {},
   "source": [
    "### 2. **Creating new features:**\n",
    "**Our working assumption is that when considering whether a candidate will be hired, there may be some importance to his knowledge of different technologies and programming languages.** So we will check how many technologies there are in the training dataset, and if there is a correlation between knowing certain programming languages and accepting the candidate for a job:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "adfe8996",
   "metadata": {},
   "outputs": [],
   "source": [
    "def process_stack_experience(train_df, label_column, top_n=20):\n",
    "    # Split the 'stack_experience' column by ';' and flatten the list\n",
    "    technologies = train_df['stack_experience'].dropna().str.split(';').tolist()\n",
    "    flattened_technologies = [tech for sublist in technologies for tech in sublist]\n",
    "\n",
    "    # Count the occurrences of each technology\n",
    "    technology_counts = Counter(flattened_technologies)\n",
    "\n",
    "    # Count the number of unique technologies\n",
    "    unique_technologies_count = len(technology_counts)\n",
    "\n",
    "    # Create dummy variables for 'stack_experience'\n",
    "    stack_experience_dummies = train_df['stack_experience'].str.get_dummies(sep=';')\n",
    "\n",
    "    # Calculate the correlation matrix between the top 20 technologies and the class '1' in label\n",
    "    correlation_with_label = stack_experience_dummies.corrwith(train_df[label_column]).sort_values(ascending=False)\n",
    "\n",
    "    # Select the top N most correlated technologies with the label\n",
    "    top_technologies = correlation_with_label.head(top_n).index\n",
    "\n",
    "    # Add the top N most correlated technologies as new columns to the original DataFrame\n",
    "    train_df = pd.concat([train_df, stack_experience_dummies[top_technologies]], axis=1)\n",
    "    \n",
    "    return train_df, unique_technologies_count, top_technologies, correlation_with_label"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "1c0ef227",
   "metadata": {},
   "outputs": [],
   "source": [
    "train_df_copy, unique_technologies_count, top_technologies, correlation_with_label = process_stack_experience(train_df_copy, 'label')\n",
    "print(f'There are {unique_technologies_count} different technologies in the dataset.')\n",
    "print()\n",
    "print(f'These are the top 20 techs correlated with class 1 in the label column:\\n{correlation_with_label.head(20)}')"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "860caad8",
   "metadata": {},
   "source": [
    "It seems that there is a significant correlation between proficiency in technologies and getting hired, so we will add the technologies as columns that would receive values of 1 or 0, for technology control or lack of control, respectively. <br>\n",
    "**We did it in order to understand if the new features could provide us with additional information that would help achieve higher results in running the models.**\n",
    "\n",
    "\n",
    "Now, we will add the top 20 technologies to the Test dataset. <br> \n",
    "\n",
    "**Important note:** since there are no labels in the test dataset, and the \"process_stack_experience\" function relies on the labels, we simply added to it the technologies that came out after running the function on the training dataset."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "ae7ae91d",
   "metadata": {},
   "outputs": [],
   "source": [
    "def add_top_technologies_to_test_df(test_df, top_technologies):\n",
    "    # Create dummy variables for 'stack_experience' in test_df\n",
    "    stack_experience_dummies_test = test_df['stack_experience'].str.get_dummies(sep=';')\n",
    "    \n",
    "    # Select only the top technologies columns\n",
    "    test_top_tech_dummies = stack_experience_dummies_test.reindex(columns=top_technologies, fill_value=0)\n",
    "    \n",
    "    # Add the top technologies as new columns to the original test DataFrame\n",
    "    test_df = pd.concat([test_df, test_top_tech_dummies], axis=1)\n",
    "    \n",
    "    return test_df"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "213646e2",
   "metadata": {},
   "outputs": [],
   "source": [
    "test_df_copy = add_top_technologies_to_test_df(test_df_copy, top_technologies)\n",
    "test_df_copy"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "f11c2f15-63cf-440e-8086-d054c3301dc2",
   "metadata": {},
   "source": [
    "### 3. **Completing missing values:**\n",
    "We will implement 2 methods to deal with the problem:\n",
    "- **Connections between features** - there are several features with connections between them:\n",
    "   - **\"B\" and \"years_of_experience\"** - as we saw in the exploration stage, there is a strong positive correlation between this features. In the vast majority of rows, the \"B\" column values are less than or equal to the \"years_of_experience\" column values. **It seems that column \"B\" represents the number of years of experience in the current job.** Therefore, every time we reach a missing cell in one of the columns, we will complete it by choosing the maximum result from two options: the value of the second cell minus the difference of the averages of the columns, or 0 when the value of the second cell minus the difference of the averages of the columns results in a negative number. \n",
    "\n",
    "  - **\"is_dev\" and \"stack_experience\"** - fill in missing values in the \"is_dev\" column according to the following rule: if there is a value in the \"stack_experience\" row that is greater than 0, the value in the \"is_dev\" row will be developer, otherwise non-developer. <br>\n",
    "  **The logic behind this course of action is that a person who has stack experience is most likely a software developer.**\n",
    "\n",
    "  - **\"age_group\" and \"years_of_experience\"** - fill in missing values in the \"age_group\" column according to the following rule: a person with over 20 years of experience will be defined as old, otherwise he will be defined as young.\n",
    "\n",
    "\n",
    "  - **\"worked_in_the_past\" and \"years_of_experience\"** - fill in missing values in the \"worked_in_the_past\" column according to the following rule: if the value in the \"years_of_experience\" column is greater than 0, the value in the \"worked_in_the_past\" column is True, otherwise it is False. <br>\n",
    "  It makes sense that if someone has experience in working, he must have worked somewhere. <br><br>\n",
    "  \n",
    "- **Proportion** - we decided to fill the missing slots in each column (apart from the columns \"is_dev\", \"age_group\", \"worked_in_the_past\") with values that would preserve the original data ratio. **We chose this way because we did not want arbitrary data filling to affect the distribution of features.**  "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "39583088-1096-4811-ac92-2e0dddd36e00",
   "metadata": {
    "scrolled": true
   },
   "outputs": [],
   "source": [
    " def fill_b_and_experience(df, b_col, exp_col):\n",
    "    # Calculate the mean values for each column\n",
    "    b_mean = df[b_col].mean()\n",
    "    exp_mean = df[exp_col].mean()\n",
    "    \n",
    "    # Define a function to fill NaN values based on the given logic\n",
    "    def fill_b(row):\n",
    "        if pd.isna(row[b_col]):\n",
    "            if pd.isna(row[exp_col]):\n",
    "                # Both values are NaN, use the mean values\n",
    "                return b_mean\n",
    "            else:\n",
    "                # If years_of_experience is not NaN, use max(years_of_experience - 5, 0)\n",
    "                return max(row[exp_col] - 5, 0)\n",
    "        else:\n",
    "            # If B is not NaN, return its value\n",
    "            return row[b_col]\n",
    "\n",
    "    def fill_exp(row):\n",
    "        if pd.isna(row[exp_col]):\n",
    "            if pd.isna(row[b_col]):\n",
    "                # Both values are NaN, use the mean values\n",
    "                return exp_mean\n",
    "            else:\n",
    "                # If B is not NaN, use B + 5\n",
    "                return row[b_col] + 5\n",
    "        else:\n",
    "            # If years_of_experience is not NaN, return its value\n",
    "            return row[exp_col]\n",
    "\n",
    "    # Apply the functions to each row in the DataFrame\n",
    "    df[b_col] = df.apply(lambda row: fill_b(row), axis=1)\n",
    "    df[exp_col] = df.apply(lambda row: fill_exp(row), axis=1)\n",
    "    return df\n",
    "\n",
    "\n",
    "def replace_words_with_count(df, column_name):\n",
    "    df[column_name] = df[column_name].apply(lambda cell: len(cell.split(';')) if pd.notna(cell) else 0)\n",
    "    return df\n",
    "\n",
    "\n",
    "def fill_na_with_ratio(df):\n",
    "    for column in df:\n",
    "        # Drop NA values and get their counts\n",
    "        if column == 'is_dev' or column == 'age_group' or column == 'worked_in_the_past' :\n",
    "            continue\n",
    "        non_na_values = df[column].dropna()\n",
    "        value_counts = non_na_values.value_counts(normalize=True)\n",
    "        # Fill NA values based on the ratio\n",
    "        df[column] = df[column].apply(\n",
    "            lambda x: np.random.choice(value_counts.index, p=value_counts.values) if pd.isna(x) else x\n",
    "        )\n",
    "    return df\n",
    "\n",
    "\n",
    "def fill_is_dev_based_on_stack_experience(df, stack_experience_col, is_dev_col):\n",
    "    # Fill is_dev based on stack_experience only if is_dev is NaN\n",
    "    df[is_dev_col] = df.apply(\n",
    "        lambda row: 'developer' if pd.isna(row[is_dev_col]) and row[stack_experience_col] > 0 else (\n",
    "            'non_developer' if pd.isna(row[is_dev_col]) else row[is_dev_col]\n",
    "        ), axis=1\n",
    "    )\n",
    "    return df\n",
    "\n",
    "\n",
    "def fill_age_group_based_on_experience(df, experience_col, age_group_col):\n",
    "    # Fill age_group based on experience only if age_group is NaN\n",
    "    df[age_group_col] = df.apply(\n",
    "        lambda row: 'old' if pd.isna(row[age_group_col]) and row[experience_col] > 20 else (\n",
    "            \"young\" if pd.isna(row[age_group_col]) else row[age_group_col]\n",
    "        ), axis=1\n",
    "    )\n",
    "    return df\n",
    "\n",
    "\n",
    "def fill_worked_in_past(df, worked_col='worked_in_the_past', experience_col='years_of_experience'):\n",
    "    # Check the conditions and update the 'worked in the past' column\n",
    "    df[worked_col] = df.apply(\n",
    "        lambda row: True if (pd.isna(row[worked_col]) and row[experience_col] > 0) or pd.isna(row[experience_col]) \n",
    "        else row[worked_col] if not pd.isna(row[worked_col])\n",
    "        else False,\n",
    "        axis=1\n",
    "    )\n",
    "    return df\n",
    "    \n",
    "\n",
    "\n",
    "train_df_copy = fill_b_and_experience(train_df_copy, 'B', 'years_of_experience')\n",
    "train_df_copy = replace_words_with_count(train_df_copy, 'stack_experience')\n",
    "train_df_copy = fill_na_with_ratio(train_df_copy)\n",
    "train_df_copy = fill_is_dev_based_on_stack_experience(train_df_copy,'stack_experience','is_dev')\n",
    "train_df_copy = fill_age_group_based_on_experience(train_df_copy,'years_of_experience','age_group')\n",
    "train_df_copy = fill_worked_in_past(train_df_copy)\n",
    "\n",
    "train_df_copy"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "64455bc1",
   "metadata": {},
   "source": [
    "### 4. **Categorical variables:**\n",
    "In the columns containing categorical variables, we will convert the categorical values to numerical values. <br>\n",
    "**Converting categorical values to numeric values may result in loss of information, and we did not want arbitrary numeric values to be obtained.** Therefore, in order to maintain the importance of the order of each categorical value, we built a function that will check for each categorical value the ratio between the times it appears in the same row with label 1 compared to the times it appears with label 0, so that the number we place in place of the categorical value is the same ratio , between 0 and 1. Thus, categorical values with a higher frequency of label '1' will receive a higher \"value\" in training and test datasets. <br>\n",
    "**It is important to note that since the test file has no labels, and the function relies on the labels, we simply streamed the numerical values created for the train file into the test file.**"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "ea6f4f8b-f0c9-4d5d-be7f-c24ecf048427",
   "metadata": {
    "scrolled": true
   },
   "outputs": [],
   "source": [
    "def calculate_punishment_metric(train_df, label_column, feature_columns):\n",
    "    if label_column not in train_df.columns:\n",
    "        print(f\"Error: '{label_column}' column does not exist in the training DataFrame.\")\n",
    "        return train_df, {}\n",
    "    \n",
    "    punishment_mappings = {}\n",
    "    \n",
    "    for feature in feature_columns:\n",
    "        if feature not in train_df.columns:\n",
    "            print(f\"Feature column {feature} not found in the training DataFrame.\")\n",
    "            continue\n",
    "        \n",
    "        # Group by feature and count the total number of labels\n",
    "        total_labels = train_df.groupby(feature)[label_column].count()\n",
    "        \n",
    "        # Group by feature and count the number of label 1 occurrences\n",
    "        label_1_counts = train_df[train_df[label_column] == 1].groupby(feature)[label_column].count()\n",
    "        \n",
    "        # Combine the results into a single DataFrame\n",
    "        comparison_df = pd.DataFrame({'Total Labels': total_labels, 'Label 1 Counts': label_1_counts})\n",
    "        \n",
    "        # Fill NaN values with 0 (in case some categories do not have label 1 occurrences)\n",
    "        comparison_df = comparison_df.fillna(0)\n",
    "        \n",
    "        # Calculate the proportion of label 1 occurrences\n",
    "        comparison_df['Proportion of Label 1'] = comparison_df['Label 1 Counts'] / comparison_df['Total Labels']\n",
    "        \n",
    "        # Calculate the punishment metric (Proportion)\n",
    "        comparison_df['Punishment Metric'] = comparison_df['Proportion of Label 1']\n",
    "        \n",
    "        # Create a mapping from feature value to punishment metric\n",
    "        punishment_mapping = comparison_df['Punishment Metric'].to_dict()\n",
    "        punishment_mappings[feature] = punishment_mapping\n",
    "        \n",
    "        # Replace the values in the feature column with the punishment metric in train_df\n",
    "        train_df[feature] = train_df[feature].map(punishment_mapping)\n",
    "    \n",
    "    return train_df, punishment_mappings\n",
    "\n",
    "# Define the columns and apply the function to train_df\n",
    "array1 = ['worked_in_the_past', 'age_group', 'disability', 'is_dev',\n",
    "          'mental_issues', 'education', 'C', 'country', 'sex']\n",
    "\n",
    "train_df_copy, punishment_mappings = calculate_punishment_metric(train_df_copy, 'label', array1)\n",
    "\n",
    "def apply_punishment_metric(test_df, punishment_mappings):\n",
    "    for feature, punishment_mapping in punishment_mappings.items():\n",
    "        if feature in test_df.columns:\n",
    "            test_df[feature] = test_df[feature].map(punishment_mapping).fillna(0)\n",
    "        else:\n",
    "            print(f\"Feature column {feature} not found in the test DataFrame.\")\n",
    "    \n",
    "    return test_df"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "6edf9b39-e70b-4a1b-9371-0549e264d3bc",
   "metadata": {},
   "source": [
    "### 5. **Normalization:**\n",
    "It seems that the features \"years_of_experience\", \"prev_salary\", \"A\", \"B\", \"D\", \"stack_experience\" and \"country\" are not normalized (Their range of values is large). **We will normalize them so that all the data would be on the same scale, otherwise a bias may arise in the data.** For the purpose of normalization we will use the StandardScaler method."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "70689379-b6c0-47a4-9e47-af6262eb367f",
   "metadata": {},
   "outputs": [],
   "source": [
    "def scale_and_split(df, cols_to_scale, label_column=None, id_column=None):\n",
    "    # Split the dataset into 'features' and 'label' if label_column is provided\n",
    "    if label_column and label_column in df.columns:\n",
    "        labels = df[label_column]\n",
    "        df_features = df.drop(columns=[label_column])\n",
    "    else:\n",
    "        labels = None\n",
    "        df_features = df.copy()\n",
    "    \n",
    "    # Drop the id_column if it is provided\n",
    "    if id_column and id_column in df.columns:\n",
    "        df_features = df_features.drop(columns=[id_column])\n",
    "    \n",
    "    # Prepare the ColumnTransformer\n",
    "    preprocessor = ColumnTransformer(\n",
    "        transformers=[('scale', StandardScaler(), cols_to_scale)],\n",
    "        remainder='passthrough'\n",
    "    )\n",
    "    \n",
    "    # Apply the transformer to the data\n",
    "    df_scaled_array = preprocessor.fit_transform(df_features)\n",
    "    \n",
    "    # Get the list of all column names in the transformed data\n",
    "    all_cols = cols_to_scale + [col for col in df_features.columns if col not in cols_to_scale]\n",
    "    \n",
    "    # Convert the transformed array to a DataFrame\n",
    "    df_scaled = pd.DataFrame(df_scaled_array, columns=all_cols)\n",
    "    \n",
    "    # Reorder the DataFrame to match the original column order\n",
    "    df_scaled = df_scaled[df_features.columns]\n",
    "    \n",
    "    return df_scaled, labels\n",
    "\n",
    "# Define the columns to scale\n",
    "cols_to_scale = ['years_of_experience', 'prev_salary', 'A', 'B', 'D', 'stack_experience']\n",
    "\n",
    "# Apply the function to the training DataFrame\n",
    "train_df_copy, train_labels = scale_and_split(train_df_copy, cols_to_scale, label_column='label', id_column='ID')\n",
    "\n",
    "# Display the scaled DataFrames and labels\n",
    "train_df_copy"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "d5797987",
   "metadata": {},
   "source": [
    "### 6. **Dimensionality:**\n",
    "After creating the features of the technology types, we had a total amount of 133 features. **Such a large amount of features may cause overfitting,** so that the model will adapt itself too much to the training set. It means that the variance will be very high, the bias will be low, and the performance of the model on the 'test' set will be less good. Thats is why we will implement two ways to reduce the number of features:\n",
    "- **Manual download** - in the beginning of the preprocessing stage, we dropped the \"ID\" and \"label\" columns, because they are not relevant for training the model. Also, since we created 116 new features of technology types, we selected only 20 technologies with the highest correlation to the label \"1\" for the purpose of training the models.\n",
    "<br>\n",
    "- **PCA** - before implementing it, we were debating which method to choose. We were afraid that using PCA will cost in losing information and geting lower peformance. On the other hand, we noticed that the \"Feature Selection\" method takes too much time to run the models. Finally, we decided to use the PCA method, with the main reason being due to runtime considerations of model training. In order to lose as little information as possible, we defined in the PCA implementation that the variance will keep 99% of the variance of the original dataset."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "a5399c3b-b6df-4f98-ad54-e5b8ab810cd0",
   "metadata": {},
   "source": [
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "\n",
    "# **Part C - Running models**\n",
    "In this part, we will train 4 selected models.\n",
    "First, we will implement PCA and split the training dataset to \"train\" and \"validation\":"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "686cac14",
   "metadata": {},
   "outputs": [],
   "source": [
    "## Check and drop 'ID' and 'label' columns if they exist\n",
    "columns_to_drop = ['ID', 'label']\n",
    "train_df_copy = train_df_copy.drop(\n",
    "    columns=[col for col in columns_to_drop if col in train_df_copy.columns], inplace=False)\n",
    "\n",
    "# Perform PCA for dimensionality reduction\n",
    "pca = PCA(0.99)  # Adjust the number of components as needed\n",
    "X_pca = pca.fit_transform(train_df_copy)\n",
    "\n",
    "# Split the data into training and test sets\n",
    "X_train, X_test, y_train, y_test = train_test_split(\n",
    "    X_pca, train_df['label'], test_size=0.20, random_state=4242)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "af4c00a3",
   "metadata": {},
   "source": [
    "## Basic models"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "72f6cfe8-dbcb-4209-94e3-6ad73cc3ee3b",
   "metadata": {},
   "source": [
    "## 1. Logistic Regression model:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "5e5f4224-27b5-45da-b186-263100f2f777",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Initialize the Logistic Regression model\n",
    "logistic_model = LogisticRegression()\n",
    "\n",
    "# Define the parameter grid for GridSearchCV\n",
    "param_grid = {\n",
    "    'max_iter': [100],          # Maximum number of iterations\n",
    "    'penalty': ['l1'],          # L1 regularization\n",
    "    'C': [0.1],                 # Inverse of regularization strength\n",
    "    'solver': ['saga'],         # Solver for handling L1 penalty\n",
    "    'l1_ratio': [1]             # Only used if penalty is 'elasticnet'\n",
    "}\n",
    "\n",
    "# Set up GridSearchCV to find the best parameters using AUC as the scoring metric\n",
    "optimizer_logistic = GridSearchCV(logistic_model, param_grid, scoring='roc_auc', cv=5, verbose=1, n_jobs=-1)\n",
    "\n",
    "# Fit the model on the training data\n",
    "optimizer_logistic.fit(X_train, y_train)\n",
    "\n",
    "# Retrieve the best model found by GridSearchCV\n",
    "best_logistic_model = optimizer_logistic.best_estimator_\n",
    "\n",
    "# Predict probabilities on the test set for the positive class\n",
    "y_prob = best_logistic_model.predict_proba(X_test)[:, 1]\n",
    "\n",
    "# Calculate and print the AUC score\n",
    "auc_score = roc_auc_score(y_test, y_prob)\n",
    "print(f\"Optimized AUC: {auc_score:.3f}\")\n",
    "\n",
    "# Plot the ROC curve for the best model\n",
    "RocCurveDisplay.from_estimator(optimizer_logistic, X_test, y_test)\n",
    "\n",
    "# Print the best parameters found by GridSearchCV\n",
    "best_parameters = optimizer_logistic.best_params_\n",
    "print(\"Best parameters found: \", best_parameters)\n",
    "\n",
    "# Predict the class labels on the test set\n",
    "y_pred = optimizer_logistic.predict(X_test)\n",
    "\n",
    "# Compute and display the confusion matrix\n",
    "conf_matrix = confusion_matrix(y_test, y_pred)\n",
    "disp = ConfusionMatrixDisplay(confusion_matrix=conf_matrix)\n",
    "disp.plot(cmap=plt.cm.Blues)\n",
    "plt.title(\"Confusion Matrix\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "317c119a-e0ff-472c-8575-68d008e51e5f",
   "metadata": {},
   "source": [
    "## 2. Naive Bayes model:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "6a1a3be8-8577-4975-92e8-e6ee6c642552",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Define the Gaussian Naive Bayes model\n",
    "NB_model = GaussianNB()\n",
    "\n",
    "# Define the parameter grid\n",
    "param_grid = {\n",
    "    'var_smoothing': [1e-7]  # Example values for var_smoothing\n",
    "}\n",
    "\n",
    "# Initialize GridSearchCV with more cross-validation folds and verbosity\n",
    "optimizer_nb = GridSearchCV(NB_model, param_grid, scoring='roc_auc', cv=5, verbose=1, n_jobs=-1)\n",
    "\n",
    "# Fit the optimizer to the training data\n",
    "optimizer_nb.fit(X_train, y_train)\n",
    "\n",
    "# Retrieve the best model found by GridSearchCV\n",
    "best_nb_model = optimizer_nb.best_estimator_\n",
    "\n",
    "# Predict probabilities and calculate AUC score\n",
    "y_prob = best_nb_model.predict_proba(X_test)[:, 1]\n",
    "auc_score = roc_auc_score(y_test, y_prob)\n",
    "print(f\"Optimized AUC: {auc_score:.3f}\")\n",
    "\n",
    "# Plot ROC curve\n",
    "RocCurveDisplay.from_estimator(optimizer_nb, X_test, y_test)\n",
    "plt.show()\n",
    "\n",
    "# Best parameters\n",
    "best_parameters = optimizer_nb.best_params_\n",
    "print(\"Best parameters found: \", best_parameters)\n",
    "\n",
    "# Predict labels\n",
    "y_pred = optimizer_nb.predict(X_test)\n",
    "\n",
    "# Compute and display the confusion matrix\n",
    "conf_matrix = confusion_matrix(y_test, y_pred)\n",
    "disp = ConfusionMatrixDisplay(confusion_matrix=conf_matrix)\n",
    "disp.plot(cmap=plt.cm.Blues)\n",
    "plt.title(\"Confusion Matrix\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "41d4cee9-e79f-41d9-8b50-4392d5b995aa",
   "metadata": {},
   "source": [
    "## Advanced models"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "828a3813-12ad-4d01-9483-99a204dd6c6d",
   "metadata": {},
   "source": [
    "## 1. Random Forest model:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "720f8974-43e9-46d0-b055-01c033b011a5",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Define the random forest model\n",
    "forest_model = RandomForestClassifier()\n",
    "\n",
    "# Define the parameter grid for GridSearchCV\n",
    "param_grid = {\n",
    "    'n_estimators': [100],           # Number of trees in the forest\n",
    "    'max_depth': [20],               # Maximum depth of the tree\n",
    "    'min_samples_split': [5],        # Minimum number of samples required to split an internal node\n",
    "    'min_samples_leaf': [1],         # Minimum number of samples required to be at a leaf node\n",
    "    'bootstrap': [True],             # Whether bootstrap samples are used when building trees\n",
    "    'max_features': [None],          # Number of features to consider when looking for the best split\n",
    "    'class_weight': ['balanced', None]  # Weighting of classes for imbalance handling\n",
    "}\n",
    "\n",
    "# Set up GridSearchCV for hyperparameter tuning using AUC as the scoring metric\n",
    "optimizer_forest = GridSearchCV(forest_model, param_grid, cv=5, scoring='roc_auc', n_jobs=-1)\n",
    "\n",
    "# Fit the model on the training data\n",
    "optimizer_forest.fit(X_train, y_train)\n",
    "\n",
    "# Retrieve the best model found by GridSearchCV\n",
    "best_forest_model = optimizer_forest.best_estimator_\n",
    "\n",
    "# Predict probabilities on the test set for the positive class\n",
    "y_prob = best_forest_model.predict_proba(X_test)[:, 1]\n",
    "\n",
    "# Calculate and print the AUC score\n",
    "auc_score = roc_auc_score(y_test, y_prob)\n",
    "print(f\"Optimized Random Forest AUC: {auc_score:.3f}\")\n",
    "\n",
    "# Plot the ROC curve for the best model\n",
    "RocCurveDisplay.from_estimator(optimizer_forest, X_test, y_test)\n",
    "\n",
    "# Print the best parameters found by GridSearchCV\n",
    "best_parameters = optimizer_forest.best_params_\n",
    "print(\"Best parameters found: \", best_parameters)\n",
    "\n",
    "# Predict the class labels on the test set\n",
    "y_pred = optimizer_forest.predict(X_test)\n",
    "\n",
    "# Compute the confusion matrix to evaluate the performance of the classifier\n",
    "conf_matrix = confusion_matrix(y_test, y_pred)\n",
    "\n",
    "# Display the confusion matrix as a heatmap\n",
    "disp = ConfusionMatrixDisplay(confusion_matrix=conf_matrix)\n",
    "disp.plot(cmap=plt.cm.Blues)\n",
    "plt.title(\"Confusion Matrix\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "87b37c91-8e11-4897-b888-825f236f6fda",
   "metadata": {},
   "source": [
    "## 2. Multi-Layer Perceptron model:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "fb618fae-bbbf-475f-bfa7-302205cf1fef",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Define the Multi-Layer Perceptron (MLP) model\n",
    "MLP_model = MLPClassifier()\n",
    "\n",
    "# Expanded parameter grid for GridSearchCV to explore more hyperparameter combinations\n",
    "param_grid = {\n",
    "    'hidden_layer_sizes': [(100, 100)],  # Configuration of hidden layers\n",
    "    'activation': ['tanh'],              # Activation function for the hidden layers\n",
    "    'solver': ['sgd'],                   # Solver for weight optimization\n",
    "    'alpha': [0.1],                      # L2 penalty (regularization term) parameter\n",
    "    'learning_rate': ['adaptive'],       # Learning rate schedule for weight updates\n",
    "    'learning_rate_init': [1],           # Initial learning rate\n",
    "    'max_iter': [1000]                   # Maximum number of iterations\n",
    "}\n",
    "\n",
    "# Initialize GridSearchCV with the MLP model and the defined parameter grid\n",
    "# Scoring is based on the AUC, using 5-fold cross-validation\n",
    "optimizer_mlp = GridSearchCV(MLP_model, param_grid, scoring='roc_auc', cv=5, verbose=2, n_jobs=-1)\n",
    "\n",
    "# Fit the optimizer to the training data to find the best hyperparameters\n",
    "optimizer_mlp.fit(X_train, y_train)\n",
    "\n",
    "# Retrieve the best model found by GridSearchCV\n",
    "best_mlp_model = optimizer_mlp.best_estimator_\n",
    "\n",
    "# Predict probabilities on the test set for the positive class\n",
    "y_prob = best_mlp_model.predict_proba(X_test)[:, 1]\n",
    "\n",
    "# Calculate and print the AUC score for the best model\n",
    "auc_score = roc_auc_score(y_test, y_prob)\n",
    "print(f\"Multi-Layer Perceptron AUC: {auc_score:.3f}\")\n",
    "\n",
    "# Plot the ROC curve for the best model\n",
    "RocCurveDisplay.from_estimator(optimizer_mlp, X_test, y_test)\n",
    "\n",
    "# Print the best hyperparameters found by GridSearchCV\n",
    "best_parameters = optimizer_mlp.best_params_\n",
    "print(\"Best parameters found: \", best_parameters)\n",
    "\n",
    "# Predict the class labels on the test set\n",
    "y_pred = optimizer_mlp.predict(X_test)\n",
    "\n",
    "# Compute the confusion matrix to evaluate the performance of the classifier\n",
    "conf_matrix = confusion_matrix(y_test, y_pred)\n",
    "\n",
    "# Display the confusion matrix as a heatmap\n",
    "disp = ConfusionMatrixDisplay(confusion_matrix=conf_matrix)\n",
    "disp.plot(cmap=plt.cm.Blues)\n",
    "plt.title(\"Confusion Matrix\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "9788703d",
   "metadata": {},
   "source": [
    "### It seems that the MLP is our best model!\n",
    "So we will analyze its performance and later on run it with the Test dataset."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "42d0f1cd",
   "metadata": {},
   "source": [
    "## Feature Importance\n",
    "We want to know which features are mostly contribute the model's performance. <br>\n",
    "Since we learned in class only \"Feature Importance\" method which is used only for Random Forest model, we will use a technique called **\"Permutation Feature Importance\"**. This method helps to measure how much the model’s performance **decreases** when the values of a particular feature are randomly shuffled or permuted while keeping other variables unchanged."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "3ec5a7a1",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Calculate permutation feature importance\n",
    "result = permutation_importance(best_mlp_model, X_test, y_test, n_repeats=10, random_state=42, n_jobs=-1)\n",
    "\n",
    "# Get feature importances and their corresponding indices\n",
    "importance_means = result.importances_mean\n",
    "importance_std = result.importances_std\n",
    "indices = np.argsort(importance_means)[::-1]\n",
    "\n",
    "# Get PCA components and original feature names\n",
    "pca_components = pca.components_\n",
    "original_feature_names = train_df_copy.columns\n",
    "\n",
    "# Calculate feature importance based on PCA loadings\n",
    "feature_importance = np.abs(pca_components.T @ importance_means)\n",
    "sorted_indices = np.argsort(feature_importance)[::-1]\n",
    "\n",
    "# Plot feature importances with original feature names\n",
    "plt.figure(figsize=(12, 8))\n",
    "plt.title(\"Feature Importance using Permutation\")\n",
    "plt.bar(range(len(original_feature_names)), feature_importance[sorted_indices], color=\"b\", align=\"center\")\n",
    "plt.xticks(range(len(original_feature_names)), [original_feature_names[i] for i in sorted_indices], rotation=90)\n",
    "plt.xlim([-1, len(original_feature_names)])\n",
    "plt.xlabel(\"Features\")\n",
    "plt.ylabel(\"Importance\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "4e6ba342",
   "metadata": {},
   "source": [
    "As you can see, the most important feature is \"stack_experience\", and right after it come the different technologies. <br> <br>\n",
    "**This plot supports our assumption that knowing different technologies and programming languages increases a person's chances to get hired for a job.**"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "88ce4a16-b081-4863-a6cd-ab3eec10fc94",
   "metadata": {},
   "source": [
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "\n",
    "# **Part D - Model Evaluating**"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "6401172c-cb5a-4418-8532-ed0646dca645",
   "metadata": {},
   "source": [
    "### Here we can see again the Confusion Matrix of the MLP model:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "316c096e-0926-4ac9-aba4-24eb75596aa5",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Plot the ROC curve for the best MLP model found by GridSearchCV\n",
    "RocCurveDisplay.from_estimator(optimizer_mlp, X_test, y_test)\n",
    "\n",
    "# Retrieve and print the best hyperparameters found by GridSearchCV\n",
    "best_parameters = optimizer_mlp.best_params_\n",
    "print(\"Best parameters found: \", best_parameters)\n",
    "\n",
    "# Predict the class labels on the test set using the best MLP model\n",
    "y_pred = optimizer_mlp.predict(X_test)\n",
    "\n",
    "# Compute the confusion matrix to evaluate the performance of the model\n",
    "conf_matrix = confusion_matrix(y_test, y_pred)\n",
    "\n",
    "# Display the confusion matrix as a heatmap for visual inspection\n",
    "disp = ConfusionMatrixDisplay(confusion_matrix=conf_matrix)\n",
    "disp.plot(cmap=plt.cm.Blues)\n",
    "plt.title(\"Confusion Matrix\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "2bb80a9d-e6cd-4a69-98c0-50c28ca9eb4f",
   "metadata": {},
   "source": [
    "Here is a breakdown of each cell in the matrix:<br>\n",
    "- **TN** (True Negatives, upper left corner, 4614): The model correctly predicted 0 (negative classification) when the actual class was 0. <br>\n",
    "- **FP** (False Positives, upper right corner, 497): The model falsely predicted 1 (positive classification) when the actual class was 0.<br>\n",
    "- **FN** (False Negatives, lower left corner, 437): The model falsely predicted 0 (negative classification) when the actual class was 1.<br>\n",
    "- **TP** (True Positives, lower right corner, 5545): The model correctly predicted 1 (positive classification) when the actual class was 1.<br>\n",
    "<br> From these values we can calculate several important performance indicators:\n",
    "\n",
    "1. **Accuracy:** <br>\n",
    "(TP+TN) / TP+TN+FP+FN = (5554+4604) / 5554+4604+507+428 ≈ 0.916 <br>\n",
    "It means that in 91.6% of the cases, the model correctly predicts the classifications.\n",
    "\n",
    "2. **Precision:** <br>\n",
    "TP / (TP+FP) = 5554 / (5545+507) ≈ 0.916 <br>\n",
    "It means that when the model predicts a positive classification, it is correct about 91.6% of the time, indicating a low rate of false positive predictions.\n",
    "\n",
    "3. **Sensitivity:** <br>\n",
    "TP / (TP+FN) = 5545 / (5545+428) ≈ 0.928 <br>\n",
    "It means that the model correctly identifies 92.8% of the actual positive cases, which indicates a low rate of false negative predictions.\n",
    "\n",
    "4. **Specificity:** <br>\n",
    "TN / (TN+FP) = 4604 / (4604+507) ≈ 0.90 <br>\n",
    "It means that the model correctly identifies 90% of the actual negative cases, indicating good performance in preventing false positive predictions."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "cb3140f4-573a-40c2-82bf-146a59c6e963",
   "metadata": {},
   "source": [
    "### Here we will run our MLP model without using K-Cross validation:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "0867e8b1-c5d1-458b-8cdd-ec67628d362b",
   "metadata": {},
   "outputs": [],
   "source": [
    "MLP_model = MLPClassifier(\n",
    "    hidden_layer_sizes=(100, 100),\n",
    "    activation='tanh',\n",
    "    solver='sgd',\n",
    "    alpha=0.1,\n",
    "    learning_rate='adaptive',\n",
    "    learning_rate_init=0.1,\n",
    "    max_iter=1000\n",
    ")\n",
    "\n",
    "# Fit the model to the training data\n",
    "MLP_model.fit(train_df_copy, train_df['label'])\n",
    "\n",
    "# Display the ROC curve\n",
    "RocCurveDisplay.from_estimator(MLP_model, train_df_copy, train_df['label'])\n",
    "plt.title(\"ROC Curve\")\n",
    "plt.show()\n",
    "\n",
    "# Predict the test data\n",
    "y_pred = MLP_model.predict(train_df_copy)\n",
    "\n",
    "# Compute and display the confusion matrix\n",
    "conf_matrix = confusion_matrix(train_df['label'], y_pred)\n",
    "disp = ConfusionMatrixDisplay(confusion_matrix=conf_matrix)\n",
    "disp.plot(cmap=plt.cm.Blues)\n",
    "plt.title(\"Confusion Matrix\")\n",
    "plt.show()\n",
    "\n",
    "# Calculate and print the AUC score\n",
    "y_proba = MLP_model.predict_proba(train_df_copy)[:, 1]\n",
    "auc_score = roc_auc_score(train_df['label'], y_proba)\n",
    "print(f\"AUC score: {auc_score:.3f}\")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "2652e72f-cd1b-4243-ae19-4146da91c9fd",
   "metadata": {},
   "source": [
    "As you can see, the model's AUC without K-Cross validation is only 0.001 higher than the model's AUC using K-Cross validation. <br>\n",
    "**It indicates that the model is not overfitted.** <br>\n",
    "It would have been considered as overfitted **if the AUC was much lower after using K-Cross validation**. <Br>\n",
    "After all, we expect to get high AUC without using K-Cross validation, so getting almost the same results after testing on the validation set is a good sign."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "586122e2",
   "metadata": {},
   "source": [
    "## Here you can see all the 4 ROC curves of the model above:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "b330aec4-e68d-4410-b6c0-a338631c35c0",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Dictionary to store the different models and their corresponding optimized versions\n",
    "models = {\n",
    "    \"Logistic Regression\": optimizer_logistic,\n",
    "    \"Naive Bayes\": optimizer_nb,\n",
    "    \"Random Forest\": optimizer_forest,\n",
    "    \"MLP\": optimizer_mlp\n",
    "}\n",
    "\n",
    "# Set up the plot for ROC curves for each model\n",
    "plt.figure(figsize=(10, 8))\n",
    "\n",
    "# Iterate over each model in the dictionary to calculate and plot the ROC curve\n",
    "for name, model in models.items():\n",
    "    y_proba = model.predict_proba(X_test)[:, 1]  # Get the predicted probabilities for the positive class\n",
    "    fpr, tpr, _ = roc_curve(y_test, y_proba)     # Compute the false positive rate and true positive rate\n",
    "    auc_score = roc_auc_score(y_test, y_proba)   # Calculate the AUC score for the ROC curve\n",
    "    plt.plot(fpr, tpr, label=f'{name} (AUC = {auc_score:.3f})')  # Plot the ROC curve for this model\n",
    "\n",
    "# Add labels and title to the ROC plot\n",
    "plt.xlabel('False Positive Rate')\n",
    "plt.ylabel('True Positive Rate')\n",
    "plt.title('ROC Curves')\n",
    "plt.legend(loc='lower right')  # Add a legend to identify each model's ROC curve\n",
    "plt.show()  # Display the plot"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "e2def94b",
   "metadata": {},
   "source": [
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "<br>\n",
    "\n",
    "\n",
    "\n",
    "# **Part E - Prediction**"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "497ea671-12eb-4e77-a9a5-cff3a18cb2d2",
   "metadata": {},
   "outputs": [],
   "source": [
    "import warnings\n",
    "warnings.filterwarnings('ignore')\n",
    "import numpy as np\n",
    "import pandas as pd\n",
    "import matplotlib.pyplot as plt\n",
    "import sklearn\n",
    "import seaborn as sns\n",
    "from sklearn.decomposition import PCA\n",
    "from sklearn.preprocessing import StandardScaler,FunctionTransformer, LabelEncoder\n",
    "from sklearn.ensemble import RandomForestClassifier\n",
    "from sklearn.linear_model import LogisticRegression\n",
    "from sklearn.metrics import auc, roc_curve, RocCurveDisplay, accuracy_score, roc_auc_score, confusion_matrix, classification_report,ConfusionMatrixDisplay\n",
    "from sklearn.model_selection import StratifiedKFold, GridSearchCV, train_test_split\n",
    "from sklearn.compose import ColumnTransformer\n",
    "from sklearn.naive_bayes import GaussianNB\n",
    "from sklearn.neural_network import MLPClassifier\n",
    "from collections import Counter\n",
    "from sklearn.inspection import permutation_importance\n",
    "import os\n"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "de8fc75f-039a-46bd-a28d-cc355c2ae56b",
   "metadata": {},
   "source": [
    "### First, we will create a function that units all the functions from the preprocessing stage:"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "ed556841",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Unit all the functions from the preprocessing stage and apply them on the \"train\" and \"test\" datasets.\n",
    "\n",
    "def preprocess_all(train_df, test_df):\n",
    "    \n",
    "    def cap_outliers_percentiles_multi(df, columns, lower_percentile=0.01, upper_percentile=0.99):\n",
    "        for column in columns:\n",
    "            lower_bound = df[column].quantile(lower_percentile)\n",
    "            upper_bound = df[column].quantile(upper_percentile)\n",
    "            df[column] = df[column].apply(lambda x: upper_bound if x > upper_bound else (lower_bound if x < lower_bound else x))\n",
    "        return df\n",
    "    numeric_columns = ['A', 'B', 'D']\n",
    "    \n",
    "    train_df = cap_outliers_percentiles_multi(train_df, numeric_columns)\n",
    "    \n",
    "    def process_stack_experience(train_df, label_column, top_n=20):\n",
    "        technologies = train_df['stack_experience'].dropna().str.split(';').tolist()\n",
    "        flattened_technologies = [tech for sublist in technologies for tech in sublist]\n",
    "        technology_counts = Counter(flattened_technologies)\n",
    "        unique_technologies_count = len(technology_counts)\n",
    "        stack_experience_dummies = train_df['stack_experience'].str.get_dummies(sep=';')\n",
    "        correlation_with_label = stack_experience_dummies.corrwith(train_df[label_column]).sort_values(ascending=False)\n",
    "        top_technologies = correlation_with_label.head(top_n).index\n",
    "        train_df = pd.concat([train_df, stack_experience_dummies[top_technologies]], axis=1)\n",
    "        return train_df, unique_technologies_count, top_technologies, correlation_with_label\n",
    "    \n",
    "    train_df, unique_technologies_count, top_technologies, correlation_with_label = process_stack_experience(train_df, 'label')\n",
    "    \n",
    "    def add_top_technologies_to_test_df(test_df, top_technologies):\n",
    "        stack_experience_dummies_test = test_df['stack_experience'].str.get_dummies(sep=';')\n",
    "        test_top_tech_dummies = stack_experience_dummies_test.reindex(columns=top_technologies, fill_value=0)\n",
    "        test_df = pd.concat([test_df, test_top_tech_dummies], axis=1)\n",
    "        return test_df\n",
    "    \n",
    "    test_df = add_top_technologies_to_test_df(test_df, top_technologies)\n",
    "        \n",
    "    \n",
    "    def fill_b_and_experience(df, b_col, exp_col):\n",
    "        b_mean = df[b_col].mean()\n",
    "        exp_mean = df[exp_col].mean()\n",
    "\n",
    "        def fill_b(row):\n",
    "            if pd.isna(row[b_col]):\n",
    "                if pd.isna(row[exp_col]):\n",
    "                    return b_mean\n",
    "                else:\n",
    "                    return max(row[exp_col] - 5, 0)\n",
    "            else:\n",
    "                return row[b_col]\n",
    "\n",
    "        def fill_exp(row):\n",
    "            if pd.isna(row[exp_col]):\n",
    "                if pd.isna(row[b_col]):\n",
    "                    return exp_mean\n",
    "                else:\n",
    "                    return row[b_col] + 5\n",
    "            else:\n",
    "                return row[exp_col]\n",
    "\n",
    "        df[b_col] = df.apply(lambda row: fill_b(row), axis=1)\n",
    "        df[exp_col] = df.apply(lambda row: fill_exp(row), axis=1)\n",
    "        return df\n",
    "\n",
    "\n",
    "    def replace_words_with_count(df, column_name):\n",
    "        df[column_name] = df[column_name].apply(lambda cell: len(cell.split(';')) if pd.notna(cell) else 0)\n",
    "        return df\n",
    "\n",
    "\n",
    "    def fill_na_with_ratio(df):\n",
    "        for column in df:\n",
    "            if column == 'is_dev' or column == 'age_group' or column == 'worked_in_the_past' :\n",
    "                continue\n",
    "            non_na_values = df[column].dropna()\n",
    "            value_counts = non_na_values.value_counts(normalize=True)\n",
    "            df[column] = df[column].apply(\n",
    "                lambda x: np.random.choice(value_counts.index, p=value_counts.values) if pd.isna(x) else x\n",
    "            )\n",
    "        return df\n",
    "\n",
    "\n",
    "    def fill_is_dev_based_on_stack_experience(df, stack_experience_col, is_dev_col):\n",
    "        df[is_dev_col] = df.apply(\n",
    "            lambda row: 'developer' if pd.isna(row[is_dev_col]) and row[stack_experience_col] > 0 else (\n",
    "                'non_developer' if pd.isna(row[is_dev_col]) else row[is_dev_col]\n",
    "            ), axis=1\n",
    "        )\n",
    "        return df\n",
    "\n",
    "\n",
    "    def fill_age_group_based_on_experience(df, experience_col, age_group_col):\n",
    "        df[age_group_col] = df.apply(\n",
    "            lambda row: 'old' if pd.isna(row[age_group_col]) and row[experience_col] > 20 else (\n",
    "                \"young\" if pd.isna(row[age_group_col]) else row[age_group_col]\n",
    "            ), axis=1\n",
    "        )\n",
    "        return df\n",
    "\n",
    "\n",
    "    def fill_worked_in_past(df, worked_col='worked_in_the_past', experience_col='years_of_experience'):\n",
    "        df[worked_col] = df.apply(\n",
    "            lambda row: True if (pd.isna(row[worked_col]) and row[experience_col] > 0) or pd.isna(row[experience_col]) \n",
    "            else row[worked_col] if not pd.isna(row[worked_col])\n",
    "            else False,\n",
    "            axis=1\n",
    "        )\n",
    "        return df\n",
    "        \n",
    "        \n",
    "    train_df = fill_b_and_experience(train_df, 'B', 'years_of_experience')\n",
    "    test_df = fill_b_and_experience(test_df, 'B', 'years_of_experience')\n",
    "    train_df = replace_words_with_count(train_df, 'stack_experience')\n",
    "    test_df = replace_words_with_count(test_df, 'stack_experience')\n",
    "    train_df = fill_na_with_ratio(train_df)\n",
    "    test_df = fill_na_with_ratio(test_df)\n",
    "    train_df = fill_is_dev_based_on_stack_experience(train_df,'stack_experience','is_dev')\n",
    "    test_df = fill_is_dev_based_on_stack_experience(test_df,'stack_experience','is_dev')\n",
    "    train_df = fill_age_group_based_on_experience(train_df,'years_of_experience','age_group')\n",
    "    test_df = fill_age_group_based_on_experience(test_df,'years_of_experience','age_group')\n",
    "    train_df = fill_worked_in_past(train_df)\n",
    "    test_df = fill_worked_in_past(test_df)\n",
    "    \n",
    "    def calculate_punishment_metric(train_df, label_column, feature_columns):\n",
    "        if label_column not in train_df.columns:\n",
    "            print(f\"Error: '{label_column}' column does not exist in the training DataFrame.\")\n",
    "            return train_df, {}\n",
    "        punishment_mappings = {}\n",
    "        for feature in feature_columns:\n",
    "            if feature not in train_df.columns:\n",
    "                print(f\"Feature column {feature} not found in the training DataFrame.\")\n",
    "                continue\n",
    "            total_labels = train_df.groupby(feature)[label_column].count()\n",
    "            label_1_counts = train_df[train_df[label_column] == 1].groupby(feature)[label_column].count()\n",
    "            comparison_df = pd.DataFrame({'Total Labels': total_labels, 'Label 1 Counts': label_1_counts})\n",
    "            comparison_df = comparison_df.fillna(0)\n",
    "            comparison_df['Proportion of Label 1'] = comparison_df['Label 1 Counts'] / comparison_df['Total Labels']\n",
    "            comparison_df['Punishment Metric'] = comparison_df['Proportion of Label 1']\n",
    "            punishment_mapping = comparison_df['Punishment Metric'].to_dict()\n",
    "            punishment_mappings[feature] = punishment_mapping\n",
    "            train_df[feature] = train_df[feature].map(punishment_mapping)\n",
    "        return train_df, punishment_mappings\n",
    "    \n",
    "    array1 = ['worked_in_the_past', 'age_group', 'disability', 'is_dev',\n",
    "          'mental_issues', 'education', 'C', 'country', 'sex']\n",
    "    train_df, punishment_mappings = calculate_punishment_metric(train_df, 'label', array1)\n",
    "    \n",
    "    def apply_punishment_metric(test_df, punishment_mappings):\n",
    "        for feature, punishment_mapping in punishment_mappings.items():\n",
    "            if feature in test_df.columns:\n",
    "                test_df[feature] = test_df[feature].map(punishment_mapping).fillna(0)\n",
    "            else:\n",
    "                print(f\"Feature column {feature} not found in the test DataFrame.\")\n",
    "\n",
    "        return test_df\n",
    "\n",
    "    test_df = apply_punishment_metric(test_df, punishment_mappings)\n",
    "    \n",
    "    def scale_and_split(df, cols_to_scale, label_column=None, id_column=None):\n",
    "        if label_column and label_column in df.columns:\n",
    "            labels = df[label_column]\n",
    "            df_features = df.drop(columns=[label_column])\n",
    "        else:\n",
    "            labels = None\n",
    "            df_features = df.copy()\n",
    "        if id_column and id_column in df.columns:\n",
    "            df_features = df_features.drop(columns=[id_column])\n",
    "        preprocessor = ColumnTransformer(\n",
    "            transformers=[('scale', StandardScaler(), cols_to_scale)],\n",
    "            remainder='passthrough'\n",
    "        )\n",
    "        df_scaled_array = preprocessor.fit_transform(df_features)\n",
    "        all_cols = cols_to_scale + [col for col in df_features.columns if col not in cols_to_scale]\n",
    "        df_scaled = pd.DataFrame(df_scaled_array, columns=all_cols)\n",
    "        df_scaled = df_scaled[df_features.columns]\n",
    "        return df_scaled, labels\n",
    "    \n",
    "    cols_to_scale = ['years_of_experience', 'prev_salary', 'A', 'B', 'D', 'stack_experience']\n",
    "    train_df, train_labels = scale_and_split(train_df, cols_to_scale, label_column='label', id_column='ID')\n",
    "    test_df, _ = scale_and_split(test_df, cols_to_scale, label_column='label', id_column='ID')\n",
    "        \n",
    "    return train_df, test_df"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "053fa460",
   "metadata": {},
   "source": [
    "# Pipline\n",
    "Here, you can run the important things:\n",
    "1. Loading the datasets and applying the preprocessing stage.\n",
    "2. Defining our best model and training it.\n",
    "3. Creating predictions on the data from the \"test\" dataset, and save it in a file called \"result_11.csv\""
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "db04b850",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Loading the datasets, doing preprocessing on the datasets, training the best model and create predictions.\n",
    "\n",
    "def pipeline(train_csv, test_csv, results_11_csv):\n",
    "    # Load the training and test datasets from CSV files\n",
    "    train_df = pd.read_csv(train_csv)\n",
    "    test_df = pd.read_csv(test_csv)\n",
    "    \n",
    "    # Extract labels from the training data and IDs from the test data\n",
    "    train_labels = train_df.label\n",
    "    test_ID = test_df.ID\n",
    "    \n",
    "    # Preprocess the data (assumes preprocess_all is a defined function)\n",
    "    train_df_run, test_df_run = preprocess_all(train_df, test_df)\n",
    "    \n",
    "    # Set up the training and test datasets for the model\n",
    "    X_train = train_df_run\n",
    "    y_train = train_labels\n",
    "    X_test = test_df_run.drop(columns=['ID'], errors='ignore')  # Drop ID column from test set if it exists\n",
    "    \n",
    "    # Define and train the MLP model with specified parameters\n",
    "    mlp_model = MLPClassifier(activation='tanh', alpha=0.1, hidden_layer_sizes=(100, 100), \n",
    "                              learning_rate='adaptive', learning_rate_init=0.1, solver='sgd')\n",
    "    mlp_model.fit(X_train, y_train)\n",
    "    \n",
    "    # Predict probabilities for the test set\n",
    "    probabilities = mlp_model.predict_proba(X_test)\n",
    "    \n",
    "    # Extract probabilities for the positive class (class 1)\n",
    "    positive_class_probabilities = probabilities[:, 1]\n",
    "    \n",
    "    # Remove the existing results file if it exists to avoid overwriting issues\n",
    "    if os.path.exists(results_11_csv):\n",
    "        os.remove(results_11_csv)\n",
    "    \n",
    "    # Create a DataFrame with test IDs and predicted probabilities\n",
    "    prediction_file = pd.DataFrame({'ID': test_ID, 'probability': positive_class_probabilities})\n",
    "    \n",
    "    # Save the predictions to a CSV file\n",
    "    prediction_file.to_csv(results_11_csv, index=False)\n",
    "    \n",
    "    # Print confirmation that the results file has been created\n",
    "    print(f\"The file '{results_11_csv}' is created and ready to use!\")\n",
    "\n",
    "# Run the pipeline function with specified CSV files\n",
    "pipeline('train.csv', 'test.csv', 'results_11.csv')"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3 (ipykernel)",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.11.3"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 5
}
