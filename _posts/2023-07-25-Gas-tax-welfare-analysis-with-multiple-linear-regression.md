---
title: R gas tax project
author: Ross Khelemsky
mathjax: true
date: June 2023
categories: media
---

Today I am going to analyze the proposed $1 increase in the gas tax in 2005 using Rstudio and time-series data on income, gas consumption, gas prices, and the prices of all other goods. I will use OLS multiple linear regression to estimate a relationship between gas consumption, real income, and the real price of gas (and find out how much gas consumption would decrease from a $1 increase in price, the effect of the tax). Then I will fit the data to the indirect utility function from Hausman (1981) to estimate the mean consumer's loss of utility from the tax. Then I will rearrange the function into the expenditure function to estimate how much is required in compensation for the consumer to be indifferent to the price change. After that, I will estimate the new consumption of gas after the tax and rebate. Then I will estimate the per capita tax revenue with and without compensation, and carbon emissions reduction with and without compensation. Finally, I will discuss the conditions under which the tax and rebate would be welfare-enhancing.[^1]

Let's begin:

{% highlight r %} 
library(haven)

gastaxdata <- (SAS_data)

summary(gastaxdata)
{% endhighlight %}

SAS_data is the dataset I am going to use for this analysis. Since it is in SAS format, I have to use the haven package to import it into Rstudio. It contains time-series data on annual per capita gas consumption (in gallons), annual per capita nominal income (disposable less savings), average nominal price of gas (per gallon), and price of all other goods (CPI less energy) from 1984-2005.


Summary(gastaxdata) displays summary statistics for the variables in the dataset (minimum, 1st quartile, median mean, 3rd quartile, and maximum). The summary statistics for gastaxdata are displayed in the figure below:


