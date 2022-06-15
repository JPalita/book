## Decision Making Under Model Uncertainty

We are closing this chapter by presenting the last topic, decision making under model uncertainty. We have seen that under the Bayesian framework, we can use different prior distributions for coefficients, different model priors for models, and we can even use stochastic exploration methods for complex model selections. After selecting these coefficient priors and model priors, we can obtain the marginal posterior inclusion probability for each variable in the full model, which may provide some information about whether or not to include a particular variable in the model for further model analysis and predictions. With all the information presented in the results, which model would be the most appropriate model? 

In this section, we will talk about different methods for selecting models and decision making for posterior distributions and predictions. We will illustrate this process using the US crime data `UScrime` as an example and process it using the `BAS` package. 

We first prepare the data as in the last section and run `bas.lm` on the full model


```r
data(UScrime, package="MASS")

# take the natural log transform on the variables except the 2nd column `So`
UScrime[, -2] = log(UScrime[, -2])

# run Bayesian linear regression
library(BAS)
crime.ZS =  bas.lm(y ~ ., data = UScrime,
                   prior = "ZS-null", modelprior = uniform()) 
```


### Model Choice

For Bayesian model choice, we start with the full model, which includes all the predictors. The uncertainty of selecting variables, or model uncertainty that we have been discussing, arises when we believe that some of the explanatory variables may be unrelated to the response variable. This corresponds to setting a regression coefficient $\beta_j$ to be exactly zero. We specify prior distributions that reflect our uncertainty about the importance of variables. We then update the model based on the data we obtained, resulting in posterior distributions over all models and the coefficients and variances within each model.

Now the question has become, how to select a single model from the posterior distribution and use it for furture inference? What are the objectives from inference?

**BMA Model**

We do have a single model, the one that is obtained by averaging all models using their posterior probabilities, the Bayesian model averaging model, or BMA. This is referred to as a hierarchical model and it is composed of many simpler models as building blocks. This represents the full posterior uncertainty after seeing the data. 

We can obtain the posterior predictive mean by using the weighted average of all of the predictions from each sub model

$$\hat{\mu} = E[\hat{Y}~|~\text{data}] = \sum_{M_m \in \text{ model space}}\hat{Y}\times p(M_m~|~\text{data}).$$
This prediction is the best under the squared error loss $L_2$. From `BAS`, we can obtain predictions and fitted values using the usual `predict` and `fitted` functions. To specify which model we use for these results, we need to include argument `estimator`.


```r
crime.BMA = predict(crime.ZS, estimator = "BMA")
mu_hat = fitted(crime.ZS, estimator = "BMA")
```

`crime.BMA`, the object obtained by the `predict` function, has additional slots storing results from the BMA model.


```r
names(crime.BMA)
```

```
##  [1] "fit"         "Ybma"        "Ypred"       "postprobs"   "se.fit"     
##  [6] "se.pred"     "se.bma.fit"  "se.bma.pred" "df"          "best"       
## [11] "bestmodel"   "best.vars"   "estimator"
```

Plotting the two sets of fitted values, one obtained from the `fitted` function, another obtained from the `fit` attribute of the `predict` object `crime.BMA`, we see that they are in perfect agreement.


```r
# Load library and prepare data frame
library(ggplot2)
output = data.frame(mu_hat = mu_hat, fitted = crime.BMA$fit)

# Plot result from `fitted` function and result from `fit` attribute
ggplot(data = output, aes(x = mu_hat, y = fitted)) + 
  geom_point(pch = 16, color = "blue", size = 3) + 
  geom_abline(intercept = 0, slope = 1) + 
  xlab(expression(hat(mu[i]))) + ylab(expression(hat(Y[i])))
```



\begin{center}\includegraphics[width=0.7\linewidth]{08-MCMC-04-decisions_files/figure-latex/BMA-fit-1} \end{center}


**Highest Probability Model**

If our objective is to learn what is the most likely model to have generated the data using a 0-1 loss $L_0$, then the highest probability model (HPM) is optimal. 


```r
crime.HPM = predict(crime.ZS, estimator = "HPM")
```


The variables selected from this model can be obtained using the `bestmodel` attribute from the `crime.HPM` object. We extract the variable names from the `crime.HPM`


