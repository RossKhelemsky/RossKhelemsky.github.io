---
title: R gastax project
author: Ross Khelemsky
date: June 2023
---

Today I am going to analyze the proposed $1 increase in the gas tax in 2005 using Rstudio and time-series data on income, gas consumption, gas prices, and the prices of all other goods. I will use OLS multiple linear regression to estimate a relationship between gas consumption, real income, and the real price of gas (and find out how much gas consumption would decrease from a $1 increase in price, the effect of the tax). Then I will fit the data to the indirect utility function from Hausman (1981) to estimate the mean consumer's loss of utility from the tax. Then I will rearrange the function into the expenditure function to estimate how much is required in compensation for the consumer to be indifferent to the price change. After that, I will estimate the new consumption of gas after the tax and rebate. Then I will estimate the per capita tax revenue with and without compensation, and carbon emissions reduction with and without compensation. Finally, I will discuss the conditions under which the tax and rebate would be welfare-enhancing.[^1]

Let's begin:

{% highlight R %} 
library(haven)

gastaxdata <- (SAS_data)

summary(gastaxdata)
{% endhighlight %}

SAS_data is the dataset I am going to use for this analysis. Since it is in SAS format, I have to use the haven package to import it into Rstudio. It contains time-series data on annual per capita gas consumption (in gallons), annual per capita nominal income (disposable less savings), average nominal price of gas (per gallon), and price of all other goods (CPI less energy) from 1984-2005.

Summary statistics for the transformed data:


{% highlight R %} 
gastaxdata$G <- gastaxdata$G_PC_AN / 12
gastaxdata$I_N <- gastaxdata$I_N_AN / 12

gastaxdata$P_G_R <- gastaxdata$P_G_N / gastaxdata$P_AOG
gastaxdata$I_R <- gastaxdata$I_N / gastaxdata$P_AOG

summary(gastaxdata[c("G", "P_G_R", "I_R")])
{% endhighlight %}
Next, I run OLS multiple linear regression, where monthly gas consumption is dependent on the real price of gas and real monthly income. This fits a plane to the data, minimizing the sum of squared residuals:


{% highlight R %} 
fit <- lm(G ~ P_G_R + I_R, data = gastaxdata)
summary(fit)
{% endhighlight %}

Now I can plot demand curves for gas (consumption as a function of price) at different levels of real monthly income held constant:

{% highlight R %} 
library(ggplot2)

p1 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() + 
      geom_abline(intercept = 4.4506, slope = -0.0707)

p1 + ggtitle("Estimated demand curve with I_R held constant at its mean value (2006.60)")
{% endhighlight %}


{% highlight R %} 
p2 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() +
      geom_abline(intercept = 4.3956, slope = -0.0707)

p2 + ggtitle("Estimated demand curve with I_R held constant at its median value (1941.01)")
{% endhighlight %}


{% highlight R %} 
p3 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() + 
      geom_abline(intercept = 4.8277, slope = -0.0707)

p3 + ggtitle("Estimated demand curve with I_R held constant at its 2005 value (2455.67)")
{% endhighlight %}

Now that I have the regression model, I can plug in the 2005 values for I_R and P_G_R (2455.67 and 1.828, respectively) to estimate the pre-tax consumption amount. Let's call this G_notax.

G_notax = 39.09754 - (14.1364 * 1.828) + (2455.67 * 0.01187) = 42.405 gal/mo

The actual amount was 44.113, so the error is not very large for this estimate.

To get the consumption amount with a $1 tax increase (and no rebate), I plug in the same I_R and 2.828 for P_G_R. Let's call this amount G_tax_uncomp.

G_tax_uncomp = 39.09754 - (14.1364 * 2.828) + (2455.67 * 0.01187) = 28.2686 gal/mo

Now I can estimate the change in utility using the indirect utility function from JA Hausman's paper ("Exact Consumer’s Surplus and Deadweight Loss” American Economic Review, 71:4, Sept. 1981, p. 668):

The indirect utility function indexes the consumer's utility based on the real price of a good (P_G_R), real income (I_R), and the regression slopes and intercepts above for those variables. The higher (more positive) it is, the better off the consumer:

Indirect utility = U(P_G_R,I_R) = e^{-\gamma*P_G_R} \cdot \left(I_R + \left(\frac{1}{\gamma} \cdot \left(\beta \cdot P_G_R + \frac{\beta}{\gamma} + \alpha\right)\right)\right)

where $\gamma$ is the regression slope coefficient for I_R (0.01187), $\beta$ is the regression slope coefficient for P_G_R (-14.1364), and $\alpha$ is the intercept (39.09754). Plugging in the 2005 values for P_G_R and I_R without the tax (U_notax), and the 2005 values with the tax and no compensation (U_tax_uncomp), we get:


