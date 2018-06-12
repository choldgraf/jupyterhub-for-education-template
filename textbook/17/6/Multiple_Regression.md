---
layout: textbook
interact_link: notebooks/17/6/Multiple_Regression.ipynb
previous:
  url: textbook/17/5/Accuracy_of_the_Classifier
  title: '17.5 The Accuracy of the Classifier'
next:
  url: textbook/18/Updating_Predictions
  title: '18. Updating Predictions'
sidebar:
  nav: sidebar-textbook
---

Now that we have explored ways to use multiple attributes to predict a categorical variable, let us return to predicting a quantitative variable. Predicting a numerical quantity is called regression, and a commonly used method to use multiple attributes for regression is called *multiple linear regression*.

## Home Prices

The following dataset of house prices and attributes was collected over several years for the city of Ames, Iowa. A [description of the dataset appears online](http://ww2.amstat.org/publications/jse/v19n3/decock.pdf). We will focus only a subset of the columns. We will try to predict the sale price column from the other columns.


{:.input_area}
```python
all_sales = Table.read_table(path_data + 'house.csv')
sales = all_sales.where('Bldg Type', '1Fam').where('Sale Condition', 'Normal').select(
    'SalePrice', '1st Flr SF', '2nd Flr SF', 
    'Total Bsmt SF', 'Garage Area', 
    'Wood Deck SF', 'Open Porch SF', 'Lot Area', 
    'Year Built', 'Yr Sold')
sales.sort('SalePrice')
```




<div markdown="0">
<table border="1" class="dataframe">
    <thead>
        <tr>
            <th>SalePrice</th> <th>1st Flr SF</th> <th>2nd Flr SF</th> <th>Total Bsmt SF</th> <th>Garage Area</th> <th>Wood Deck SF</th> <th>Open Porch SF</th> <th>Lot Area</th> <th>Year Built</th> <th>Yr Sold</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>35000    </td> <td>498       </td> <td>0         </td> <td>498          </td> <td>216        </td> <td>0           </td> <td>0            </td> <td>8088    </td> <td>1922      </td> <td>2006   </td>
        </tr>
    </tbody>
        <tr>
            <td>39300    </td> <td>334       </td> <td>0         </td> <td>0            </td> <td>0          </td> <td>0           </td> <td>0            </td> <td>5000    </td> <td>1946      </td> <td>2007   </td>
        </tr>
    </tbody>
        <tr>
            <td>40000    </td> <td>649       </td> <td>668       </td> <td>649          </td> <td>250        </td> <td>0           </td> <td>54           </td> <td>8500    </td> <td>1920      </td> <td>2008   </td>
        </tr>
    </tbody>
        <tr>
            <td>45000    </td> <td>612       </td> <td>0         </td> <td>0            </td> <td>308        </td> <td>0           </td> <td>0            </td> <td>5925    </td> <td>1940      </td> <td>2009   </td>
        </tr>
    </tbody>
        <tr>
            <td>52000    </td> <td>729       </td> <td>0         </td> <td>270          </td> <td>0          </td> <td>0           </td> <td>0            </td> <td>4130    </td> <td>1935      </td> <td>2008   </td>
        </tr>
    </tbody>
        <tr>
            <td>52500    </td> <td>693       </td> <td>0         </td> <td>693          </td> <td>0          </td> <td>0           </td> <td>20           </td> <td>4118    </td> <td>1941      </td> <td>2006   </td>
        </tr>
    </tbody>
        <tr>
            <td>55000    </td> <td>723       </td> <td>363       </td> <td>723          </td> <td>400        </td> <td>0           </td> <td>24           </td> <td>11340   </td> <td>1920      </td> <td>2008   </td>
        </tr>
    </tbody>
        <tr>
            <td>55000    </td> <td>796       </td> <td>0         </td> <td>796          </td> <td>0          </td> <td>0           </td> <td>0            </td> <td>3636    </td> <td>1922      </td> <td>2008   </td>
        </tr>
    </tbody>
        <tr>
            <td>57625    </td> <td>810       </td> <td>0         </td> <td>0            </td> <td>280        </td> <td>119         </td> <td>24           </td> <td>21780   </td> <td>1910      </td> <td>2009   </td>
        </tr>
    </tbody>
        <tr>
            <td>58500    </td> <td>864       </td> <td>0         </td> <td>864          </td> <td>200        </td> <td>0           </td> <td>0            </td> <td>8212    </td> <td>1914      </td> <td>2010   </td>
        </tr>
    </tbody>
</table>
<p>... (1992 rows omitted)</p
</div>



A histogram of sale prices shows a large amount of variability and a distribution that is clearly not normal. A long tail to the right contains a few houses that had very high prices. The short left tail does not contain any houses that sold for less than $35,000.


{:.input_area}
```python
sales.hist('SalePrice', bins=32, unit='$')
```


![png]({{ site.baseurl }}/images/textbook/17/6/Multiple_Regression_3_0.png)


#### Correlation

No single attribute is sufficient to predict the sale price. For example, the area of first floor, measured in square feet, correlates with sale price but only explains some of its variability.


{:.input_area}
```python
sales.scatter('1st Flr SF', 'SalePrice')
```


![png]({{ site.baseurl }}/images/textbook/17/6/Multiple_Regression_5_0.png)



{:.input_area}
```python
correlation(sales, 'SalePrice', '1st Flr SF')
```




{:.output_data_text}
```
0.64246625410302249
```



In fact, none of the individual attributes have a correlation with sale price that is above 0.7 (except for the sale price itself).


{:.input_area}
```python
for label in sales.labels:
    print('Correlation of', label, 'and SalePrice:\t', correlation(sales, label, 'SalePrice'))
```

{:.output_stream}
```
Correlation of SalePrice and SalePrice:	 1.0
Correlation of 1st Flr SF and SalePrice:	 0.642466254103
Correlation of 2nd Flr SF and SalePrice:	 0.35752189428
Correlation of Total Bsmt SF and SalePrice:	 0.652978626757
Correlation of Garage Area and SalePrice:	 0.638594485252
Correlation of Wood Deck SF and SalePrice:	 0.352698666195
Correlation of Open Porch SF and SalePrice:	 0.336909417026
Correlation of Lot Area and SalePrice:	 0.290823455116
Correlation of Year Built and SalePrice:	 0.565164753714
Correlation of Yr Sold and SalePrice:	 0.0259485790807

```

However, combining attributes can provide higher correlation. In particular, if we sum the first floor and second floor areas, the result has a higher correlation than any single attribute alone.


{:.input_area}
```python
both_floors = sales.column(1) + sales.column(2)
correlation(sales.with_column('Both Floors', both_floors), 'SalePrice', 'Both Floors')
```




{:.output_data_text}
```
0.7821920556134877
```



This high correlation indicates that we should try to use more than one attribute to predict the sale price. In a dataset with multiple observed attributes and a single numerical value to be predicted (the sale price in this case), multiple linear regression can be an effective technique.

## Multiple Linear Regression 

In multiple linear regression, a numerical output is predicted from numerical input attributes by multiplying each attribute value by a different slope, then summing the results. In this example, the slope for the `1st Flr SF` would represent the dollars per square foot of area on the first floor of the house that should be used in our prediction. 

Before we begin prediction, we split our data randomly into a training and test set of equal size.


{:.input_area}
```python
train, test = sales.split(1001)
print(train.num_rows, 'training and', test.num_rows, 'test instances.')
```

{:.output_stream}
```
1001 training and 1001 test instances.

```

The slopes in multiple regression is an array that has one slope value for each attribute in an example. Predicting the sale price involves multiplying each attribute by the slope and summing the result.


{:.input_area}
```python
def predict(slopes, row):
    return sum(slopes * np.array(row))

example_row = test.drop('SalePrice').row(0)
print('Predicting sale price for:', example_row)
example_slopes = np.random.normal(10, 1, len(example_row))
print('Using slopes:', example_slopes)
print('Result:', predict(example_slopes, example_row))
```

{:.output_stream}
```
Predicting sale price for: Row(1st Flr SF=1092, 2nd Flr SF=1020, Total Bsmt SF=952.0, Garage Area=576.0, Wood Deck SF=280, Open Porch SF=0, Lot Area=11075, Year Built=1969, Yr Sold=2008)
Using slopes: [  9.99777721   9.019661    11.13178317   9.40645585  11.07998556
  11.03830075  10.26908341  10.42534332  11.00103437]
Result: 195583.275784

```

The result is an estimated sale price, which can be compared to the actual sale price to assess whether the slopes provide accurate predictions. Since the `example_slopes` above were chosen at random, we should not expect them to provide accurate predictions at all.


{:.input_area}
```python
print('Actual sale price:', test.column('SalePrice').item(0))
print('Predicted sale price using random slopes:', predict(example_slopes, example_row))
```

{:.output_stream}
```
Actual sale price: 206900
Predicted sale price using random slopes: 195583.275784

```

#### Least Squares Regression

The next step in performing multiple regression is to define the least squares objective. We perform the prediction for each row in the training set, and then compute the root mean squared error (RMSE) of the predictions from the actual prices.


{:.input_area}
```python
train_prices = train.column(0)
train_attributes = train.drop(0)

def rmse(slopes, attributes, prices):
    errors = []
    for i in np.arange(len(prices)):
        predicted = predict(slopes, attributes.row(i))
        actual = prices.item(i)
        errors.append((predicted - actual) ** 2)
    return np.mean(errors) ** 0.5

def rmse_train(slopes):
    return rmse(slopes, train_attributes, train_prices)

print('RMSE of all training examples using random slopes:', rmse_train(example_slopes))
```

{:.output_stream}
```
RMSE of all training examples using random slopes: 69653.9880638

```

Finally, we use the `minimize` function to find the slopes with the lowest RMSE. Since the function we want to minimize, `rmse_train`, takes an array instead of a number, we must pass the `array=True` argument to `minimize`. When this argument is used, `minimize` also requires an initial guess of the slopes so that it knows the dimension of the input array. Finally, to speed up optimization, we indicate that `rmse_train` is a smooth function using the `smooth=True` attribute. Computation of the best slopes may take several minutes.


{:.input_area}
```python
best_slopes = minimize(rmse_train, start=example_slopes, smooth=True, array=True)
print('The best slopes for the training set:')
Table(train_attributes.labels).with_row(list(best_slopes)).show()
print('RMSE of all training examples using the best slopes:', rmse_train(best_slopes))
```

{:.output_stream}
```
The best slopes for the training set:

```


<div markdown="0">
<table border="1" class="dataframe">
    <thead>
        <tr>
            <th>1st Flr SF</th> <th>2nd Flr SF</th> <th>Total Bsmt SF</th> <th>Garage Area</th> <th>Wood Deck SF</th> <th>Open Porch SF</th> <th>Lot Area</th> <th>Year Built</th> <th>Yr Sold</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>73.7779   </td> <td>72.3057   </td> <td>51.8885      </td> <td>46.5581    </td> <td>39.3267     </td> <td>11.996       </td> <td>0.451265</td> <td>538.243   </td> <td>-534.634</td>
        </tr>
    </tbody>
</table>
</div>


{:.output_stream}
```
RMSE of all training examples using the best slopes: 31146.4442711

```

#### Interpreting Multiple Regression 

Let's interpret these results. The best slopes give us a method for estimating the price of a house from its attributes. A square foot of area on the first floor is worth about \$75 (the first slope), while one on the second floor is worth about \$70 (the second slope). The final negative value describes the market: prices in later years were lower on average.

The RMSE of around \$30,000 means that our best linear prediction of the sale price based on all of the attributes is off by around \$30,000 on the training set, on average.  We find a similar error when predicting prices on the test set, which indicates that our prediction method will generalize to other samples from the same population.


{:.input_area}
```python
test_prices = test.column(0)
test_attributes = test.drop(0)

def rmse_test(slopes):
    return rmse(slopes, test_attributes, test_prices)

rmse_linear = rmse_test(best_slopes)
print('Test set RMSE for multiple linear regression:', rmse_linear)
```

{:.output_stream}
```
Test set RMSE for multiple linear regression: 31105.4799398

```

If the predictions were perfect, then a scatter plot of the predicted and actual values would be a straight line with slope 1. We see that most dots fall near that line, but there is some error in the predictions.


{:.input_area}
```python
def fit(row):
    return sum(best_slopes * np.array(row))

test.with_column('Fitted', test.drop(0).apply(fit)).scatter('Fitted', 0)
plots.plot([0, 5e5], [0, 5e5]);
```


![png]({{ site.baseurl }}/images/textbook/17/6/Multiple_Regression_24_0.png)


A residual plot for multiple regression typically compares the errors (residuals) to the actual values of the predicted variable. We see in the residual plot below that we have systematically underestimated the value of expensive houses, shown by the many positive residual values on the right side of the graph.


{:.input_area}
```python
test.with_column('Residual', test_prices-test.drop(0).apply(fit)).scatter(0, 'Residual')
plots.plot([0, 7e5], [0, 0]);
```


![png]({{ site.baseurl }}/images/textbook/17/6/Multiple_Regression_26_0.png)


As with simple linear regression, interpreting the result of a predictor is at least as important as making predictions. There are many lessons about interpreting multiple regression that are not included in this textbook. A natural next step after completing this text would be to study linear modeling and regression in further depth.

## Nearest Neighbors for Regression

Another approach to predicting the sale price of a house is to use the price of similar houses. This *nearest neighbor* approach is very similar to our classifier. To speed up computation, we will only use the attributes that had the highest correlation with the sale price in our original analysis.


{:.input_area}
```python
train_nn = train.select(0, 1, 2, 3, 4, 8)
test_nn = test.select(0, 1, 2, 3, 4, 8)
train_nn.show(3)
```


<div markdown="0">
<table border="1" class="dataframe">
    <thead>
        <tr>
            <th>SalePrice</th> <th>1st Flr SF</th> <th>2nd Flr SF</th> <th>Total Bsmt SF</th> <th>Garage Area</th> <th>Year Built</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>240000   </td> <td>1710      </td> <td>0         </td> <td>1710         </td> <td>550        </td> <td>2004      </td>
        </tr>
    </tbody>
        <tr>
            <td>229000   </td> <td>1302      </td> <td>735       </td> <td>672          </td> <td>472        </td> <td>1996      </td>
        </tr>
    </tbody>
        <tr>
            <td>136500   </td> <td>864       </td> <td>0         </td> <td>864          </td> <td>336        </td> <td>1978      </td>
        </tr>
    </tbody>
</table>
<p>... (998 rows omitted)</p
</div>


The computation of closest neighbors is identical to a nearest-neighbor classifier. In this case, we will exclude the `'SalePrice'` rather than the `'Class'` column from the distance computation. The five nearest neighbors of the first test row are shown below.


{:.input_area}
```python
def distance(pt1, pt2):
    """The distance between two points, represented as arrays."""
    return np.sqrt(sum((pt1 - pt2) ** 2))

def row_distance(row1, row2):
    """The distance between two rows of a table."""
    return distance(np.array(row1), np.array(row2))

def distances(training, example, output):
    """Compute the distance from example for each row in training."""
    dists = []
    attributes = training.drop(output)
    for row in attributes.rows:
        dists.append(row_distance(row, example))
    return training.with_column('Distance', dists)

def closest(training, example, k, output):
    """Return a table of the k closest neighbors to example."""
    return distances(training, example, output).sort('Distance').take(np.arange(k))

example_nn_row = test_nn.drop(0).row(0)
closest(train_nn, example_nn_row, 5, 'SalePrice')
```




<div markdown="0">
<table border="1" class="dataframe">
    <thead>
        <tr>
            <th>SalePrice</th> <th>1st Flr SF</th> <th>2nd Flr SF</th> <th>Total Bsmt SF</th> <th>Garage Area</th> <th>Year Built</th> <th>Distance</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>150000   </td> <td>1299      </td> <td>0         </td> <td>967          </td> <td>494        </td> <td>1954      </td> <td>51.9711 </td>
        </tr>
    </tbody>
        <tr>
            <td>144000   </td> <td>1344      </td> <td>0         </td> <td>1024         </td> <td>484        </td> <td>1958      </td> <td>60.8358 </td>
        </tr>
    </tbody>
        <tr>
            <td>183500   </td> <td>1299      </td> <td>0         </td> <td>1001         </td> <td>486        </td> <td>1979      </td> <td>68.6003 </td>
        </tr>
    </tbody>
        <tr>
            <td>140000   </td> <td>1283      </td> <td>0         </td> <td>931          </td> <td>506        </td> <td>1962      </td> <td>76.5049 </td>
        </tr>
    </tbody>
        <tr>
            <td>173000   </td> <td>1287      </td> <td>0         </td> <td>957          </td> <td>541        </td> <td>1977      </td> <td>77.2464 </td>
        </tr>
    </tbody>
</table>
</div>



One simple method for predicting the price is to average the prices of the nearest neighbors.


{:.input_area}
```python
def predict_nn(example):
    """Return the majority class among the k nearest neighbors."""
    return np.average(closest(train_nn, example, 5, 'SalePrice').column('SalePrice'))

predict_nn(example_nn_row)
```




{:.output_data_text}
```
158100.0
```



Finally, we can inspect whether our prediction is close to the true sale price for our one test example. Looks reasonable!


{:.input_area}
```python
print('Actual sale price:', test_nn.column('SalePrice').item(0))
print('Predicted sale price using nearest neighbors:', predict_nn(example_nn_row))
```

{:.output_stream}
```
Actual sale price: 146000
Predicted sale price using nearest neighbors: 158100.0

```

#### Evaluation

To evaluate the performance of this approach for the whole test set, we apply `predict_nn` to each test example, then compute the root mean squared error of the predictions. Computation of the predictions may take several minutes.


{:.input_area}
```python
nn_test_predictions = test_nn.drop('SalePrice').apply(predict_nn)
rmse_nn = np.mean((test_prices - nn_test_predictions) ** 2) ** 0.5

print('Test set RMSE for multiple linear regression: ', rmse_linear)
print('Test set RMSE for nearest neighbor regression:', rmse_nn)
```

{:.output_stream}
```
Test set RMSE for multiple linear regression:  30232.0744208
Test set RMSE for nearest neighbor regression: 31210.6572877

```

For these data, the errors of the two techniques are quite similar! For different data sets, one technique might outperform another. By computing the RMSE of both techniques on the same data, we can compare methods fairly. One note of caution: the difference in performance might not be due to the technique at all; it might be due to the random variation due to sampling the training and test sets in the first place.

Finally, we can draw a residual plot for these predictions. We still underestimate the prices of the most expensive houses, but the bias does not appear to be as systematic. However, fewer residuals are very close to zero, indicating that fewer prices were predicted with very high accuracy. 


{:.input_area}
```python
test.with_column('Residual', test_prices-nn_test_predictions).scatter(0, 'Residual')
plots.plot([0, 7e5], [0, 0]);
```


![png]({{ site.baseurl }}/images/textbook/17/6/Multiple_Regression_39_0.png)
