---
date: "2020"
external_link: ""
image:
  caption: 
  focal_point: Smart
links:
- icon: twitter
  icon_pack: fab
  name: Follow
  url: https://twitter.com/georgecushen
#slides: example
summary: An example of using the in-built project page.
tags:
- Deep Learning
title: Chi-Square Tests
url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""
---



```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = F)

# load packages -----------------------------------------------

library(tidyverse)
library(janitor)
library(dplyr)
library(kableExtra)
library(ggbeeswarm)
library(car)
library(effsize)

# add data ---------------------------------------------------

mack_creek_verts <- read_csv("mack_creek_vertebrates.csv") %>% 
  clean_names()

```

<br>
<br>

### **Is there a significant difference in salamander counts in channel locations in old growth and clear cut forests?**

<br>

Used a chi-square test to determine if there is a significant difference in what channel Pacific giant salamanders are located (pool, cascade or side channel) between the two old growth and clear cut forests.

**Null Hypothesis:** There is no siginificant difference in what channel Pacific giant salamander are located between old growth and clear cut forests

**Alternative Hypothesis:** There is a significant difference in what channel Pacific giant salamander are located between old growth and clear cut forests

```{r, include = T}

#subdata: only include 2017 and salamader observations-------------------------------------

salamander_channel_2017 <- mack_creek_verts %>% 
  filter (year == "2017") %>% 
  filter (species == "DITE") %>% 
  filter (unittype != "IP") %>% 
  mutate(channel_type= case_when(
    unittype %in% "C" ~ "Cascade",
    unittype %in% "P" ~"Pool",
    unittype %in% "SC" ~ "Side Channel"
  )) %>% 
  select (species, channel_type, section)

# counts
salamander_channel_counts_2017 <- salamander_channel_2017 %>% 
  group_by(channel_type, section) %>% 
  count(species) %>% 
  select (channel_type, section, n)

# create contigency table of ONLY counts
salamander_ct <- salamander_channel_counts_2017 %>% 
  pivot_wider(names_from = section, values_from = n) %>%
  column_to_rownames('channel_type')

# chi-square test --------------------------------------------------------------------

salamander_chi <- chisq.test(salamander_ct)

```

<br> 

A Pearson's chi-squared test was performed to examine whether or not there was a significant difference in where in the channel Pacific giant salamanders are located between the two sections of forest (old growth and clear cut). The results from our test found that there was no siginificant difference in what channel Pacific giant salamander are located in forest section ($\chi$^2^(`r salamander_chi$parameter`) = `r round(salamander_chi$statistic,2)`, *p* = `r round(salamander_chi$p.value,2)`). 