![Summary statistics for all variables](https://github.com/niklasbuschmann/contrast/assets/137047194/cb3db72d-5d57-454e-a981-789ee3666439)

Next I convert annual per capita gasoline consumption and nominal income
to monthly. Then I convert nominal monthly income and the nominal price of
gas into real monthly income and the real price of gas by deflating by the CPI:

{% highlight r %}
gastaxdata$G <- gastaxdata$G_PC_AN / 12
gastaxdata$I_N <- gastaxdata$I_N_AN / 12

gastaxdata$P_G_R <- gastaxdata$P_G_N / gastaxdata$P_AOG
gastaxdata$I_R <- gastaxdata$I_N / gastaxdata$P_AOG

summary(gastaxdata[c("G", "P_G_R", "I_R")])
{% endhighlight %}

I then retrieve summary statistics for the transformed data:

![Summary statistics for transformed variables](https://github.com/niklasbuschmann/contrast/assets/137047194/2b0d0751-85c8-49c3-9f5e-3058378cba99)

Then I run OLS multiple linear regression, where monthly gas consumption (G) is dependent on the real price of gas (P_G_R) and real monthly income (I_R). This fits a plane to the data which minimizes the sum of squared residual errors between the observed and predicted values of G, of the form: 

$$\mathbf{G}=\beta_1 \mathbf{I_R} + \beta_2 \mathbf{P_{GR}} + \beta_0$$

![Summary statistics of OLS multiple linear regression](https://github.com/niklasbuschmann/contrast/assets/137047194/115cb969-73ef-481c-a50d-aac086141493)

{% highlight r %} 
fit <- lm(G ~ P_G_R + I_R, data = gastaxdata)
summary(fit)
{% endhighlight %}

β1, the slope estimate for I_R, is 0.01187. This means that
for a 1$ rise in real monthly income (with the real price of gas held constant),
monthly gas consumption is expected to increase by 0.01187 gal/mo.

β2, the slope estimate for P_G_R, is −14.1364. This means
that for a 1$ rise in the real price of gas (with real monthly income held con-
stant), monthly gas consumption is expected to decrease by 14.136 gal/mo.


β0, the intercept estimate, is 39.09754. This means that if real incomes and
the real price of gas were 0$, per capita monthly gas consumption would be
39.09754 gal/mo. While it may not have a real-world interpretation(since the
real price of gas isn't going to ever be 0), it is still necessary
to compute to find the best fit line(or plane, in this instance).


This vector of residuals, 

$$\vec{\beta}= \begin{matrix}
    \beta_0\\
    \beta_1\\
    \beta_2
\end{matrix} = \begin{matrix}
    39.09754\\
    0.01187\\
    -14.1364
\end{matrix}$$ 

can also be derived analytically, using some matrix algebra and calculus, as shown in Appendix A.


The adjusted R^2 of 0.8979 implies that 89.79% of the variation in gas consumption can be explained by changes in the real price of gas and real income.


Now I use gglot to plot demand curves for gas (monthly gas consumption in gal/mo as a function of real price) at different levels of real monthly income held constant:

{% highlight r %} 
library(ggplot2)

p1 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() + 
      geom_abline(intercept = 4.4506, slope = -0.0707)

p1 + ggtitle("Estimated demand curve with I_R held constant at its mean value (2006.60)")
{% endhighlight %}

![regression I_R held at mean value](https://github.com/niklasbuschmann/contrast/assets/137047194/daf00939-7e6c-4290-8d6d-3876b32338dd)

{% highlight r %} 
p2 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() +
      geom_abline(intercept = 4.3956, slope = -0.0707)

p2 + ggtitle("Estimated demand curve with I_R held constant at its median value (1941.01)")
{% endhighlight %}

![regression I_R held at median value](https://github.com/niklasbuschmann/contrast/assets/137047194/51266789-3007-450b-8627-1ee331d9d919)

{% highlight r %} 
p3 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() + 
      geom_abline(intercept = 4.8277, slope = -0.0707)

p3 + ggtitle("Estimated demand curve with I_R held constant at its 2005 value (2455.67)")
{% endhighlight %}

![regression I_R held constant at 2005 value](https://github.com/niklasbuschmann/contrast/assets/137047194/4fcb6c3c-2287-4f64-a918-e1fcfd4371b2)

Also, we can rerun this regression in python and use plotly create an interactive 3d plot of the plane fit to the data, which I have linked to and attached a picture of below: 

[Interact with the 3D plot shown below!](/3d_plot.html)

![Interactive 3D plot](https://github.com/niklasbuschmann/contrast/assets/137047194/d6736bba-b56f-4c2a-9d6c-7438f5edb5c7)
Code is shown in Appendix B.

Now that I have the regression model, I can plug in the 2005 values for I_R and P_G_R (2455.67 and 1.828, respectively) to estimate the pre-tax consumption amount. Let's call this G_notax.

$$\mathbf{G_{notax}} = 39.09754 - (14.1364 * 1.828) + (2455.67 * 0.01187) = 42.405 gal/mo$$

The actual amount was 44.113, so the error is not very large for this estimate.

To get the consumption amount with a $1 tax increase (and no rebate), I plug in the same I_R and 2.828 for P_G_R. Let's call this amount G_taxuncomp.

$$ \mathbf{G_{taxuncomp}} = 39.09754 − (14.1364 ∗ 2.828) + (2455.67 ∗ 0.01187) = 28.2686
gal/mo $$


After that, I can estimate the change in consumer utility from the tax using the indirect utility function from JA Hausman’s paper(”Exact Consumer’s Surplus and Deadweight Loss” American Economic Review, 71:4, Sept. 1981, p. 668): The indirect utility function indexes the consumer's utility(happiness/pleasure/satisfaction) based on the real price of a good(P_G_R), real income(I_R), and the regression slopes and intercepts above for those variables. The higher(more positive) it is, the better off the consumer:
Indirect Utility =

$$\mathbf{U(P_{GR}, I_R)} = e^{-\beta_1 P_{GR}} * (\mathbf{I_R} + (\frac{1}{\beta_1}  (\beta_2 P_{GR} + \frac{\beta_2}{\beta_1} + \beta_0)))$$

where β1 is the regression slope coefficient for I_R (0.01187), β2 is the regression slope coefficient for P_G_R (-14.1364), and β0 is the intercept (39.09754). Plugging in the 2005 values for P_G_R and I_R without the tax (for U_notax), and the 2005 values with the tax and no compensation (for U_taxuncomp), we get consumer utility with and without the tax:

$$ \mathbf{U_{notax}} = \mathbf{U(1.828, 2455.67)} = e^{−0.01187∗1.828} ∗ (2455.67 + ( \frac{1}{0.01187} ∗ (−14.1364 ∗ 1.828 + \frac{−14.1364}{0.01187} + 39.09754))) = −94682.17$$

$$ \mathbf{U_{taxuncomp}} = \mathbf{U(2.828, 2455.67)} = e^{−0.01187∗2.828} ∗ (2455.67 + ( \frac{1}{0.01187} ∗ (−14.1364 * 2.828 + \frac{−14.1364}{0.01187} + 39.09754))) = −94716.56$$

I can rearrange the terms of the indirect utility function(shown in appendix B) to isolate I_R and get the expenditure function, which gives the income required to achieve some level of utility, given a utility function and prices:

$$ \mathbf{I_R} = \mathbf{Expend(P_{GR}, U)} = (e^{\beta_1 * P_{GR}}* U) -(\frac{1}{\beta_1} * (\beta_2 * P_{GR} + \frac{\beta_2}{\beta_1} + \beta_0))$$


Plugging in the pre-tax utility level(U_{notax}), the post-tax P_G_R, and the same β2, β1, and β0, I get that the required expenditure to be at the same utility level as before the tax is:

$$\mathbf{Expend(2.828, -94682.17)} = (e^{0.01187*2.828} * -94682.17) -(\frac{1}{0.01187} * (-14.1364 * 2.828 + \frac{-14.1364}{0.01187} + 39.09754)) = 2491.23$$

This means that the required monthly compensation per capita is 2491.23 − 2455.67 = 35.36

To get G(monthly gas consumption per capita) after the tax and compensa-
tion, simply plug in 2.828 for P_G_R and 2491.23 for I_R back into the regression:

$$G_{taxcomp} = 39.09754 − (14.1364 ∗ 2.828) + (2491.23 ∗ 01187) = 28.6907gal/mo$$

Monthly tax revenue per capita without compensation is (tax amount per gallon * gallons consumed) = 1*28.2686 = 28.2686

Monthly tax revenue per capita with compensation is (tax amount per gallon*gallons consumed)-rebate amount= (1 ∗ 28.6907) − 35.36 = −6.669

Now we can use the EPA’s estimate for the amount(in metric tons) of carbon dioxide emitted per gallon of gasoline consumed(0.00889) to calculate the change in consumption emissions between no tax and tax with compensation:

Consumption difference between compensated tax and no tax= 28.6907 - 42.405 = -13.71 gal/mo

Monthly carbon emissions reduction(MTCO2/mo) from tax with compensation
= −13.71 ∗ 0.00889 = −0.121882

This means that for the tax and compensation to be welfare increasing(in that society benefits, which is hopefully what the government wants):

$$\frac{loss.tax.revenue}{carbon.emissions.reduction} < social.cost.carbon $$

This is the main welfare consideration since consumers are indifferent with com-
pensation.

The social cost of carbon is the net present value to society of future dam-
ages caused by 1 additional ton of carbon being emitted(via climate change). It
can also be reinterpreted as the benefit society would gain by emitting 1 metric
ton less CO2.

The main cost to consider is the loss of tax revenue by the government after imposing the tax.

I can then get the formula that if welfare is positive, given consumers indifferent and government in deficit, then: loss tax revenue < (amount reduction CO2 emissions in metric tons)∗ (reduction in cost to society of 1 fewer metric ton CO2 being emitted)

Therefore for the tax and compensation to be welfare enhancing, the social cost of carbon must be at or above:

$$




Appendix A:Deriving the OLS estimators analytically

I am trying to fit a linear model to the data. This means that I'm trying to predict a given level of G for a given level of P\_G\_R and I\_R. \\

This is done by formulating the relationship between a 22x1 vector of observations of the dependent variable $\vec{y}$ (monthly gas consumption for the 22 time periods measured in the dataset from 1984-2005) as a multiplication of a 22x3 matrix $\mathbf{X}$ (with the first column being all 1s to represent the constant term, and the second 2 columns filled with observations of the independent variables, P\_G\_R and I\_R) multiplied by a 3x1 vector of OLS estimators $\vec{\beta}$ and added to a 22x1 vector of errors for each prediction. 

This is of the form:

$$\vec{y} = \mathbf{X} \vec{\beta} + \vec{\epsilon}$$

Appendix B: Python code for regression and interactive 3d plot


{% highlight py %} 
import pandas as pd
import statsmodels.api as sm
import numpy as np
import plotly.graph_objects as go

# Load the SAS file
df = pd.read_sas(r'C:\Users\rossk\OneDrive\Desktop\SASdata.sas7bdat', format='sas7bdat')

# Transform annual gas consumption and nominal income data into monthly data
df['G'] = df['G_PC_AN'] / 12
df['I_N'] = df['I_N_AN'] / 12

# Divide the price of gas and nominal income by the CPI for all other goods
df['P_G_R'] = df['P_G_N'] / df['P_AOG']
df['I_R'] = df['I_N'] / df['P_AOG']

# Define the dependent variable and independent variables
Y = df['G']
X = df[['I_R', 'P_G_R']]

# Add a constant to the independent variables matrix
X = sm.add_constant(X)

# Fit the OLS model
model = sm.OLS(Y, X)
results = model.fit()

# Get the coefficients
coefficients = results.params
print(coefficients)

# Create a range of values for I_R and P_G_R
x_range = np.arange(df['I_R'].min(), df['I_R'].max(), (df['I_R'].max() - df['I_R'].min())/10)
y_range = np.arange(df['P_G_R'].min(), df['P_G_R'].max(), (df['P_G_R'].max() - df['P_G_R'].min())/10)

# Create a meshgrid for I_R_MO and P_G_R
x, y = np.meshgrid(x_range, y_range)

# Compute corresponding z values (G_PC_MO) for the plane
z = coefficients.const + coefficients.I_R*x + coefficients.P_G_R*y

# Create a 3D scatter plot
scatter = go.Scatter3d(x=df['I_R'], y=df['P_G_R'], z=df['G'], mode='markers', 
                        marker=dict(size=3, color='blue', opacity=0.5), name='Data')

# Create a surface plot for the plane
plane = go.Surface(x=x, y=y, z=z, colorscale=[[0, 'red'], [1, 'red']], opacity=0.3, name='Fit')

# Layout
layout = go.Layout(scene=dict(xaxis_title='Real Monthly Income', yaxis_title='Real Price of Gas', 
                              zaxis_title='Monthly Gas Consumption'), 
                   width=700, margin=dict(r=20, b=10, l=10, t=10))

# Figure
fig = go.Figure(data=[scatter, plane], layout=layout)

# Save the plot as an HTML file
fig.write_html("3d_plot.html")
{% endhighlight %}

Appendix B:


Appendix C: Considerations on improving this analysis

If you've made it this far, I'm open to hearing your feedback to improve on this analysis for future applications. Should I add any of the following?:
- An explanation of the standard errors and significance of the t values for the slopes/intercept in the explanation of the regression
- Welfare analysis for the uncompensated tax
- Consideration of other social costs avoided like congestion
- More discussion of Limitations of the analysis, i.e heterogeneous incomes and preferences, supply side effects, potential omitted variables in regression, problems of measuring utility, distributional effects, etc.
- Extension of this analysis to current year
- Extension of this analysis with different tax rates

Or would that be too much clutter? This already feels like a lot for an undergrad writing sample, but also many (most?)competitive applicants already have multiple coauthored papers/Phd level econ coursework/A's in real analysis and topology by my age so I think it needs to be extensive lol




