# Weather-Analysis
We've taken data for the weather changes in Perth, Australia every day for the last five years. Our goal is to create a working predictive model that predicts whether or not there is rain tomorrow in the city. First we change the data to a form that the neural network can work on. We remove NA values, strings and unecessary columnns of data. Then we begin splitting the data into training and testing while establishing the input and output of each. We fit both into a scaling funtion that changes the data so that the mean will be 0 and the standard deviation will be 1 so that the dataset given to the network is consistent. Then, using MLPClassifier, we train the data in question i.e; X_train and y_train. After this, we name a new variable "y_pred", that contains an array of predicted values that we got from using the .predict function on the test input data (X_test). An accuracy metric can be calculated using the accuracy_score() function comparing the y_pred values and the output test values (y_test).

Posting the actual code here. Code is in jupyter in the other file as well.

# Importing the libraries

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix
from sklearn.preprocessing import StandardScaler
from sklearn.neural_network import MLPClassifier
from sklearn.model_selection import GridSearchCV

# Reading the data from the csv file

df = pd.read_csv("weatherperth.csv")
exclude = ["Date","Location", "RISK_MM"]
for attributes in exclude:
    del df[attributes]

df = df.dropna()  # Dropping the Not applicable values from the data set

# Mapping of non-numeric (Neural network isn't gonna recognize non-numeric values) to a numeric representation.

Mapping_array = ["RainToday", "RainTomorrow"]
for i in Mapping_array:
    if i in df.columns:
        df[i] = df[i].map({
        "Yes" : 1,
        "No" : 0,
    })
    else: print("Column {i} doesn't exist")

# Mapping of cyclical data. 
First glance, I thought I'd just take the 360/16 = 22.5 degree and use that to represent all the values in the 
compass from North to North North West (left side of north). But the issue here is that when I do that, it won't be right. North and North-North West, 
which are supposed to be beside each other have significantly different values when represented like that. This isn't good for the model, we need it
as accurate as possible and make sense of the data. Hence we take the sin and cos values, this way N and NNE values are vlose to each other
i.e; 0 and 360 degree values are represented close to each other as it should be since we're representing a cyclical dataset (directions).

directions = ['N','NNE','NE','ENE','E','ESE','SE','SSE','S','SSW','SW','WSW','W','WNW','NW','NNW']
angles = np.arange(0, 2*np.pi,2*np.pi/16)
dir_ang = dict(zip(directions, angles))
print(dir_ang)
mappable_attributes = ['WindGustDir','WindDir9am', 'WindDir3pm']
for i in mappable_attributes:
    df[i] = df[i].map(dir_ang)
    df[i+ '_cos'] = np.cos(df[i])
    df[i + '_sin'] = np.sin(df[i])
df = df.drop(columns = mappable_attributes)
y = df['RainTomorrow']
X = df.drop(columns = 'RainTomorrow')
# We split the data into testing and training sets.
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size = 0.33,
    random_state = 0
    )

# Apply scaler transformation.
What we do here is getting the mean and standard deviation of X_train data and use it to transform X_train and X_test.
Why not X_test? To avoid data leakage. Model ain't supposed to know the X_test statistical values during testing. If we do fit X_test, then we'll
have an overly optimistic model. We need the data to predict on never seen before data so that it can be an ideal test.

scaler = StandardScaler()
scaler.fit(X_train)
X_train = scaler.transform(X_train)
X_test = scaler.transform(X_test)

# Training syntax

nn = MLPClassifier(
    hidden_layer_sizes = (50,50),
    random_state = 0,
    max_iter = 500
)
nn.fit(X_train,y_train)
y_pred = nn.predict(X_test)
score = accuracy_score(y_pred,y_test)
print("The accuracy of the model is:",round((score*100),2), "%")

# We can use grid layer search to fine tune our model accuracy. 
We can change the number of hidden layers our model has or the number of nodes each hidden layer has
The grid layer search will find an ideal or the best configuration.

p = {
 'hidden_layer_sizes' : (
     (10,5),(10,), (20, 6), (100, 10), (5,)
 )
}
nn = MLPClassifier (
    max_iter = 5000,
    random_state = 0
)
gs = GridSearchCV(nn, p, cv = 3)
gs.fit(X_train,y_train)
print(gs.cv_results_['params'])
print(gs.cv_results_['mean_test_score'])
best_nn = gs.best_estimator_
y_pred = best_nn.predict(X_test)
best_acc = accuracy_score(y_pred,y_test)
print("The new improved accuracy of the model using GridSearch is:", round((best_acc*100),2), "%")
