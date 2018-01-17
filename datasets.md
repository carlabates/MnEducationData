---
output: 
  html_document: 
    keep_md: yes
    self_contained: no
---
## [](#header-2)Minnesota K12: Datasets
* Compiled and analyzed by Carla Bates.
* Site created January 2018; this page updated 2018-01-17

Development of following data sets included here:
- Student enrollment across districts:  s_byDistrict
- Student 
- Teacher
- Finances
- Expenses



#### [](#header-4)Student data

Data downloaded from MDE > Data Center > Data Reports and Analytics > [Student](http://w20.education.state.mn.us/MDEAnalytics/DataTopic.jsp?TOPICID=2) in October 2017.

Files downloaded were named "<data year> Enrollment of Special Populations"  The student files are from 2005-06 to 2016-17.  All files were downloaded as Excel workbooks; the 'School' worksheet was saved as a csv file named with as "mde_specpop_<data year>"

The files detail student demographics on a school by school basis.  This information is organized by unique school Numbers and contains the following columns:  DataYear, District County Num, District County Name, district Number, district Type, District Name, school Number, School Name, ECSUNumber and Economic Development Region, Grade, K12Enr (K12 Enrollment), FreeK12 (Free lunch K12), RedK12 (Reduced lunch K12), LEPIdentifiedK12 (English Learner), LEPServedK12 (Served English Learner), SPEK12 (Special Education K12), HMLK12 (Homeless K12).  Demographic info on early childhood is also included. 

The data files are read into R and then cleaned. No changes were made outside of R.


```r
mdeFiles <- as.list(list.files(path="data/", pattern="^mde_spec_pop.+"))
for(i in seq_along(mdeFiles)) {
  tmpName <- paste0("specpops", gsub("[(mde_spec_pop_)|(.csv)]", "", mdeFiles[i]))
  tmpPath <- paste0("data/", mdeFiles[i])
  tmpFile <- data.frame(read.csv(tmpPath,stringsAsFactors = FALSE,strip.white = TRUE))
  tmpFile <- tmpFile[-nrow(tmpFile), ]
  tmpFile <- rename(tmpFile, District=DistrictName)
  tmpFile <- rename(tmpFile, Region=EconomicDevelopmentRegion)
  tmpFile$District <- trimws(tmpFile$District)
  assign(tmpName, tmpFile)
}
```

Most 'data cleaning' decisions are benign analytically, but a few determine the scope of the analysis here.  Within R the columns removed from the student special populations data include:
- "x" (unknown) from 0506 and 0607;
- all items ending in EC12 in all lists;
- "lepidentifiedk12" from 0910-1617;
- renamed "lepservedk12" to "lepk12"; and 
- "hmlk12" in 1516 and 1617.

Given the uneven development and different funding structure for early childhood, a separate study of EC data will be included later; the decision to only keep "LEPServedK12" and not "LEPIdentifiedK12" will require a separate study; and the exclusion of HMLK12 necessitates a separate study on homeless data.  


```r
dflist <- list(specpops0506,specpops0607,specpops0708,specpops0809,specpops0910,
              specpops1011,specpops1112,specpops1213,specpops1314,specpops1415,
              specpops1516,specpops1617)

specpop <- lapply(dflist, function(x) x %>% select(-ends_with("EC12"), 
                                                   -matches("X"),
                                                   -matches("LEPIdentifiedK12"),
                                                   -matches("HMLK12")))

for (i in seq_along(specpop)) {
  tmp_list <- names(specpop[[i]])
  if ("LEPServedK12" %in% tmp_list) {
    colnames(specpop[[i]])[15] <- "lepk12"
  }
}
for (i in seq_along(specpop)) { #extract dataframes per year
  tmp_name <- ls(pattern="^specpops.+")
  tmp_df <- as.data.frame(specpop[i], use.names = FALSE)
  colnames(tmp_df) <- tolower(colnames(tmp_df))
  assign(tmp_name[i], tmp_df)
}
```

To be useful for analysis the numeric signifiers for 'regions' and for 'districttypes' needed to be spelt out.  This code used information from 'sheet' help files' downloaded from MDE at the same time as the data.


```r
#refresh r cache of dflist
dflist <- list(specpops0506,specpops0607,specpops0708,specpops0809,specpops0910,
               specpops1011,specpops1112,specpops1213,specpops1314,specpops1415,
               specpops1516,specpops1617)

students <- do.call(rbind, dflist)
for (i in seq_along(students$districttype)) {
  students$districttype[[i]] <- switch (students$districttype[[i]],
                                        "1" = "Independent",
                                        "3" = "Special",
                                        "6" = "Intermediate",
                                        "7" = "Charter",
                                        "8" = "Integration",
                                        "50" = "Cooperatives",
                                        "51" = "Voc Coops",
                                        "52" = "Sped Coops",
                                        "53" = "Voc and Sped",
                                        "60" = "Correctional Facilities",
                                        "61" = "Ed Districts",
                                        "62" = "Secondary Facilities",
                                        "70" = "State Operated",
                                        "83" = "Service Coop",
                                        'NA' = 'NA')
}
for (i in seq_along(students$region)) {
     students$region[[i]] <- switch(students$region[[i]],
                                     "01" = "Northwest",
                                      "1" = "Northwest",
                                     "02" = "Headwater",
                                      "2" = "Headwater",
                                     "03" = "Arrowhead",
                                      "3" = "Arrowhead",
                                     "04" = "West Central",
                                      "4" = "West Central",
                                     "05" = "North Central",
                                     "5" = "North Central",
                                     "06W" = "Upper Minnesota Valley",
                                     "06E" = "Mid-Minnesota",
                                     "06" =  "Mn Valley Mid-Mn",
                                     "07W" = "Central",
                                     "07E" = "East Central",
                                     "07" = "East Cntr and Cntr",
                                     "08" = "Southwest",
                                     "8" = "Southwest",
                                     "09" = "South Central",
                                      "9" = "South Central",
                                     "10" = "Southeast",
                                     "11" = "Twin Cities",
                                    "011" = "Twin Cities",
                                     'NA' = 'NA')
}

students$freek12 <- as.numeric(students$freek12)
students$redk12 <- as.numeric(students$redk12)
students$spek12 <- as.numeric(students$spek12)
students$k12enr <- as.numeric(students$k12enr)
students$lepk12 <- as.numeric(students$lepk12)
```

The data is now ready to be grouped and then filtered for the summary rows "All Grades."  The data files contain information for all grades in a school and all schools in a district.  The "All Grades" field summarizes enrollment numbers by school which allows them to be added by district and region or district typs or data year.


```r
s_byDistrict <- students %>% group_by(datayear,region,districttype,district) %>% 
  filter(grade == "All Grades") %>% 
  summarise(free = sum((freek12+redk12), na.rm=TRUE), 
            sped = sum(spek12, na.rm=TRUE),
            lep = sum(lepk12, na.rm=TRUE),
            enrll = sum(k12enr, na.rm=TRUE))
```


More to come ...
