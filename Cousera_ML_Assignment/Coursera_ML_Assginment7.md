K-means Clustering and PCA
================

In this exercise, we will implement the K-means clustering algorithm and apply it to compress an image.

In the second part, we will use principal component analysis to find a low-dimensional representation of face images.

1.K-means Clustering
====================

### 1.1 Implementing K-means

#### 1.1.1 Data loading

``` r
# Suppressing warning message
options(warn = -1)

# library for loading matlab file
library(rmatio) 
data = read.mat("C:/Users/user/Documents/Basic-ML-with_R/data/ex7data2.mat")
list2env(data, .GlobalEnv)
```

    ## <environment: R_GlobalEnv>

``` r
rm(data)
```

#### 1.1.2 Finding closest centroids

Using 'findClosestCentroids' function, we assigns every training examples x(i) to its closest centroid, given the current positions.

``` r
findClosestCentroids = function(X, centroids) {
  
  m = dim(X)[1]
  c = rep(0, m)
  k = dim(centroids)[1]
  
  for (i in 1:m){
    # Calculate distances between sample x and each centroid
    X_i_matrix = rep(1, k) %*% t(X[i, ])
    diff = X_i_matrix - centroids # K by n, where n is the dimension of X
    diff_2 = diff^2
    distance = apply(diff_2, 1, sum) 
    # Assign every training example to closest centroid's index
    c[i] = which.min(distance)
  }
  # return c
  c
}
```

#### 1.1.3 Computing centroid means

Using 'computeCentroids' function, we recomputes, for each centroid, the mean of the points that were assigned to it.

``` r
computeCentroids = function(X, idx, K){
  
  n = dim(X)[2]
  centroids = matrix(0, nrow = K, ncol = n)
    
  for (i in 1:K){
    centroids[i, ] = apply(X[which(idx == i), ], 2, mean)
  }
  centroids
}
```

#### 1.1.4 K-means on example dataset

We use 'runKmeans' function to implement K-means algorithm with iterations.

``` r
ini_cen = matrix(c(3, 3, 6, 2, 8, 5), nrow = 3, byrow = TRUE)
max_iter = 10
```

Make 'runKmeans' function first

``` r
runKmeans = function(X, init_centroids, max_iter){
  
  # Settings
  m = dim(X)[1]
  n = dim(X)[2]
  K = dim(init_centroids)[1]
  
  previous_centroids = array(0, dim = c(dim(init_centroids), max_iter +1))
  previous_centroids[, , 1] = init_centroids
  
  # Run K-means algorithm with iterations 
  for (i in 1:max_iter){
    idx = findClosestCentroids(X, previous_centroids[, , i])
    previous_centroids[, , i + 1] = computeCentroids(X, idx, K)
  }
  list(centroids = previous_centroids[, , max_iter + 1], idx = idx)
}
```

Run K-means algorithm

``` r
K_means_algorithm = runKmeans(X, ini_cen, 10)

# Save results
centroids = K_means_algorithm$centroids
idx = K_means_algorithm$idx
```

Visualize the results

``` r
# Convert matrix to data.frame first
data_plot = data.frame(X, idx)
centroids_plot = as.data.frame(centroids)

# Create plot
library(ggplot2)
p = ggplot(data_plot, aes(X1, X2), color = idx) + geom_point(aes(color = idx))+
  geom_point(data = centroids_plot, aes(V1, V2), color = 'red', pch = 4,
             size = 3, lwd = 2) +
  ylab("X2") + xlab("X1") +
  scale_x_continuous(breaks = seq(-1, 9, 1)) +
  scale_y_continuous(breaks = seq(0, 6, 1)) +
  theme_bw() + ggtitle("The result of K-means algorithm") +
  theme(legend.title = element_blank(), 
        panel.grid.major.x =element_blank(),
        panel.grid.minor.y = element_blank(),  
        panel.grid.minor.x = element_blank(), 
        panel.grid.major.y = element_blank()) +
  theme(legend.position = 'none')
p
```

![](Coursera_ML_Assginment7_files/figure-markdown_github/visualization-1.png)

2.Principal Component Analysis
==============================

We will first experiment with an example 2D dataset to get intuition on how PCA works and then use it on a bigger dataset of 5000 face image dataset.

### 2.1 Example dataset

``` r
data = read.mat("C:/Users/user/Documents/Basic-ML-with_R/data/ex7data1.mat")
list2env(data, .GlobalEnv)
```

    ## <environment: R_GlobalEnv>