```r
crime.HPM$best.vars
```

```
## [1] "Intercept" "M"         "Ed"        "Po1"       "NW"        "U2"       
## [7] "Ineq"      "Prob"      "Time"
```

We see that, except the intercept, which is always in any models, the highest probability model also includes `M`, percentage of males aged 14-24; `Ed`, mean years of schooling; `Po1`, police expenditures in 1960; `NW`, number of non-whites per 1000 people; `U2`, unemployment rate of urban males aged 35-39; `Ineq`, income inequlity; `Prob`, probability of imprisonment, and `Time`, average time in state prison.

To obtain the coefficients and their posterior means and posterior standard deviations, we tell give an optional argument to `coef` to indicate that we want to extract coefficients under the HPM. 


```r
# Select coefficients of HPM

# Posterior means of coefficients
coef.crime.ZS = coef(crime.ZS, estimator="HPM")
coef.crime.ZS
```

```
## 
##  Marginal Posterior Summaries of Coefficients: 
## 
##  Using  HPM 
## 
##  Based on the top  1 models 
##            post mean  post SD   post p(B != 0)
## Intercept   6.72494    0.02623   1.00000      
## M           1.42422    0.42278   0.85357      
## So          0.00000    0.00000   0.27371      
## Ed          2.14031    0.43094   0.97466      
## Po1         0.82141    0.15927   0.66516      
## Po2         0.00000    0.00000   0.44901      
## LF          0.00000    0.00000   0.20224      
## M.F         0.00000    0.00000   0.20497      
## Pop         0.00000    0.00000   0.36961      
## NW          0.10491    0.03821   0.69441      
## U1          0.00000    0.00000   0.25258      
## U2          0.27823    0.12492   0.61494      
## GDP         0.00000    0.00000   0.36012      
## Ineq        1.19269    0.27734   0.99654      
## Prob       -0.29910    0.08724   0.89918      
## Time       -0.27616    0.14574   0.37180
```

We can also obtain the posterior probability of this model using

```r
postprob.HPM = crime.ZS$postprobs[crime.HPM$best]
postprob.HPM
```

```
## [1] 0.01824728
```

we see that this highest probability model has posterior probability of only 0.018. There are many models that have comparable posterior probabilities. So even this model has the highest posterior probability, we are still pretty unsure about whether it is the best model.

**Median Probability Model**

Another model that is frequently reported, is the median probability model (MPM). This model includes all predictors whose marginal posterior inclusion probabilities are greater than 0.5. If the variables are all uncorrelated, this will be the same as the highest posterior probability model. For a sequence of nested models such as polynomial regression with increasing powers, the median probability model is the best single model for prediction.  

However, since in the US crime example, `Po1` and `Po2` are highly correlated, we see that the variables included in MPM are slightly different than the variables included in HPM.


```r
crime.MPM = predict(crime.ZS, estimator = "MPM")
crime.MPM$best.vars
```

```
## [1] "Intercept" "M"         "Ed"        "Po1"       "NW"        "U2"       
## [7] "Ineq"      "Prob"
```

As we see, this model only includes 7 variables, `M`, `Ed`, `Po1`, `NW`, `U2`, `Ineq`, and `Prob`. It does not include `Time` variable as in HPM. 

When there are correlated predictors in non-nested models, MPM in general does well. However, if the correlations among variables increase, MPM may miss important variables as the correlations tend to dilute the posterior inclusing probabilities of related variables.  

To obtain the coefficients in the median probability model, we specify that the estimator is now "MPM":


```r
# Obtain coefficients of the  Median Probabilty Model
coef(crime.ZS, estimator = "MPM")
```

