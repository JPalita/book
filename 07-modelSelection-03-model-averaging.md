## Bayesian Model Averaging

In the last section, we explored model uncertainty using posterior probability of models based on BIC. In this section, we will continue the kid's cognitive score example to see how to obtain an Bayesian model averaging results using model posterior probability.

### Visualizing Model Uncertainty

Recall that in the last section, we used the `bas.lm` function in the `BAS` package to obtain posterior probability of all models in the kid's cognitive score example. 
$$ \text{score} ~\sim~ \text{hq} + \text{IQ} + \text{work} + \text{age} $$

We have found the posterior distribution under model uncertainty using all possible combinations of the predictors, the mother's high school status `hs`, mother's IQ score `IQ`, whether the mother worked during the first three years of the kid's life `work`, and mother's age `age`. With 4 predictors, there are $2^4 = 16$ possible models. In general, for linear regression model with $p$ predictor variables
$$ y_i = \beta_0+\beta_1(x_{p,i}-\bar{x}) + \cdots + \beta_p(x_{p,i}-\bar{x}_p)+\epsilon_i,\qquad i = 1, \cdots,n,$$
there will be in total $2^p$ possible models.

We can also visualize model uncertainty from the `bas` object `cog_bas` that we generated in the previous section. 



In R, the image function may be used to create an image of the model space that looks like a crossword puzzle. 

```r
image(cog_bas, rotate = F)
```

```
## Warning in par(par.old): argument 1 does not name a graphical parameter
```

![](07-modelSelection-03-model-averaging_files/figure-latex/visualize-1.pdf)<!-- --> 

To obtain a clearer view for model comparison, we did not rotate the image. Here, the predictors, including the intercept, are on the $y$-axis, while the $x$-axis corresponds to each different model. Each vertical column corresponds to one model. For variables that are not included in one model, they will be represented by black blocks. For example, model 1 includes the intercept, `hs`, and `IQ`, but not `work` or `age`. These models are ordered according to the log of posterior odd over the null model (model with only the intercept). The log of posterior odd is calculated as
$$\ln(\PO[M_m:M_0]) = \ln (\BF[M_m:M_0]\times \Odd[M_m:M_0]).$$ 
Since we assing same prior probability for all models, $\text{O}[M_m:M_0] = 1$ and therefore, the log of posterior odd is the same as the log of the Bayes factor. The color of each column is proportional to the log of the posterior probability. Models with same colors have similar posterior probabilities. This allows us to view models that are clustered together, when the difference within a cluster is not worth a bare mention.

If we view the image by rows, we can see whether one variable is included in a particular model. For each variable, there are only 8 models in which it will appear. For example, we see that `IQ` appears in all the top 8 models with larger posterior probabilities, but not the last 8 models. The `image` function shows up to 20 models by default.

### Bayesian Model Averaging Using Posterior Probability

Once we have obtained the posterior probability of each model, we can make inference and obtain weighted averages of quantities of interest using these probabilities as weights. Models with higher posterior probabilities receive higher weights, while models with lower posterior probabilities receive lower weights. This gives the name "Bayesian  Model Averaging" (BMA). For example, the probability of the next prediction $\hat{Y}^*$ after seeing the data can be calculated as a "weighted average" of the prediction of next observation $\hat{Y}^*_j$ under each model $M_j$, with the posterior probability of $M_j$ being the "weight"
$$ \hat{Y}^* = \sum_{j=1}^{2^p}\hat{Y}^*_j\ p(M_j~|~\text{data}). $$

In general, we can use this weighted average formula to obtain the value of a quantity of interest $\Delta$. $\Delta$ can $Y^*$, the next observation; $\beta_j$, the coefficient of variable $X_j$; $p(\beta_j~|~\text{data})$, the posterior probability of $\beta_j$ after seeing the data. The posterior probability of $\Delta$ seeing the data can be calculated using the formula

