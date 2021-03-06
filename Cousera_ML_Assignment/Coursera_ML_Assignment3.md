Multi-class Classification and Neural Networks
================

1.Multi-class Classification
============================

### 1.1 Visualizing the data

#### 1.1.1 Data loading

``` r
library(rmatio) # library for loading matlab file
```

    ## Warning: package 'rmatio' was built under R version 3.4.4

``` r
data = read.mat("C:/Users/user/Documents/Basic-ML-with_R/data/ex3data1.mat")
list2env(data, .GlobalEnv)
```

    ## <environment: R_GlobalEnv>

``` r
rm(data)
m = dim(X)[1]
```

#### 1.1.2 Function setting

Special thanks to github.com/faridcher

``` r
# displayData function 

displayData = function(X, example_width = round(sqrt(dim(X)[2]))) {
  # Displaydata display 2D data in a nice grid
  # [h, display_array] = displayData(X, example_width) displays 2D data
  # stored in X in a nice grid. It returns the figure handle h and the
  # displayed array if requested
  
  if (is.vector(X)) {
    X = t(X)
  }
  
  # Compute rows, cols
  m = dim(X)[1]
  n = dim(X)[2]
  
  example_height = (n / example_width) # 20
  
  # Compute number of items to display
  display_rows = floor(sqrt(m)) # 10, floor(x) returns integer below or equal x
  display_cols = ceiling(m / display_rows) # 10, celing(x) returns integer above or equal x
  
  # Between images padding
  pad = 1
  
  # Setup blank display
  display_array = 
    matrix(0, pad + display_rows * (example_height + pad), 
            pad + display_cols * (example_width + pad))
  
  # Copy each example into a patch on the display array
  curr_ex = 1
  for (j in 1:display_rows) {
    for(i in 1:display_cols) {
      if (curr_ex > m)
        break
      # Copy the patch
      
      # Get the max value of the patch
      max_val = max(abs(X[curr_ex, ]))
      display_array[pad + (j - 1) * (example_height + pad) + (1:example_height),
                    pad + (i - 1) * (example_width + pad) + (1:example_width)] =
        matrix(X[curr_ex, ], example_height, example_width) / max_val
      curr_ex = curr_ex + 1
    }
    if (curr_ex > m)
      break
  }
  
  # Display image
  op = par(bg = "gray")
  
  # image draws by row from bottom up, but R indexes matrices by column, top down
  dispArr = t(apply(display_array, 2, rev))
  
  image(z = dispArr, col = gray.colors(100),
        xaxt = 'n', yaxt = 'n')
  
  grid(nx = display_cols, display_rows,
       col = 'black', lwd = 2, lty = 1)
  box()
  par(op)
}
```

#### 1.1.3 Display

``` r
# Randomly select 100 data points to display
rand_indices = sample(m)
sel = X[rand_indices[1:100], ]

# Displaying randomly selected data
displayData(sel)
```

![](Coursera_ML_Assignment3_files/figure-markdown_github-ascii_identifiers/display-1.png)

### 1.2 One-vs-all Classification

In this part of the exercise, we will implement one-vs-all classification by training multiple regularized logistic regression classifiers, one for each of the K classes in our dataset.

In particular, we should return all the classifier parameters into the matrix THETA(K by N+1), where each row of THETA corresponds to the learned logistic regression parameters for one class.

#### 1.2.1 Settings

``` r
input_layer_size = 400
num_labels = 10 # 10 lebels, from 1 to 10, 
                # note that we have mapped "0" to label 10  
lambda = 0.1
```

#### 1.2.2 OnevsAll function

we use 'costFunction.R', 'computeGradient.R' function from last exercise.

``` r
# Function loading
source("C:/Users/user/Documents/Basic-ML-with_R/function/computeGradient.R")
source("C:/Users/user/Documents/Basic-ML-with_R/function/costFunction.R")

oneVsAll = function(X, y, num_labels, lambda) {
  
  X = cbind(rep(1, dim(X)[1]), X) # add bias unit 
  m = length(y)
  n = dim(X)[2] # number of features
  
  # matrix for saving classifiers for each K
  theta = matrix(0, num_labels, n)  
  
  for (i in 1:num_labels) {
    initial_theta = c(rep(0, n))
    
  # transforming y to y_n, where j-th element of y_n indicates
  # whether the j-th element of y belongs to class i 
    y_n = c(rep(0, m))
    y_n[y == i] = 1 
    
    optimRes = optim(par = initial_theta,
                     fn = costFunction(X, y_n, lambda), 
                     gr = computeGradient(X, y_n, lambda),
                     method = "BFGS", control = list(maxit = 80))
    
    theta[i, ] = t(optimRes$par)
  }
  # saving theta in list form
  list(theta = theta)
}
```