```
## 
##  Marginal Posterior Summaries of Coefficients: 
## 
##  Using  MPM 
## 
##  Based on the top  1 models 
##            post mean  post SD   post p(B != 0)
## Intercept   6.72494    0.02713   1.00000      
## M           1.46180    0.43727   1.00000      
## So          0.00000    0.00000   0.00000      
## Ed          2.30642    0.43727   1.00000      
## Po1         0.87886    0.16204   1.00000      
## Po2         0.00000    0.00000   0.00000      
## LF          0.00000    0.00000   0.00000      
## M.F         0.00000    0.00000   0.00000      
## Pop         0.00000    0.00000   0.00000      
## NW          0.08162    0.03743   1.00000      
## U1          0.00000    0.00000   0.00000      
## U2          0.31053    0.12816   1.00000      
## GDP         0.00000    0.00000   0.00000      
## Ineq        1.18815    0.28710   1.00000      
## Prob       -0.18401    0.06466   1.00000      
## Time        0.00000    0.00000   0.00000
```



**Best Predictive Model**

If our objective is prediction from a single model, the best choice is to find the model whose predictions are closest to those given by BMA. "Closest" could be based on squared error loss for predictions, or be based on any other loss functions. Unfortunately, there is no nice expression for this model. However, we can still calculate the loss for each of our sampled models to try to identify this best predictive model, or BPM.

Using the squared error loss, we find that the best predictive model is the one whose predictions are closest to BMA. 


```r
crime.BPM = predict(crime.ZS, estimator = "BPM")
crime.BPM$best.vars
```

```
##  [1] "Intercept" "M"         "So"        "Ed"        "Po1"       "Po2"      
##  [7] "M.F"       "NW"        "U2"        "Ineq"      "Prob"
```

The best predictive model includes not only the 7 variables that MPM includes, but also `M.F`, number of males per 1000 females, and `Po2`, the police expenditures in 1959. 

Using the `se.fit = TRUE` option with `predict` we can calculate standard deviations for the predictions or for the mean. Then we can use this as input for the `confint` function for the prediction object. Here we only show the results of the first 20 data points.




```r
crime.BPM = predict(crime.ZS, estimator = "BPM", se.fit = TRUE)
crime.BPM.conf.fit = confint(crime.BPM, parm = "mean")
crime.BPM.conf.pred = confint(crime.BPM, parm = "pred")
cbind(crime.BPM$fit, crime.BPM.conf.fit, crime.BPM.conf.pred)
##                    2.5%    97.5%     mean     2.5%    97.5%     pred
##  [1,] 6.668988 6.513238 6.824738 6.668988 6.258715 7.079261 6.668988
##  [2,] 7.290854 7.151787 7.429921 7.290854 6.886619 7.695089 7.290854
##  [3,] 6.202166 6.039978 6.364354 6.202166 5.789406 6.614926 6.202166
##  [4,] 7.661307 7.490608 7.832006 7.661307 7.245129 8.077484 7.661307
##  [5,] 7.015570 6.847647 7.183493 7.015570 6.600523 7.430617 7.015570
##  [6,] 6.469547 6.279276 6.659818 6.469547 6.044966 6.894128 6.469547
##  [7,] 6.776133 6.555130 6.997135 6.776133 6.336920 7.215346 6.776133
##  [8,] 7.299560 7.117166 7.481955 7.299560 6.878450 7.720670 7.299560
##  [9,] 6.614927 6.482384 6.747470 6.614927 6.212890 7.016964 6.614927
## [10,] 6.596912 6.468988 6.724836 6.596912 6.196374 6.997449 6.596912
## [11,] 7.032834 6.877582 7.188087 7.032834 6.622750 7.442918 7.032834
## [12,] 6.581822 6.462326 6.701317 6.581822 6.183896 6.979748 6.581822
## [13,] 6.467921 6.281998 6.653843 6.467921 6.045271 6.890571 6.467921
## [14,] 6.566239 6.403813 6.728664 6.566239 6.153385 6.979092 6.566239
## [15,] 6.550129 6.388987 6.711270 6.550129 6.137779 6.962479 6.550129
## [16,] 6.888592 6.746097 7.031087 6.888592 6.483166 7.294019 6.888592
## [17,] 6.252735 6.063944 6.441526 6.252735 5.828815 6.676654 6.252735
## [18,] 6.795764 6.564634 7.026895 6.795764 6.351369 7.240160 6.795764
## [19,] 6.945687 6.766289 7.125086 6.945687 6.525866 7.365508 6.945687
## [20,] 7.000331 6.840374 7.160289 7.000331 6.588442 7.412220 7.000331
## [...]

```

