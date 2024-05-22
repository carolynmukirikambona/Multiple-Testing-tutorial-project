---
title: 'False discovery rate'
teaching: 10
exercises: 2
---

:::::::::::::::::::::::::::::::::::::: questions 

- How does correcting for the family-wise error rate (FWER) affect the number of significant hits in large-scale data, such as RNA-Seq analysis of 20,000 human genes?
- What is the interpretation of a p-value histogram, and how can it be used to assess the distribution of p-values in multiple testing scenarios?
- How can the Benjamini-Hochberg method be applied to control the false discovery rate (FDR) in RNA-Seq data, and what are the benefits of using this method over FWER correction?

::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::: objectives

- Demonstrate how correcting for the family-wise error rate (FWER) using methods like Bonferroni correction can lead to few or no significant hits in large-scale testing scenarios.
- Introduce the concept of p-value histograms, explain their interpretation, and illustrate how they can be used to visualize the distribution of p-values in multiple testing.
- Explain the Benjamini-Hochberg method for controlling the false discovery rate (FDR).
- Provide practical examples and R code to apply the Benjamini-Hochberg method to RNA-Seq data.
- Discuss the advantages of controlling the FDR over FWER in the context of large-scale genomic data. 

::::::::::::::::::::::::::::::::::::::::::::::::

# Introduction

In high-throughput experiments like RNA-Seq, we often conduct thousands of statistical tests simultaneously. This large number of tests increases the risk of false positives. The __False Discovery Rate (FDR)__ is the expected proportion of false positives among the rejected hypotheses. While controlling the family-wise error rate (FWER), the probability of making at least one type I error, is one approach to address false positives, it can be too conservative, leading to few or no significant results. An alternative approach is to control the false discovery rate (FDR), which offers a balance between identifying true positives and limiting false positives. 

## Example: RNA-Seq Analysis with 20,000 Genes

Here we will consider an example where we analyze RNA-Seq data from 20,000 human genes to identify differentially expressed genes between two conditions.


```r
library(ggplot2) # Load necessary library
```


```r
set.seed(123) # Simulate p-values for 20,000 genes

p_values <- runif(19000, min = 0, max = 1) # 95% of p-values are uniformly distributed (noise)

p_values <- c(p_values, rbeta(1000, 0.5, 1)) # 5% of p-values are drawn from a beta distribution (signal)

# Histogram of p-values
ggplot(data.frame(p_values), aes(x = p_values)) +
  geom_histogram(bins = 50, fill = "lightblue", color = "black") +
  labs(title = "P-value Histogram", x = "P-value", y = "Frequency")
```

<img src="fig/False discovery rate-rendered-unnamed-chunk-2-1.png" style="display: block; margin: auto;" />

```r
# Finding the number of significant differentially expressed genes before FWER correction
alpha <- 0.05
significant_hits <- p_values < alpha
significant_genes<-sum(significant_hits) 
significant_genes
```

```{.output}
[1] 1167
```

## The Concept of P-value Histograms

A p-value histogram is a graphical representation that displays the distribution of p-values obtained from multiple hypothesis tests. When conducting a large number of statistical tests, such as in genome-wide association studies or RNA-Seq analysis, p-value histograms provide a visual tool to assess the behavior and distribution of p-values across all tests.

To create a p-value histogram, we plot the p-values on the x-axis and the frequency (or count) of those p-values on the y-axis. This visualization helps in understanding how the p-values are distributed across the range from 0 to 1.

### Interpretation of P-value Histograms
A p-value histogram provides insights into the distribution of p-values and helps identify patterns that might indicate issues or meaningful findings in our data. If all the null hypotheses are true (i.e., there is no effect), p-values should follow a uniform distribution. This means the histogram should show a relatively flat distribution of p-values across the range from 0 to 1. If there are true effects present, you would expect to see an excess of low p-values (close to 0) compared to what would be expected under a uniform distribution. This indicates potential significant findings. A large spike near zero can indicate that many tests are finding significant results, suggesting the presence of true positives. A right-skewed distribution, with most p-values clustering towards 1, might suggest that most tests are not finding significant results, indicating that there may be few true effects in the data.

Therefore, by visualizing the distribution, we can get a sense of the overall significance landscape of our tests. In cases where the histogram shows a sharp spike at p-values near zero and another spike near one, it might indicate issues like p-hacking or poor experimental design. Understanding the distribution of p-values can inform the choice of multiple testing correction methods. For example, if there is an excess of low p-values, controlling the false discovery rate (FDR) might be more appropriate than controlling the family-wise error rate (FWER). After applying a multiple testing correction (such as the Benjamini-Hochberg method), a p-value histogram of the adjusted p-values can help evaluate the effectiveness of the correction in controlling false positives.

# The impact of correcting for the family-wise error rate (FWER) using Bonferroni correction

