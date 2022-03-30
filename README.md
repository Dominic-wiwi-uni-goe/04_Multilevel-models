# Chapter 4

## Multivariate regression and multilevel models

In the previous chaper we calculated estimates with simple linear regression models and multivariate regression models. However, these models do not include worker-specific effets. Therefore, we can create models that incorporate one regression line per worker (e.g., order picker or truck driver). These regression models range from a conventional fixed effects model to a fully random effects model:

![image](https://user-images.githubusercontent.com/102478331/160404934-f18e491b-6a8c-493f-9991-6f9128f86ba2.png)

## What we will learn in this chapter

In this chapter we will deal with mixed effects and random effects multilevel models that we apply to a dataset with professional truck drivers. They are working in the grocery retailing sector and drive one or more routes per day. The underlying vehicle routing problem (VRP) can be best described as a multiple trip VRP with time window.

### Loading the relevant packages in R

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

### Applicaiton of multilevel models: The mixed effects model with random intercepts and fixed slopes

In a first step, we run a mixed effects multilevel model that will allow to estimate worker-specific effects. Such a model is easily conducted in R, specifically with the package `lme4`. In the following, the code will look just like what you used for regression with lm, but with an additional component specifying the group, i.e. worker-specific effect. The `(1|Fahrer)` means that we are allowing the intercept, represented by 1, to vary by `Fahrer` which is the anonymous identification number of a professional truck driver in the given setting. 

(1) Insert the empirical dataset for your analysis and name it `dataset_final`. Then we will have a look at the first lines of the final dataset:

```
View(dataset_final)
```

(2) Define the dependent variable:

Our dependent variable is the duration time per route. The clock starts at the depot and ends after the delivery route at the deopt. The variable is named `Tourdauer_Ist` with is the real route duration from a track and trace system.

(3) Define the control variables :

We use various control variables in our multilevel model. The idea is to best describe what happens during a given route:

- `Anzahl_Stopps`: Defines the number of stops (equal to supermarket stores) on the respective route. 
- `GTE`: Measures the number of transport units transported on the respective route.
- `hour`: Relects a proxy variable for the traffic density as it operationalizes the starting time of the route.
- `Plan_Kap`: Defines the availible capacity of a truck. The comapny uses small (=30 capacity) and big (=60 capacity) trucks
- `KM`: The kilometers driven to fulfill the respective route.  


(4) Hereafter, we formulate a multilevel model by adding our depenent variables stepwise to the model

```
model1 <- lmer(Tourdauer_Ist ~  (1 | Fahrer), data = dataset_final)
model2 <- lmer(Tourdauer_Ist ~  Anzahl_Stopps + (1 | Fahrer), data = dataset_final)
model3 <- lmer(Tourdauer_Ist ~  Anzahl_Stopps + GTE + (1 | Fahrer), data = dataset_final)
model4 <- lmer(Tourdauer_Ist ~  Anzahl_Stopps + GTE + hour + (1 | Fahrer), data = dataset_final)
model5 <- lmer(Tourdauer_Ist ~  Anzahl_Stopps + GTE + hour + Plan_Kap + (1 | Fahrer), data = dataset_final)
model6 <- lmer(Tourdauer_Ist ~  Anzahl_Stopps + GTE + hour + Plan_Kap + KM + (1 | Fahrer), data = dataset_final)
```

(5) R-squared value

We can print the R-squared value by `r.squared`. Herein we find a good model fit where our `model6` can explain 84.4% of the total variance in our multi level model. When we only look at the first level describing the route, we find that 72.63% of the variance can be explained. This is already a hint that the level 2 variable (the professional truck driver) may be relevant. But there is another measure to prove this.

```
r.squaredGLMM(model6)
```

![image](https://user-images.githubusercontent.com/102478331/160665868-71773eb4-645f-4172-859a-6cc7e5b43653.png)


(6) Intra-class correlation coefficient 

The intra-class correlation coefficient (ICC) indicates how much level 1 variance can be explained by level 2 (as a percentage). For example, if the ICC is 0.118, we would say that 11.8% of the variation is between individual workers and 88.2% is within the workers. While the adjusted ICC only relates to the random effects, the conditional ICC also takes the fixed effects variances into account. Typically, the adjusted ICC is of interest when the analysis of random effects is of interest. We can calculate the ICC with

```
performance::icc(model6)
```

![image](https://user-images.githubusercontent.com/102478331/160666237-044ba68e-d5e1-487d-8e42-2fdfd49e8160.png)

We are lucky that it makes sense here: The R-squared of the level 1 (for the route iself) is 72.63 (R2m) and when we add the ICC of 11.8 which descirbed the variance chaused by level 2, we will end up with the total R-squared of the ultilevel-model (R2c) which is 84.44. 

Now, we are ready to look at the estimated.

(7) Summary of the final model

We can also print a summary of the model by

```
summary(model6)
```

![image](https://user-images.githubusercontent.com/102478331/160666497-989f4b60-0065-4f67-bc2c-fdccc170897f.png)

(8) For the classical regression tables we can use the following lines of code

```
htmlreg(list(model1, model2, model3, model4, model5, model6),
        custom.model.names = c("Nullmodel", "Model 1", "Model 2","Model 3", "Model 4", "Model 5", "Model 6"),          
        file="D:/raw data for science/03_truck loading datasets/05_BO_deep dive/Model_all1.html")
```
or

```
stargazer(model1, model2, model3, model4, model5, model6,
          type = "html", 
          digits=4, 
          title="", 
          order=c(""),
          dep.var.labels=c("Route duration"),
          no.space=TRUE,
          summary=TRUE,
          single.row=TRUE,
          rownames=TRUE,
          align=TRUE,
          out="D:/raw data for science/03_truck loading datasets/05_BO_deep dive/Model_all1b.html",
          flip = FALSE)
```

Giving as a regression table which looks more like what we know from scientific journals.

![image](https://user-images.githubusercontent.com/102478331/160666775-323293a5-4164-48f5-aa97-3663b3d02ad0.png)


### Applicaiton of multilevel models: The random effects model with random intercepts and random slopes

For this step, our dataset and variables are the same. However, we allow random effecnts on the variable `Anzahl_Stopps`. We formulate the following models and we (again) add stepwise all independent variables:

```
model20 <- lmer(Tourdauer_Ist ~ (Anzahl_Stopps | Fahrer), data = dataset_final)
model21 <- lmer(Tourdauer_Ist ~ Anzahl_Stopps + (Anzahl_Stopps | Fahrer), data = dataset_final)
model22 <- lmer(Tourdauer_Ist ~ Anzahl_Stopps + GTE + (Anzahl_Stopps | Fahrer), data = dataset_final)
model23 <- lmer(Tourdauer_Ist ~ Anzahl_Stopps + GTE + hour + (Anzahl_Stopps | Fahrer), data = dataset_final)
model24 <- lmer(Tourdauer_Ist ~ Anzahl_Stopps + GTE + hour + Plan_Kap + (Anzahl_Stopps | Fahrer), data = dataset_final)
model25 <- lmer(Tourdauer_Ist ~ Anzahl_Stopps + GTE + hour + Plan_Kap + KM + (Anzahl_Stopps | Fahrer), data = dataset_final)
```

And we print the final model:

```
htmlreg(list(model0, model20, model21, model22, model23, model24, model25),
        file="D:/raw data for science/03_truck loading datasets/05_BO_deep dive/Model_all3.html")

stargazer(model0, model20, model21, model22, model23, model24, model25,
          type = "html", 
          digits=4, 
          title="", 
          order=c(""),
          dep.var.labels=c("Survival object of picking time (sec.)"),
          no.space=TRUE,
          summary=TRUE,
          single.row=TRUE,
          rownames=TRUE,
          align=TRUE,
          out="D:/raw data for science/03_truck loading datasets/05_BO_deep dive/Model_all3b.html",
          flip = FALSE)
```

Giving us the following output:

![image](https://user-images.githubusercontent.com/102478331/160667319-08000357-dfae-49db-aad7-0c478060c7af.png)


When we are interested in the truck driver specific effects, we can also get the individual regression weights or model betas by

```
coef(model25)$Fahrer
```

Looking like this:

![image](https://user-images.githubusercontent.com/102478331/160667517-b7cf085b-3306-4c56-b647-a6eba960110c.png)

When we now want to plot all regression lines of the drivers in our random effects model (random intercepts AND random slopes), we use the following code lines:

```
coef(model14)$Fahrer
intercepts <- coef(model14)$Fahrer[,1]   # Specifying the first column only
slopes <- coef(model14)$Fahrer[,2]       # Specifying the second column only

geom_abline(slope=slopes, intercept=intercepts)

ggplot(dataset_final, aes(x=random.coefficients.predictions, y=Tourdauer_Ist, group=Fahrer))+
  stat_smooth(method="lm", se=FALSE, size=.5, color="red")
stat_smooth(aes(group=1), method="lm", color="blue", size=1.5)
```

We get this graph:

![image](https://user-images.githubusercontent.com/102478331/160667838-c161ec05-3776-47da-8469-5433fe1ded3f.png)



**Sources:** 
- https://m-clark.github.io/mixed-models-with-R/random_intercepts.html
- https://bookdown.org/steve_midway/DAR/random-effects.html
- https://rpubs.com/favstats/multilevel
