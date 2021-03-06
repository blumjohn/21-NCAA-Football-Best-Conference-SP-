# 21-NCAA-Football-Best-Conference-SP-

---
title: "Conference Analysis, 2021 NCAA Football using SP+"
author: "John Blum"
date: "1/3/2022"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = FALSE, warning = FALSE, message=FALSE)
```

# Summary
A statistical analysis of SP+ rankings for the 2021 NCAA football season, reveals that the best power 5 conferences in 2021 are:  
1. SEC  
2. Big10   
3. Big12  
4. ACC  
5. Pac12.

# Motivation
Fans often beat their chest in bowl season about the superiority of their conference, with exhibition bowl games provided as proof of superiority or weakness. But conferences are not equal in how and why they make this beat. It's well considered that the SEC is the best conference given it's geographic recruiting talent advantages and representation in recent National Championships in the BCS and playoff eras. Yet bowl and championship season often create interesting fan takes.

For example, when an SEC team loses a bowl game, it's now a meme the excuse is, "We would have won, but we didn't want to be there." And, "It was insulting to be matched up against such a lowly team that doesn't compare to anyone in our conference." Or, "The only reason we lost is because we were tired due to the long grind of the SEC." Never mind that the SEC only plays 8 conference games and other leagues play 9. There is also the fact that these teams chant "SEC, SEC" during their games, something other conferences find odd because they want their rivals to lose. This bowl season, I read a Tennessee fan claiming he didn't care their 3rd place SEC East team lost to 4th place Big10 West team playing walk on wide receivers, because two SEC teams were in the playoff and won. In other words, they'd rather their team lose and their rivals win, than their own team win. Meanwhile, other conferences may or may not want to see their conference teams succeed. Some fans adopt an attitude of "I want my rivals to lose every game, ever," with others wanting their conference to do well but specific rivals lose. Sometimes the take is "The worst place team in the SEC would go undefeated in other conferences. The only reason they sometimes lose bowl games is because ..." the reasons mentioned above. 

Not surprisingly, fanbases from other conferences get defensive - even while acknowledging the success of the SEC - but wanting acknowledgment their leagues too have great football teams, an acknowledgement that rarely comes.

The purpose of this analysis seeks to address which conference was best in 2021, and how competitive and comparative the conferences are aside from any outlying teams. 

# Data
SP+ Data were taken from Bill Connely at ESPN. The data are not directly presented due to subscription rights. Only derivative data products will be shown. This analysis assumes that SP+ is an accurate, comparative representation of football teams. SP+, like many advanced statstical methods, adjusts for things like quality of opponent, or is reflective of how good teams are relative to a hypothetical average teams based on a set of predictive variables.

[Data are avaliable here.](https://www.espn.com/college-football/insider/story/_/id/32986253/college-football-sp+-rankings-following-new-year-six-bowl-games)

In that article, Bill Connely uses the mean SP+ points for each conference to determine best. This analysis goes further and looks at other metrics and the full distribution.

```{r}
library(readxl)
library(knitr)
library(kableExtra)

# Load Data
df <- read_xlsx("spplus.xlsx")

```


# Summary Statistics

SP+ ranks teams according to Points, where Points is expected offense points minus expected defensive points allowed plus expected special teams points (which can be plus or minus). Statistics for points by conference are below.

```{r}
library(tidyverse)
library(moments)
library(bootstrap)
library(magrittr)



df2 <- df %>% group_by(Conference) %>% summarize(Mean=mean(Points),Median=median(Points),
                                                 P25=quantile(Points,0.25),
                                                 P75=quantile(Points,.75),
                                                 SD=sd(Points),
                                                 Skewness=skewness(Points),
                                                 Kurtosis=kurtosis(Points))
                                                 

df2 <- df2[order(-df2$Mean),]

# print table
df2 %>% kbl(,digits=2,caption="2021 SP+ Conference Statistics") %>% kable_styling() 
# save table is image
df2 %>% kbl(,digits=2,caption="2021 SP+ Conference Statistics") %>% kable_styling() %>% save_kable(file="table1.png")


