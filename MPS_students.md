---
title: "MPS students"
author: "Carla"
date: "May 1, 2018"
output: 
  html_document: 
    keep_md: yes
---



## Steady student enrollment


```r
ggplot(mmps, aes(x=datayear, y=value, fill=variable)) +
  geom_bar(stat="identity", position="dodge")
```

![](MPS_students_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

##Loss of market share

## [MPS teachers] (MPS_teachers.md)
