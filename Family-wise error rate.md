---
title: "Family-wise error rate"
teaching: 10
exercises: 2
---

:::::::::::::::::::::::::::::::::::::: questions

- What is the family-wise error rate (FWER), and why is it important in multiple testing scenarios?
- How does the Bonferroni procedure adjust p-values to control the FWER, and what are its limitations?
::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: objectives

- Understand the concept of the family-wise error rate (FWER) and its significance in multiple testing, including the implications of making multiple comparisons without controlling for FWER.
- Learn the Bonferroni procedure for adjusting p-values to maintain the FWER at a specified level, and recognize when alternative methods may be more appropriate or effective in controlling for multiple comparisons.
:::::::::::::::::::::::::::::::::::::::::::


In statistical hypothesis testing, conducting multiple tests simultaneously increases the likelihood of making at least one false-positive error. In this episode, we will explore the family-wise error rate (FWER), and discuss methods for adjusting p-values to account for multiple comparisons, using practical examples.

## A Multi-Hypothesis Testing Framework

In multiple testing scenarios, researchers often have an overarching hypothesis that encompasses several individual hypotheses, each examining specific aspects or relationships within the data. This approach allows researchers to explore various facets of the research question comprehensively.

Going back to our study investigating the effects of air pollution on the prevalence of a disease, the overarching hypothesis could be formulated as follows:

_Exposure to air pollution is associated with increased prevalence of the disease_ 

Under this overarching hypothesis, several individual hypotheses can be formulated to examine different aspects of the relationship between air pollution exposure and disease prevalence. These individual hypotheses may focus on various pollutants, different health outcomes, or specific populations.

An example using three individual null hypotheses:

**$H_{0,1}$**: Exposure to particulate matter  is not associated with increased disease prevalence.

**$H_{0,2}$**: Exposure to nitrogen dioxide is not associated with increased disease prevalence.

**$H_{0,3}$**: Long-term exposure to ozone (O3) is not associated with an increased prevalence of the disease.

The three null hypotheses can be combined to the following overall null hypothesis:

**$H_{0}$**: Air pollution is not associated with an increased prevalence of the disease.

:::: callout ::::::::::::
As soon as one of the individual null hypotheses is rejected, we also reject the overall null hypothesis. Therefore, we'll want to make sure that we have not a single false positive among our individual hypothesis tests.
:::::::::::::::::::::::

![Figure_6: Relationship between Overall Hypothesis and Individual Hypotheses (Effects of Air Pollution on Disease
Prevalence)](fig/Relationship between Overall Hypothesis and Individual Hypotheses. Effects of Air Pollution on Rdisease prevalence.png)


In this illustration, each individual hypothesis delves into a distinct facet of the overarching research inquiry, enabling researchers to thoroughly examine the intricate connection between air pollution exposure and disease prevalence. By scrutinizing the impacts of various air pollutants on the disease, this method encourages a systematic exploration of diverse factors and aids in revealing potential associations or patterns within the data set.



Now, let us assume that after data collection, for hypothesis 1, we  find that 15 out of 100 individuals exposed to high levels of particulate matter develop the disease, for hypothesis 2, 20 out of 100 individuals exposed to high levels of nitrogen dioxide develop the disease and for hypothesis 3, 5 out of 100 individuals exposed to high levels of ozone develop the disease.

Let us assume we conduct statistical tests for each of these hypotheses, resulting in p-values for each test. For simplicity, let us maintain our significance level at $\alpha=0.05$ for each individual test. We can conduct binomial tests for each hypothesis and calculate the p-values in R.


```r
n = 100 # number of test persons
p = 0.04 # Known prevalence of the disease in the general population

individuals_suffered = c(15, 20, 5) # number of individuals who suffered from the disease for each hypothesis

p_values = sapply(individuals_suffered, function(x) {
  p_value = dbinom(x, size = n, prob = p)
  return(p_value)
})#Calculate the p-values for each hypothesis using the binomial probability mass function


for (i in 1:length(p_values)) {
  cat(sprintf("Hypothesis %d: p = %.4f\n", i, p_values[i]))
}# Print the p-values for each hypothesis
```

```{.output}
Hypothesis 1: p = 0.0000
Hypothesis 2: p = 0.0000
Hypothesis 3: p = 0.1595
```


## The Bonferroni correction 

Now, to calculate the probability of having any false-positive within the set of tests (also known as familywise error rate or FWER), we can use methods such as the Bonferroni correction. This method adjust the significance level for each test to control for multiple testing.

The Bonferroni procedure adjusts the significance level for each individual test by dividing the desired overall significance level by the number of tests conducted (m).

$$α_{\text{Bonf}}= α/m $$
 
Where:

α is the desired overall significance level (usually set to 0.05). 
m is the number of hypothesis tests conducted.


```r
FWER <- 0.05# Define the desired Family-wise error rate

m <- length(p_values)# Calculate the number of hypothesis tests conducted (m)

alpha_bonf <- FWER / m # Calculate Bonferroni adjusted significance level

alpha_bonf
```

```{.output}
[1] 0.01666667
```


Since in our example above we have three tests (three hypotheses), the Bonferroni corrected significance level is $\alpha_{\text{bonf}} = 0.05/3 \approx 0.0167$.
 
### Interpretation

For each individual test, we compare the calculated p-value to this adjusted significance level. If the p-value is less than or equal to 0.0167, we reject the null hypothesis for that test.

Based on the Bonferroni correction, we reject the null hypothesis for Hypotheses 1 and 2, indicating significant associations between particulate matter and nitrogen dioxide exposure with disease prevalence. However, for Hypothesis 3 (ozone), we fail to reject the null hypothesis, suggesting no significant association with disease prevalence at the adjusted significance level. This adjustment for multiple testing helps control the overall probability of making at least one false-positive error across all tests conducted.

In this example, while the evidence supports associations between certain air pollutants and disease prevalence, it does not provide conclusive evidence to reject the overarching null hypothesis entirely. Instead, it suggests a nuanced interpretation wherein the relationship between air pollution exposure and disease prevalence may vary depending on the specific pollutant considered. Therefore, further investigation and analysis may be necessary to fully elucidate the relationship between air pollution exposure and disease prevalence and to refine the overarching null hypothesis accordingly. This could involve exploring additional factors, conducting more comprehensive analyses, or considering alternative statistical approaches to account for potential confounding variables or sources of variability in the data.

Instead of changing the significance level, another (equivalent) calculation is adjusting the p-values obtained from individual hypothesis tests, followed by comparing the adjusted p-values to the desired FWER. With this procedure, we can reject those individual null hypotheses that have a p-value below the desired FWER ($p < \text{FWER}$).

$$p_{\text{Bonf}}= p \times m$$

In R, this can be done as follows:

```r
p.adjust(p_values, method = "bonferroni")
```

```{.output}
[1] 2.539669e-05 6.747937e-09 4.785324e-01
```

We can check which of the individual hypotheses can will be rejected at a $\text{FWER}=0.05$:


```r
p.adjust(p_values, method = "bonferroni") < 0.05
```

```{.output}
[1]  TRUE  TRUE FALSE
```


The conclusion is the same as when changing the significance level for each test.

::::::::::::::::::::::::::::::::::::::: instructor

Inline instructor notes can help inform instructors of timing challenges
associated with the lessons. They appear in the "Instructor View"
::::::::::::::::::