The option `estimator = "BPM` is not yet available in `coef()`, so we will need to work a little harder to get the coefficients by refitting the BPM.
First we need to extract  a vector of zeros and ones representing which variables are included in the BPM model.

```r
# Extract a binary vector of zeros and ones for the variables included 
# in the BPM
BPM = as.vector(which.matrix(crime.ZS$which[crime.BPM$best],
                             crime.ZS$n.vars))
BPM
```

```
##  [1] 1 1 1 1 1 1 0 1 0 1 0 1 0 1 1 0
```

Next, we will refit the model with `bas.lm` using the optional argument `bestmodel = BPM`.  In general, this is the starting model for the stochastic search, which is helpful for starting the MCMC.
 We will also specify that want to have 1 model by setting `n.models = 1`. In this way, `bas.lm` starts with the BPM and fits only that model. 


```r
# Re-run regression and specify `bestmodel` and `n.models`
crime.ZS.BPM = bas.lm(y ~ ., data = UScrime,
                      prior = "ZS-null",
                      modelprior = uniform(),
                      bestmodel = BPM, n.models = 1)
```
Now since we have only one model in our new object representing the BPM, we can use the `coef` function to obtain the summaries.


```r
# Obtain coefficients of MPM
coef(crime.ZS.BPM)
```

```
## 
##  Marginal Posterior Summaries of Coefficients: 
## 
##  Using  BMA 
## 
##  Based on the top  1 models 
##            post mean  post SD   post p(B != 0)
## Intercept   6.72494    0.02795   1.00000      
## M           1.28189    0.49219   1.00000      
## So          0.09028    0.11935   1.00000      
## Ed          2.24197    0.54029   1.00000      
## Po1         0.70543    0.75949   1.00000      
## Po2         0.16669    0.76781   1.00000      
## LF          0.00000    0.00000   0.00000      
## M.F         0.55521    1.22456   1.00000      
## Pop         0.00000    0.00000   0.00000      
## NW          0.06649    0.04244   1.00000      
## U1          0.00000    0.00000   0.00000      
## U2          0.28567    0.13836   1.00000      
## GDP         0.00000    0.00000   0.00000      
## Ineq        1.15756    0.30841   1.00000      
## Prob       -0.21012    0.07452   1.00000      
## Time        0.00000    0.00000   0.00000
```

Note the posterior probabilities that coefficients are zero is either zero or one since we have selected a model.  

**Comparison of Models**

After discussing all 4 different models, let us compare their prediction results. 


```r
# Set plot settings
par(cex = 1.8, cex.axis = 1.8, cex.lab = 2, mfrow = c(2,2), mar = c(5, 5, 3, 3),
    col.lab = "darkgrey", col.axis = "darkgrey", col = "darkgrey")

# Load library and plot paired-correlations
library(GGally)
ggpairs(data.frame(HPM = as.vector(crime.HPM$fit),  
                   MPM = as.vector(crime.MPM$fit),  
                   BPM = as.vector(crime.BPM$fit),  
                   BMA = as.vector(crime.BMA$fit))) 
```



\begin{center}\includegraphics[width=0.7\linewidth]{08-MCMC-04-decisions_files/figure-latex/paired-cor-1} \end{center}

From the above paired correlation plots, we see that the correlations among them are extremely high. As expected, the single best predictive model (BPM) has the highest correlation with MPM, with a correlation of 0.998. However, the highest posterior model (HPM) and the Bayesian model averaging model (BMA) are nearly equally as good.

 
### Prediction with New Data

Using the `newdata` option in the `predict` function, we can obtain prediction from a new data set. Here we pretend that `UScrime` is an another new data set, and we use BMA to obtain the prediction of new observations. Here we only show the results of the first 20 data points.


