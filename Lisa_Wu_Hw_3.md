Lisa_Wu_HW_3
================

## Load data

Load the following data: + applications from `app_data_sample.parquet` +
edges from `edges_sample.csv`

    ## # A tibble: 2,018,477 × 16
    ##    application_number filing_date examiner_name_last examiner_name_first
    ##    <chr>              <date>      <chr>              <chr>              
    ##  1 08284457           2000-01-26  HOWARD             JACQUELINE         
    ##  2 08413193           2000-10-11  YILDIRIM           BEKIR              
    ##  3 08531853           2000-05-17  HAMILTON           CYNTHIA            
    ##  4 08637752           2001-07-20  MOSHER             MARY               
    ##  5 08682726           2000-04-10  BARR               MICHAEL            
    ##  6 08687412           2000-04-28  GRAY               LINDA              
    ##  7 08716371           2004-01-26  MCMILLIAN          KARA               
    ##  8 08765941           2000-06-23  FORD               VANESSA            
    ##  9 08776818           2000-02-04  STRZELECKA         TERESA             
    ## 10 08809677           2002-02-20  KIM                SUN                
    ## # ℹ 2,018,467 more rows
    ## # ℹ 12 more variables: examiner_name_middle <chr>, examiner_id <dbl>,
    ## #   examiner_art_unit <dbl>, uspc_class <chr>, uspc_subclass <chr>,
    ## #   patent_number <chr>, patent_issue_date <date>, abandon_date <date>,
    ## #   disposal_type <chr>, appl_status_code <dbl>, appl_status_date <chr>,
    ## #   tc <dbl>

## Get gender for examiners

