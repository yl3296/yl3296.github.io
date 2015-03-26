---
layout: post
title: Bad Chart Critique
---
####Introduction
This map was published by the Bureau of Labor Statistics (BLS), and used in a recent article in Vox. Vox took this map an dget the conclusion that Mississippi has the worst unemployment rate. 
Source: http://www.vox.com/2014/8/18/6032965/mississippi-unemployment-highest-state
The following analysis is based on dataset on BLS: http://www.bls.gov/web/laus/laumstrk.htm

Here should be the original chart
![badchart](https://cloud.githubusercontent.com/assets/10662777/6846631/0c197038-d396-11e4-9c60-e0fdc9555562.gif)

*Here comes my critique:
1. One state in black,purple and dark brown and only one state in red. The range is unreasonable then.Three lable is useless and the remaining 4 lable is not clear enough to give a distribution of the rate.
2. Map is good to give a statebased picture, but not accurate when we need to compare different states.
3. The conclusion that Mississippi is the worst can not be get. Search the dataset, we find hte worst rate is Puerto Rico's 13.1%. But both BLS and Vox omitted it in the graph.


*Here comes my improve:
1. Rearrange the color range to 1)include all the rate value; 2) every range has rates fell in it.  I use the most recent data set for Jan 2015 to plot by ggplot. 

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
2. Make a bar chart in increasing order to have a clear mind of rate in different state. It's pretty important when we want to compare states. By this, we can easily read the lowest and highest unemployment rate. 

Here comes my barchart.
![bar](https://cloud.githubusercontent.com/assets/10662777/6846672/63943c26-d396-11e4-99da-41e6c85ac9bf.png)

Bar chart is always a good way to show comparation between classes. 


Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below :point_down:).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