Now, let us say we go ahead and use Bonferroni method to correct Family-wise error rate (FWER). We then check to see what is the number of significant differentially expressed genes after Bonferroni correction. 


```r
# Applying Bonferroni correction
alpha <- 0.05
bonferroni_threshold <- alpha / length(p_values)
significant_bonferroni <- p_values < bonferroni_threshold

significant_genes<-sum(significant_bonferroni) # Number of significant differentially expressed genes after Bonferroni correction
significant_genes
```

```{.output}
[1] 1
```

## Interpretation

Applying the Bonferroni correction in this example results in one (1) significant differentially expressed gene due to the stringent threshold, demonstrating how FWER correction can be too conservative in large-scale testing.

# Controlling FDR Using the Benjamini-Hochberg Method

The Benjamini-Hochberg (BH) method is a statistical procedure used to control the FDR in multiple hypothesis testing, where the chance of obtaining false positives increases. The BH method helps control the proportion of false positives (false discoveries) among the rejected hypotheses, thus providing a more balanced approach than traditional methods like the __Bonferroni correction__, which can be overly conservative. The BH procedure adjusts the p-values from multiple tests to control the FDR. These adjusted p-values can be compared to a significance threshold to determine which results are significant.

## Steps of the Benjamini-Hochberg Procedure
First the observed p-values are arranged in ascending order. Next, ranks are assigned to the p-values, with the smallest p-value getting rank 1, the next smallest rank 2, and so on. Then, for each p-value, the BH critical value is calculated as;

BH critical value=𝑖/𝑚*Q

where, i is the rank, m is the total number of tests, and 
Q is the desired FDR level (e.g., 0.05).

After this calculation, the largest p-value that is less than or equal to its BH critical value is identified. This p-value and all smaller p-values are considered significant. Optionally, one can adjust the p-values to reflect the BH correction using software functions.

We will continue with our simulated p-values for 20,000 genes i to apply the Benjamini-Hochberg correction in r, and also plot the adjusted p values for comparison. 


```r
# Applying Benjamini-Hochberg correction
p_adjusted <- p.adjust(p_values, method = "BH")

# Plotting the adjusted p-values
ggplot(data.frame(p_adjusted), aes(x = p_adjusted)) +
  geom_histogram(bins = 50, fill = "lightblue", color = "black") +
  labs(title = "Adjusted P-value Histogram", x = "Adjusted P-value", y = "Frequency")
```

<img src="fig/False discovery rate-rendered-unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

```r
significant_bh <- p_adjusted < alpha

sum(significant_bh) # Number of significant hits after Benjamini-Hochberg correction
```

```{.output}
[1] 3
```

## Interpretation

The Benjamini-Hochberg method controls the FDR, allowing for a greater number of significant differentially expressed genes (3) compared to the Bonferroni correction (1). This approach provides a more balanced and powerful method for identifying true positives in large-scale data.

# Advantages of controlling the FDR over FWER in the context of large-scale genomic data

Controlling the False Discovery Rate (FDR) offers several advantages over controlling the Family-Wise Error Rate (FWER) in the context of large-scale genomic data analysis. 

FDR control typically results in __higher statistical power__ compared to FWER control. In large-scale genomic studies, where thousands or even millions of hypotheses are tested simultaneously (such as in transcriptomic analyses), FDR control allows for the detection of more true positives while still controlling the proportion of false discoveries. This increased power is crucial for identifying biologically relevant findings amidst the vast amount of data.

FDR control is __less conservative__ than FWER control. While FWER methods, such as the Bonferroni correction, set a stringent threshold for each individual test to maintain the overall Type I error rate, they often result in a high rate of false negatives. In contrast, FDR methods, balance between controlling false discoveries and maximizing the detection of true positives, leading to a more balanced approach.

FDR control provides a __more interpretable framework__ for large-scale genomic studies. By controlling the proportion of false discoveries among all rejected hypotheses, we can prioritize significant findings based on their estimated false discovery rate. This allows for a more nuanced interpretation of results, focusing on those hypotheses most likely to be true positives while still acknowledging the potential for false discoveries.

FDR control allows for __greater flexibility in hypothesis testing__ and exploration of large-scale genomic data. Researchers can conduct exploratory analyses without overly stringent correction methods limiting their ability to detect meaningful associations. This flexibility encourages hypothesis generation and discovery, leading to advancements in our understanding of complex biological systems.

FDR control facilitates __consistency__ in statistical inference across different studies and datasets. By controlling the false discovery rate at a predetermined threshold (e.g., 5%), we can compare results across studies and meta-analyses more reliably, regardless of variations in sample sizes or experimental designs.

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

Inline instructor notes can help inform instructors of timing challenges
associated with the lessons. They appear in the "Instructor View"

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