```
![ALT test](sptable1.png)


To make sense of these, consider this primer on statistical metrics.

* **Mean** - The mean is the average of the population.  
* **Median** - The median is the midpoint of the population. This metric is often favored because outliers, such as an exceptionally strong or bad team, can otherwise distort the mean from a reprensentation of the population.   
* **P25** - The 25th percentile, meaning 75% of the population will be above this number.  
* **P75** - the 75th pervcentile, meaning only 25% of the population will be above this number.  
* **SD** - Standard deviation, a measure of the variance around the mean. A higher standard deviation means the population has more variance, e.g. a greater spread of good or bad teams in the conference. A lower standard deviaiton means the population is more concentrated around the mean team in the conference, which may make the conference very competitive with itself but does not guarantee external competitiveness.  
* **Skewness** - A truly symmetrical conference would have a skewness equal to 0. A positive skew means the right hand tail is larger than the left hand (lagging) tail, and a negative skew means the lagging tail is larger than the leading tail. In general, skewness between -0.5 and 0.5 is symmetric; between -1 and -0.5 or 0.5 and 0.1 moderately skewed; and < -1 or > 1 highly skewed. Skewness is rarely a predictive or informative metric.  
* **Kurtosis** - Kurtosis is a measure of the combined weight of the tails relative to the rest of the distribution. The kurtosis for a normal distribution is 3. Data sets with high kurtosis tend to have outliers.  Kurtosis is also rarely a predictive or informative metric.

## Initial Interpretation
The top 5 conferences by mean and their difference from the top (SEC) in () are:
1. SEC - 11.88 (0)  
2. Big 10 - 9.51 (-2.37)  
3. Big 12 - 7.53 (-4.35)  
4. ACC - 6.14 (-5.74)  
5. Pac 12 - 2.60 (-9.28)  

For median, these are
1. SEC - 12.25 (0)  
2. Big 12 - 11.25 (-1)  
3. Big 10 - 10.75 (-1.5)  
4. ACC - 7.4 (-4.85)  
5. Pac 12 - 5.05 (-7.2)  

The median for each conference is actually higher than each mean, meaning these conferences have some bad teams that pull the mean below the median, more than good teams that pull it above. 

The SEC is the best in both metrics, but the Big12 is closer to the SEC in median. 

The SEC is the best conference according to SP+, and ACC and Pac12 are 4th and 5th. **Which conference is 2nd?** According to mean, the Big 10. According to median, Big 12.

Neither metric is necessarily correct. The Big 12 is more highly skewed compared to the Big10 - arguably because Kansas is very bad. The Big 12 also has a higher kurtosis, meaning more weight of its tails - also possibly because Kansas is very bad.

What is true is the medians of the SEC, Big10, and Big12 are close - meaning these leagues as a whole should be competitive with each other and any betterness is marginal if outlying teams are excluded. Of course, it's not exactly fair to remove outliers from one conferenc and not the others (i.e. remove the top 2 teams and compare). 

Visualitions of these populations may be more illuminating than raw numbers.

# Visualization
Two compare conferneces, the populations can be plotted. Two methods are a ridgeline plot and a boxplot.

* **Ridgeline plot** - This essentially shows a smoothed histogram of each conference on the y axis with SP+ points on the X-axis. This will show the spread of each conferences team along the SP+ points dimension.  
* **Boxplot** - This plot is more difficult to initilaly read but more informative. It contains a box surrounding the 25th and 75h percentiles, or the middle 50% of the confernece. The median for that conference is within the box in a vertical bar. The individual teams are plotted along the X-axis as small points. Lines extend outside the boxes representing the expected population minimum and maximums.Finally, statistical outliers are plotted as red circles. [A good primer on boxplot is here.](https://www.data-to-viz.com/caveat/boxplot.html)   


### Ridgeline Plot

```{r}
library(ggplot2)
library(ggridges)

ggplot(data=df,aes(x=Points,y=reorder(Conference,Points),fill=Conference)) + geom_density_ridges() + theme_ridges() + ylab("Conference") + labs(caption="SP+ Rankings")
ggsave("sp_ridgeview.png")

