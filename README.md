# Glassnode - Data science challenge

In this repository, I am presenting my approach to solve the Glassnode data science challenge, which detailed description can be found here: https://github.com/glassnode/code-challenges/blob/master/data-scientist-challenge.md.
First I will give a brief reminder on the problem at hand. I will then prensent the proposed solution, uncomplete, along with the reasoning and assumptions behind.

## Description of the problem

The aim of this exercise is to build an application able to predict future price movements of a given cryptocurrency, up to the next 6 hours. From the wording, it seems that an "up" or "down" output is expected, however I am not sure this is the spirit of this exercise. Therefore I am taking the liberty to introduce the amplitude of the movement as well. Solving this problem can be done in two steps.

* It is important to determine how to model accurately the evolution of the price for this asset. Therefore, the first task is to retrieve the historical time series for the chosen cryptocurrency, as well as any other kind of data that may influence its price, analyse the retrieved data, and build a model that fits the time series. The model should be able to predict the 1 hour and 6 hour movements. To determine whether the chosen model is accurate, it is important to test it as well, by calibrating it on a given time window, then running the predictions outside of this window and comparing them to actual realisations.

* Once a model is chosen, it can be integrated in the final application. The model may need to be calibrated for each run or may not. Depending on that the calibration can be integrated or separated, depending on the model choice.

## Part 1 - model building

Although there is total liberty around the choice of the cryptocurrency, I decided to focus on the very popular bitcoin. There are 2 reasons behind this choice:
- BTC being widely known, developing an application around it might serve more users than an application around less known cryptocurrencies;
- The high liquidity of BTC makes the data more sensible and more reliable than less liquid currencies for which new investors have a higher impact on the prices.

Due to time constraint, decision was made to build a model focus only on the historical open prices (the close price for last hour is adjusted in real time). Most certainly, it could be beneficial to explore the relation between the open price and other factors (such as volume traded, high price, low price...), but this analysis would require more time.

### Data retrieval

Data is retrieved via the **get_hourly_series** function, taking as parameters the cryptocurrency, the currency to convert into (typically USD), the number of data points to retrieve and the cutoff timestamp. The definition of a function for data retrieval allows more flexibility which can be useful also for future requests. For the purpose of this exercise, I applied the function to retrieve BTC prices in USD for the last 5,000 hours.

### Data analysis

As a first step, a simple plot of the evolution of the BTC prices with time may give an idea on what class of model may fit. Although it does not seem like stationarity conditions are satisfied, practical considerations such as lack of knowledge about more complicated models and limited time to dive into research force me to go for simple autoregressive model. However, instead of modelling the evolution of the price directly, I chose as base process to model the natural logarithm of the price. This choice is justified by the will of keeping the prices positive, whatever are the regression coefficient and noise introduced in the model.

### AR model calibration and testing

For the calibration and testing of the model, I used the python package **statsmodels**. I compared predictions using the AR model against the realisation on 100 points for the 1h prediction, 95 points for the 6h. To do so, calibration and predictions were done 100 times. This model is completely deterministic, some stochasticity can be added via a noise term (gaussian or taking the empirical distribution of the error on the training data for example). 

The order of the AR model determined by the **fit** method coming from the **statsmodels** package is around 30. However, fitting an AR(1) model gives comparable results. This is excpected as the prices are closely related to recent history. For this exercise I used the same model to generate 1h and 6h predictions. In a general case, different models may be used for different time steps. 

As expected, the 6 hours predictions are much less accurate that the 1h predictions. There are two reasons behind it:
- the further away, the more uncertainty there is and errors are cumulating;
- lower reactivity (a jump in the stock price will create important error for the following 5 predictions as well).


## Part 2 - Dockerised API

This part is untouched.


## Conclusion

Predicting cryptocurrency price movements is a challenging problem. Machine learning may be a a powerful approach and there are numerous librairies to do so. However, I prefered to focus on a simple approach that I understand rather than playing with complex built-in functions for models I do not know the details (context, performance, hypothesis...). This lead me to use simple autoregressive models, not very well performing for this exercise. Indeed, prices seem quite close between predictions and realisations, however we are more interested about the variations, which are inaccurate.

Here I decided to consider only the time series of historical prices to build the model, however, different data can be used in complement or replacement. This would require time to explore the connection between them. But we notice by simple plots a correlation between various cryptocurrencies. It could be an element to explore further, as activity of the whole crypto world may translate, earlier or later, on a specific crypto currency.
