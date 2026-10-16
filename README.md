# Netflix_Movie_Data_Analysis.ipynb
# IMPORTING LIB
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
df = pd.read_csv('/content/mymoviedb.csv', lineterminator='\n')
df.head()

# Viewing Dataset
df.info()
looks like our dataset has no NaNs! • Overview, Original_Language and Poster-Url wouldn't be so useful during analysis • Release_Date column needs to be casted into date time and to extract only the year value
# Exploring Geners Column
df['Genre'].head()
df.shape
Exploration Summary

• we have a dataframe consisting of 9827 rows and 9 columns.

• our dataset looks a bit tidy with no NaNs nor duplicated values.

• Release_Date column needs to be casted into date time and to extract only the year value

• Overview, Original_Languege and Poster-Url wouldn't be so useful during analysis

• there is noticable outliers in Popularity column

• Vote_Average bettter be categorised for proper analysis.

• Genre column has comma saperated values and white spaces that needs to be handc
df.head()
df['Release_Date'] = pd.to_datetime(df['Release_Date'])
df['Release_Date'].head()
# Conforming Changes
print(df['Release_Date'].dtypes)
df['Release_Date'] = df['Release_Date'].dt.year
df['Release_Date'].dtypes
df.info()
df.head()
# **Dropping Overview, Original_Language and Poster_Url **
# List column to be dropped
cols = ['Overview', 'Original_Language', 'Poster_Url']
df.drop(cols, axis=1, inplace=True, errors='ignore')
df.columns
df.head()
# categorizing Vote_Average **column**
We would cut the Vote_Average values and make 4 categories: popular average below_avg not_popular to describe it more using catigorize_col() function provided above.

def catigorize_col(df, col, labels):
    """
    catigorizes a certain column based on its quartiles

    Args:
        (df)  df  - dataframe we are proccesing
        (col) str - to be catigorized column's name
        (labels) list - list of labels from min to max

    Returns:
        (df)  df  - dataframe with the categorized col
    """

    # setting the edges to cut the column accordingly
    edges = [df[col].describe()['min'],
             df[col].describe()['25%'],
             df[col].describe()['50%'],
             df[col].describe()['75%'],
             df[col].describe()['max']]
             # 1. Reload the data from the beginning and clean it
df = pd.read_csv('/content/mymoviedb.csv', lineterminator='\n')
df['Release_Date'] = pd.to_datetime(df['Release_Date']).dt.year
df.drop(['Overview', 'Original_Language', 'Poster_Url'], axis=1, inplace=True, errors='ignore')

# 2. Recategorize Vote_Average
labels = ['not_popular', 'below_avg', 'average', 'popular']
edges = [df['Vote_Average'].describe()['min'],
         df['Vote_Average'].describe()['25%'],
         df['Vote_Average'].describe()['50%'],
         df['Vote_Average'].describe()['75%'],
         df['Vote_Average'].describe()['max']]
df['Vote_Average_Levels'] = pd.cut(df['Vote_Average'], edges, labels=labels, duplicates='drop', include_lowest=True)
df.dropna(inplace=True)

# 3. Split and explode the Genre (run this only once!)
df['Genre'] = df['Genre'].astype(str).str.split(', ')
df = df.explode('Genre')
df = df.reset_index(drop=True)

df.head()
def catigorize_col(df, col, labels):
    """
    catigorizes a certain column based on its quartiles
    """
    edges = [
        df[col].describe()["min"],
        df[col].describe()["25%"],
        df[col].describe()["50%"],
        df[col].describe()["75%"],
        df[col].describe()["max"],
    ]
    df["Vote_Average_Levels"] = pd.cut(
        df[col], edges, labels=labels, duplicates="drop", include_lowest=True
    )

    # ADD THIS LINE:
    return df

    df.head()
    #Exploring column
df['Vote_Average_Levels'].value_counts()

# define levels and edges directly
labels = ['not_popular', 'below_avg', 'average', 'popular']
edges = [
    df['Vote_Average'].describe()['min'],
    df['Vote_Average'].describe()['25%'],
    df['Vote_Average'].describe()['50%'],
    df['Vote_Average'].describe()['75%'],
    df['Vote_Average'].describe()['max']
]

df['Vote_Average_Levels'] = pd.cut(
    df['Vote_Average'],
    edges,
    labels=labels,
    duplicates='drop',
    include_lowest=True
)

df['Vote_Average_Levels'].head()

# droping NANs
df.dropna(inplace = True)

# confirming
df.isna().sum()

df.head()

# **we'd split genres into a list and then explode our dataframe to have only one genre per row for ezch movie**
# split the strings into lists safely by ensuring we operate on strings
df['Genre'] = df['Genre'].astype(str).str.split(', ')

# explode the lists
df = df.explode('Genre')

# reset the index
df = df.reset_index(drop=True)

df.head()

df['Genre'].dtypes
df.info()
df.nunique()
Now that our dataset is clean and tidy, we are left with a total of 6 columns and 25551 rows to dig into during our analysis

# **Data Visualization**
Here, we would employ plotting libraries like Matplotlib and Seaborn to generate insightful visualizations that help us better understand our data.
# Setting up Seaborn configuration
sns.set_style('whitegrid')

# Q1: What is the most frequent genre in the dataset?
df['Genre'].describe()

# visualizing genre column
sns.catplot(
    y='Genre',
    data=df,
    kind='count',
    order=df['Genre'].value_counts().index,
    color='#4287f5',
)
plt.title('genre column distribution')
plt.show()

we can notice from the above visual that Drama genre is the most frequent genre in our dataset and has appeared more than 14% of the times among 19 other genres

## **Q2: What genres has highest votes?**
# Visualizing Vote_Average distribution using a histogram/KDE
plt.figure(figsize=(10, 5))
sns.histplot(df['Vote_Average'], bins=20, kde=True, color='#4287f5')
plt.title('Vote Average Distribution')
plt.xlabel('Vote Average')
plt.ylabel('Frequency')
plt.show()

# Visualizing Vote Average Levels instead of raw Vote_Average
plt.figure(figsize=(8, 5))
sns.countplot(
    y='Vote_Average_Levels',
    data=df,
    order=['not_popular', 'below_avg', 'average', 'popular'],
    palette='Blues_r',
)
plt.title('Vote Average Levels Distribution')
plt.xlabel('Count')
plt.ylabel('Vote Average Level')
plt.show()

# Q3: What movie got the highest popularity? what's its genre?
# checking max popularity in dataset
df[df['Popularity'] == df['Popularity'].max()]

# Q4: What movie got the lowest popularity? what's its genre?
# checking min popularity in dataset
df[df['Popularity'] == df['Popularity'].min()]

# Q5: Which year has the most filumed movies?
# checking distribution of release dates
df['Release_Date'].hist()
plt.title('Release_Date column distribution')
plt.show()

# **Conclusion**
Q1: What is the most frequent genre in the dataset?

Drama genre is the most frequent genre in our dataset and has appeared more than 14% of the times among 19 other genres.**bold text**

Q2: What genres has highest votes?

We have 25.5% of our dataset with popular vote (6520 rows). Drama again gets the highest popularity among fans by having more than 18.5% of movies popularities.

Q3: What movie got the highest popularity? what's its genre?

Spider-Man: No Way Home has the highest popularity rate in our dataset and it has genres of Action, Adventure and Science Fiction.

Q4: What movie got the lowest popularity? what's its genre?

The United States vs. Billie Holiday has the lowest rate in our dataset and it has genres of music, drama, war, sci-fi and history.

Q5: Which year has the most filmed movies?

Year 2020 has the highest filming rate in our dataset.
