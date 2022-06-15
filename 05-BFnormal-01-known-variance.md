## Bayes Factors for Testing a Normal Mean: variance known {#sec:known-var}






Now we show how to obtain Bayes factors for testing hypothesis about a normal mean, where **the variance is known**. To start, let's consider a random sample of observations from a normal population with mean $\mu$ and pre-specified variance $\sigma^2$. We consider testing whether the population mean $\mu$ is equal to $m_0$ or not.

Therefore, we can formulate the data and hypotheses as below:

**Data**
$$Y_1, \cdots, Y_n \iid \No(\mu, \sigma^2)$$

**Hypotheses**

* $H_1: \mu = m_0$
* $H_2: \mu \neq m_0$

**Priors**

We also need to specify priors for $\mu$ under both hypotheses. Under $H_1$, we assume that $\mu$ is exactly $m_0$, so this occurs with probability 1 under $H_1$. Now under $H_2$, $\mu$ is unspecified, so we describe our prior uncertainty with the conjugate normal distribution centered at $m_0$ and with a variance $\sigma^2/\mathbf{n_0}$. This is centered at the hypothesized value $m_0$, and it seems that the mean is equally likely to be larger or smaller than $m_0$, so a dividing factor $n_0$ is given to the variance. The hyper parameter $n_0$ controls the precision of the prior as before.

In mathematical terms, the priors are:

* $H_1: \mu = m_0  \text{  with probability 1}$
* $H_2: \mu \sim \No(m_0, \sigma^2/\mathbf{n_0})$

**Bayes Factor**

Now the Bayes factor for comparing $H_1$ to $H_2$ is the ratio of the distribution, the data under the assumption that $\mu = m_0$ to the distribution of the data under $H_2$.

$$\begin{aligned}
\BF[H_1 : H_2] &= \frac{p(\data \mid \mu = m_0, \sigma^2 )}
 {\int p(\data \mid \mu, \sigma^2) p(\mu \mid m_0, \mathbf{n_0}, \sigma^2)\, d \mu} \\
\BF[H_1 : H_2] &=\left(\frac{n + \mathbf{n_0}}{\mathbf{n_0}} \right)^{1/2} \exp\left\{-\frac 1 2 \frac{n }{n + \mathbf{n_0}} Z^2 \right\} \\
 Z   &=  \frac{(\bar{Y} - m_0)}{\sigma/\sqrt{n}}
\end{aligned}$$

The term in the denominator requires integration to account for the uncertainty in $\mu$ under $H_2$. And it can be shown that the Bayes factor is a function of the observed sampled size, the prior sample size $n_0$ and a $Z$ score.

Let's explore how the hyperparameters in $n_0$ influences the Bayes factor in Equation \@ref(eq:BayesFactor). For illustration we will use the sample size of 100. Recall that for estimation, we interpreted $n_0$ as a prior sample size and considered the limiting case where $n_0$ goes to zero as a non-informative or reference prior.

