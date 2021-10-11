# Time Series Analysis of Eve Online Participation Data

In this project, we were asked to look at the raw war participation data for the various large groups of players involved in The Beeitnam War (also known as "World War Bee 2") in Eve Online, and attempt to forecast the next few weeks. Additionally, the client wanted to see if the historical numbers could tell us anything about the nature of the individual major groups involved.  

For the players involved in the war, it's clear from the recent numbers that since the siege of 1DQ began, Imperium participation is rising, while PAPI (a coalition composed of primarily Legacy and Panfam) participation is falling. But what (if anything) does the past and current data predict about future participation? Using time series forecasting techniques, we can answer that question.

You can find more information about Eve Online and the Beeitnam War [here](https://www.polygon.com/2020/9/15/21436851/ever-online-world-war-bee-2-interview-the-mittani-vily "Polygon Article on WWB2").

## Understanding Player Participation in Eve

The first step in any analysis is to understand your data. Using a common-sense approach that should ring true for any long-time player, we know:

1. **Fleet participation is partly based upon previous participation.** When a group is successful (winning objectives, winning the ISK war, or even just making a "good" attempt at one of the two), the next fleet has a greater chance of having more participants. This sort of success feeds upon itself, and of course, the reverse is true as well. This is why "failscades" are a thing. Swings and momentum are extremely important in Eve.

2. **N+1 matters.** Absolute numbers are king- if an alliance can field more members, then they typically have a better chance of winning. This is why measuring participation in anything other than raw numbers doesn't make sense. We know this intuitively- a fleet of 100 that adds 100 with another ping is more valuable than a fleet of 25 that adds 25, even if the percentage change is the same. This was the biggest problem (among many...) I had with u/holyyakker 's analysis- showing whether or not a group's participation percentage increase/decrease is statistically significant is not important, due to the nature of N+1.

3. **Not all alliances are equal.** Every alliance and corp has it's own individual culture that factors into fleet turnout. An alliance could have a historically high participation rate based upon motivation, culture, or pap requirements. Trying to draw direct comparisons between groups without accounting for these factors doesn't work, because you assume that each group has the same motivations for playing.

4. **Events affect future participation.** Whether it's as simple as a pre-pinged fleet, or the most destructive event in the history of Eve (Massacre at M2), these game-changing events can cause huge changes in participation. We want to factor in a group's capacity to respond to these events in a smart way, instead of trying to just "smooth away" these data points.

5. **Seasonality is important.** COVID and it's associated restrictions had an overall positive effect on player participation across the game. But even seasonal shifts between winter and summer have an effect. We should obviously try to factor this in to our forecast (if possible).

## Forecast Methodology

With the above understanding, we then move to selecting our forecasting model. We'll look at the Imperium, PAPI as a whole, Legacy, Panfam, and Imperium/PAPI without capital ships. For each alliance, we want a purely numbers-based model that factors in prior performance, with more emphasis given to recent numbers (momentum). We also want to factor in how a group responds to event-based changes, all while keeping in mind that each group responds to all of these changes in a different way. For these reasons, we will use an ARIMA (Autoregressive Integrated Moving Average) model. Included in this model is something called autocorrelation, or the degree to which each point in a series is correlated with earlier values in the series.

We'll also include the following events as transformation functions:

* Northern conflict
* War begins
* FWST
* M2
* 1DQ siege

*(Note: Panfam joined the war later than the others, which is reflected in their "War begins" tranformation function)*

Each of these factors affected the groups in slightly different ways, which is taken into account through our modeling. Since we know the interruptions affected player participation, we should not remove them from the model based on significance alone. We should instead use the relative strength/weakness of these values to draw further conclusions.

There are a ton of different ways to choose the variables and their form of impact in these types of models- but luckily, most statistics programs can automatically calculate and choose the best values for you. This is important since as we know, while Eve players as a whole may act in similar ways, each specific group can have different motivating factors and influences for participation.

Finally, the further out you go in your prediction, the greater the chances for error. We will be conservative and forecast 10 weeks out from the latest available data point.

### Why Can't We Just Use a Simple Moving Average or Simple Exponential Smoothing?

The reason people use SMAs for, say, long term stock trading, is because we purposely want to assign less significance to rapid swings and shifts. Clearly, choosing a long term investment based on short-term trends would be a mistake. Additionally, we make the assumption that each point of data within the calculated range is equal (they have the same "reason" behind the value).

These ideas fail when translated to Eve player activity in a common sense way: we all know that game events can and often do lead to rapid shifts in player activity. We also know that activity levels at any given point are less important the further they are away from the present. It makes more sense to gradually decrease the weights placed on older values. We also want to factor in any trends in our data, which SES fails to do.

*Most importantly, SMA and SES are lagging indicators- they don't make predictions, which is what we want to do here.*

## Results

We were able to create models for each of the three major groups individually, PAPI as a whole, and the two factions without caps. The models all have one key value- that of AR, or the Autoregression Coefficient. This value ranges from -1 to 1, but in simple terms: the closer this value is to 1 or -1, the more important the most recent values are. *In Eve terms, this represents how important recent momentum is to each group.*

The modeling software did not see Seasonality as an important factor in any of the groups. This could be contributed to two issues: the lack of length of our data, as well as confounding between war participation, COVID participation and summer participation.

Here are the overall and individual modeling results:

### "Big 3" (Imperium, Legacy, and Panfam)

![Combined ARIMA](https://github.com/antonyebrown/Eve_TimeSeries/blob/main/combined_arima.png)

### Imperium

![Imperium ARIMA](https://github.com/antonyebrown/Eve_TimeSeries/blob/main/imperium_arima.png)

|                   | Imperium | P-values |
|-------------------|----------|----------|
| const             | 4371.722 | 0        |
| northern_conflict | 552.9825 | 0.3725   |
| war_begins        | 1052.669 | 0        |
| fwst              | 975.5213 | 0.0023   |
| m2                | 2003.957 | 0        |
| 1dq_siege         | 478.7347 | 0.252    |
| ar.L1             | 0.677751 | 0        |

### Legacy

![Legacy ARIMA](https://github.com/antonyebrown/Eve_TimeSeries/blob/main/legacy_arima.png)

|                   | Legacy   | P-values |
|-------------------|----------|----------|
| const             | 3210.31  | 0        |
| northern_conflict | 567.3983 | 0        |
| war_begins        | 245.9752 | 0.1695   |
| fwst              | 775.8515 | 0.2621   |
| m2                | 682.5072 | 0        |
| 1dq_siege         | -253.66  | 0.4694   |
| ar.L1             | 0.77897  | 0        |

### Panfam

![Panfam ARIMA](https://github.com/antonyebrown/Eve_TimeSeries/blob/main/panfam_arima.png)

|                   | Panfam   | P-values |
|-------------------|----------|----------|
| const             | 3516.099 | 0        |
| northern_conflict | 254.2358 | 0.4682   |
| war_begins        | 528.3235 | 0.0004   |
| fwst              | 955.3835 | 0.2828   |
| m2                | 455.1407 | 0.2145   |
| 1dq_siege         | -420.838 | 0.1392   |
| ar.L1             | 0.685363 | 0        |

### PAPI

![PAPI ARIMA](https://github.com/antonyebrown/Eve_TimeSeries/blob/main/papi_arima.png)

|                   | PAPI     | P-values |
|-------------------|----------|----------|
| const             | 9580.234 | 0        |
| northern_conflict | 1742.81  | 0        |
| war_begins        | 891.9184 | 0.0151   |
| fwst              | 2054.074 | 0.0968   |
| m2                | 1133.411 | 0.0221   |
| 1dq_siege         | -874.699 | 0.3154   |
| ar.L1             | 0.714206 | 0        |

### Imperium Subcaps

![Imperium Subcaps ARIMA](https://github.com/antonyebrown/Eve_TimeSeries/blob/main/impsub_arima.png)

|                   | Imperium Subcaps | P-values |
|-------------------|------------------|----------|
| const             | 4140.94          | 0        |
| northern_conflict | 507.9919         | 0.1738   |
| war_begins        | 920.6601         | 0        |
| fwst              | 737.4214         | 0.0001   |
| m2                | 1123.722         | 0.0001   |
| 1dq_siege         | 481.2323         | 0.055    |
| ar.L1             | 0.752436         | 0        |

### PAPI Subcaps

![PAPI Subcaps ARIMA](https://github.com/antonyebrown/Eve_TimeSeries/blob/main/papisub_arima.png)

|                   | PAPI Subcaps | P-values |
|-------------------|--------------|----------|
| const             | 8670.415     | 0        |
| northern_conflict | 1500.548     | 0        |
| war_begins        | 866.0177     | 0.001    |
| fwst              | 783.1124     | 0.2688   |
| m2                | 770.1185     | 0.17     |
| 1dq_siege         | -336.776     | 0.5698   |
| ar.L1             | 0.793114     | 0        |

## AR Coefficient Comparison

Comparing the AR coefficients (momentum impact- the closer this value is to 1, the more important the most recent values are) from each group yields interesting results as well: while pvp participation numbers for everyone are affected by more recent numbers, **Legacy is by far the most impacted individually.** As a whole, PAPI cares a lot more about recent results than the Imperium:

|       Group      | AR Coefficient |
|:----------------:|:--------------:|
| Imperium         | 0.678          |
| Panfam           | 0.685          |
| PAPI             | 0.714          |
| Imperium Subcaps | 0.752          |
| Legacy           | 0.779          |
| PAPI Subcaps     | 0.793          |

## Conclusion

Something that surprised me to discover was just how differently each group reacted to the 1DQ siege. The start of this event had a significant positive impact on the Imperium, while it had an almost meaningless negative impact on PAPI. This is because PAPI was already in steep decline before the start of the siege. On the other hand, the continuing effects of the siege have impacted PAPI much more than the Imperium- with a significant, ongoing negative effect for PAPI and a slight positive effect on the Imperium. While the participation numbers will tend to revert toward their individual means, the ongoing negative effects of the 1DQ siege are impacting Panfam much more than Legacy. 

**It's plain to see that our models predict good news for the Imperium, and a dire warning for PAPI.** If their strategy doesn't change, the Imperium's numbers will continue to rise, while PAPI's numbers will stagnate. It's important to take note of the confidence bands around these predictions- in the case of the Imperium, the tighter band predicts, at worst, slightly lower numbers, while PAPI has a larger capacity to underperform.


## References

[1]: Hyndman RJ, Athanasopoulos G. Forecasting: Principles and Practice. Melbourne: O Texts; 2018. (Available [here](https://otexts.com/fpp3/ "Forecasting: Principles and Practice").)

[2]: Strickland, Jeffrey. Time Series Analysis and Forecasting Using Python & R. S.l.: LULU COM; 2020. (Available [here](https://www.amazon.com/Time-Analysis-Forecasting-using-Python/dp/1716451132 "Time Series Analysis and Forecasting Using Python & R").)


### Skills used:
	
**Python**
* Data cleaning
* Data blending
* ARIMA modeling (including major interventions)
* Time Series forecasting
* Data visualization
* Data analysis