```r
BMA.new = predict(crime.ZS, newdata = UScrime, estimator = "BMA",
                  se.fit = TRUE, nsim = 10000)
crime.conf.fit.new = confint(BMA.new, parm = "mean")
crime.conf.pred.new = confint(BMA.new, parm = "pred")

# Show the combined results compared to the fitted values in BPM
cbind(crime.BPM$fit, crime.conf.fit.new, crime.conf.pred.new)
##                    2.5%    97.5%     mean     2.5%    97.5%     pred
##  [1,] 6.668988 6.516973 6.815005 6.661770 6.242515 7.067698 6.661770
##  [2,] 7.290854 7.130743 7.452047 7.298827 6.869772 7.714767 7.298827
##  [3,] 6.202166 5.958549 6.400221 6.179308 5.719030 6.624379 6.179308
##  [4,] 7.661307 7.364186 7.821790 7.610585 7.150669 8.043400 7.610585
##  [5,] 7.015570 6.848175 7.257065 7.054238 6.611699 7.503637 7.054238
##  [6,] 6.469547 6.292949 6.751244 6.514064 6.044151 6.956464 6.514064
##  [7,] 6.776133 6.519746 7.078274 6.784846 6.284624 7.270964 6.784846
##  [8,] 7.299560 7.042870 7.488791 7.266344 6.832924 7.705520 7.266344
##  [9,] 6.614927 6.481290 6.779405 6.629448 6.213962 7.031541 6.629448
## [10,] 6.596912 6.457099 6.739807 6.601246 6.189525 7.015332 6.601246
## [11,] 7.032834 6.871894 7.241696 7.055003 6.634697 7.500822 7.055003
## [12,] 6.581822 6.424074 6.720593 6.570625 6.164031 6.989779 6.570625
## [13,] 6.467921 6.208733 6.711778 6.472327 6.011658 6.936831 6.472327
## [14,] 6.566239 6.394331 6.767615 6.582374 6.163065 7.021503 6.582374
## [15,] 6.550129 6.363211 6.758263 6.556880 6.113900 7.005157 6.556880
## [16,] 6.888592 6.744583 7.064158 6.905017 6.482980 7.319464 6.905017
## [17,] 6.252735 5.983572 6.468477 6.229073 5.787267 6.697356 6.229073
## [18,] 6.795764 6.538742 7.093427 6.809572 6.342467 7.290903 6.809572
## [19,] 6.945687 6.746839 7.126626 6.943294 6.509860 7.362240 6.943294
## [20,] 7.000331 6.775210 7.149052 6.961980 6.540823 7.388084 6.961980
## [...]

```

## Summary

In this chapter, we have introduced one of the common stochastic exploration methods, Markov Chain Monte Carlo, to explore the model space to obtain approximation of posterior probability of each model when the model space is too large for theoretical enumeration. We see that model selection can be sensitive to the prior distributions of coefficients, and introduced Zellner's $g$-prior so that we have to elicit only one hyper-parameter to specify the prior.  Still model selection can be sensitive to the choice of $g$ where values that are too large may unintentially lead to the null model receiving high probability in Bartlett's paradox. To resolve this and other paradoxes related to the choice of $g$, we recommend default choices that have improved robustness to prior misspecification such as the unit information $g$-prior, the Zellner-Siow Cauchy prior, and the hyper-$g/n$ prior. 

We then demonstrated a multiple linear regression process using the `BAS` package and the US crime data `UScrime` using the Zellner-Siow cauchy prior, and have tried to understand the importance of variables. 
Finally, we have compared the prediction results from different models, such as the ones from Bayesian model average (BMA), the highest probability model (HPM), the median probability model (MPM), and the best predictive model (BPM). For the comparison, we have used the Zellner-Siow Cauchy prior. But of course there is not one single best prior that is the best overall. If you do have prior information about a variable, you should include it. If you expect that there should be many predictors related to the response variable $Y$, but that each has a small effect, an alternate prior may be better. Also, think critically about whether model selection is important. If you believe that all the variables should be relevant but are worried about over fitting, there are alternative priors that will avoid putting probabilities on coefficients that are exactly zero and will still prevent over fitting by shrinkage of coefficients to prior means. Examples include the Bayesian lasso or Bayesian horseshoe.

There are other forms of model uncertainty that you may want to consider, such as linearity in the relationship between the predictors and the response, uncertainty about the presence of outliers, and uncertainty about the distribution of the response. These forms of uncertainty can be incorporated by expanding the models and priors similar to what we have covered here. 

Multiple linear regression is one of the most widely used statistical methods, however, this is just the tip of the iceberg of what you can do with Bayesian methods. 
