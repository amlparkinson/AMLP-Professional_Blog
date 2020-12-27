---
date: "2019"
external_link: ""
image:
  caption: P
  focal_point: Smart
links:
- icon: twitter
  icon_pack: fab
  name: Follow
  url: https://twitter.com/georgecushen
#slides: example
summary: Examples of a couple statistical tests such as ANOVAs, regressions, and t-tests.
tags:
- Deep Learning
title: Statistical Tests
url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""
---

### Task 1: Predicting Penguin Mass


```{r}
# load data and remove NA's
penguin <- read_csv("lter_penguin_species.csv") %>% 
  drop_na(body_mass_g) %>% 
  drop_na(flipper_length_mm) %>% 
  drop_na(sex) %>% 
  filter (sex != ".")

```

```{r}

# visualize data
penguin_sub <- penguin %>% 
  select(body_mass_g, species, flipper_length_mm, sex)

ggplot(penguin_sub, aes(x=flipper_length_mm, y=body_mass_g)) +
  geom_point(aes(color=fct_rev(species))) +
  facet_wrap(~sex) +
  labs(x = "\nFlipper Length (mm)", 
       y = "Body Mass (g)\n", 
       title = "Weight and Flipper Length\n of Three Antarctic Penguin Species") +
  theme_bw() +
  scale_color_discrete(name="Species", 
                       labels=c("Gentoo Penguin (Pygoscelis papua)",
                                'Chinstrap Penguin (Pygoscelis antarctica)',
                                "Adelie Penguin (Pygoscelis adeliae)")) +
  theme(legend.title.align=0.5, 
        plot.title = element_text(hjust=0.5),
        panel.spacing=unit(2, "lines")) +
  scale_x_continuous(lim=c(150, 250), 
                     expand=c(0,0),
                     breaks=seq(150, 250, by=50)) +
  scale_y_continuous(lim=c(2500, 6500), 
                     expand=c(0,0),
                     breaks=seq(3000, 6500, by=1000))

```

