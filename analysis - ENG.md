# Quantile regression analysis

## Introduction ##

**Research objective**: To determine how different diamond characteristics affect diamond prices in various distribution quantiles, thereby assessing the differences between cheaper and more expensive diamonds.

Research objectives:
1. Perform initial data analysis and present descriptive statistics.
2. After selecting appropriate regressors, construct a quantile regression model and examine the characteristics that influence diamond price growth.
3. To evaluate the accuracy of the constructed model.
4. To present the results and final conclusions.

## Initial data analysis ##

The analysis used the R package, *ggplot2* library data set *diamonds*. Missing N\A and zero values were checked and the distribution of the dependent variable *price* was observed. Based on the graph below, an asymmetrical price distribution can be observed. It can be said that the majority of the data set consists of cheaper diamonds. The data is not normally distributed, but quantile regression analysis is resistant to non-normality, so the data is well suited for further investigation.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/cedb97e9-f8b2-481a-9768-2651511bcd41" />

The scatter plot below shows zero values for variables X, Y, and Z. These are likely to be erroneous and will have a significant impact on our model, so we will not include these variables in the quantile regression model. It can also be seen that as the weight of carats increases, so does the dispersion of prices. This indicates that the scatter plots presented are dominated by non-linear relationships.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/9a46c52a-6abc-4678-b25c-355e124a881d" />

The distribution of categorical variable values was observed using rectangular diagrams. The diagrams show that there are many outliers in the data, so it was decided to apply quantile regression, as it is not sensitive to outliers.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/921cb032-85f8-4062-880f-3eaa238380b6" />

## Checking assumptions ##

After performing the initial analysis, the next step is to check the assumptions. In quantile regression analysis, unlike other types of analysis, there is no need to check for normality or heteroscedasticity, as it does not use any assumptions about the distribution of errors or their uniform dispersion. Therefore, even if the residuals:

- are not normal,
- their dispersion varies (heteroscedasticity),

quantile regression still works. However, it is necessary to check whether the variables are correlated with each other. A correlation matrix and VIF values are used to identify multicollinearity.

### Correlation matrix ###

The correlation matrix shows that the variables *carat*, *X*, *Y*, and *Z* are most strongly correlated with each other. Including the variables *X*, *Y*, and *Z* in the model may lead to multicollinearity. This is another reason why this type of data is not included when constructing a quantile regression model.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/2909c3b9-83ca-4046-b981-c43fe36ca261" />

### VIF values ###

Another multicollinearity check was performed using VIF values. VIF shows how much the standard error of the coefficient of the created model increases due to the correlation between variables. The theory states that if VIF > 4, then there is a relationship between the variables and it is likely that multicollinearity may cause problems. In this case, the problem should be solved by adding new variables or removing inappropriate ones.

The table below shows the results obtained for the VIF values.

| Variables  | VIF |
| ------------- | ------------ - |
| carat   | 1.32  |
| depth  | 1.38  |
| table  | 1.79  |
| color  | 1.17  |
| cut  | 1.93  |
| clarity  | 1.30  |

All VIF values obtained are < 4, indicating a low correlation. The variables can be used in the model.

## Quantile regression ##
> [!NOTE]
> The data set was divided into training and testing sets in a 90/10 ratio, where 90% of the data was used for model training and the remaining 10% for testing.
> The value τ (tau) is the quantile level, which is between 0 and 1.

During the analysis, τ values ranging from 0.1, 0.2, ..., 0.9 were used. An empty vector was created to store the MSE metric values. The length of this vector corresponds to the number of τ values (since the mean square error (MSE) results for each τ were recorded in this vector).
```
taus <- seq(0.1, 0.9, by = 0.1)
mse_values <- numeric(length(taus))
```
Next, a cycle is created that goes through each τ element. During the cycle, quantile regression models are also created, which are trained with the training set data. After the model was created, the response value was predicted (using test data) and the mean square error was calculated, the value of which was recorded in the previously created vector.
```
for (i in seq_along(taus)) {
  tau <- taus[i]
  model <- rq(price ~ carat + depth + table + color + cut + clarity, tau = tau, data = train_data)
  preds <- predict(model, newdata = test_data)
  mse_values[i] <- mean((test_data$price - preds)^2)
}
```

