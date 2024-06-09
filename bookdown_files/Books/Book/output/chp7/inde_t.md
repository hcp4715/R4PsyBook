
Independent-Samples t-test

Hypothesis: two-sided (μ2 - μ1 ≠ 0)

Descriptives:
───────────────────────────────────────
 Variable Factor Level    N Mean (S.D.)
───────────────────────────────────────
 ECR_mean    sex     1  442 3.32 (0.83)
 ECR_mean    sex     2 1042 3.25 (0.91)
───────────────────────────────────────

Levene’s test for homogeneity of variance:
────────────────────────────────────────────────────
                       Levene’s F df1  df2     p    
────────────────────────────────────────────────────
ECR_mean: sex (2 - 1)        6.49   1 1482  .011 *  
────────────────────────────────────────────────────
Note: H0 = equal variance (homoscedasticity).
If significant (violation of the assumption),
then you should better set `var.equal=FALSE`.

Results of t-test:
────────────────────────────────────────────────────────────────────────────────────────────
                           t   df     p     Difference [95% CI]  Cohen’s d [95% CI]     BF10
────────────────────────────────────────────────────────────────────────────────────────────
ECR_mean: sex (2 - 1)  -1.54 1482  .124     -0.08 [-0.18, 0.02] -0.09 [-0.20, 0.02] 2.05e-01
────────────────────────────────────────────────────────────────────────────────────────────

