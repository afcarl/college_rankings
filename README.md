# College Ranking System Based on Earnings Outcomes
## Using Federal College Scorecard Data

### Background

For three decades the _U.S. News_ college rankings have heavily influenced students' college choices. Its methodology, which uses measures of financial resources, alumni giving, selectivity, and assessment by peer institutions, has affected colleges' behavior as they attempt to improve their rankings. Undergraduate academic reputation, a subjective measure, is responsible for 22.5 percent of its [ratings](http://www.usnews.com/education/best-colleges/articles/how-us-news-calculated-the-rankings). Faculty salary and per-student spending are used as inputs, pushing colleges to spend more money in order to improve their rank.

This project presents a ranking system based on [College Scorecard data](https://collegescorecard.ed.gov/data/) released by the U.S. Department of Education in 2015. It uses student earnings ten years after matriculation as the outcome of interest and school and student characteristics like SAT scores and demographic information as inputs. The governing assumption behind this model is that most students go to college in order to earn more money, or at least income after graduation is a chief consideration. In this ranking system, colleges are rated on student earnings after graduating relative to predicted earnings. While most schools perform as predicted, this model highlights colleges that exceed expectations and flags colleges performing worse than predicted.

In August 2013 President Obama launched the College Scorecard in order to fight rising college costs and ensure quality. The U.S. Department of Education ultimately declined to rate or rank schools using the data it gathered. Instead it released information and metrics about schools through an online portal and redesigned College Scorecard. The hope is that greater transparency will enable students and parents to make better decisions when choosing a college.

### Methods

All data used for this project come from the College Scorecard technical site and are pulled using the [API](https://github.com/18F/open-data-maker/blob/api-docs/API.md). The U.S. Treasury and Education Department cross-walked their data to map 2011 student median earnings outcomes to 2000-01 and 2001-02 pooled student cohorts. Logged median student earnings are used to adjust for skew in income numbers. Earnings are in 2014 dollars. Student and school characteristics come from 2001, the year the 2000-01 cohort started college. The sample is limited to schools currently operating in 2011. It is restricted to public and non-profit bachelor degree granting institutions because ACT and SAT scores are missing for other types of schools. Vocational art, nursing, and bible schools are removed from the sample. Schools with suppressed earnings due to privacy or missing SAT and ACT scores are removed from the sample. The sample size is 1,078 colleges. _The Economist_ released a [college ranking system](http://www.economist.com/blogs/graphicdetail/2015/10/value-university) in 2015. Its approach informed the decision to remove some vocational schools and include enrollment as a feature. 

School characteristics include number of students enrolled, a flag for high-income region, and a flag for urban or suburban area. These variables come from U.S. Department of Education data. If a school is located in the Northeast, Mideast, or Far West, it is labeled as a high-income region college, because those regions have significantly higher incomes. Similarly, cities and suburbs tend to have higher incomes, so an urban area flag is used. The model also controls for student enrollment. Admission rate, completion rate, and net price measures are dropped from the model because they are either not statistically significant or highly correlated with other attributes. Some school attributes, like academic program and ownership (public or private), are not included in the model because they may play a role in school quality.  

Since there is limited data about the earnings cohort, school characteristics are used to infer information about the students in the cohort. Each school's average score on the SAT and ACT, percentage female, and percentage Pell Grant recipients come from U.S. Department of Education data. The number of test-takers for SAT and ACT is not available, so a weighted average is not possible. Instead, standardized SAT score is used. If it is missing, standardized ACT score is used. Students who attend schools with higher average scores on the SAT and ACT are generally expected to perform better in the labor market. 

Student demographic characteristics for the pooled 2000-01 and 2001-02 cohort are provided by the Treasury Department. They include percentage of census tract born in the United States, percentage of census tract black or Hispanic, and percentage over age 23 at entry. Percentage female, Pell grants, and born in U.S. are all negatively correlated with earnings after ten years. The percentage born in U.S. measure appears to be picking up whether the student lives in a metropolitan area with more immigrants and higher incomes on average. Poverty rate, percentage black or Hispanic, and median household income measures are dropped from the model because they are highly correlated with the Pell Grant measure.

```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:      log_earnings_10yr   R-squared:                       0.707
Model:                            OLS   Adj. R-squared:                  0.705
Method:                 Least Squares   F-statistic:                     258.1
Date:                Tue, 14 Mar 2017   Prob (F-statistic):          5.94e-222
Time:                        18:10:19   Log-Likelihood:                 711.40
No. Observations:                 863   AIC:                            -1405.
Df Residuals:                     854   BIC:                            -1362.
Df Model:                           8                                         
Covariance Type:            nonrobust                                         
===================================================================================
                      coef    std err          t      P>|t|      [95.0% Conf. Int.]
-----------------------------------------------------------------------------------
score_sat_act       0.0628      0.005     11.773      0.000         0.052     0.073
female_pct         -0.0510      0.004    -11.937      0.000        -0.059    -0.043
born_in_usa_pct    -0.0572      0.005    -12.047      0.000        -0.067    -0.048
pell_grant_pct     -0.0849      0.006    -15.378      0.000        -0.096    -0.074
enrollment          0.0258      0.004      6.315      0.000         0.018     0.034
overage23           0.0360      0.004      8.388      0.000         0.028     0.044
region_high_inc     0.0559      0.009      5.991      0.000         0.038     0.074
urban_area          0.0251      0.010      2.624      0.009         0.006     0.044
intercept          10.6264      0.009   1204.571      0.000        10.609    10.644
==============================================================================
Omnibus:                       92.981   Durbin-Watson:                   1.908
Prob(Omnibus):                  0.000   Jarque-Bera (JB):              398.425
Skew:                          -0.405   Prob(JB):                     3.04e-87
Kurtosis:                       6.229   Cond. No.                         5.35
==============================================================================
```

The project uses a linear regression model to predict each school's cohort earnings after ten years. Model variables are statistically significant and account for 71% of the variation in logged median income after ten years. The regression specification is validated using [k-fold cross-validation](http://scikit-learn.org/stable/modules/cross_validation.html#k-fold). School ratings and ranks are based on the premise that school quality is reflected in the extent to which students in the ten-year cohort outperform their expected earnings. The residual of predicted average earnings subtracted from actual median logged earnings is standardized to produce a score for each school. Schools with a score of 1.5 and above are given a rating of 5, 0.5 and above are given 4, and between -0.5 and 0.5 earn a 3. Schools below -0.5 receive a rating of 2 and below -1.5 receive a 1. These ratings are constructed so that about half of schools earn a 3, indicating that they more or less meet expectations according to the model.

### Results

The 64 colleges with the highest rating (5) have the highest SAT and ACT scores and the highest census tract median household income. Their net price is higher than average, and they have the lowest admission rate and highest completion rate on average. These colleges' average predicted earnings are $46,790, versus $58,833 actual earnings. 

Colleges with the lowest rating have above-average SAT and ACT scores, the lowest student enrollment numbers, and the highest average net price. 89% of these schools are private institutions, compared to 38% overall. These colleges' average predicted earnings are $46,014, compared to $36,207 actual. 

University of Chicago, the tied for third best national university according to the 2017 _U.S. News_ rankings, receives a rating of 3 and a rank of 546, with its students underperforming predictions by $158 on average. University of Pennsylvania, tied for eighth place in _U.S. News_, has a rank of 89 and has a rating of 4. Its students outperform expectations by $9,970. According to the model, a U. Penn. education is higher quality than a U. Chicago one because Penn's students perform better than anticipated in the labor market. 

Amherst and Williams, the top-ranked liberal arts colleges on the 2017 _U.S. News_ list, earned ratings of 2 and ranks of 790 and 856 because their students have lower earnings than expected after ten years. Nevertheless, both of these colleges are on a U.S. Dept. of Education [list](http://blog.ed.gov/2015/09/schools-with-low-costs-and-high-incomes/) of schools with low costs and high outcomes.

### Conclusions and Limitations

This model has several limitations. First, it is difficult to draw conclusions about an entire college or university based upon a single student cohort. More research is needed to support these findings using other cohorts in the College Scorecard data and other data sources. Moreover, the ten-year earnings cohort is made up of students eligible for financial aid, so schools with low percentages of these students might be inaccurately represented. Second, the model does not include for-profit and community colleges because they are missing test score data. Third, this model makes the assumption that students primarily go to college because they want to make more money and ignores outcome measures like completion rate, net price, and learning outcomes. Attending college has many intangible benefits that are hard to measure in dollar terms. Finally, school-level N counts by SAT and ACT exam or test results by earnings cohort would make the SAT/ACT measure more accurate. 

This college ranking system should be used alongside other measures of college performance, like [net price](https://collegecost.ed.gov/netpricecenter.aspx) metrics. Students, parents, and college administrators go astray when they make decisions using a single ranking system like the _U.S. News_ college rankings. This model, with its focus on labor market outcomes, provides a valuable alternative measure of college performance. Most importantly, it minimizes perverse incentives. The _U.S. News_ rankings encourage colleges to inflate their reputation, spend more money, and admit fewer students. In contrast, these new rankings incentivize colleges to accept more students, particularly those from disadvantaged backgrounds, and do a better job of preparing them for the workforce.