``` r
rm(data)
```

#### 2.1.1 Visualization

``` r
example_dataset = data.frame(V1 = X[, 1], V2 = X[, 2])
p1 = ggplot(example_dataset, aes(V1, V2)) + geom_point(color = 'blue') +
  scale_x_continuous(breaks = seq(0, 7, 1)) + 
  scale_y_continuous(breaks = seq(2, 8, 1)) + theme_bw() +
  ggtitle("Example Dataset 1") +
  theme(legend.title = element_blank(), panel.grid.major.x = element_blank() ,
        panel.grid.minor.y = element_blank(),
        panel.grid.major.y = element_blank(),
        panel.grid.minor.x = element_blank())
p1
```

![](Coursera_ML_Assginment7_files/figure-markdown_github/visu-1.png)

### 2.2 Implementing PCA

PCA consists of two computational steps: First, we compute the covariance matrix of the data. Second, we use 'svd' function to compute the eigenvectors. These will correspond to the principal components of variation in the data.

Before using PCA, it is important to first normalize the data. we use 'featureNormalize' function that we made in EX1.

#### 2.2.1 Normalize data

``` r
# Function loading
source("C:/Users/user/Documents/Basic-ML-with_R/function/featureNormalize.R")

normalization = featureNormalize(X)
X_norm = normalization$X_norm
```

#### 2.2.2 Make 'PCA' function and implement

``` r
PCA = function(X) {
  
  m = dim(X)[1]
  n = dim(X)[2]
  
  # Make variance-covariance matrix for X
  v_c_matrix = (t(X) %*% X) / m
  
  # Eigenvalue decomposition v_c_matrix
  eigendecomp = svd(v_c_matrix) # svd function returns the eigenvectors u,
                                # the eigenvalues in d as a list.
  eigendecomp
}

# Implements PCA function
eigendecomp = PCA(X_norm)

# Save the results
principal_components = eigendecomp$u
variation = eigendecomp$d

# Show top principal component
sprintf('Top principal component: %.3f, %.3f', principal_components[1, 1],
        principal_components[1, 2])
```

    ## [1] "Top principal component: -0.707, -0.707"

### 2.3 Dimensionality reduction with PCA

We can use PCA results to reduce the feature dimension of our dataset by projecting eah example onto a lower dimensional space. In this part, we will project the example dataset into a 1-dimensional space.

#### 2.3.1 Make 'projectData' function

``` r
projectData = function(X, U, K){
  # U indicates principal components and K means desired number of dimensions
  # to reduce.
  
  n = dim(X)[2] # number of features(dimensions)
  m = dim(X)[1] # number of example
  
  U_reduce = U[, 1 : K] # n by K
  Z = X %*% U_reduce # Z contains results of projection, m by K
  Z
}
```

#### 2.3.2 Implements projection

``` r
K = 1
projection = projectData(X_norm, principal_components, K)
```

### 2.4 Reconstructing an approximation of the data

We can approximately recover the data by projecting them back onto the original high dimensional space.

#### 2.4.1 Make 'recoverData' function

``` r
recoverData = function(Z, U, K){
 X_rec = Z %*% t(U[, 1 : K])
 X_rec
}
```

#### 2.4.2 Reconstruct data

``` r
recov = recoverData(projection, principal_components, K)
```

### 2.5 Visualizing the projections

We can see how the projection affects the data by visualization. Note that we should use normalized dataset to see the effect of projection.

``` r
# Make data.frame for projected dataset
projected_dataset = data.frame(V1 = recov[, 1], V2 = recov[, 2])

# Make data.frame for normalized sample dataset
n_example_dataset = data.frame(V1 = X_norm[, 1], V2 = X_norm[, 2])

p2 = ggplot(n_example_dataset, aes(V1, V2)) + geom_point(color = 'blue') +
  scale_x_continuous(breaks = seq(-4, 3, 1)) + 
  scale_y_continuous(breaks = seq(-4, 3, 1)) + theme_bw() +
  ggtitle("The normalized and projected data after PCA") +
  theme(legend.title = element_blank(), panel.grid.major.x = element_blank() ,
        panel.grid.minor.y = element_blank(),
        panel.grid.major.y = element_blank(),
        panel.grid.minor.x = element_blank())

p2 = p2 + geom_point(data = projected_dataset, aes(V1, V2), color = "red")
p2
```

![](Coursera_ML_Assginment7_files/figure-markdown_github/visualizeeffect-1.png) The projection effectively only retains the information in the direction given by top principal component.