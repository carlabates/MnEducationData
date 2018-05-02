---
title: "MPS students"
author: "Carla"
date: "May 1, 2018"
output: 
  html_document: 
    keep_md: yes
---



## Slowly declining enrollment
The MPS total enrollment has ebbed and flowed and it currently is {r enrll_diff enrll_direction} from 2005. 




```r
ggplot(mps_tmp, aes(x=datayear, y=value, fill=variable)) +
  geom_bar(stat="identity") +
  annotate("text", x="06-07", y=37000, label=paste0("N = ", total_2005)) +
  annotate("text", x="15-16", y=35000, label=paste0("N = ", total_n)) 
```

![](MPS_students_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

## Loss of market share

## [MPS teachers](MPS_teachers.md)
