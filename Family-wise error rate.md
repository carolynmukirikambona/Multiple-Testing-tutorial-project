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
# Family-wise Error Rate (FWER)

In statistical hypothesis testing, conducting multiple tests simultaneously increases the likelihood of making at least one false-positive error. In this tutorial, we will explore FWER, discuss methods for adjusting p-values to account for multiple comparisons, and examine practical examples.

## A Multi-Hypothesis Testing Framework

In multiple testing scenarios, researchers often have an overarching hypothesis that encompasses several individual hypotheses, each examining specific aspects or relationships within the data. This approach allows researchers to explore various facets of the research question comprehensively.

Going back to our study investigating the effects of air pollution on the prevalence of a disease, the overarching hypothesis could be formulated as follows:

_Exposure to air pollution is associated with increased prevalence of the disease_ 

Under this overarching hypothesis, several individual hypotheses can be formulated to examine different aspects of the relationship between air pollution exposure and disease prevalence. These individual hypotheses may focus on various pollutants, different health outcomes, or specific populations.

An example using three individual hypothesis:

_Hypothesis 1: Exposure to particulate matter  is associated with increased disease prevalence_

_Hypothesis 2: Exposure to nitrogen dioxide is associated with increased disease prevalence_

_Hypothesis 3: Long-term exposure to ozone (O3) is associated with an increased prevalence of the disease_

![Figure_5: Relationship between Overall Hypothesis and Individual Hypotheses: Effects of Air Pollution on Disease
Prevalence"](fig/Relationship between Overall Hypothesis and Individual Hypotheses. Effects of Air Pollution on Rdisease prevalence.png)

In this illustration, each individual hypothesis delves into a distinct facet of the overarching research inquiry, enabling researchers to thoroughly examine the intricate connection between air pollution exposure and disease prevalence. By scrutinizing the impacts of various air pollutants on the disease, this method encourages a systematic exploration of diverse factors and aids in revealing potential associations or patterns within the dataset.

Now, let us assume that after data collection, for hypothesis 1, we  find that 15 out of 100 individuals exposed to high levels of particulate matter develop the disease, for hypothesis 2, 20 out of 100 individuals exposed to high levels of nitrogen dioxide develop the disease and for hypothesis 3, 5 out of 100 individuals exposed to high levels of ozone develop the disease.

Let us assume we conduct statistical tests for each of these hypotheses, resulting in p-values for each test. For simplicity, let us maintain our significance level (alpha) at 0.05 for each individual test. We can conduct binomial tests for each hypothesis and calculate the p-values in R.


``` r
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

``` output
Hypothesis 1: p = 0.0000
Hypothesis 2: p = 0.0000
Hypothesis 3: p = 0.1595
```


## The Bonferroni correction 

Now, to calculate the probability of having any false-positive within the set of tests (also known as familywise error rate or FWER), we can use methods such as the Bonferroni correction. This method adjust the significance level for each test to control for multiple testing.

The Bonferroni procedure adjusts the significance level for each individual test by dividing the desired overall significance level (alpha) by the number of tests conducted (m).

Bonferroni adjusted significance level (αBonf)= α/m​
 
Where:

α is the desired overall significance level (usually set to 0.05).
m is the number of hypothesis tests conducted.


``` r
α <- 0.05# Define the desired overall significance level (alpha)

m <- length(p_values)# Calculate the number of hypothesis tests conducted (m)

αBonf <- α / m # Calculate Bonferroni adjusted significance level
```


Since in our example above we have three tests (three hypotheses), the Bonferroni corrected significance level would be:

alpha_bonf = 0.05/3​
 ≈0.0167
 
### _Interpretation_ 

For each individual test, we would compare the calculated p-value to this adjusted significance level. If the p-value is less than or equal to 0.0167, we would reject the null hypothesis for that test.

Based on the Bonferroni correction, we would reject the null hypothesis for Hypotheses 1 and 2, indicating significant associations between particulate matter and nitrogen dioxide exposure with disease prevalence. However, for Hypothesis 3 (ozone), we would fail to reject the null hypothesis, suggesting no significant association with disease prevalence at the adjusted significance level. This adjustment for multiple testing helps control the overall probability of making at least one false-positive error across all tests conducted.

In this example, while the evidence supports associations between certain air pollutants and disease prevalence, it does not provide conclusive evidence to reject the overarching null hypothesis entirely. Instead, it suggests a nuanced interpretation wherein the relationship between air pollution exposure and disease prevalence may vary depending on the specific pollutant considered. therefore, further investigation and analysis may be necessary to fully elucidate the relationship between air pollution exposure and disease prevalence and to refine the overarching null hypothesis accordingly. This could involve exploring additional factors, conducting more comprehensive analyses, or considering alternative statistical approaches to account for potential confounding variables or sources of variability in the data.

Even if we could change the significance level, another approach to address the issue of multiple comparisons is to adjust the p-values obtained from individual hypothesis tests. This adjustment is equivalent to controlling the overall familywise error rate (FWER) and ensures that the probability of making at least one false positive error across all tests remains at or below a predetermined threshold, typically 0.05. Common methods for adjusting p-values include the Bonferroni correction and False Discovery Rate (FDR) procedures. These methods modify the threshold for statistical significance for each individual test based on the number of comparisons conducted, thereby maintaining the integrity of the overall statistical analysis while accounting for multiple comparisons. Adjusting p-values offers a flexible and widely used approach to control the type I error rate and enhance the reliability of statistical inference in multiple testing scenarios.

::::::::::::::::::::::::::::::::::::::: instructor

Inline instructor notes can help inform instructors of timing challenges
associated with the lessons. They appear in the "Instructor View"
::::::::::::::::::