#### 1.2.3 Training and predicting

``` r
# Training multiple regularized logistic regression classifiers 
# for each class
training = oneVsAll(X, y, num_labels, lambda)
theta = training$theta # from list vaialbe to gloval env variable
```

For each input, we compute the probability that it belongs to each calss using the trained logistic regression classifiers.

'predictOneVsAll function will pick the class for which the corresponding logistic regression classifier outputs the highest probability and return the class label as the prediction for the input example.

``` r
predictOneVsAll = function(X, y, theta) {
  
  m = length(y)
  X = cbind(rep(1, m), X) # add bias unit
  X_theta = X %*% t(theta) # 5000 by 10
  h = 1 / (1 + exp(-X_theta)) # 5000 by 10, each row of h represents 
                              # probabilities for each class
  pred = c(apply(h, 1, which.max)) 
  # which.max returns index of maximum element
  accuracy = mean(pred == y)
  list(pred = pred, accuracy = accuracy)
}

do_predict = predictOneVsAll(X, y, theta)
pred = do_predict$pred
accuracy = do_predict$accuracy

sprintf('Training set accuracy: %.3f', accuracy)
```

    ## [1] "Training set accuracy: 0.945"

2.Neural Networks
=================

Logistic regression cannot form more complex hypotheses as it is only a linear classifier.(we could add more features to logistic regression, but that can be very expensive to train.)

The neural network will be able to represent complex models that form non-linear hypotheses.

In this part, we will use trained parameters from a neural network.

### 2.1 Model representation

Our neural network has 3 layers - an input layer, a hidden layer(with 25 units) and an output layer(with 10 classes).

``` r
# parameters loading
theta = read.mat("C:/Users/user/Documents/Basic-ML-with_R/data/ex3weights.mat")
list2env(theta, .GlobalEnv)
```

    ## <environment: R_GlobalEnv>

### 2.2 Feedforward propagation and prediction

#### 2.2.1 Prediction function

``` r
predict_w_nn = function(theta1, theta2, X) {
  
  if (is.vector(X)) {
    X = t(X) # if input X was a vector, we should transform 
             # X to 1 by n matrix
  }
  
  m = dim(X)[1]
  X = cbind(c(rep(1, m)), X) # add bias unit to X
  z_2 = theta1 %*% t(X) # 25 by 5000
  a_2 = 1 / (1 + exp(-z_2)) # 25 by 5000
  a_2 = rbind(c(rep(1, dim(a_2)[2])), a_2) # add bias unit to a_2, 26 by 5000
  z_3 = theta2 %*% a_2 # 10 by 5000
  a_3 = 1 / (1 + exp(-z_3)) # 10 by 5000
  
  pred =  c(apply(a_3, 2, which.max)) # 5000 by 1
  list(pred = pred)
}
```

#### 2.2.2 Prediction

``` r
# Using prediction function
prediction = predict_w_nn(Theta1, Theta2, X)

# saving result
pred = prediction$pred

# Print Accuracy 
sprintf('Training set accuracy: %.3f', mean(pred == y))
```

    ## [1] "Training set accuracy: 0.975"

To give you an idea of the network's output, you can also run through the examples one at the time to see what it is predicting.

``` r
# Choose random number in 1:5000
i = sample(1:5000, 1)

# Display
displayData(X[i, ])
```

![](Coursera_ML_Assignment3_files/figure-markdown_github-ascii_identifiers/sample-1.png)

``` r
# Predicting for i-th element in sample and compare it to i-th element of y   
pred = predict_w_nn(Theta1, Theta2, X[i, ])$pred
sprintf('Neural Network prediction:%d (when y = %d)', pred, y[i])
```

    ## [1] "Neural Network prediction:6 (when y = 6)"
