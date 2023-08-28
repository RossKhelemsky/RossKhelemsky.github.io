---
title: 2005 Gas Tax Welfare Analysis
author: Ross Khelemsky
mathjax: true
date: June 2023
categories: media
---

## Abstract

I am going to analyze the proposed $1 increase in the gas tax in 2005 using R and time-series data on income, gas consumption, gas prices, and the prices of all other goods. I will use OLS multiple linear regression to estimate a relationship between gas consumption, real income, and the real price of gas (and find out how much gas consumption would decrease from a $1 increase in price, the effect of the tax). Then I will fit the data to the indirect utility function from Hausman (1981) to estimate the mean consumer's loss of utility from the tax. Then I will rearrange the function into the expenditure function to estimate how much is required in compensation for the consumer to be indifferent to the price change. After that, I will estimate the new consumption of gas after the tax and rebate. Then I will estimate the per capita tax revenue with and without compensation, and carbon emissions reduction with and without compensation. Finally, I will discuss the conditions under which the tax and rebate would be welfare-enhancing.(^1)(^2)

1. A pdf version of this analysis is available to download [here](https://github.com/RossKhelemsky/RossKhelemsky.github.io/files/12450562/2005_Gas_Tax_Welfare_Analysis.4.pdf)

2. Special thanks to Dr.Joseph Hughes, who taught me for intermediate micro, compiled the data for this analysis, and came up with a significant proportion of the methodology.


## Importing and transforming the data

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

## Fitting, interpreting, and visualizing the model

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


This vector of coefficients, 

$$\vec{\beta}= \begin{pmatrix}
    \beta_0\\
    \beta_1\\
    \beta_2
\end{pmatrix} = \begin{pmatrix}
    39.09754\\
    0.01187\\
    -14.1364
\end{pmatrix}$$ 

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

Also, we can rerun this regression in python and use plotly to create an interactive 3d plot of the plane fit to the data, which I have linked to and attached a picture of below: 

[Interact with the 3D plot shown below!](/3d_plot.html)

![Interactive 3D plot](https://github.com/niklasbuschmann/contrast/assets/137047194/d6736bba-b56f-4c2a-9d6c-7438f5edb5c7)
Code is shown in Appendix B.

## Calibrating the model to 2005 and estimating the consumer effects of the gas tax and the required compensation for the consumer to be indifferent

Now that I have the regression model, I can plug in the 2005 values for I_R and P_G_R (2455.67 and 1.828, respectively) to estimate the pre-tax consumption amount. Let's call this G_notax.

$$\mathbf{G_{notax}} = 39.09754 - (14.1364 * 1.828) + (2455.67 * 0.01187) = 42.405 gal/mo$$

The actual amount was 44.113, so the error is not very large for this estimate.

To get the consumption amount with a $1 tax increase (and no rebate), I plug in the same I_R and 2.828 for P_G_R. Let's call this amount G_taxuncomp.

$$ \mathbf{G_{taxuncomp}} = 39.09754 − (14.1364 ∗ 2.828) + (2455.67 ∗ 0.01187) = 28.2686
gal/mo $$


After that, I can estimate the change in consumer utility from the tax using the indirect utility function from JA Hausman’s 1981 paper(”Exact Consumer’s Surplus and Deadweight Loss”): The indirect utility function indexes the consumer's utility(happiness/pleasure/satisfaction) based on the real price of a good(P_G_R), real income(I_R), and the regression slopes and intercepts above for those variables. The higher(more positive) it is, the better off the consumer:
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

## Estimating the effect of the tax and compensation on tax revenue

To get G(monthly gas consumption per capita) after the tax and compensa-
tion, simply plug in 2.828 for P_G_R and 2491.23 for I_R back into the regression:

$$G_{taxcomp} = 39.09754 − (14.1364 ∗ 2.828) + (2491.23 ∗ 01187) = 28.6907gal/mo$$

Monthly tax revenue per capita without compensation is (tax amount per gallon * gallons consumed) = 1*28.2686 = 28.2686

Monthly tax revenue per capita with compensation is (tax amount per gallon*gallons consumed)-rebate amount= (1 ∗ 28.6907) − 35.36 = −6.669

## Welfare analysis using EPA estimates of carbon emissions from gasoline consumption

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

Therefore for the tax and compensation to be welfare enhancing, the social cost of carbon must be above:

$$Social.cost.carbon > \frac{-6.669}{-0.121882} = $54.71/MTCO2$$

which is actually not very far off from recent estimates of SCCO2: Wang et al 2019 conducted a meta-analysis on estimates of the social cost of carbon and found a mean estimate of \$54.70 per ton of CO2 emitted(Though I'm not sure if this is in real dollars or 2019 dollars. The 2005 tax $54.71 threshold is in real dollars). (Wang et al 2019)


## Appendix A: Deriving the OLS estimators analytically

I am trying to fit a linear model to the data. This means that I'm trying to predict a given level of G for a given level of P_G_R and I_R. 

This is done by formulating the relationship between a 22x1 vector of observations of the dependent variable y (monthly gas consumption for the 22 time periods measured in the dataset from 1984-2005) as a multiplication of a 22x3 matrix X (with the first column being all 1s to represent the constant term, and the second 2 columns filled with observations of the independent variables, P_G_R and I_R) multiplied by a 3x1 vector of OLS estimators β and added to a 22x1 vector of errors for each prediction. 

This is of the form:

$$\vec{y} = \mathbf{X} \vec{\beta} + \vec{\epsilon}$$

Where 

$$ \vec{y} =\begin{pmatrix}
    36.41\\
    36.95\\
    43.54\\
    43.29\\
    41.56\\
    43.25\\
    41.20\\
    43.97\\
    46.32\\
    48.51\\
    50.42\\
    49.37\\
    47.79\\
    49.38\\
    55.96\\
    53.99\\
    47.52\\
    49.03\\
    50.43\\
    48.58\\
    45.90\\
    44.11
\end{pmatrix} \mathbf{X} = \begin{pmatrix}
    1 , 1681.08 , 1.75\\
    1 , 1742.29 , 1.68\\
    1 , 1771.71 , 1.12\\
    1 , 1803.62 , 1.15\\
    1 , 1852.83 , 1.17\\
    1 , 1880.23 , 1.19\\
    1 , 1886.81 , 1.32\\
    1 , 1847.62 , 1.14\\
    1 , 1877.33 , 1.08\\
    1 , 1899.13 , 1.00\\
    1 , 1934.56 , 0.94\\
    1 , 1947.46 , 0.95\\
    1 , 1978.68 , 1.03\\
    1 , 2013.85 , 0.99\\
    1 , 2062.86 , 0.77\\
    1 , 2135.40 , 0.87\\
    1 , 2212.29 , 1.21\\
    1 , 2230.91 , 1.11\\
    1 , 2249.71 , 1.00\\
    1 , 2301.05 , 1.18\\
    1 , 2380.18 , 1.45\\
    1 , 2455.67 , 1.83
\end{pmatrix}$$

and while I know the squared-error minimizing β to be 

$$\begin{pmatrix}
    \beta_0\\
    \beta_1\\
    \beta_2
\end{pmatrix} = \begin{pmatrix}
    39.09754\\
    0.01187\\
    -14.1364
\end{pmatrix}$$

the point of this appendix is to derive this vector analytically, knowing only the observations.

So, assuming I don't already know β, to get the best fit plane and derive the estimator vector, I minimize the sum of squared errors, S. Remembering that ϵ is a 22x1 vector of the errors for any given fit, the objective function is thus:


$$ S = (\vec{\epsilon})^{T} \vec{\epsilon} $$

$$=(\vec{y} - \mathbf{X}\vec{\beta})^{T} (\vec{y} - \mathbf{X}\vec{\beta})$$

$$=((\vec{y})^{T} - (\vec{\beta})^{T} \mathbf{X}^{T}) (\vec{y} - \mathbf{X}\vec{\beta})$$

$$=(\vec{y})^{T} \vec{y} - (\vec{y})^{T}\mathbf{X}\vec{\beta} - (\vec{\beta})^{T} \mathbf{X}^{T}\vec{y} + (\vec{\beta})^{T} \mathbf{X}^{T} \mathbf{X}\vec{\beta}  $$

 Before I minimize this sum, I need to establish some matrix calculus definitions:
 
 For an nx1 vector of variables x, and an nxm matrix \textbf{A} with entries that do not depend on x,
if:

$$\mathbf{A}\vec{x} = \vec{y} \to \frac{\partial \vec{y}}{\partial \vec{x}} = \mathbf{A}^{T}$$

if:
 
 $$\vec{x}^{T}\mathbf{A} = \vec{y} \to \frac{\partial \vec{y}}{\partial \vec{x}} = \mathbf{A}$$
 
if \textbf{A} is symmetric, which the matrix we will be working with is (as shown below), and:

$$\vec{x}^{T}\mathbf{A} \vec{x} = \vec{y} \to \frac{\partial \vec{y}}{\partial \vec{x}} = 2\mathbf{A}\vec{x}$$

To minimize the sum, I set the partial derivative of S with respect to β equal to zero.

$$\frac{\partial S}{\partial \vec{\beta}} = 0 - ((\vec{y})^{T}\mathbf{X})^{T} - \mathbf{X}^{T}\vec{y} + 2 \mathbf{X}^{T}\mathbf{X} \vec{\beta} = 0$$

$$-2\mathbf{X}^{T}\vec{y} + 2 \mathbf{X}^{T}\mathbf{X} \vec{\beta}=0$$

$$ \mathbf{X}^{T} \mathbf{X} \vec{\beta} = \mathbf{X}^{T}\vec{y}$$

$$\vec{\beta} = (\mathbf{X}^{T}\mathbf{X})^{-1} \mathbf{X}^{T}\vec{y} $$

Plugging X and y back in, I get:

$$\vec{\beta} = (\begin{pmatrix}
    1 , 1681.08 , 1.75\\
    1 , 1742.29 , 1.68\\
    1 , 1771.71 , 1.12\\
    1 , 1803.62 , 1.15\\
    1 , 1852.83 , 1.17\\
    1 , 1880.23 , 1.19\\
    1 , 1886.81 , 1.32\\
    1 , 1847.62 , 1.14\\
    1 , 1877.33 , 1.08\\
    1 , 1899.13 , 1.00\\
    1 , 1934.56 , 0.94\\
    1 , 1947.46 , 0.95\\
    1 , 1978.68 , 1.03\\
    1 , 2013.85 , 0.99\\
    1 , 2062.86 , 0.77\\
    1 , 2135.40 , 0.87\\
    1 , 2212.29 , 1.21\\
    1 , 2230.91 , 1.11\\
    1 , 2249.71 , 1.00\\
    1 , 2301.05 , 1.18\\
    1 , 2380.18 , 1.45\\
    1 , 2455.67 , 1.83
\end{pmatrix}^{T} \begin{pmatrix}
    1 , 1681.08 , 1.75\\
    1 , 1742.29 , 1.68\\
    1 , 1771.71 , 1.12\\
    1 , 1803.62 , 1.15\\
    1 , 1852.83 , 1.17\\
    1 , 1880.23 , 1.19\\
    1 , 1886.81 , 1.32\\
    1 , 1847.62 , 1.14\\
    1 , 1877.33 , 1.08\\
    1 , 1899.13 , 1.00\\
    1 , 1934.56 , 0.94\\
    1 , 1947.46 , 0.95\\
    1 , 1978.68 , 1.03\\
    1 , 2013.85 , 0.99\\
    1 , 2062.86 , 0.77\\
    1 , 2135.40 , 0.87\\
    1 , 2212.29 , 1.21\\
    1 , 2230.91 , 1.11\\
    1 , 2249.71 , 1.00\\
    1 , 2301.05 , 1.18\\
    1 , 2380.18 , 1.45\\
    1 , 2455.67 , 1.83
\end{pmatrix})^{-1} \begin{pmatrix}
    1 , 1681.08 , 1.75\\
    1 , 1742.29 , 1.68\\
    1 , 1771.71 , 1.12\\
    1 , 1803.62 , 1.15\\
    1 , 1852.83 , 1.17\\
    1 , 1880.23 , 1.19\\
    1 , 1886.81 , 1.32\\
    1 , 1847.62 , 1.14\\
    1 , 1877.33 , 1.08\\
    1 , 1899.13 , 1.00\\
    1 , 1934.56 , 0.94\\
    1 , 1947.46 , 0.95\\
    1 , 1978.68 , 1.03\\
    1 , 2013.85 , 0.99\\
    1 , 2062.86 , 0.77\\
    1 , 2135.40 , 0.87\\
    1 , 2212.29 , 1.21\\
    1 , 2230.91 , 1.11\\
    1 , 2249.71 , 1.00\\
    1 , 2301.05 , 1.18\\
    1 , 2380.18 , 1.45\\
    1 , 2455.67 , 1.83
\end{pmatrix}^{T} \begin{pmatrix}
    36.41\\
    36.95\\
    43.54\\
    43.29\\
    41.56\\
    43.25\\
    41.20\\
    43.97\\
    46.32\\
    48.51\\
    50.42\\
    49.37\\
    47.79\\
    49.38\\
    55.96\\
    53.99\\
    47.52\\
    49.03\\
    50.43\\
    48.58\\
    45.90\\
    44.11
\end{pmatrix}$$

$$= \begin{pmatrix}
    22, 44145.27, 25.94 \\
    44145.27, 89570662.40, 52080.10 \\
    25.94, 52080.10, 32.20 \\
\end{pmatrix}^{-1} \begin{pmatrix}
   1017.50 \\
   2052968.20 \\
   1177.15 \\
\end{pmatrix}$$

$$= \begin{pmatrix}
    4.879879, -0.002006, -0.686597 \\
    -0.002006, 0.000001, -0.000021 \\
    -0.686597, -0.000021, 0.618954 \\
\end{pmatrix} \begin{pmatrix}
    1017.50 \\
    2052968.20 \\
    1177.15 \\
\end{pmatrix}
$$

$$= \begin{pmatrix}
    39.0976\\
    0.01187\\
    -14.1364
\end{pmatrix}$$

which is very close to what I got originally!

## Appendix B: Python code for regression and interactive 3d plot


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

## Appendix C: Deriving Expenditure function from Indirect Utility

I'm going to isolate I\_R in the formula for indirect utility(consumer satisfaction as a function of real income and the real price of gas) to get the expenditure function(the amount of income needed to reach a certain level of utility, given a utility function and prices):

$$ U= e^{-\beta_1 * P\_G\_R} * (I\_R + (\frac{1}{\beta_1}* (\beta_2*P\_G\_R+\frac{\beta_2}{\beta_1}+\beta_0))) $$


$$ U* e^{\beta_1 * P\_G\_R} = I\_R + (\frac{1}{\beta_1}* (\beta_2*P\_G\_R+\frac{\beta_2}{\beta_1}+\beta_0)) $$


$$ \mathbf{Expend(P\_G\_R, U)} = \mathbf{I_R} =  (U * e^{\beta_1 * P\_G\_R}) -(\frac{1}{\beta_1} * (\beta_2 * P_{GR} + \frac{\beta_2}{\beta_1} + \beta_0))$$

## Appendix D: Considerations on improving this analysis

If you've made it this far, I'm open to hearing your feedback to improve on this analysis for future applications. Should I add any of the following?:
- An explanation of the standard errors and significance of the t values for the slopes/intercept in the explanation of the regression
- Welfare analysis for the uncompensated tax
- Consideration of other social costs avoided like congestion
- More discussion of Limitations of the analysis, i.e heterogeneous incomes and preferences, supply side effects, potential omitted variables in regression, problems of measuring utility, distributional effects, etc.
- Extension of this analysis to current year
- Extension of this analysis with different tax rates

Or would that be too much clutter? This already feels like a lot for an undergrad writing sample, but also many (most?)competitive applicants already have multiple coauthored papers/Phd level econ coursework/A's in real analysis and topology by my age so I think it needs to be extensive lol

##References:

Hausman 1981->
Hausman, J. A. (1981). Exact Consumer’s Surplus and Deadweight Loss. The American Economic Review, 71(4), 662–676. Retrieved from

\url{http://www.jstor.org/stable/1806188}

\url{http://www.econ.uiuc.edu/~econ536/Papers/hausman81.pdf}


EPA->
Environmental Protection Agency. (2023). Greenhouse Gases Equivalencies Calculator - Calculations and References. EPA. Retrieved from

\url{https://www.epa.gov/energy/greenhouse-gases-equivalencies-calculator-calculations-and-references}


Wang et al 2019 ->
Wang, P., Deng, X., Zhou, H., \& Yu, S. (2019). Estimates of the Social Cost of carbon: A review based on meta-analysis. Journal of Cleaner Production, 209, 1494–1507. Retrieved from

\url{https://doi.org/10.1016/j.jclepro.2018.11.058}

\url{https://www.sciencedirect.com/science/article/pii/S0959652618334589?casa_token=r_RXW-1Yus4AAAAA:aIC8-lXSao5_RcTnmEkhNopUC-3c5BBWoDBY7pEdNdBpv7ReI5fOL_jwuaXZ3VRkYHcwe4OK7uc}



