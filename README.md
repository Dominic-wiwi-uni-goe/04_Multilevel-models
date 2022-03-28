## 04 Multilevel models

In the previous chaper we calculated estimated with simple linear regression models and multivariate regression models. However, these models do not include worker-specific effets. Therefore, we can create models that incorporate one regression line per worker (e.g., order picker or truck driver). These regression models range from a conventional fixed effects model to a fully random effects model:

![image](https://user-images.githubusercontent.com/102478331/160404934-f18e491b-6a8c-493f-9991-6f9128f86ba2.png)

### Applicaiton of multilevel models: The mixed effects model with random intercepts and fixed slopes

In a first step, we run a mixed model that will allow for a worker specific effect. Such a model is easily conducted in R, specifically with the package `lme4`. In the following, the code will look just like what you used for regression with lm, but with an additional component specifying the group, i.e. worker-specific effect. The `(1|Picker)` means that we are allowing the intercept, represented by 1, to vary by `MDENR` which is the anonymoud identification number of a picker. 

(1) Load the relevant packages
```
library("sjPlot")
library("sjmisc")
library("sjstats")
library("stats")
library("lme4")
library("haven")
library("MuMIn")
library("devtools")
library("corrplot")
library("data.table")
library("eha")
library("flexsurv")
library("fs")
library("ggplot2")
library("ggpubr")
library("JM")
library("lattice")
library("PerformanceAnalytics")
library("plyr")
library("Rcpp")
library("readxl")
library("reshape2")
library("rms")
library("splines")
library("stargazer")
library("survival")
library("survminer")
library("tidyverse")
library("mosaicData")
library("dplyr")
library("treemapify")
library("scales")
library("ggcorrplot")
library("factoextra")
library("superheat")
library("texreg")
```

(2) Insert the empirical dataset for your analysis and name it `dataset`:
```
dataset <- BON1
dataset
```

(3) For our model, we try to explain the dependent variable `picktime` by several independent variables:
- `picktime` = Time per pick in seconds
- `ANZ_PICK` = Number of picks in stock keepting units (SKUs)
- `level` = Level of pick where 0=ground level and 1=chest level
- `weight` = Weight of a SKU picked in kilograms
- `distance` = Travel distance of picker from pick i to i + 1
- `packes_SKU` = Primary packages in one SKU (= secondary package)
- `volume` = Volume of a SKU in litres
- `AUFTRAGSPOS` = The position within the batch for a given pick
- `MDENR` = The anonymous identification number of the order pickers

(4) Hereafter, we formulate a multilevel model by adding our depenent variables stepwise to the model
```
multilevel01 <- lmer(picktime  ~  ANZ_PICK  + (1 | MDENR ), data = dataset)
multilevel02 <- lmer(picktime  ~  ANZ_PICK  + level + (1 | MDENR ), data = dataset)
multilevel03 <- lmer(picktime  ~  ANZ_PICK  + level + weight + (1 | MDENR ), data = dataset)
multilevel04 <- lmer(picktime  ~  ANZ_PICK  + level + weight + distance + (1 | MDENR ), data = dataset)
multilevel05 <- lmer(picktime  ~  ANZ_PICK  + level + weight + distance + packes_SKU + (1 | MDENR ), data = dataset)
multilevel06 <- lmer(picktime  ~  ANZ_PICK  + level + weight + distance + packes_SKU + volume + (1 | MDENR ), data = dataset)                     
multilevel07 <- lmer(picktime  ~  ANZ_PICK  + level + weight + distance + packes_SKU + volume + AUFTRAGSPOS + (1 | MDENR ), data = dataset)
```

(5) Summary of the final model

We can print a summary of the final model with the `summary` function.

```
summary(multilevel07)
```

Here, we receive the following output.

![image](https://user-images.githubusercontent.com/102478331/160454773-eb465ee4-f833-4c0c-8e01-a320656a2607.png)

(6) For a regression table with estimated, p-values, and model fit measures (AIC, BIC) we use the `htmlreg` function

```
htmlreg(list(multilevel01, multilevel02, multilevel03, multilevel04, multilevel05, multilevel06, multilevel07),
        custom.model.names = c("Model 1", "Model 2","Model 3", "Model 4", "Model 5", "Model 6", "Model 7"),  
        custom.coef.names = c("intercept", "number of picks", "level", "weight", "distance", "packes per SKU", "volume", "batch position"), 
        file="C:/Users/domin/Desktop/Reference_Model.html")
```

![image](https://user-images.githubusercontent.com/102478331/160453696-b62d361d-b658-42c7-94df-293676cd0496.png)

(7) Intra-class correlation coefficient 

The intra-class correlation coefficient (ICC) indicates how much level 1 variance can be explained by level 2 (as a percentage). For example, if the ICC is 0.2, we would say that 20% of the variation is between individual workers and 80% is within the workers. We can calculate the ICC with.

```
icc(multilevel07)
```

The output in R is: 

![image](https://user-images.githubusercontent.com/102478331/160455406-da3cd151-86cc-4ae7-8acc-6171455c442a.png)


### Applicaiton of multilevel models: The random effects model with random intercepts and random slopes












### Calculating worker-specific effects: Individual estimates of the random effects

After running the model, we can actually get estimates of the worker-specific effects. I show two ways for the first five students, both as random effect and as random intercept (i.e. intercept + random effect).

```
ranef(gpa_mixed)$student %>% head(5)
```

```
coef(gpa_mixed)$student %>% head(5)
```




**Sources:** 
- https://m-clark.github.io/mixed-models-with-R/random_intercepts.html
- https://bookdown.org/steve_midway/DAR/random-effects.html
- https://rpubs.com/favstats/multilevel
