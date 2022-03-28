## 04 Multilevel models

In the previous chaper we calculated estimated with simle linear regression models and multivariate regression models. However, these models do not include worker-specific effets. Therefore, we can create models that incorporate one regression line per worker (e.g., order picker, truck driver). These regression models range from a conventional fixed effects model to a fully random effects model.

![image](https://user-images.githubusercontent.com/102478331/160404934-f18e491b-6a8c-493f-9991-6f9128f86ba2.png)

### Applicaiton of multilevel models

We run a mixed model that will allow for a worker specific effect. Such a model is easily conducted in R, specifically with the package `lme4`. In the following, the code will look just like what you used for regression with lm, but with an additional component specifying the group, i.e. worker-specific effect. The `(1|worker-ID)` means that we are allowing the intercept, represented by 1, to vary by worker. 

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
