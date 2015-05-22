---
layout: post
title: "Bad Chart Critiuqe"
author: "Yang(Claire) Liu"
date: "March 26, 2015"
---
###Introduction
This article presents a critique of heatmap, a chart type commonly used to display value by colours,highlighting how they are poorly designed to effectively communicate in the underlying data, and presents a number of more effective alternatives.
Basically, heat maps allow users to understand and analyze complex data sets with large number of classes. There can be many ways to display heatmaps, but they all share one thing in common -- they use color to communicate relationships between data values that would be would be much harder to understand if presented numerically in a spreadsheet.

###My Bad Chart
This map was published by the Bureau of Labor Statistics (BLS), and used in a recent article in Vox. Vox took this map and get the conclusion that Mississippi has the highest unemployment rate. 
Source: http://www.vox.com/2014/8/18/6032965/mississippi-unemployment-highest-state
The following analysis is based on dataset on BLS: http://www.bls.gov/web/laus/laumstrk.htm

Here's the original chart
![badchart](https://cloud.githubusercontent.com/assets/10662777/7760126/22aaf12e-ffe6-11e4-9799-ebcf32b11c53.gif)


####What's the problem? 

1. Three out of the seven colors are not found on the map at all and only one state is in red. Three lables are useless and the remaining four lables are not clear enough to give a distribution of the rate. In other words, the legend range is unreasonable.
2. Map is good to give a state based picture, but not accurate when we need to compare different states.In this article, the author aimed to know the highest unemployment rate. Generally, heatmap is not an accuracy way to get this kind of information.
3. Let's look at the dataset. Obviously, Mississippi is not the worst one. Both BLS and Vox omitted the Puerto Rico,  which has the highest unemployment rate 13.1%. Then the conclusion is of course wrong.


![image](https://cloud.githubusercontent.com/assets/10662777/6850372/30908190-d3af-11e4-8483-a772997f75a7.png)



#####Let's improve it


**1. Reset the color range:**

1) make sure the color range include all the rate value; 

2) every range has at least one value falling in it.

Then I use the most recent data set in March 2015 to improve by ggplot.

![final_map](https://cloud.githubusercontent.com/assets/10662777/7759284/1a906c10-ffde-11e4-8a40-ed0ba52af2cc.png)

Here's the [dataset](https://github.com/yl3296/yl3296.github.io/blob/master/data/data_ggplot.csv)

Here's the R code:

```
library(ggplot2)
library(plyr)
library(maps)

unemp <- read.csv('unem_rate.csv', head = TRUE)
names(unemp) <- c('rank', 'region', 'rate')

unemp$region <- tolower(unemp$region)
us_state_map <- map_data('state')
map_data <- merge(unemp, us_state_map, by = 'region')
map_data <- arrange(map_data, order)
states <- data.frame(state.center, state.abb)

p1 <- ggplot(data = map_data, aes(x = long, y = lat, group = group))
p1 <- p1 + geom_polygon(aes(fill = cut_number(rate, 7)))
p1 <- p1 + geom_path(colour = 'gray', linestyle = 2)
p1 <- p1 + scale_fill_brewer('Unemployment Rate (Mar 2015)', palette  = 'PuRd')
p1 <- p1 + coord_map()
p1 <- p1 + geom_text(data = states, aes(x = x, y = y, label = state.abb, group = NULL), size = 2)
p1 <- p1 + theme_bw()
p1
# Save the dataset as csv. It will be used in the following interactive map by leafletR.
write.csv(map_data, file = "map_unem.csv")

```

**2. Make it iteractive by leafletR**

  Here's the [dataset](https://github.com/yl3296/yl3296.github.io/blob/master/data/data_leaflet.csv)
  
  Here's the code :
  
```
library(ggplot2)
library(plyr)
library(maps)
library(leafletR)

unemp <- read.csv("map_unem.csv", head = TRUE)  

unem_data <- toGeoJSON(data = unemp, dest = tempdir(),name = "Unemployment")

unem_style <- styleGrad(prop="rate", breaks=seq(2, 8, by=1), 
                     style.val=rev(heat.colors(6)), leg="Unemployment Rate", fill.alpha=0.7)
popup = c("rate")
map <- leaflet(data=unem_data, dest = tempdir(), title=" 2015 Unemployment Rate", base.map= "osm", style=unem_style, popup="popup")
```

**3. Make a bar chart in order to find the state with highest unemployment rate**

One of the goal of the article in Vox is to find the state with highest rate, then the most straight forward method is barchart. By doing it, we can easily  see the maximum, minimum and compare different states. Basically, Bar chart is always a better way to make comparation between classes than pie chart.
Here comes my barchart.

![bar](https://cloud.githubusercontent.com/assets/10662777/6846672/63943c26-d396-11e4-99da-41e6c85ac9bf.png)

**4. Look at trends by line chart and barchart**

Basically, heatmap shows status at a time point without trend information. To know more about the trend and compare between years, I made the following line chart and bar chart using unemployment rate of California. 

In the line chart, we can clearly see the the unemployment rate greatly increased since 2008 and reached the peak in about 2010. After 2010, it began to gradually decrease but still in a relatively high level. 

In the bar chart, we can see the trend in two elements: hight of the bar and the gradient color. The same conclusion can be get.


![final_line](https://cloud.githubusercontent.com/assets/10662777/7760208/ff81ae44-ffe6-11e4-9c2e-44c740131031.png)
![final_bar](https://cloud.githubusercontent.com/assets/10662777/7760225/341920e2-ffe7-11e4-98f7-45974eac2386.png)


Here's the R code:

```
library(ggplot2)
library(KernSmooth)
library(plyr)
library(scales)
library(mgcv)

unem_cali <- read.csv(file="/Users/ClaireL/Documents/W4701Visual/Blogcritique/unem_cali.csv",head=TRUE)
p1 <- qplot(Year, Unemployment_rate, data = unem_cali, geom = c("point","smooth"),method = "gam",formula = y~s(x))
p2 <-qplot(Year, Unemployment_rate, data = unem_cali, geom = "histogram",stat="identity",binwidth=0.1,fill = Unemployment_rate)
```

###Some thoughts
1. Bar chart is an excellent tool to see the maximum, minimum and make comparision. It still works when the number of classes is getting large. But a little less cute.
2. Heatmap shows its priority with large number of classes. 
3. Nice tool to present iteractive maps: leaflet, leafletR, ggmap and rMaps.