The mathematical response and covariance notation, as well as the structure of the quantile regression model, which was constructed based on the formulas in the theoretical description, are presented in the figure below. Here, *y* is the response, *x* is the vector of covariance and intercept (free member) vector components.
> [!NOTE]
> The intercept ensures that even if all other variables are equal to zero, the model still has a constant initial value and a constant offset.

<img width="614" height="256" alt="Image" src="https://github.com/user-attachments/assets/fa3ddca5-8e2d-4c6d-b67b-31d10dfbc6f1" />

Next, we observed how accurately the model makes predictions. The model was evaluated using the MSE (mean squared error) metric, which shows the average difference between the actual and predicted values. Based on the results of the graph, it can be seen that the smallest error was obtained when τ = 0.6, which indicates that the predicted 60th percentile quantile provided the smallest forecast dispersion compared to the actual prices in the test data. In this quantile, the forecasts best minimize the square error. 

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/4d4d10d2-ec5c-4d07-a1cb-4a1b31966c2b" />

It is also important to check whether the model's predicted quantiles correspond to the actual data distribution.
The black line represents the empirical probability, which is calculated using test data.
- The red line shows the perfect match between the model's predicted quantiles and the theoretical quantiles. In other words, the empirical probability completely matches the theoretical probability (what should be in theory is what is in practice).

The first graph shows that both lines are very close to each other. This is a good sign, indicating that the empirical probabilities almost coincide with the theoretical ones and that the selected quantile regression performs stable predictions.

When analyzing the graph of the differences between empirical and theoretical probabilities, it is also clear that the difference between the probabilities is extremely small, and at τ = 0.8, the empirical probability matches the theoretical probability.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/ed41c1d0-03bc-4059-bac2-246cb347b591" />

Based on theoretical mathematical expressions, the average quantile loss function was calculated, which essentially shows how inaccurate the quantile 𝜏 forecasts are on average.

The lower the value, the better the model reflects the selected quantile.

<img width="612" height="667" alt="Image" src="https://github.com/user-attachments/assets/684641ef-32f5-45f0-a536-bd6c345af320" />

The graph of the quantile loss function shows that the smallest loss is observed when τ = 0.1, τ = 0.2, and τ = 0.9. Therefore, it can be said that the model is best at predicting cheaper and more expensive diamonds, and most often errs in predicting medium-priced diamonds at the τ = 0.6 quantile.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/04f90c1c-405d-48f7-9aa1-855818ae0107" />

The following graph shows a comparison of quantile regression with OLS (Ordinary Least Squares).
The points show the distribution of actual data, with *x* representing diamond carats and *y* representing price.

- $${\color{orange}OLS \space \color{orange}regression}$$ - predicts the average price based on *carat*.
- $${\color{red}0.1 \space \color{red}quantile}$$ - shows the price behavior in the group of cheaper diamonds.
- $${\color{limegreen}0.6 \space \color{limegreen}quantile}$$ - shows the price behavior in the group of diamonds that are slightly more expensive than average.
- $${\color{darkorchid}0.9 \space \color{darkorchid}quantile}$$ - shows the price of the most expensive diamonds per carat.

Since quantiles model different levels of price distribution, it can be said that $${\color {red}0.1 \space \color{red}quantile}$$ separates diamonds so that approximately 10% of data points (diamonds) are below this line and about 90% are above it. Diamonds below the line are among the 10% cheapest (compared to others with similar carats). Above the line are diamonds that are more expensive than the 10% cheapest, i.e., they fall into the more expensive 90% group.