\begin{equation}
\textsf{BF}[H_1 : H_2] = \left(\frac{n + \mathbf{n_0}}{\mathbf{n_0}}\right)^{1/2} \exp\left\{-\frac{1}{2} \frac{n }{n + \mathbf{n_0}} Z^2 \right\}
(\#eq:BayesFactor)
\end{equation}

Figure \@ref(fig:vague-prior) shows the Bayes factor for comparing $H_1$ to $H_2$ on the y-axis as $n_0$ changes on the x-axis. The different lines correspond to different values of the $Z$ score or how many standard errors $\bar{y}$ is from the hypothesized mean. As expected, larger values of the $Z$ score favor $H_2$.

\begin{figure}

{\centering \includegraphics{05-BFnormal-01-known-variance_files/figure-latex/vague-prior-1} 

}

\caption{Vague prior for mu: n=100}(\#fig:vague-prior)
\end{figure}

But as $n_0$ becomes smaller and approaches 0, the first term in
the Bayes factor goes to infinity, while the exponential term involving the
data goes to a constant and is ignored. In the limit as $n_0 \rightarrow 0$ under this noninformative prior, the Bayes factor paradoxically ends up favoring $H_1$ regardless of the value of $\bar{y}$.

The takeaway from this is that we cannot use improper priors with $n_0 = 0$, if we are going to test our hypothesis that $\mu = n_0$. Similarly, vague priors that use a small value of $n_0$ are not recommended due to the sensitivity of the results to the choice of an arbitrarily small value of $n_0$.

This problem arises with vague priors -- the Bayes factor favors the null model $H_1$ even when the data are far away from the value under the null -- are known as the Bartlett's paradox or the Jeffrey's-Lindleys paradox.

Now, one way to understand the effect of prior is through the standard effect size

$$\delta = \frac{\mu - m_0}{\sigma}.$$
The prior of the standard effect size is

$$\delta \mid   H_2  \sim \No(0, \frac{1}{\mathbf{n_0}})$$

This allows us to think about a standardized effect independent of the units of the problem. One default choice is using the unit information prior, where the prior sample size $n_0$ is 1, leading to a standard normal for the standardized effect size. This is depicted with the blue normal density in Figure \@ref(fig:effect-size). This suggested that we expect that the mean will be within $\pm 1.96$ standard deviations of the hypothesized mean **with probability 0.95**. (Note that we can say this only under a Bayesian setting.)

In many fields we expect that the effect will be small relative to $\sigma$. If we do not expect to see large effects, then we may want to use a more informative prior on the effect size as the density in orange with $n_0 = 4$. So they expected the mean to be within $\pm 1/\sqrt{n_0}$ or five standard deviations of the prior mean.

\begin{figure}

{\centering \includegraphics{05-BFnormal-01-known-variance_files/figure-latex/effect-size-1} 

}

\caption{Prior on standard effect size}(\#fig:effect-size)
\end{figure}

::: {.example #unnamed-chunk-2}
To illustrate, we give an example from parapsychological research. The case involved the test of the subject's claim to affect a series of randomly generated 0's and 1's by means of extra sensory perception (ESP). The random sequence of 0's and 1's are generated by a machine with
probability of generating 1 being 0.5. The subject claims that his ESP would make the sample mean differ significantly from 0.5.
:::

Therefore, we are testing $H_1: \mu = 0.5$ versus $H_2: \mu \neq 0.5$. Let's use a prior that suggests we do not expect a large effect which leads
the following solution for $n_0$. Assume we want a standard effect of 0.03, there is a 95% chance that it is between $(-0.03/\sigma, 0.03/\sigma)$, with $n_0 = (1.96\sigma/0.03)^2 = 32.7^2$.

Figure \@ref(fig:prior-effect) shows our informative prior in blue, while the unit information prior is in orange. On this scale, the unit information
prior needs to be almost uniform for the range that we are interested.



\begin{figure}

{\centering \includegraphics{05-BFnormal-01-known-variance_files/figure-latex/prior-effect-1} 

}

\caption{Prior effect in the extra sensory perception test}(\#fig:prior-effect)
\end{figure}

A very large data set with over 104 million trials was collected to test this hypothesis, so we use a normal distribution to approximate the distribution the sample mean.

* Sample size: $n = 1.0449 \times 10^8$
* Sample mean: $\bar{y} =  0.500177$, standard deviation $\sigma = 0.5$
* $Z$-score: 3.61

Now using our prior in the data, the Bayes factor for $H_1$ to $H_2$ was 0.46, implying evidence against the hypothesis $H_1$ that $\mu = 0.5$.

* Informative $\BF[H_1:H_2] = 0.46$
* $\BF[H_2:H_1] = 1/\BF[H_1:H_2] = 2.19$

Now, this can be inverted to provide the evidence in favor of $H_2$. The evidence suggests that the hypothesis that the machine operates with a probability that is not 0.5, is 2.19 times more likely than the hypothesis
the probability is 0.5. Based on the interpretation of Bayes factors from Table \@ref(tab:jeffreys1961), this is in the range of "not worth the bare mention".

To recap, we present expressions for calculating Bayes factors for a normal model with a specified variance. We show that the improper reference priors for $\mu$ when $n_0 = 0$, or vague priors where $n_0$ is arbitrarily small,
lead to Bayes factors that favor the null hypothesis regardless of the data, and thus should not be used for hypothesis testing.

Bayes factors with normal priors can be sensitive to the choice of the $n_0$. While the default value of $n_0 = 1$ is reasonable in many cases, this may be too non-informative if one expects more effects. Wherever possible, think about how large an effect you expect and use that information to help select the $n_0$.

All the ESP examples suggest weak evidence and favored the machine generating random 0's and 1's with a probability that is different from 0.5. Note that ESP is not the only explanation -- a deviation from 0.5 can also occur if the random number generator is biased. Bias in the stream of random numbers in our pseudorandom numbers has huge implications for numerous fields that depend on simulation. If the context had been about detecting a small bias in random numbers what prior would you use and how would it change the outcome? You can experiment it in `R` or other software packages that generate random Bernoulli trials.

Next, we will look at Bayes factors in normal models with unknown variances using the Cauchy prior so that results are less sensitive to the choice of $n_0$.
