---
title: R gastax project
author: Ross Khelemsky
date: June 2023
---

Today I am going to analyze the proposed $1 increase in the gas tax in 2005 using Rstudio and time-series data on income, gas consumption, gas prices, and the prices of all other goods. I will use OLS multiple linear regression to estimate a relationship between gas consumption, real income, and the real price of gas (and find out how much gas consumption would decrease from a $1 increase in price, the effect of the tax). Then I will fit the data to the indirect utility function from Hausman (1981) to estimate the mean consumer's loss of utility from the tax. Then I will rearrange the function into the expenditure function to estimate how much is required in compensation for the consumer to be indifferent to the price change. After that, I will estimate the new consumption of gas after the tax and rebate. Then I will estimate the per capita tax revenue with and without compensation, and carbon emissions reduction with and without compensation. Finally, I will discuss the conditions under which the tax and rebate would be welfare-enhancing.[^1]

Let's begin:

```r
library(haven)

gastaxdata <- (SAS_data)

summary(gastaxdata)
SAS_data is the dataset I am going to use for this analysis. Since it is in SAS format, I have to use the haven package to import it into Rstudio. It contains time-series data on annual per capita gas consumption (in gallons), annual per capita nominal income (disposable less savings), average nominal price of gas (per gallon), and price of all other goods (CPI less energy) from 1984-2005.

Summary statistics for the transformed data:

r
Copy code
gastaxdata$G <- gastaxdata$G_PC_AN / 12
gastaxdata$I_N <- gastaxdata$I_N_AN / 12

gastaxdata$P_G_R <- gastaxdata$P_G_N / gastaxdata$P_AOG
gastaxdata$I_R <- gastaxdata$I_N / gastaxdata$P_AOG

summary(gastaxdata[c("G", "P_G_R", "I_R")])
Next, I run OLS multiple linear regression, where monthly gas consumption is dependent on the real price of gas and real monthly income. This fits a plane to the data, minimizing the sum of squared residuals:

r
Copy code
fit <- lm(G ~ P_G_R + I_R, data = gastaxdata)
summary(fit)
Now I can plot demand curves for gas (consumption as a function of price) at different levels of real monthly income held constant:

r
Copy code
library(ggplot2)

p1 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() + 
      geom_abline(intercept = 4.4506, slope = -0.0707)

p1 + ggtitle("Estimated demand curve with I_R held constant at its mean value (2006.60)")
r
Copy code
p2 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() +
      geom_abline(intercept = 4.3956, slope = -0.0707)

p2 + ggtitle("Estimated demand curve with I_R held constant at its median value (1941.01)")
r
Copy code
p3 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() + 
      geom_abline(intercept = 4.8277, slope = -0.0707)

p3 + ggtitle("Estimated demand curve with I_R held constant at its 2005 value (2455.67)")
Now that I have the regression model, I can plug in the 2005 values for I_R and P_G_R (2455.67 and 1.828, respectively) to estimate the pre-tax consumption amount. Let's call this G_notax.

G_notax = 39.09754 - (14.1364 * 1.828) + (2455.67 * 0.01187) = 42.405 gal/mo

The actual amount was 44.113, so the error is not very large for this estimate.

To get the consumption amount with a $1 tax increase (and no rebate), I plug in the same I_R and 2.828 for P_G_R. Let's call this amount G_tax_uncomp.

G_tax_uncomp = 39.09754 - (14.1364 * 2.828) + (2455.67 * 0.01187) = 28.2686 gal/mo

Now I can estimate the change in utility using the indirect utility function from JA Hausman's paper ("Exact Consumer’s Surplus and Deadweight Loss” American Economic Review, 71:4, Sept. 1981, p. 668):

The indirect utility function indexes the consumer's utility based on the real price of a good (P_G_R), real income (I_R), and the regression slopes and intercepts above for those variables. The higher (more positive) it is, the better off the consumer:

Indirect utility = U(P_G_R,I_R) = e^{-\gamma*P_G_R} \cdot \left(I_R + \left(\frac{1}{\gamma} \cdot \left(\beta \cdot P_G_R + \frac{\beta}{\gamma} + \alpha\right)\right)\right)

where $\gamma$ is the regression slope coefficient for I_R (0.01187), $\beta$ is the regression slope coefficient for P_G_R (-14.1364), and $\alpha$ is the intercept (39.09754). Plugging in the 2005 values for P_G_R and I_R without the tax (U_notax), and the 2005 values with the tax and no compensation (U_tax_uncomp), we get:

U_notax = U(1.828, 2455.67) = 
�
−
0.01187
⋅
1.828
⋅
(
2455.67
+
(
1
0.01187
⋅
(
−
14.1364
⋅
1.828
+
−
14.1364
0.01187
+
39.09754
)
)
)
=
−
94682.17
e 
−0.01187⋅1.828
 ⋅(2455.67+( 
0.01187
1
​
 ⋅(−14.1364⋅1.828+ 
0.01187
−14.1364
​
 +39.09754)))=−94682.17

U_tax_uncomp = U(2.828, 2455.67) = 
�
−
0.01187
⋅
2.828
⋅
(
2455.67
+
(
1
0.01187
⋅
(
−
14.1364
⋅
2.828
+
−
14.1364
0.01187
+
39.09754
)
)
)
=
−
94716.56
e 
−0.01187⋅2.828
 ⋅(2455.67+( 
0.01187
1
​
 ⋅(−14.1364⋅2.828+ 
0.01187
−14.1364
​
 +39.09754)))=−94716.56

I can rearrange the terms of the indirect utility function to isolate I_R and get the expenditure function, which gives the income required to achieve some level of utility, given a utility function and prices:

I_R = Expend(P_G_R, U) = (e^{\gamma \cdot P_G_R} \cdot U) - \left(\frac{1}{\gamma} \cdot \left(\beta \cdot P_G_R + \frac{\beta}{\gamma}\right) + \alpha\right)

Plugging in the pre-tax utility level (U_notax), the post-tax P_G_R, and the same $\beta$, $\gamma$, and $\alpha$, I get that the required expenditure to be at the same utility level as before the tax is:

Expend(2.828, -94682.17) = 
(
�
0.01187
⋅
2.828
⋅
−
94682.17
)
−
(
1
0.01187
⋅
(
−
14.1364
⋅
2.828
+
−
14.1364
0.01187
+
39.09754
)
)
=
2491.23
(e 
0.01187⋅2.828
 ⋅−94682.17)−( 
0.01187
1
​
 ⋅(−14.1364⋅2.828+ 
0.01187
−14.1364
​
 +39.09754))=2491.23

This means that the required monthly compensation per capita is $2491.23 - 2455.67 = 35.36$.

To get G (monthly gas consumption per capita) after the tax and compensation, simply plug in 2.828 for P_G_R and 2491.23 for I_R back into the regression:

G_tax_comp = 39.09754 - (14.1364 * 2.828) + (2491.23 * 0.01187) = 28.6907 gal/mo

Monthly tax revenue per capita without compensation is tax amount per gallon * gallons consumed = $1 \cdot 28.2686 = 28.2686$.

Monthly tax revenue per capita with compensation is (tax amount per gallon * gallons consumed) - rebate amount = $(1 \cdot 28.6907) - 35.36 = -6.669$.

Now we can use the EPA's estimate for the amount (in metric tons) of carbon dioxide emitted per gallon of gasoline consumed (0.00889) to calculate the change in consumption emissions between no tax and tax with compensation:

Consumption difference between no tax and compensated tax = $28.6907 - 42.405 = -13.71$ gal/mo.

Monthly carbon emissions reduction (MTCO2/mo) from tax with compensation = $-13.71 \cdot 0.00889 = -0.121882$.

From my understanding, this means that for the tax and compensation to be welfare increasing (in that society benefits, which is hopefully what the government wants): 
�
�
�
�
_
�
�
�
_
�
�
�
�
�
�
�
�
�
�
�
�
�
_
�
�
�
�
�
�
�
�
�
_
�
�
�
�
�
�
�
�
�
<
�
�
�
�
�
�
_
�
�
�
�
_
�
�
�
�
�
�
.
carbon_emissions_reduction
loss_tax_revenue
​
 <social_cost_carbon. This is the main welfare consideration since consumers are indifferent with compensation.

The social cost of carbon is the net present value to society of future damages caused by 1 additional ton of carbon being emitted (via climate change). It can also be reinterpreted as the benefit society would gain by emitting 1 metric ton less CO2.

I can then get the equation that if welfare is positive, given consumers indifferent, then: 
�
�
�
�
_
�
�
�
_
�
�
�
�
�
�
�
<
(
�
�
�
�
�
�
_
�
�
�
�
�
�
�
�
�
_
�
�
2
_
�
�
�
�
�
�
�
�
�
_
�
�
_
�
�
�
�
�
�
_
�
�
�
�
)
⋅
(
�
�
�
�
�
�
�
�
�
_
�
�
_
�
�
�
�
_
�
�
_
�
�
�
�
�
�
�
_
�
�
_
1
_
�
�
�
�
�
_
�
�
�
�
�
�
_
�
�
�
_
�
�
2
_
�
�
�
�
�
_
�
�
�
�
�
�
�
)
loss_tax_revenue<(amount_reduction_CO2_emissions_in_metric_tons)⋅(reduction_in_cost_to_society_of_1_fewer_metric_ton_CO2_being_emitted).

Therefore, for the tax and compensation to be welfare-enhancing, the social cost of carbon must be at or above Social\_cost\_carbon > \frac{-6.669}{-0.121882} = $54.71/MTCO2 which is actually not very far off from recent estimates of SCCO2: Wang et al. 2019 conducted a meta-analysis on estimates of the social cost of carbon and found a mean estimate of $54.70 per ton of CO2 emitted (Pei Wang, Xiangzheng Deng, Huimin Zhou, Shangkun Yu, Estimates of the social cost of carbon: A review based on meta-analysis, Journal of Cleaner Production) (Though I'm not sure if this is in real dollars or 2019 dollars. The 2005 tax $54.71 threshold is in real dollars.)

If you've made it this far, I have a question to improve on this analysis for future applications. Should I add any of the following?:

An explanation of the standard errors and significance of the t-values for the slopes/intercept in the explanation of the regression,
Welfare analysis for the uncompensated tax,
Consideration of other social costs avoided like congestion,
More discussion of Limitations of the analysis, supply-side effects, potential omitted variables in regression, problems of measuring utility, distributional effects, etc.,
Extension of this analysis to the current year,
Extension of this analysis with different tax rates,
Or would that be too much clutter? This already feels like a lot for an undergrad writing sample, but also many (most?) competitive applicants already have multiple coauthored papers/PhD level econ coursework/A's in real analysis and topology by my age, so I think it needs to be extensive lol.