$${\color{limegreen}0.6 \space \color{limegreen}quantile}$$ separates the data at a ratio of 60:40. Below this line, it is likely that 60% of diamonds are cheaper and have similar carat characteristics. Analyzing the graph, we can see that at 1 carat, the model expects that 60% of diamonds cost no more than $5,000, and 60% of 2-carat diamonds cost no more than $125,000.

> [!IMPORTANT]
> The density of points shown in the graph is NOT directly proportional to the quantile. That is, although the model mathematically states that "60% of observations are likely to be below the line," this does not mean that we will visually see 60% of the points at the bottom.

It is clear that the prices of cheap, average, and expensive diamonds depend on their weight, as the quantile lines gradually rise. It can be said that as the weight of diamonds increases, so does their price.

Diamonds weighing 1 carat or more have a wider range of prices. It can be seen that the data is more scattered, which means that weight no longer unambiguously determines the price. Perhaps other quality characteristics such as color, transparency, or cut quality have a greater influence. This may also be influenced by other external factors (perhaps the market offers many different options in this weight range).

The OLS regression line also rises sharply. It can be said that as the carat increases, the average price also increases, and the relationship between these variables is positive and strong.

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/84bc984b-09ba-41ec-8905-9c289f25e20d" />

The following graph shows the relationship between the percentage depth of a diamond and its price. There is a noticeable negative effect of depth at the lowest ($${\color{red}0.1}$$) and highest ($${\color {darkorchid}0.9}$$) quantiles, which suggests that in the case of the most expensive and cheapest diamonds, greater depth can be considered a disadvantage that negatively affects the price. Since the $${\color{darkorchid}0.9 \space \color{darkorchid}quantile}$$ experiences a steeper slope in the graph, the most expensive diamonds are the most affected by price changes. Excessive depth is considered undesirable in terms of proportions and compromises aesthetics, which may be one of the reasons why products are valued lower than usual.

There is also a large vertical spread of data, which is concentrated between approximately 58-65 percent depth. Based on this information, it is assumed that the price of diamonds of the same depth can vary greatly, ranging from a few hundred to tens of thousands of dollars. Therefore, the percentage of diamond depth is not a decisive factor and one of the main factors that significantly affect the price of diamonds. 

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/aa4438c6-0bb4-413e-86e1-11f686b921f3" />

Another graph showing the analysis of the variable *table* (the width of the top of the diamond relative to its widest point) and price. As in the previous graph, there is a wide vertical spread, suggesting that the prices of diamonds of similar width can vary significantly. Diamonds with width values below 50 are quite rare and can be considered exceptions. The variable *table* is not strongly correlated with the price of a diamond and is not sufficient on its own to explain the price. For more expensive diamonds ($${\color{darkorchid}0.9 \space \color{darkorchid}quantile}$$ and $${\color{limegreen}0.6 \space \color{limegreen}quantile}$$ lines), a larger diamond top width may be associated with a higher price, but the effect is still not strong. 

In the case of classic linear regression, the $${\color{orange}OLS \space \color{orange}line}$$ rises, indicating that changes in *table* have a very weak positive effect on the average price of diamonds. However, due to the large vertical dispersion, it is said that $${\color{orange}OLS}$$ is not very informative in this case. 

<img width="963" height="708" alt="Image" src="https://github.com/user-attachments/assets/d937298e-fcce-4bcc-9d7f-efd011c78eb0" />

## Conclusions ##

- Diamonds weighing 1 carat or more have significantly more varied prices
- There is a noticeable wide range of prices for diamonds weighing 1 carat or more. This shows that buyers value not only weight, but also many other characteristics, which determines the wide range of prices.
- For the most expensive and cheapest diamonds, greater depth reduces the value of the product.
- Diamond weight is the main factor influencing price changes.
- Wider diamonds (in the expensive category) are more highly valued than narrower gemstones.