***Figure 1:** Flipper lengths (mm) and body mass (g) by sex of three penguin species in Antarctica - [Gentoo](https://doi.org/10.6073/pasta/409c808f8fc9899d02401bdb04580af7), [Chinstrap](https://doi.org/10.6073/pasta/2b1cff60f81640f182433d23e68541ce), and [Adelie](https://doi.org/10.6073/pasta/abc50eed9138b75f54eaada0841b9b86) penguins. Data: Palmer Station LTER.*

<br>
General trends from Figure 1:

- Male penguins appear to have higher body mass and slightly longer flipper lengths than females across all three species. 
- Gentoo penguins have the highest body mass and longest flipper length than Chinstrap and Adelie penguins. 

<br>

```{r, include = F}
#Multiple linear regression model 

# model trials --------------------------------------------------

mass_model1 <- lm(body_mass_g ~ species + flipper_length_mm + sex, data=penguin)
summary(mass_model1)

mass_model2 <- lm(body_mass_g ~ flipper_length_mm + sex, data=penguin) 
summary(mass_model2)

mass_model3 <- lm(body_mass_g ~ species + flipper_length_mm, data=penguin)
summary(mass_model3)

# compare model AICs

AIC (mass_model1) #4740 --> lowest AIC
AIC (mass_model2) #4862
AIC (mass_model3) #4895

# check assumptions (residual normality and homoscedasticity)

plot(mass_model1)

# statement about how the model passes the assumptions here #########


```

***Table 1:** Regression table displaying the relationship between Antarctic penguins species, sex, and flipper length to predict body mass. (body mass(sex, flipper length, species) = -365.8 + 530.38(MALE) + 20.03(flipper length) + 836.26(Gentoo Penguin) - 87.63(Chinstrap Penguin)*
```{r, results='asis'}
#table to report model results (using stargazer)

stargazer(mass_model1, type="html")
```

<br>

The assumptions for using multiple linear regression were met by plotting the residuals of the multiple linear regression model. The QQ plot showed that the residuals follow a normal distribution so the assumption of residual normality is met, while the residuals vs. fitted plot showed that the residuals appear randomly distributed with no change in range so the assumption of homoscedasticity is also met. 

<br>

The adjusted R^2^ (R^2^ = 0.865) evaluates how much variance in body mass is explained by the explanatory variables (species, flipper length, sex), so in this case 86.5% of the variance in penguin body mass can be explained by species, flipper length, and sex.

<br>

Interpretation of the model coefficients:

- If everything else about the penguin is the same (species, flipper length), then I expect male penguins to weigh 530.38 grams more than a female penguin on average
- If everything else is held constant, then I expect boby mass to increase 20.03 grams for every 1 mm increase in flipper length.
-  If everything else about the penguin is the same (sex, flipper length), then I expect Gentoo penguins weigh 836.26 grams more than Adelie penguins. 
-  If everything else about the penguin is the same (sex, flipper length), then I expect Chinstrap penguins weigh 87.63 grams less than Adelie penguins. 



<br>

***Table 2:** Predicted weights of three penguins (right column) using model: (body mass(sex, flipper length, species) = -365.8 + 530.38(MALE) + 20.03(flipper length) + 836.26(Gentoo Penguin) - 87.63(Chinstrap Penguin)*
```{r}
# Use the model predict the masses 

# create data frame
new_df <- data.frame(
  flipper_length_mm = rep(c(195, 205, 220)),
  sex = rep(c("FEMALE", "MALE", "MALE")),
  species = rep(c("Chinstrap penguin (Pygoscelis antarctica)", "Adelie Penguin (Pygoscelis adeliae)", "Gentoo penguin (Pygoscelis papua)"))
)

# predict values in data frame
predict_df <- predict(mass_model1, newdata = new_df)

# bind predictions with new_df
new_df_predictions <- data.frame(new_df, predict_df)

# make new_df_preiditions into a nice table

new_df_predictions %>% 
  kable (col.names = c("Flipper Length (mm)", "Sex", "Species", "Body Mass (g) Predictions"), align="c") %>%
 # add_header_above(c(" "= 1, "Low Birthweight?"=2)) %>% 
  kable_styling(
    bootstrap_options = c("striped", "hover", "condensed", "responsive"), 
    full_width=T,
    position="center") 

```


<br>
<br>

### Task 2: Smoking effects on baby birth weight

```{r}
# load data and only keep tpounds, lowbw, and smoke columns

birthweights <- read_csv('nc_birthweights.csv') %>% 
  select (tpounds, lowbw, smoke) %>% 
  drop_na(smoke)

```


***Table 3:** Data was collected on babies birthweight (categorically classified as low or not low) and whther or not the mother smoked during pregnancy. 1,000 babies from North Carolina were randomly selected for the study.*
```{r}
# table: Does the proportion of babies born at “low birth weight” differ depending on whether the mother smoked or did not smoke during pregnancy

# step 1: create sub data

birthweights_table_data <- birthweights %>% 
  group_by(smoke) %>% 
  count(lowbw) %>% 
  mutate(low_birth_weight = case_when( 
         lowbw == 1 ~ "Yes",
         lowbw == 0 ~ "No")) %>% 
  mutate("Mother Smoke?" = case_when(
    smoke == 1 ~"Yes",
    smoke == 0 ~"No"
  )) %>% 
  ungroup(smoke) %>% 
  select(-lowbw, -smoke)

# step 2 create contigency  table

birthweights_table_counts <- birthweights_table_data %>% 
  pivot_wider(names_from = low_birth_weight, values_from = n) 

birthweights_table_proportions <- birthweights_table_counts %>% 
  adorn_percentages(denominator = "row") %>% 
  adorn_pct_formatting(digits=1) %>% 
  adorn_ns(position = "front")

# step 3: R markdown friendly contigency table with counts and proportions

birthweights_table_proportions %>% 
  kable (col.names = c("Mother Smoke?", "No", "Yes")) %>%
  add_header_above(c(" "= 1, "Low Birthweight?"=2)) %>% 
  #add_header_above(c("North Carolina babies: Does mother smoking affect baby's birthweight?"=3)) %>% 
  kable_styling(
    bootstrap_options = c("striped", "hover", "condensed", "responsive"), 
    full_width=F,
    position="center") %>% 
  column_spec(1, width='4cm') %>%  #control column 1's row width
  column_spec(2, width="8em") %>% 
  column_spec(3, width="8em")


```

```{r, include = F}
# statistically significant difference between the proportion of babies born at low birth weights for smoking vs. non-smoking mothers? 

# H null: there is no difference in birthweights of babies from mothers who did and did not smoke
# H alt : there is a difference in birthweights of babies from mothers who did and did not smoke

# step 1: get cocntigency table of just counts
birthweight_counts <- birthweights_table_counts %>% 
  column_to_rownames('Mother Smoke?')           

# step 2: run chi-square test
birthweights_chi <- chisq.test(birthweight_counts)
birthweights_chi

# p = 0.2696
```

<br>

A Pearson's chi-squared test was performed to examine whether or not babies with low birthweights are independent of having mothers who smoked during pregnancy. The results from our test found that babies with low bithweights are independent of being born from mothers who smoked during pregnancy ($\chi$^2^(`r birthweights_chi$parameter`) = `r round(birthweights_chi$statistic,2)`, *p* = `r round(birthweights_chi$p.value,2)`). 

To improve future surveys, researchers might consider changing the "smoking during pregnancy" category from binomial (yes or no) to an ordinal response with more than two categories (e.g. frequently, sometimes, rarely, never). Some women might have smoked once or twice throughout their pregnancy, while others might have smoked every day, or some other frequency. Mothers who smoked once or twice may not have had as much of an effect on their babies birthweight than mothers who smoked frequently, so researchers would want to distinguish between these differences if there are any.

<br>

### Task 3: Visualizing UCSB campus profiles

```{r, include = F}
ucsb <- read_csv('ucsb_campus_profiles.csv') %>% 
  pivot_longer('2018 - 2019' : '2008 - 2009',
               names_to = 'year',
               values_to = 'value') %>% 
  clean_names()
```

In this section, I will explore the undergraduate enrollment rates of minorities at University of California, Santa barbara across all years of available data (2008-2019). Enrollment rates of whites, others, and unknowns were excluded from the analysis and some ethnicities were combined into similar groups (Filipinos were combined with Asian and Pacific Islanders, Chicanos and Latinos were combined into a new group called Hispanics). 

```{r}
# for now, explore what relationship to explore
# subdata: combine groups and remove other/unknown

ucsb_ethnicity <- ucsb %>% 
  filter (student_level == "Undergraduate (individuals)") %>% 
  group_by(ethnicity_domestic_enrollment,year) %>% 
  summarise(enrollment = sum(value, na.rm=T)) %>% 
  ungroup(ethnicity_domestic_enrollment)

ucsb_ethnicity_combined <- ucsb_ethnicity %>% 
  filter (ethnicity_domestic_enrollment != "White") %>% 
  filter (ethnicity_domestic_enrollment != "Other") %>%
  filter (ethnicity_domestic_enrollment != "Unknown") %>% 
  mutate (ethnicity_domestic_enrollment = case_when(
    ethnicity_domestic_enrollment %in% 'American Indian / Alaskan'  ~ 'American Indian/Alaskan', 
    ethnicity_domestic_enrollment %in% c("Asian / Pacific Islander", "Filipino" ) ~ "Asian/Pacific Islander", 
    ethnicity_domestic_enrollment %in% "Black / African American" ~ "African American",
    ethnicity_domestic_enrollment %in% c("Chicano", "Latino") ~ "Hispanic",
    ethnicity_domestic_enrollment %in% "E. Indian / Pakistani" ~ "E.Indian/Pakistani"
    )) %>% 
  group_by(year, ethnicity_domestic_enrollment) %>% 
  summarise(enrollment = sum(enrollment)) 


#plot

ggplot (ucsb_ethnicity_combined, aes(x= year, y=enrollment)) +
  geom_line (aes(group = ethnicity_domestic_enrollment, 
                 color=ethnicity_domestic_enrollment), 
             show.legend=F) +
  geom_point(aes(color=ethnicity_domestic_enrollment), show.legend=F) +
  theme_bw() +
  labs(x= "\nYear", 
       y= "Enrollment\n", 
       title=" Undergraduate Enrollment of Minorities at UC Santa Barbara", 
       subtitle="2008-2019\n") +



   coord_cartesian(clip="off") +
   theme(plot.title = element_text(hjust=0.5),
        plot.subtitle = element_text(hjust=0.5),
        plot.margin = unit(c(1,7,1,1),"lines")) +
  scale_x_discrete (expand=c(0,0),
                    labels = c('08-09', '09-10', '10-11', 
                               '11-12', '12-13', '13-14', 
                               '14-15', '15-16', '16-17', 
                               '17-18', '18-19')) +
  scale_y_continuous(lim=c(0, 7000), 
                     expand=c(0,0),
                     breaks=seq(0,7000, by=1000)) +
  annotate("text", label= "American Indian/Alaskan", 
           x = Inf, 
           y = 150, 
           size = 3,
           hjust = -0.12,
           vjust = 0) +
  annotate("text", label = "Asian/Pacific Islander",
           x = Inf,
           y = 5000,
           size = 3,
           hjust = -0.1,
           vjust = 0) +
  annotate("text", label = "African American",
           x = Inf,
           y = 1010,
           size = 3,
           hjust = -0.175,
           vjust = 0) +
  annotate("text", label = "Hispanic",
           x = Inf,
           y = 6020,
           size = 3,
           hjust = -0.25,
           vjust = 0) +
  annotate("text", label = "E.Indian/Pakistani",
           x = Inf,
           y = 650,
           size = 3,
           hjust = -0.18,
           vjust = 0) 
  
```
***Figure x:** Annual undergraduate enrollment rates of minorities and underrepresented groups at University of California, Santa Barbara. Years are based on a school calendar year which starts in September of one year and ends in June of the following year. Data: [UCSB](http://bap.ucsb.edu/institutional.research/campus.profiles/)*

<br>

General takeways from Figure x:

- Line graphs with groups that have large differences in numbers (Hispanics and Asian/Pacific Islander compared to African Americans, E. Indian/Pakistani, and American Indian/Alaskan) can result in incorrectly misinterpreting  information minorities have had the largest growth in undergradutes.
- While it appears that the enrollment for the other three minorities has been relatively stagnant, African American enrollment has almost doubled and East Indian and Pakistani enrollment has increased by 226% since 2008-2009 school year. American Indian and Alaskan enrollment has very similar percent change: 52% and 54% respectively.    

<br>

### Task 4: Purple urchins on the Santa Barbara coast

```{r, include = F}
# load data and modify so only have purple urchin observations in control treatments 

urchin <- read_csv('sbc_lter_urchins.csv') 
urchin <-  urchin %>% 
  clean_names() %>% 
  filter (common_name == "Purple Urchin") %>% 
  filter (treatment == "CONTROL") %>% 
  uncount(count) %>% 
  select (year, site, size) %>% 
  mutate(site = case_when(
    site %in% c("CARP") ~ "Carpinteria" ,
    site %in% c("IVEE") ~ "Isla Vista",
    site %in% c("AQUE") ~ "Arroyo Quemado",
    site %in% c("MOHK") ~ "Mohawk", 
    site %in% c("NAPL") ~ "Naples")) %>% 
  mutate(year=as.character(year))

```

<br> 
 
***Table 4** Summary statistics for purple urchin (*Strongylocentrotus purpuratus*) population sizes in control plots at five sites along the Santa Barbara coast from 2008-2018. Data: [SBC LTER](https://doi.org/10.6073/pasta/846569acc8745c133a44eac40245e6f0). *

```{r}

# summary statistics for urchin sizes at the 5 different sites

urchin_stats <- urchin %>% 
  group_by(site) %>% 
  summarize (mean = mean(size, na.rm=T),
             sd = sd(size, na.rm=T),
             se = sd((size) / sqrt(n()), na.rm=T),
             sample_size = n())
# kable of urchin stats

urchin_stats %>% 
  kable (col.names = c("Site", "Mean", "Standard Deviation", "Standard Error", "Sample Size"),
         digits=2) %>%
  kable_styling(
    bootstrap_options = c("striped", "hover", "condensed", "responsive"), 
    full_width=T,
    position="center") 


```
<br>
<br>
```{r}
# exploratory graphs of purple urchin size distributions at each site

ggplot (urchin, aes(x=size)) +
  geom_density(aes(fill=site), show.legend=F) +
  labs ( x= '\nSize (cm)', 
         y = 'Density\n', 
         title="Sampling distribution of purple urchins",
         subtitle = "2008-2018\n") +
  scale_x_continuous(expand = c(0,0),
                     lim = c(0,10),
                     breaks = seq(0,10, by=2)) +
  scale_y_continuous(expand = c(0,0)) +
  theme_bw() +
  facet_wrap(~site, scales="free_x") +
  theme(plot.title = element_text(hjust=0.5),
        panel.spacing=unit(2, "lines"),
        plot.subtitle = element_text(hjust=0.5))



```

Observed trends in sampe distribution graphs:

- Urchin sizes at Arroyo Quemado, Isla Vista, and Mohawk appear normally distributed, while urchin sizes are slightly right skewed.


<br>

Observations of exploratory graphs:

- Distribution of sizes appears to be normal for all five sites.
- There is a large sample size for all sites, so it is unlikely that all urchins at the sites fall within these bounds. These gaps in the graph are likely due to sampling error --> check metadata

```{r, include = F}
# is there a significant difference in mean purple urchin size in control treatments at the 5 sites

# H null: there is no significant difference in mean purple urchin size in control treatments at the 5 sites
# H alt: there is a significant difference in mean purple urchin size in control treatments at the 5 sites

# anova
urchin_aov <- aov(size ~ site, data=urchin)
summary(urchin_aov)
# p<2e-16

# tukey
urchin_tukey <- TukeyHSD(urchin_aov)
urchin_tukey

# results: all sites have a significant difference except for 

# To report your one-way ANOVA, need to unlist the outcomes isla vista and carpenteria

urchin_outputs <- unlist(summary(urchin_aov))
urchin_outputs

# Create a data frame of Tukey to reference p-values through in-line referencing

df <- data.frame(urchin_tukey$site)
df
```

<br>

To determine whether or not there is a difference in total mean sizes for purple urchins (*Strongylocentrotus purpuratus*) found in five different sites along the Santa Barbara coast, a one-way ANOVA test was performed. The results  F(`r urchin_outputs[1]`, `r urchin_outputs[2]`) = `r round(urchin_outputs[7],2)`, *p* = `r round(urchin_outputs[9],4)`) indicate that there is a significant difference between at least two of the mean sizes for purple urchins in different sites A Tukey's HSD post-hoc test was performed to determine which sites had purple urhcin populations with significant differences between total mean sizes. The results show that there is a significant difference in all but one pairwise comparison: total mean sizes between purple urchins in Isla Vista (`r round(urchin_stats$mean[3], 2)`cm $\pm$ `r round(urchin_stats$sd[3], 2)`cm)(mean $\pm$ SD) and Carpinteria (`r round(urchin_stats$mean[2], 2)`cm $\pm$ `r round(urchin_stats$sd[2], 2)`cm)(*p* = `r round(df$p.adj[5],3)`). The actual difference between the population means was 0.08cm.

All other pairwaise comparisons had significant differences in purple urchin mean sizes(*p* = `r round(df$p.adj[1],3)`). Mohawk had the largest mean population size (`r round(urchin_stats$mean[4], 2)`cm $\pm$ `r round(urchin_stats$sd[4], 2)`cm)(mean $\pm$ SD), which was 0.71cm, 1.11cm, 1.03cm, and 1.23cm larger than mean population sizes ar Arroyo Quemado (`r round(urchin_stats$mean[1], 2)`cm $\pm$ `r round(urchin_stats$sd[1], 2)`cm) (mean $\pm$ SD), Carpinteria (`r round(urchin_stats$mean[2], 2)`cm $\pm$ `r round(urchin_stats$sd[2], 2)`cm), Isla Vista (`r round(urchin_stats$mean[3], 2)`cm $\pm$ `r round(urchin_stats$sd[3], 2)`cm)(mean $\pm$ SD), and Naples (`r round(urchin_stats$mean[5], 2)`cm $\pm$ `r round(urchin_stats$sd[5], 2)`cm) (mean $\pm$ SD), respectively. Difference in mean urchin sizes at the other sites, Carpinteria, Isla Vista, and Naples, was 0.4cm, 0.32cm, and 0.52cm. The difference in mean sizes for urchins in Naples from Isla Vista and Carpinteria was 0.2cm and 0.12cm.