```
![ALT test](sp_ridgeview.png)

This plot shows a smoothed density of teams with the y-axis indicating the relative concentraion of teams around that pionts value. In this plot, the SEC and Big10 look similar, with simlar peaks and tails, the SEC a bit wider and fatter. The Big12 in this reprensetation appears to have a peak higher than the SEC and Big10, but a short tail. It's highly left skewed with a collection of very good but not elite teams. 


### Boxplot
A boxplot may show more information. 
```{r,fig.height=5.5,,fig.width=8,dpi=300}
# ggplot(data=subset(df,Conference %in% c("SEC","Big10","Big12","ACC","Pac12")),aes(x=Points,fill=Conference,color=Conference)) + geom_density(alpha=0.1) + facet_wrap(~Conference) 
# 
# ggplot(data=df,aes(x=Points,fill=Conference,color=Conference)) + geom_density(alpha=0.5) + facet_wrap(~Conference)

# Boxplot
ggplot(data=df,aes(x=Points,y=reorder(Conference,Points),fill=Conference)) + geom_boxplot(alpha=0.8,outlier.shape=16,outlier.size=4,outlier.color="red") + ylab("Conference") + geom_point(alpha=0.25) + labs(title="SP+ by conference", subtitle="POST CFP Semifinal",caption=str_wrap("Boxes span the 25th to 75th percentile. The vertical line in the boxes is the median. Lines extending from the boxes span the expected minimum and maximum for the population. Points mark the individual teams within the conference. Red circles are statistical outliers from the conference's population.",110)) + theme(plot.caption=element_text(hjust=0))

ggsave("sp_boxplot.png")



```
![ALT test](sp_boxplot.png)


Again, the box represents the 25th to 75th percentiles, dots are the individual teams, the vertical line in the box the median of that conference's population, and the horizontal lines the expected statistical minimum and maximums. Points outside those lines are plotted as red circles indicating outliers.

Focussing on the Power 5, the SEC has outliers as red circles in Georgia (good) and Vanderbilt (terrible). The Big 10 has outliers in Ohio State (good) and Northwestern. The Big 12 has a negative outlier in Kansas, and the ACC in Duke. 

Looking at just the boxes, the SEC and Big 10 have similar width boxes, with the SEC slightly to the right of the Big 10 and the Big 12. The medians are all similar. This means the middle of these conferences will be highly competitive with each other, with a slight but not substantial edge to the SEC. 

The Big 12 is tightly bunched with many teams competitive with each other.

# Division Analysis
The above are conferences as a whole, but some of the P5 conferences have divisions. The anlalysis will be redone for P5 conferences only comparing divisions if applicable.

```{r}

df3 <- df %>% filter(Conference %in% c("SEC","Big10","Big12","ACC","Pac12")) %>% group_by(Division) %>% summarize(Mean=mean(Points),Median=median(Points),
                                                 P25=quantile(Points,0.25),
                                                 P75=quantile(Points,.75),
                                                 SD=sd(Points),
                                                 Skewness=skewness(Points),
                                                 Kurtosis=kurtosis(Points))
                                                 

df3 <- df3[order(-df3$Mean),]

# plot table
df3 %>% kbl(,digits=2,caption="2021 SP+ Conference Statistics by Division") %>% kable_styling()
# save table as image
df3 %>% kbl(,digits=2,caption="2021 SP+ Conference Statistics by Division") %>% kable_styling() %>% save_kable(file="table2.png")


# Density Polot
ggplot(data=subset(df,Conference %in% c("SEC","Big10","Big12","Pac12","ACC")), aes(x=Points,y=reorder(Division,Points),fill=Division)) + geom_density_ridges() + theme_ridges() + ylab("Division") + labs(caption="SP+ Rankings by Division")

ggsave("div_ridgeview.png")

# Boxplot
ggplot(data=subset(df,Conference %in% c("SEC","Big10",
                                        "Big12","Pac12",
                                        "ACC")),aes(x=Points,y=reorder(Division,Points),fill=Division)) + geom_boxplot(alpha=0.8,outlier.shape=16,outlier.size=4,outlier.color="red") + ylab("Conference") + geom_point(alpha=0.25) + labs(title="SP+ by Division", subtitle="POST CFP Semifinal",caption=str_wrap("Boxes span the 25th to 75th percentile. The vertical line in the boxes is the median. Lines extending from the boxes span the expected minimum and maximum for the population. Points mark the individual teams within the conference. Red circles are statistical outliers from the conference's population.",110)) + theme(plot.caption=element_text(hjust=0))