We’ll get gender based on the first name of the examiner, which is
recorded in the field `examiner_name_first`. We’ll use library `gender`
for that, relying on a modified version of their own
[example](https://cran.r-project.org/web/packages/gender/vignettes/predicting-gender.html).

Note that there are over 2 million records in the applications table –
that’s because there are many records for each examiner, as many as the
number of applications that examiner worked on during this time frame.
Our first step therefore is to get all *unique* names in a separate list
`examiner_names`. We will then guess gender for each one and will join
this table back to the original dataset. So, let’s get names without
repetition:

    ## # A tibble: 2,595 × 1
    ##    examiner_name_first
    ##    <chr>              
    ##  1 JACQUELINE         
    ##  2 BEKIR              
    ##  3 CYNTHIA            
    ##  4 MARY               
    ##  5 MICHAEL            
    ##  6 LINDA              
    ##  7 KARA               
    ##  8 VANESSA            
    ##  9 TERESA             
    ## 10 SUN                
    ## # ℹ 2,585 more rows

Now let’s use function `gender()` as shown in the example for the
package to attach a gender and probability to each name and put the
results into the table `examiner_names_gender`

    ## # A tibble: 1,822 × 3
    ##    examiner_name_first gender proportion_female
    ##    <chr>               <chr>              <dbl>
    ##  1 AARON               male              0.0082
    ##  2 ABDEL               male              0     
    ##  3 ABDOU               male              0     
    ##  4 ABDUL               male              0     
    ##  5 ABDULHAKIM          male              0     
    ##  6 ABDULLAH            male              0     
    ##  7 ABDULLAHI           male              0     
    ##  8 ABIGAIL             female            0.998 
    ##  9 ABIMBOLA            female            0.944 
    ## 10 ABRAHAM             male              0.0031
    ## # ℹ 1,812 more rows

Finally, let’s join that table back to our original applications data
and discard the temporary tables we have just created to reduce clutter
in our environment.

    ##            used  (Mb) gc trigger  (Mb) limit (Mb) max used  (Mb)
    ## Ncells  4477991 239.2    8237225 440.0         NA  4895694 261.5
    ## Vcells 49463440 377.4   95367219 727.6      16384 79779256 608.7

## Guess the examiner’s race

We’ll now use package `wru` to estimate likely race of an examiner. Just
like with gender, we’ll get a list of unique names first, only now we
are using surnames.

    ## # A tibble: 3,806 × 1
    ##    surname   
    ##    <chr>     
    ##  1 HOWARD    
    ##  2 YILDIRIM  
    ##  3 HAMILTON  
    ##  4 MOSHER    
    ##  5 BARR      
    ##  6 GRAY      
    ##  7 MCMILLIAN 
    ##  8 FORD      
    ##  9 STRZELECKA
    ## 10 KIM       
    ## # ℹ 3,796 more rows

We’ll follow the instructions for the package outlined here
<https://github.com/kosukeimai/wru>.

    ## Warning: Unknown or uninitialised column: `state`.

    ## Proceeding with last name predictions...

    ## ℹ All local files already up-to-date!

    ## 701 (18.4%) individuals' last names were not matched.

    ## # A tibble: 3,806 × 6
    ##    surname    pred.whi pred.bla pred.his pred.asi pred.oth
    ##    <chr>         <dbl>    <dbl>    <dbl>    <dbl>    <dbl>
    ##  1 HOWARD       0.597   0.295    0.0275   0.00690   0.0741
    ##  2 YILDIRIM     0.807   0.0273   0.0694   0.0165    0.0798
    ##  3 HAMILTON     0.656   0.239    0.0286   0.00750   0.0692
    ##  4 MOSHER       0.915   0.00425  0.0291   0.00917   0.0427
    ##  5 BARR         0.784   0.120    0.0268   0.00830   0.0615
    ##  6 GRAY         0.640   0.252    0.0281   0.00748   0.0724
    ##  7 MCMILLIAN    0.322   0.554    0.0212   0.00340   0.0995
    ##  8 FORD         0.576   0.320    0.0275   0.00621   0.0697
    ##  9 STRZELECKA   0.472   0.171    0.220    0.0825    0.0543
    ## 10 KIM          0.0169  0.00282  0.00546  0.943     0.0319
    ## # ℹ 3,796 more rows

As you can see, we get probabilities across five broad US Census
categories: white, black, Hispanic, Asian and other. (Some of you may
correctly point out that Hispanic is not a race category in the US
Census, but these are the limitations of this package.)

Our final step here is to pick the race category that has the highest
probability for each last name and then join the table back to the main
applications table. See this example for comparing values across
columns: <https://www.tidyverse.org/blog/2020/04/dplyr-1-0-0-rowwise/>.
And this one for `case_when()` function:
<https://dplyr.tidyverse.org/reference/case_when.html>.

    ## # A tibble: 3,806 × 8
    ##    surname    pred.whi pred.bla pred.his pred.asi pred.oth max_race_p race 
    ##    <chr>         <dbl>    <dbl>    <dbl>    <dbl>    <dbl>      <dbl> <chr>
    ##  1 HOWARD       0.597   0.295    0.0275   0.00690   0.0741      0.597 white
    ##  2 YILDIRIM     0.807   0.0273   0.0694   0.0165    0.0798      0.807 white
    ##  3 HAMILTON     0.656   0.239    0.0286   0.00750   0.0692      0.656 white
    ##  4 MOSHER       0.915   0.00425  0.0291   0.00917   0.0427      0.915 white
    ##  5 BARR         0.784   0.120    0.0268   0.00830   0.0615      0.784 white
    ##  6 GRAY         0.640   0.252    0.0281   0.00748   0.0724      0.640 white
    ##  7 MCMILLIAN    0.322   0.554    0.0212   0.00340   0.0995      0.554 black
    ##  8 FORD         0.576   0.320    0.0275   0.00621   0.0697      0.576 white
    ##  9 STRZELECKA   0.472   0.171    0.220    0.0825    0.0543      0.472 white
    ## 10 KIM          0.0169  0.00282  0.00546  0.943     0.0319      0.943 Asian
    ## # ℹ 3,796 more rows

Let’s join the data back to the applications table.

    ##            used  (Mb) gc trigger  (Mb) limit (Mb) max used  (Mb)
    ## Ncells  4659649 248.9    8237225 440.0         NA  7367226 393.5
    ## Vcells 51798965 395.2   95367219 727.6      16384 94105604 718.0

## Examiner’s tenure

To figure out the timespan for which we observe each examiner in the
applications data, let’s find the first and the last observed date for
each examiner. We’ll first get examiner IDs and application dates in a
separate table, for ease of manipulation. We’ll keep examiner ID (the
field `examiner_id`), and earliest and latest dates for each application
(`filing_date` and `appl_status_date` respectively). We’ll use functions
in package `lubridate` to work with date and time values.

    ## # A tibble: 2,018,477 × 3
    ##    examiner_id filing_date appl_status_date  
    ##          <dbl> <date>      <chr>             
    ##  1       96082 2000-01-26  30jan2003 00:00:00
    ##  2       87678 2000-10-11  27sep2010 00:00:00
    ##  3       63213 2000-05-17  30mar2009 00:00:00
    ##  4       73788 2001-07-20  07sep2009 00:00:00
    ##  5       77294 2000-04-10  19apr2001 00:00:00
    ##  6       68606 2000-04-28  16jul2001 00:00:00
    ##  7       89557 2004-01-26  15may2017 00:00:00
    ##  8       97543 2000-06-23  03apr2002 00:00:00
    ##  9       98714 2000-02-04  27nov2002 00:00:00
    ## 10       65530 2002-02-20  23mar2009 00:00:00
    ## # ℹ 2,018,467 more rows

The dates look inconsistent in terms of formatting. Let’s make them
consistent. We’ll create new variables `start_date` and `end_date`.

Let’s now identify the earliest and the latest date for each examiner
and calculate the difference in days, which is their tenure in the
organization.

    ## # A tibble: 5,625 × 4
    ##    examiner_id earliest_date latest_date tenure_days
    ##          <dbl> <date>        <date>            <dbl>
    ##  1       59012 2004-07-28    2015-07-24         4013
    ##  2       59025 2009-10-26    2017-05-18         2761
    ##  3       59030 2005-12-12    2017-05-22         4179
    ##  4       59040 2007-09-11    2017-05-23         3542
    ##  5       59052 2001-08-21    2007-02-28         2017
    ##  6       59054 2000-11-10    2016-12-23         5887
    ##  7       59055 2004-11-02    2007-12-26         1149
    ##  8       59056 2000-03-24    2017-05-22         6268
    ##  9       59074 2000-01-31    2017-03-17         6255
    ## 10       59081 2011-04-21    2017-05-19         2220
    ## # ℹ 5,615 more rows

Joining back to the applications data.

    ##            used  (Mb) gc trigger  (Mb) limit (Mb)  max used  (Mb)
    ## Ncells  4667797 249.3    8237225 440.0         NA   8237225 440.0
    ## Vcells 57873133 441.6  114520662 873.8      16384 114305815 872.1

## Look at examiners demographics

    ## # A tibble: 1 × 1
    ##       n
    ##   <int>
    ## 1  5649

### Compare TCs by gender graphically

    ## `summarise()` has grouped output by 'tc'. You can override using the `.groups`
    ## argument.

![](Lisa_Wu_Hw_3_files/figure-gfm/compare-tcs-by-gender-1.png)<!-- -->

#### The graph reveals a higher representation of males in the organization compared to females. However, it is crucial to acknowledge the presence of a significant number of “NAs” in the data, as these figures can greatly impact the interpretation. Examining the ratio between females and males is also noteworthy. For instance, in the case of “tc 1600,” the difference between males and females is only slightly pronounced when the NAs are not considered. However, in the case of “tc 2100,” the gender gap is noticeably wider.

### Compare TCs by race graphically

    ## `summarise()` has grouped output by 'tc'. You can override using the `.groups`
    ## argument.

![](Lisa_Wu_Hw_3_files/figure-gfm/compare-tcs-by-race-1.png)<!-- -->

#### The data suggests that individuals of white ethnicity constitute the majority across all “tc” categories. However, it is essential to acknowledge the potential bias introduced by name associations. Some individuals may possess names that sound traditionally associated with white ethnicity, despite belonging to other racial backgrounds. Additionally, an intriguing observation emerges: the presence of a higher number of individuals of Asian ethnicity in “tc 2100” and “tc 2400,” both of which pertain to computer-related fields.

### Compare TCs by tenure graphically

    ## Warning: Removed 24 rows containing non-finite values (`stat_bin()`).

![](Lisa_Wu_Hw_3_files/figure-gfm/compare-tcs-by-tenure-1.png)<!-- -->

#### The presence of a considerable number of employees who have remained with the organization for more than 6000 days indicates a positive aspect for the company. Among the four different “tc” departments there appears to be a relatively similar count of individuals who have surpassed the 6000-day mark in tc 1600, 1700, and 2100. Conversely, tc 2400 exhibits the lowest number in this category. This observation provides a basis for hypothesizing about the turnover rate in each department. Despite tc 1600 and 1700 having fewer employees overall compared to 2100 and 2400, they manage to maintain a similar number of employees who have stayed for more than 6000 days, implying a higher retention rate for these two departments. However, further analysis is necessary to validate this hypothesis.

### Turnover by gender and race

``` {r-turnover}
# Step 1: set 85% sample data and 15% testing data
'''set.seed(123)
examiners <- applications %>%
  group_by(examiner_id) %>%
  summarise(
    tenure = first(tenure_days), 
    gender = first(gender),
    race = first(race),
    tc = first(tc),
    earliest_date = first(earliest_date),
    latest_date = first(latest_date)
    )
main_sample_percentage <- 0.85
examiners_split <- examiners %>%
  mutate(split = ifelse(row_number() <= round(n() * main_sample_percentage), "main", "test"))
main_sample <- examiners_split %>% filter(split == "main")
test_sample <- examiners_split %>% filter(split == "test")
write.csv(main_sample, "main_sample.csv", row.names = FALSE)
write.csv(test_sample, "test_sample.csv", row.names = FALSE)

#Step 2: define turnover: idea is to check the yearly turnover rate and average them. look at who is there the first day of the year and still there the first day of next year, and see how many people held a position at the organization that whole year. then use formula to get a turnover rate. I want to use a while loop to loop 17 years, and then avergae the turnover rate. 

#Create a copy of the dataset to test, so it doesnt mess up my main sample
'''applications <- applications %>%
  mutate(appl_status_date = dmy_hms(appl_status_date))

applications <- applications %>%
  mutate(year = year(appl_status_date))

#filter any data after 2017
applications <- applications %>%
  filter(year <= 2017)

turnover <- applications %>%
  group_by(examiner_id) %>%
  summarize(min_year = min(year), max_year = max(year), tc = first(tc), gender = first(gender), race = first(race)) %>%
  mutate(year_left = if_else(max_year<2017, max_year+1, NA_real_))

turnover_rate2 <- turnover %>%
  group_by(year_left) %>%
  summarize(turnover_count = n()) %>%
  mutate(year = year_left-1)

total_examiners <- applications %>%
  group_by(year) %>%
  summarize(previous_year_count = n_distinct(examiner_id))

turnover_rate2 <- turnover_rate2 %>%
  left_join(total_examiners) %>%
  mutate(turnover_rate = turnover_count/previous_year_count*100) %>%
  select(-year)
```
