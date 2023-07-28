---
title: R gastax project
author: Ross Khelemsky
mathjax: true
date: June 2023
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

Then I run OLS multiple linear regression, where monthly gas consumption (G) is dependent on the real price of gas (P_G_R) and real monthly income (I_R). This fits a plane to the data which minimizes the sum of squared residual errors between the observed and predicted values of $\mathbf{G}$, of the functional form: 

$$\mathbf{G}=\gamma \mathbf{I_R} + \beta \mathbf{P_{GR}} + \alpha$$


{% highlight r %} 
fit <- lm(G ~ P_G_R + I_R, data = gastaxdata)
summary(fit)
{% endhighlight %}

γ, the slope estimate for I_R, is 0.01187. This means that
for a 1$ rise in real monthly income (with the real price of gas held constant),
monthly gas consumption is expected to increase by 0.01187 gal/mo.

β, the slope estimate for P_GR, is −14.1364. This means
that for a 1$ rise in the real price of gas (with real monthly income held con-
stant), monthly gas consumption is expected to decrease by 14.136 gal/mo.


α, the intercept estimate, is 39.09754. This means that if real incomes and
the real price of gas were 0$, per capita monthly gas consumption would be
39.09754 gal/mo. While it may not have a real-world interpretation(since the
real price of gas isn't going to ever be 0), it is still necessary
to compute to find the best fit line(or plane, in this instance).
The adjusted $R^2$ of 0.8979 implies that 89.79% of the variation in gas con-
sumption can be explained by changes in the real price of gas and real income.
Then I plot demand curves for gas(consumption as a function of price), at
different levels of real monthly income held constant:

The adjusted $R^{2}$ of 0.8979 implies that 89.79% of the variation in gas con-
sumption can be explained by changes in the real price of gas and real income


Now I use gglot to plot demand curves for gas (consumption as a function of price) at different levels of real monthly income held constant:

{% highlight r %} 
library(ggplot2)

p1 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() + 
      geom_abline(intercept = 4.4506, slope = -0.0707)

p1 + ggtitle("Estimated demand curve with I_R held constant at its mean value (2006.60)")
{% endhighlight %}


{% highlight r %} 
p2 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() +
      geom_abline(intercept = 4.3956, slope = -0.0707)

p2 + ggtitle("Estimated demand curve with I_R held constant at its median value (1941.01)")
{% endhighlight %}


{% highlight r %} 
p3 <- ggplot(gastaxdata, aes(x = G, y = P_G_R)) + geom_point() + 
      geom_abline(intercept = 4.8277, slope = -0.0707)

p3 + ggtitle("Estimated demand curve with I_R held constant at its 2005 value (2455.67)")
{% endhighlight %}

Also, we can rerun this regression in python and use plotly create an interactive 3d plot of the plane fit to the data. Code is shown in Appendix A.

[View 3D Plot](/3d_plot.html)

Now that I have the regression model, I can plug in the 2005 values for I_R and P_G_R (2455.67 and 1.828, respectively) to estimate the pre-tax consumption amount. Let's call this G_notax.

$$\mathbf{G_{notax}} = 39.09754 - (14.1364 * 1.828) + (2455.67 * 0.01187) = 42.405 gal/mo$$

The actual amount was 44.113, so the error is not very large for this estimate.

To get the consumption amount with a $1 tax increase (and no rebate), I plug in the same I_R and 2.828 for P_G_R. Let's call this amount G_tax_uncomp.



Appendix A: Python code for regression and interactive 3d plot


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

Appendix B: Considerations on improving this analysis

If you've made it this far, I'm open to hearing your feedback to improve on this analysis for future applications. Should I add any of the following?:
- An explanation of the standard errors and significance of the t values for the slopes/intercept in the explanation of the regression
- Welfare analysis for the uncompensated tax
- Consideration of other social costs avoided like congestion
- More discussion of Limitations of the analysis, supply side effects, potential omitted variables in regression, problems of measuring utility, distributional effects, etc.
- Extension of this analysis to current year
- Extension of this analysis with different tax rates

Or would that be too much clutter? This already feels like a lot for an undergrad writing sample, but also many (most?)competitive applicants already have multiple coauthored papers/Phd level econ coursework/A's in real analysis and topology by my age so I think it needs to be extensive lol




