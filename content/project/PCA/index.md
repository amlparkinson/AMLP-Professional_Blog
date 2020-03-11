---
date: "2019"
#external_link: http://example.org
image:
  caption: 
  focal_point: Smart
summary: Conducting PCA to examine relationships between variables potentially contributing to fire induced conifer mortality. 
tags:
#- Demo
title: Principal Component Analysis
---


# Background 

I am currently working on a project  to understand how topographic and climatic variables affected mortality of endemic Big-cone (*Pseudotsuga macrocarpa*) conifers in Los Padres National Forest after the 2007 Zaca Fire. Knowing how these variables interact to affect fire-induced mortality will help guide restoration efforts by prioritizing out-planting in areas most likely to survive future fires. ArcGIS was used to gather mean values for topographic variables such as elevation, aspect, topographic wetness index (TWI), and topographic position index (TPI) and climate variables; weather data was obtained as 30-year normals from the PRISM Climate Group^1^.  Here I will be exploring relationships between these environmental variables. 

# PCA 
```{r, warning = F}

#sub-data: only include columns of explanatory variables 

bc_sub <- bc %>% 
  select (elev:tpi, vpd_Aug, twi, flow, vpd_jan, cwd_2) %>% 
  mutate(flow = as.numeric(flow)) %>% 
  drop_na() %>% 
  rename ('Elevation' = 'elev',
          'Slope' = 'slope',
          'TPI' = 'tpi',
          'Annual Precipitation' = 'precip',
          'Solar Radiation' = 'sr',
          'Flow Accumulation' = 'flow',
          'TWI' = 'twi',
          'CWD' = 'cwd_2',
          'Temperature' = 'temp',
          'VPD (Jan)' = 'vpd_jan',
          'VPD (Aug)' = 'vpd_Aug')

# run PCA 
z_bc_pca <- prcomp(bc_sub, scale = T)

```

```{r, echo = F, warning = F, include = T, results = 'asis'}

#plot PCA to visualize interactions 

autoplot(z_bc_pca, 
         colour = NA, 
         loadings.label = T,
         loadings.label.size = 3,
         loadings.label.colour = "black",
         loadings.label.repel = T) +
  labs(title = "PCA of Envrionmental Variables for Big-Cone Forests in Zaca Fire Scar") +
  theme_minimal() +
  scale_x_continuous(lim = c(-0.07, 0.1),
                     expand = c(0,0)) +
  scale_y_continuous(lim = c(-0.1, 0.1),
                     expand = c(0,0))
```


***Figure 1:** PCA exploring the relationships between various topographic and climatic variables that could influence fire-induced mortality of *P. macrocarpa*. 68.3% of variance is explained by the first two principal components.*  

<br> 

# Results:

- Temperature (i.e. mean maximum August temperature),VPD (August), and climatic water deficit (CWD) have a very strong positive relationship with each other and a very strong negative relationship with annual precipitation. Both CWD and VPD depend on available water in the ecosystem. Thus, this strong trend is not surprising considering that one of the hottest months of the year in which there likely has not been rainfall in months is correlated with high VPD and high CWD.  
- Elevation has a strong negative relationship with temperature, VPD (August and January), and CWD, yet a moderate positive relationship with annua precipitation. As elevation increase, temperatures cool which would in turn redcude VPD and CWD. 
- Slope has a very strong negative relationship with flow accumulation and topographic wetness index (TWI). Slope is a component used to calculate TWI^2^. High values of TWI and flow accumulation (where water accumulates) indicate wetter areas. A decrease in slope would be associated with higher TWI and flow accumulation values.


<br>

# Citations

1. PRISM Climate Group, Oregon State University, http://prism.oregonstate.edu, created 4 Feb 2004.
2. Grabs, T., Seibert, J., Bishop, K., Laudon, H. (2009). Modeling spatial patterns of saturated areas: A comparison of the topographic wetness index and a dynamic distributed model. Journal of Hydrology. 373(1-2):15-23.


