---
toc: true
layout: post
description: "Introduction to Aethos."
categories: [markdown, aethos]
image: https://miro.medium.com/max/742/1*WdxHNOegMN9Eb9fZRN1cCg.png
title: "Aethos — A Data Science Library to Automate your Workflow"
---

As a Data Scientist in industry, there are a few pain points that I’m sure a lot of other data scientists can relate to:

1.  Copy and pasting code from previous projects to your current project or from notebook to notebook is frustrating.
2.  For the same task, each data scientist will do it their own way.
3.  Managing experiments with notebook names gets unorganized very quickly and structuring data science projects can be a mess.
4.  Documenting your process while you’re in the middle of your analysis often disrupts your flow.
5.  Version controlling models, transforming it into a service, and developing a data pipeline after a model has been trained.
6.  There are tons of different libraries coming out that each fill a niche of Data Science & ML and learning a new API (especially for deep learning and NLP) or reading documentation (if there is any) every time gets a little tiresome. Ironic, I know.
7.  Making just even semi-appealing visualizations is often time consuming and sometimes difficult.

This is why I made Aethos.

[Aethos](https://github.com/Ashton-Sidhu/aethos) is a Python library of automated Data Science techniques and use cases from missing value imputation, to NLP pre-processing, feature engineering, data visualizations and all the way to modelling and generating code to deploy your model as a service and soon, your data pipeline.

Installation
============

Let’s install Aethos like any other Python package:

```
pip install aethos
```

Once Aethos is installed, we can already do a couple of things. We can install all the required NLP corpora,

```
aethos install-corpora
```

install extensions such as QGrid for a better experience when interacting with your data

```
aethos enable-extensions
```

and create a Data Science project with the full folder structure to house source code, models, data and experiments.

```
aethos create
```

Getting Started
===============

First, let’s import the necessary libraries:

```python
import aethos as at  
import pandas as pd
```

Before we start, let’s also configure some options that will make our life easier. For myself, I often write a report of the steps I took during my analysis and why I did them to better communicate my process to other data scientists on the team or to myself if I come back to an experiment after a prolonged length of time. Instead of writing reports manually, Aethos will automatically write the report as a text file as we go through our analysis (all reports are saved in %HOME%/.aethos/reports/). To enable Aethos to write the report as a word doc, we just have to enable the option:

```python
at.options.word_report = True  
at.options.interactive_table = True
```

The `interactive_table` option will display our data using the [itables](https://github.com/mwouts/itables) library, a personal favorite of mine as it has a client side search feature.

Now let’s load our data. Aethos takes in data in the form of a pandas dataframe so data is loaded just like if you were working with pandas. We’re going to use the titanic dataset for this example.

```python
data = pd.read_csv('titanic.csv')
```

To use Aethos, we just have to pass the dataframe into the Aethos Data object.

```python
df = at.Data(data, target_field='Survived', report_name='titanic')  
df
```

<img class="dj t u eh ak" src="https://miro.medium.com/max/2196/1*REYNw0ITYeRKbFoJy_2R3w.png" width="1098" height="416" role="presentation"/>

Couple of things to note about what just happened:

*   Since this is a supervised learning problem we specified the column we want to predict (‘Survived’).
*   By default since we did not pass any test data to the Data object, Aethos automatically splits the data given into a train and test set. This can be turned off by setting `split=False` when initializing the Data object.
*   By default, the split percentage is 20. This can be changed by setting `test_split_percentage` to a float between 0 and 1 (by default it is 0.2).
*   We specified the report name to `titanic`, since we specified we want a Word Doc created, a `titanic.txt` and a `titanic.docx` will be created in `%HOME%/.aethos/reports/`.

Let’s get started with some basic analysis.

Analysis
========

Using the Data object is just like working with a Pandas DataFrame.

```python
df['Age'] # returns a Pandas series of the training data in the Age column

df[df['Age'] > 25] # returns a Pandas DataFrame of the the training data where Age is > 25.

df.nunique() # You can even run pandas functions on the Data object, however they only operate on your training data.
```

To add new columns is the same as doing it in pandas. When adding a new column it will add the new data to the dataset (train or test if data is split), based on the length of the dataset. You do, however, have the ability to specify which dataset to add it to as well:

```python
# based off the length of some\iterable it will add to the train set or test set (if data is split).
df[`new_col_name`] = `some_iterable`

# You can also specify what dataset to add it to as well
df.x_train # pandas dataframe of the training set  
df.x_test # pandas dataframe of the test set
df.x_train[`new_col`] = `some_iterable`  
df.x_test[`new_col`] = `some_iterable`
```

Aethos offers a few options to get started by getting a holistic view of your training data. The first one is one that you are probably familiar with. Aethos’ `describe` function extends pandas to provide a little extra information.

```python
df.describe()
```

<img class="dj t u eh ak" src="https://miro.medium.com/max/2168/1*ih9JD9R2jpRQaek85-XTXQ.png" width="1084" height="532" role="presentation"/>

We can also get more detailed information about each column, all of this is available through the [pandas-summary](https://github.com/mouradmourafiq/pandas-summary) library.

```python
df.describe_column('Age')
```

<img class="dj t u eh ak" src="https://miro.medium.com/max/1734/1*9dLL5KfyztUE6RvhMW3lFQ.png" width="867" height="810" role="presentation"/>

The command also provides a histogram of the data. Each statistical value can be accessed by referencing its name.

```python
df.describe_column('Age')\['iqr'\]
```

The other option is to generate an EDA report available through [pandas-profiling](https://github.com/pandas-profiling/pandas-profiling).

```python
df.data_report()
```

<img class="dj t u eh ak" src="https://miro.medium.com/max/1634/1*zNWX1CZ52YGjbg5UmIawcQ.png" width="817" height="669" role="presentation"/>

We can also generate histograms, pairplots and jointplots all with one line of code (Note: image sizes were scaled to a better fit for this article and are configurable).

```python
df.jointplot('Age', 'Fare', kind='hex', output_file='age_fare_joint.png')  
df.pairplot(diag_kind='hist', output_file='pairplot.png')  
df.histogram('Age', output_file='age_hist.png')
```

<img class="dj t u eh ak" src="https://miro.medium.com/max/1648/1*a4qIxKgEm4oA5FtE8Ng1Gw.png" width="" height="" role="presentation"/>
<p align="middle">
    <img class="dj t u eh ak" src="https://miro.medium.com/max/742/1*WdxHNOegMN9Eb9fZRN1cCg.png" width="" height="" role="presentation"/>
    <img class="dj t u eh ak" src="https://miro.medium.com/max/852/1*oyXk2dPYr8XAHfUHg-vpEA.png" width="" height="" role="presentation"/>
</p>

Jointploit, Pairplot and histogram (L to R)

We can also view information about missing values in both datasets at any time.

```python
df.missing_values
```

<img class="dj t u eh ak" src="https://miro.medium.com/max/800/1*Lv_eL1kYRVtFNpoTc221hw.png" width="400" height="317" role="presentation"/>

Let’s deal with the missing values in the Cabin, Age and Embarked columns.

For the purpose of this article we will replace missing values in the `Embarked` column with the most common value and missing values in the `Age` column with the median value. We will then drop the `Cabin` column.

```python
df.replace_missing_mostcommon('Embarked')
```

A lot just happened in a few lines of code. Let’s start with the imputation.

*   Aethos uses established packages (sci-kit learn, nltk, gensim, etc.) to do all the analysis, in this case, imputation.
*   To avoid data leakage all the analytical techniques are fit to the training set and then applied to the test set.
*   In general when there is a test set, whatever is done to the training set will be applied to the test set.
*   When you run a technique a small description of the technique is written to the report.
*   All of this is consistent for every Aethos analytical technique and can be viewed in the source code [here](https://github.com/Ashton-Sidhu/aethos/tree/develop/aethos) (this function is in cleaning/clean.py).

```python
df.replace_missing_median('Age')  
df.drop('Cabin')
```

Let’s also drop the Ticket, Name, PClass, PassengerId, Parch, and SibSp columns.

```python
df.drop('Ticket', 'Name', 'PClass', 'PassengerId', 'Parch', 'SibSp')
```

This isn’t a post about why you should try avoiding the `.apply` function in pandas but in the interest of API coverage and consistency, you can use `.apply` with Aethos as well. Specify a function and an output column name and you’re good to go.

The two big differences are that the whole DataFrame is passed into the function and the [Swifter](https://github.com/jmcarpenter2/swifter) library is used to parallelize the running of the function, decreasing run time.

```python
def get_person(data):  
    age, sex = data['Age'], data['Sex']  
    return 'child' if age < 16 else sexdf.apply(get_person, 'Person')
```

<img class="dj t u eh ak" src="https://miro.medium.com/max/2164/1*gEwI6IiIy5aSGBK0VOh4mg.png" width="1082" height="424" role="presentation"/>

Just like with any other Aethos method the function `get_person` is applied to both the train and test set. Aethos provides an optimization when using `.apply` but it’s still best to use vectorization when possible.

We can now drop the `Sex` column.

```python
df.drop('Sex')
```

We need to transform `Embarked` and `Person` to numeric variables and for this we’ll just use one-hot encoding for simplicity. We also want to drop the original columns which we specify with the `keep_col` parameter. Since we’re using [scikit-learn’s OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html) class, we can pass keyword arguments to its’ constructor as well.

```python
df.onehot_encode('Embarked', 'Person', keep_col=False, drop=None)
```

<img class="dj t u eh ak" src="https://miro.medium.com/max/2184/1*Jy7G39182cgboDmIXXTH6g.png" width="1092" height="277" role="presentation"/>

Reporting
=========

A couple of things to keep in mind about the current reporting feature is that it records and writes in the order the code is executed and that it currently does not caption any images. We can also write to the report at any time by doing the following:

```python
df.log(`write_something_here`)
```

Conclusion
==========

Aethos is standardized API that allows you to run analytical techniques, train models and more, with a single line of code. What I demonstrated is just the tip of the iceberg of the analytical techniques you can do with Aethos. There are over 110 automated techniques and models in Aethos with more coming soon.

In the subsequent blog posts I will go over training models and viewing results with Aethos. For more detail, I recommend reading the [Usage](https://github.com/Ashton-Sidhu/aethos#usage) section on Github or the [documentation](https://aethos.readthedocs.io/en/latest/?badge=latest).

Coming Soon
===========

This is just an introduction to Aethos but new features are coming soon! For a full roadmap you can view [it here](https://github.com/Ashton-Sidhu/aethos#development-phases).

In the near future you can expect:

*   Experiment management with MLFlow
*   Improved existing visualizations with a better API
*   More automated visualizations
*   More automated techniques (dimensionality reduction, feature engineering)
*   Pre-trained models (GPT-2, BERT, etc.)

Feedback
========

I encourage all feedback about this post or Aethos. You can message me on [twitter](https://twitter.com/ashtonasidhu) or e-mail me at sidhuashton@gmail.com.

Any bug or feature requests, please create an issue on the [Github repo](https://github.com/Ashton-Sidhu/aethos/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc). I welcome all feature requests and any contributions. This project is a great starter if you’re looking to contribute to an open source project — you can always message me if you need assistance getting started.
