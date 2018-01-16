---
output: 
  html_document: 
    keep_md: yes
    self_contained: no
---

All datasets were downloaded from the [Minnesota Department of Education Data Center]
(http://w20.education.state.mn.us/MDEAnalytics/DataTopic.jsp?TOPICID=3) in October 2017. The raw data used is discussed at [datasets] (datasets.Rmd) 


##Minnesota K12 Education Data
######*Compiled and analyzed by Carla Bates*
######*Site created January 2018; this page updated 2018-01-15*

Extraordinary passions light up numerous debates on education today.  Data is disparaged by the left who seem aesthically opposed to numbers for some reason or used as a cudgel by the right to beat up on the latest community school in the name of progress.  Of course, these flailings mask real questions about power and indicate huge cracks in how we think about our state, about who lives in our state, about who should benefit from the bounty of our state, and about our shared future? These are not questions to be answered with data but we have little chance of forging a just and sustainable future if we continue to ignore what the data can show us about how we are currently operating.

This website offers data and some analysis which may provide common ground on which to stand when debating how to educate our children.

###Question 1:  What does the student population look like across the state?  
Exploring this question requires the dataset called 's_byDistrict' from [datasets] This dataset includes information on student demographics for students in each district in the state in the following categories:
- free is free and reduced lunch
- sped is special education
- lep is limited english proficency


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


```r
#These functions summarise data across districts and figure proportions.
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

mn <- s_byDistrict %>% group_by(datayear) %>% fn_sumEnr(.)
mn_enrll <- mn %>% filter(datayear == "05-06" | datayear == "16-17")

mn_total <- prettyNum(mn_enrll[2,2]-mn_enrll[1,2], big.mark=",")
mn_increase <- as.numeric(round((mn_enrll[2,2] - mn_enrll[1,2])/mn_enrll[1,2]*100))
mn_free <- prettyNum(mn_enrll[2,3]-mn_enrll[1,3], big.mark=",")
mn_fincrease <- as.numeric(round((mn_enrll[2,3] - mn_enrll[1,3])/mn_enrll[1,3]*100))
```


From 2005 to 2016, the k12 student population in Minnesota increased by 28,504 which is a 3 percentage increase. 



```r
propmn <- mn %>% mutate(prop_free = round((free/enrll), 2), 
                         prop_sped = round((sped/enrll), 2), 
                         prop_lep = round((lep/enrll), 2))
melted_propmn <- melt(propmn)
f_propmn <- melted_propmn %>% filter(variable == "prop_free" | 
                                     variable == "prop_sped" | 
                                     variable == "prop_lep")
```

During this same period the numbers of students receiving free and reduced lunch increased by 68,634 students, meaning that the overall proportion of the student population in poverty increased by 28 percent.


```r
ggplot(f_propmn, aes(x=datayear, y=value, group=variable)) +
  geom_line(aes(color=variable), size=2) +
  xlab("Year") +
  ylab("Proportion of total enrollment") +
  ggtitle("Proportion of total enrollment of special populations\n across Minnesota from 2005-2016") +
  scale_fill_brewer(palette = "Set1") +
  annotate("text", x="13-14", y=0.32,
    label=paste0("This percentage increase in children
              receiving free and reduced lunch
            is equivalent to ", mn_free, " children."))
```

![](index_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

Almost every region of the state experienced an increase in k12 student poverty after the 2008 Great Recession. 
