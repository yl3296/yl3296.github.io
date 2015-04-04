
---
layout: post
title: Bad Chart Critique
![iamge3](https://cloud.githubusercontent.com/assets/10662777/6994617/63cdc11c-daee-11e4-8b0d-846925396713.jpg)
---


####Introduction
This article presents a critique of heatmap, a chart type commonly used to display value by colours,highlighting how they are poorly designed to effectively communicate in the underlying data, and presents a number of more effective alternatives.
Basically, heat maps allow users to understand and analyze complex data sets with large number of classes. There can be many ways to display heatmaps, but they all share one thing in common -- they use color to communicate relationships between data values that would be would be much harder to understand if presented numerically in a spreadsheet.

####My Bad Chart
![image7](https://cloud.githubusercontent.com/assets/10662777/6994620/83f133ca-daee-11e4-9e9d-d95952a2ba21.jpg)
This map was published by the Bureau of Labor Statistics (BLS), and used in a recent article in Vox. Vox took this map and get the conclusion that Mississippi has the highest unemployment rate. 
Source: http://www.vox.com/2014/8/18/6032965/mississippi-unemployment-highest-state
The following analysis is based on dataset on BLS: http://www.bls.gov/web/laus/laumstrk.htm

Here's the original chart
![badchart](https://cloud.githubusercontent.com/assets/10662777/6846631/0c197038-d396-11e4-9c60-e0fdc9555562.gif)
![iamge3](https://cloud.githubusercontent.com/assets/10662777/6994617/63cdc11c-daee-11e4-8b0d-846925396713.jpg)

#####What's the problem? 

1. Three colour of legend do not show up at all and one state is in red. Obviosly, the range is unreasonable. Three lables are useless and the remaining four lables are not clear enough to give a distribution of the rate.
2. Map is good to give a state based picture, but not accurate when we need to compare different states.In this article, the author aimed to know the highest unemployment rate. Generally, we can not get this informaiton by a heatmap.
3. Look at the dataset. We find Mississippi is not the worst one. Both BLS and Vox omitted the Puerto Rico,  which has the highest unemployment rate 13.1%. Then the conclusion is of course wrong.
![image](https://cloud.githubusercontent.com/assets/10662777/6850372/30908190-d3af-11e4-8483-a772997f75a7.png)



####Let's improve it
![image8](https://cloud.githubusercontent.com/assets/10662777/6994621/8d0c0336-daee-11e4-92fc-3edb09b43ade.jpg)

1. Reset the color range: 1)include all the rate value; 2) every range has values fell in it.  
Then I use the most recent data set in Jan 2015 to improve by ggplot. 
![map_rplot](https://cloud.githubusercontent.com/assets/10662777/6846640/2eb61268-d396-11e4-928b-97de8ded807d.png)

Here 's the R code:

```
library(XML);
library(ggplot2);
library(maps);
library(plyr);

# read the data from the bls website with correct column formats
unemp = readHTMLTable('http://www.bls.gov/web/laus/laumstrk.htm', colClasses = c('character', 'character', 'numeric'))[[2]];

# rename columns and convert region to lowercase
names(unemp) = c('rank', 'region', 'rate');
unemp$region  = tolower(unemp$region);

# get us state map data and merge with unemp
us_state_map = map_data('state');
map_data = merge(unemp, us_state_map, by = 'region'); 

# keep data sorted by polygon order
map_data = arrange(map_data, order);

# plot map using ggplot2

p0 = ggplot(map_data, aes(x = long, y = lat, group = group)) 
p0 + ggtitle("Unemployment Rate (July 2014)") + geom_polygon(aes(fill = cut_number(rate, 5))) +geom_path(colour = 'grey', linestyle = 2)+coord_map()+scale_colour_brewer(palette='Set1')+ scale_fill_hue(l=80);
```

2. Make a bar chart with sorting.Then we can easily  see the maximum, minimum and compare different states. Basically,Bar chart is always a better way to show comparation between classes than pie chart.
Here comes my barchart.

![bar](https://cloud.githubusercontent.com/assets/10662777/6846672/63943c26-d396-11e4-99da-41e6c85ac9bf.png)



####Some thoughts
1. Bar chart is an excellent tool to see the maximum, minimum and compare any several classes. It still works when the number of classes is getting large. Just a little less cute.
2. Heatmap shows its priority with large number of classes. But not so good in comparing.
3. Heatmap is easily to plot in ggplot, D3 and Excel.
4. I tried making one more kind of heatmap. It also works good to tell the rate distritbution.

![treemap](https://cloud.githubusercontent.com/assets/10662777/6853376/41a290b0-d3c1-11e4-80ae-34f2958c30a1.jpg)

![image2](https://cloud.githubusercontent.com/assets/10662777/6994626/af2cde22-daee-11e4-81d6-5084bb82a61f.png)




