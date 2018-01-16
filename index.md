---
output: 
  html_document: 
    keep_md: yes
    self_contained: no
---

_All datasets were downloaded from the [Minnesota Department of Education Data Center](http://w20.education.state.mn.us/MDEAnalytics/DataTopic.jsp?TOPICID=3) in October 2017._ 
_The raw data used is discussed at [datasets](datasets.Rmd)_


# [](#header-1)Minnesota K12
###### [](#header-6)Compiled and analyzed by Carla Bates
###### [](#header-6)Site created January 2018; this page updated 2018-01-15

**Extraordinary passions light up numerous debates on education today.**  Data is disparaged by the left who seem aesthically opposed to numbers for some reason or used as a cudgel by the right to beat up on the latest community school in the name of progress.  Of course, these flailings mask real questions about power and indicate huge cracks in how we think about our state, about who lives in our state, about who should benefit from the bounty of our state, and about our shared future? These are not questions to be answered with data but we have little chance of forging a just and sustainable future if we continue to ignore what the data can show us about how we are currently operating.

This website offers data and analysis in the hopes of building common ground on which to stand when debating how to educate our children.

#### [](#header-4)Question 1:  What does k12 look like across the state?  
Exploring this question requires the dataset called 's_byDistrict' from [datasets](datasets.Rmd) s_byDistrict includes information on student demographics in each district in the state in the following categories:
- 'free' is free and reduced lunch
- 'sped' is special education
- 'lep' is limited English proficency


```r
s_byDistrict <- read.csv("data/s_byDistrict.csv", stringsAsFactors = FALSE)
s_toPrint <- s_byDistrict %>% select(-X, -districttype)  
print(head(s_toPrint))
```

```
##   datayear    region                          district free sped lep enrll
## 1    05-06 Arrowhead      BIRCH GROVE COMMUNITY SCHOOL   18    3   0    32
## 2    05-06 Arrowhead     DULUTH PUBLIC SCHOOLS ACADEMY  380  126   0   773
## 3    05-06 Arrowhead                GREAT EXPECTATIONS    0    6   0    36
## 4    05-06 Arrowhead HARBOR CITY INTERNATIONAL CHARTER   79   40   3   205
## 5    05-06 Arrowhead         LAKE SUPERIOR HIGH SCHOOL   48   21   0    86
## 6    05-06 Arrowhead      NORTH SHORE COMMUNITY SCHOOL   99   21   0   253
```

For each year, the enrollment data for each group in each district was totaled and then divided by the total enrollment in order to determine statewide proportions.


```r
fn_sumEnr <- function(x) {
  x %>% summarise(enrll=sum(enrll, na.rm=TRUE),
                  free=sum(free, na.rm=TRUE), 
                  sped=sum(sped, na.rm=TRUE),
                  lep=sum(lep, na.rm=TRUE))
}
fn_propEnr <- function(x) {
  x %>% mutate_at(.vars = vars(3:5), 
                  .funs = funs(round((./enrll), 2)))
}
```

From 2005 to 2016, the k12 student population in Minnesota increased by 28,504 which is a 3 percentage increase. 



During this same period the numbers of students receiving free and reduced lunch increased by 68,634 students:  not only did more children in poverty join the k12 system but more children within the system already fell into poverty.  Note the significant up-tic around the 2008-09 Great Recession and how we have not returned to pree 2008 levels.  The proportion of students receiving special education or of limited english proficency remained almost flat, increasing by 6% and 3% respectively.

![](index_files/figure-html/unnamed-chunk-5-1.png)<!-- -->


####[](#header-4)Question 2:  How did different regions of the state fare?
Almost every region of the state experienced an increase in k12 student poverty after the 2008 Great Recession. 
