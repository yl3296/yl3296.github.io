---
title: "Bad Chart Critiuqe"
author: "Claire Liu (yl3296)"
date: "March 26, 2015"
---
####Introduction
This article presents a critique of heatmap, a chart type commonly used to display value by colours,highlighting how they are poorly designed to effectively communicate in the underlying data, and presents a number of more effective alternatives.
Basically, heat maps allow users to understand and analyze complex data sets with large number of classes. There can be many ways to display heatmaps, but they all share one thing in common -- they use color to communicate relationships between data values that would be would be much harder to understand if presented numerically in a spreadsheet.

####My Bad Chart
This map was published by the Bureau of Labor Statistics (BLS), and used in a recent article in Vox. Vox took this map and get the conclusion that Mississippi has the highest unemployment rate. 
Source: http://www.vox.com/2014/8/18/6032965/mississippi-unemployment-highest-state
The following analysis is based on dataset on BLS: http://www.bls.gov/web/laus/laumstrk.htm

Here's the original chart
![badchart](https://cloud.githubusercontent.com/assets/10662777/6846631/0c197038-d396-11e4-9c60-e0fdc9555562.gif)


#####What's the problem? 

1. Three colour of legend do not show up at all and one state is in red. Obviosly, the range is unreasonable. Three lables are useless and the remaining four lables are not clear enough to give a distribution of the rate.
2. Map is good to give a state based picture, but not accurate when we need to compare different states.In this article, the author aimed to know the highest unemployment rate. Generally, we can not get this informaiton by a heatmap.
3. Look at the dataset. We find Mississippi is not the worst one. Both BLS and Vox omitted the Puerto Rico,  which has the highest unemployment rate 13.1%. Then the conclusion is of course wrong.
![image](https://cloud.githubusercontent.com/assets/10662777/6850372/30908190-d3af-11e4-8483-a772997f75a7.png)



####Let's improve it


1. Reset the color range: 1)include all the rate value; 2) every range has values fell in it.  
Then I use the most recent data set in Jan 2015 to improve by ggplot. 
![rplotmap](https://cloud.githubusercontent.com/assets/10662777/7014656/a10ac9f0-dc95-11e4-9fc0-f6efc9b1dbfe.png)

Here 's the R code:

```
library(ggplot2)
library(scales)
library(maps)

unemp <- read.csv("/Users/ClaireL/Documents/W4701Visual/Blogcritique/unemployment2015.csv", header = F, stringsAsFactors = F)
names(unemp) <- c("id", "state_fips", "county_fips", "name", "year", 
                  "?", "?", "?", "rate")
unemp$county <- tolower(gsub(" County, [A-Z]{2}", "", unemp$name))
unemp$state <- gsub("^.*([A-Z]{2}).*$", "\\1", unemp$name)

county_df <- map_data("county")
names(county_df) <- c("long", "lat", "group", "order", "state_name", "county")
county_df$state <- state.abb[match(county_df$state_name, tolower(state.name))]
county_df$state_name <- NULL

state_df <- map_data("state")

choropleth <- merge(county_df, unemp, by = c("state", "county"))
choropleth <- choropleth[order(choropleth$order), ]

choropleth$rate_d <- cut(choropleth$rate, breaks = c(seq(0, 10, by = 2), 35))

ggplot(choropleth, aes(long, lat, group = group)) +
  geom_polygon(aes(fill = rate_d), colour = alpha("white", 1/2), size = 0.2) + 
  geom_polygon(data = state_df, colour = "white", fill = NA) +
  scale_fill_brewer(palette = "PuRd")

```

2. Make a bar chart with sorting.Then we can easily  see the maximum, minimum and compare different states. Basically,Bar chart is always a better way to show comparation between classes than pie chart.
Here comes my barchart.

![bar](https://cloud.githubusercontent.com/assets/10662777/6846672/63943c26-d396-11e4-99da-41e6c85ac9bf.png)

3. To know more about the trend and compare between years, I use line chart and bar chart.

![cali_line](https://cloud.githubusercontent.com/assets/10662777/7014664/bbfb27e6-dc95-11e4-87f0-20f062218e59.png)
![rplot_cali](https://cloud.githubusercontent.com/assets/10662777/7014673/d2a45684-dc95-11e4-92d9-d43e229064f3.png)

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

####Some thoughts
1. Bar chart is an excellent tool to see the maximum, minimum and compare any several classes. It still works when the number of classes is getting large. Just a little less cute.
2. Heatmap shows its priority with large number of classes. But not so good in comparing.
3. Heatmap is easily to plot in ggplot, D3 and Excel.
4. I tried making one more kind of heatmap. It also works good to tell the rate distritbution.

![treemap](https://cloud.githubusercontent.com/assets/10662777/6853376/41a290b0-d3c1-11e4-80ae-34f2958c30a1.jpg)





