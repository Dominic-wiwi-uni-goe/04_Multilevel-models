## 04 Multilevel models

We run a mixed model that will allow for a worker specific effect. Such a model is easily conducted in R, specifically with the package `lme4`. In the following, the code will look just like what you used for regression with lm, but with an additional component specifying the group, i.e. worker-specific effect. The `(1|worker-ID)` means that we are allowing the intercept, represented by 1, to vary by worker. 

#### Estimates of the random effects
After running the model, we can actually get estimates of the worker-specific effects. I show two ways for the first five students, both as random effect and as random intercept (i.e. intercept + random effect).

```
ranef(gpa_mixed)$student %>% head(5)
```

```
coef(gpa_mixed)$student %>% head(5)
```

# showing mixedup::extract_random_effects(gpa_mixed)


**Source:** 
- https://m-clark.github.io/mixed-models-with-R/random_intercepts.html
- 