\begin{equation} 
p(\Delta~|~\text{data}) = \sum_{j=1}^{2^p}p(\Delta~|~ M_j,\ \text{data})p(M_j~|~\text{data}).
(\#eq:general)
\end{equation}

This formula is similar to the one we have seen in Week 2 lecture **Predictive Inference** when we used posterior probability of two different success rates of getting the head in a coin flip to calculate the predictive probability of getting heads in **future** coin flips. Recall in that example, we have two competing hypothese, that the success rate (also known as the probability) of getting heads in coin flips, are
$$ H_1: p = 0.7,\qquad \text{vs}\qquad H_2: p=0.4.$$

We calcualted the posterior probability of each success rate. They are
$$ 
\begin{aligned}
P(p=0.7~|~\text{data})= & P(H_1~|~\text{data})= p^* = 0.754,\\
P(p=0.4~|~\text{data}) = & P(H_2~|~\text{data}) = 1-p^* = 0.246.
\end{aligned}
$$

We can use these two probabilities to calculate the posterior probability of getting head in the next coin flip
\begin{equation}
P(\text{head}~|~\text{data}) = P(\text{head}~|~H_1,\text{data})P(H_1~|~\text{data}) + P(\text{head}~|~H_2,\text{data})P(H_2~|~\text{data}).
(\#eq:example)
\end{equation}

We can see that equation \@ref(eq:example) is just a special case of the general equation \@ref(eq:general) when the posterior probability of hypotheses $P(H_1~|~\text{data})$ and $P(H_2~|~\text{data})$ serve as weights.

Moreover, the expected value of $\Delta$ can also be obtained by a weighted average formula of expected values on each model, using conditional probability

$$ E[\Delta~|~\text{data}] = \sum_{j=1}^{2^p}E[\Delta~|~M_j,\ \text{data}]p(M_j~|~\text{data}).$$


Since the weights $p(M_j~|~\text{data})$ are probabilities and have to sum to one, if the best model had posterior probability one, all of the weights would be placed on that single best model. In this case, using BMA would be equivalent to selecting the best model with the highest posterior probability. However, if there are several models that receive substantial probability, they would all be included in the inference and account for the uncertainty about the true model. 

### Coefficient Summary under BMA

We can obtain the coefficients by the `coef` function. 


```r
cog_coef = coef(cog_bas)
cog_coef
```

```
## 
##  Marginal Posterior Summaries of Coefficients: 
## 
##  Using  BMA 
## 
##  Based on the top  16 models 
##            post mean  post SD   post p(B != 0)
## Intercept  86.79724    0.87287   1.00000      
## hs          3.59494    3.35643   0.61064      
## IQ          0.58101    0.06363   1.00000      
## work        0.36696    1.30939   0.11210      
## age         0.02089    0.11738   0.06898
```

Under Bayesian model averaging, the table above provides the posterior mean, the posterior standard deviation, and the posterior inclusion probability (pip) of each coefficient. The posterior mean of the coefficient $\hat{\beta}_j$ under BMA would be used for future predictions. The posterior standard deviation $\text{se}_{\beta_j}$ provides measure of variability of the coefficient $\beta_j$. An approximate range of plausible values for each of the coefficients may be obtained via the empirical rule
$$ (\hat{\beta}_j-\text{critical value}\times \text{se}_{\beta_j},\  \hat{\beta}_j+\text{critical value}\times \text{se}_{\beta_j}).$$

However, this only applies if the posterior distribution is symmetric or unimodal. 

The posterior mean of the intercept, $\hat{\beta}_0$, is obtained after we have centered the variables. We have discussed the effect of centering the model. One of the advantage of doing so is that the intercept $\beta_0$ represents the sample mean of the observed response $Y$. Under the reference prior, the point estimate of the intercept $\hat{\beta}_0$ is exactly the mean $\bar{Y}$.

We see that the posterior mean, standard deviation and inclusion probability are slightly different than the ones we obtained in Section \@ref(sec:Bayes-multiple-regression) when we forced the model to include all variables. Under BMA, `IQ` has posterior inclusion probability 1, suggesting that it is very likely that `IQ` should be included in the model. `hs` also has a high posterior inclusion probability of about 0.61. However, the posterior inclusion probability of mother's work status `work` and mother's age `age` are relatively small compared to `IQ` and `hs`.

We can also `plot` the posterior distributions of these coefficients to take a closer look at the distributions

```r
par(mfrow = c(2, 2))
plot(cog_coef, subset = c(2:5))
```

![](07-modelSelection-03-model-averaging_files/figure-latex/plot-dis-1.pdf)<!-- --> 


This plot agrees with the summary table we obtained above, which shows that the posterior probability distributions of `work` and `age` have a very large point mass at 0, while the distribution of `hs` has a relatively small mass at 0. There is a slighly little tip at 0 for the variable `IQ`, indicating that the posterior inclusion probability of `IQ` is not exactly 1. However, since the probability mass for `IQ` to be 0 is so small, that we are almost certain that `IQ` should be included under Bayesian model averaging.

## Summary

In this chapter, we have discussed Bayesian model uncertainty and Bayesian model averaging. We have shown how Bayesian model averaging can be used to address model uncertainty using the ensemble of models for inference, rather than selecting a single model. We applied this to the kid's cognitive score data set using `BAS` package in R. Here we illustrated the concepts using BIC and reference prior on the coefficients. In the next chapter, we will explore alternative priors for coefficients, taking into account the sensitivity of model selection to prior choices. We will also explore Markov Chain Monte Carlo algorithm for model sampling when the model space is too large for theoretical calculations.