ggsave("div_boxplot.png")

```
![ALT test](sptable2.png)

![ALT test](div_ridgeview.png)

![ALT test](div_boxplot.png)



### Interpretation of Divisions
The SEC west is the best division with the highest Mean and Median values, followed by the Big10 East. However, the Big10 East has a higher P75 value, with Ohio State rated a bit better than Alabama and Michigan greater than Ole Miss. The Big10 East does have laggards this year (Rutgers is improving, Indiana was expected to be good but after early injuries, collapsed from their strong 2020 season.) Those divisions are followed by the ACC Atlantic, SEC East, and Big10 West. 

The SEC East is interesting because Georgia is no longer classified as an outlier given the smaller population, but Vanderbilt still is. They both likely skew the mean and median given the small sample size. On a purpely mean and median level, the SEC east is comparable to the Big10 West. The ACC Atlantic appears better on a mean basis due to the lack of a bad team to pull down their result. Aside from Georgia, the SEC East is arguably equal to the Big10 West and the ACC Atlantic this year. 

Both Pac12 Divisions are the lowest. 

The Big 12 as a whole drops as it doesn't have divisions. It would be 2nd best based on median, but it's mean is held down by Kansas. Assuming it did have divisions, and Kansas was randomly assigned to only one, at least one division would rise towards the top. This could be done via a bootstrap method: create a Kansas division, and randomly assign 4 other teams to that division, and 5 to the other. Do this 100 times and calculate the results. 

I have not yet done this exercise.

```{r}
# Bootstrap -->

fmean <- function(x){mean(x)}
fmedian <- function(x){median(x)} 
fp25 <- function(x){quantile(x,.25)}
fp75 <- function(x){quantile(x,.75)} 

 # for(conf in c("SEC","Big10","Big12","ACC","Pac12")){ -->
# <!--   print(conf) -->                                           
# <!--   tf <- df %>% filter(Conference==conf) -->
# <!--   results <- bootstrap(tf$Points,100,fmean) -->
# <!--   print(mean(results$thetastar)) -->
# <!--    results <- bootstrap(tf$Points,100,fmedian) -->
# <!--   print(mean(results$thetastar)) -->
# <!-- #  results <- bootstrap(tf$Points,100,fp25) -->
# <!-- #  mean(results$fp25) -->
# <!-- #    results <- bootstrap(tf$Points,100,fp75) -->
# <!-- #  mean(results$fp75) -->

```



# Discussion
It's not possible to remove teams from a conference in the discusion of conference strength, either good or bad. The SEC is the best confernece. It's also the most top heavy, followed by the Big 10. The core of the SEC may be marginally better than the the Big 10 or Big 12, but it's slight. Those conferences would all be comptitive with each other minus Georgia and Alabama, or if the top 2 are taken away from any conference. It comes down to recruiting, and Georgia, Alabama and Ohio State have a massive recruting and talent advantage over any other team, including in the SEC.

The SEC does have fewer bottom teams than the Big 10, and is more comparable in that area to the Big 12 which only has one significantly below average team. However, Vanderbilt and Kansas are significantly worse than Northwestern, and Northwestern has a recent history of success. Further, the SEC only plays 8 conference games, whereas those other conferneces play 9, often substituting an FCS opponent prior to rivalry week. It's not known what effect an extra conference game would have on these distributions given they are already opponent adjusted but poorly sampled.



# Conclusion
The SEC was the best conference in 2021, followed by the Big 10 and the Big 12 was third. It's tempting to put the Big 12 second due to it's slightly higher median and higher P75 value, but the Big 10 has three teams ranked higher than the top Big 12 team, giving it the edge. 

What's also true is if the top teams in the league are removed, these leagues are competitive with each other. This is why they often produce great bowl matchups. A losing team has no excuses. Ball players like to play ball, and winners like to win. Any excuse that a team lost because they didn't want to be there is actually an acknowledgement that despite talent, the team is soft. 

