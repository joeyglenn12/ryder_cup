# US Ryder Cup Team Selection 2023 - Case Study

This project aims to determine the best candidates to be selected for the US Ryder Cup Team for 2023 by analyzing golfer data since 2020.

## Data Source

This data was scraped from ESPN's website using Python. Specifically, data was collected on the top PGA tour players' performance in golf tournaments from the beginning of the 2020 season, in September, to April 2023.
This data includes fields such as Earnings, Fedex Points, and individual statistics.

## Methodology

Several techniques were used to identify top candidates, including:
* Data Cleaning: I cleaned the data to remove missing or inaccurate data points and ensure consistency. 
* Exploratory Data Analysis: I conducted data analysis to understand the distribution of the data and identify any outliers or anomalies.
* Feature Engineering: I created new features, such as win percentage per event, to better capture golfer performance.

## Results

The Ryder Cup team is comprised of twelve golfers total. Six golfers automatically qualify on total Fedex points. 
The other six golfers are picked by the Ryder Cup captain.

As of April 12, 2023:

Auto Qualify:

| Name | Points | 
| --------------- | --------------- |
| Scottie Scheffler | 1844 | 
| Max Homa | 1801 | 
| Keegan Bradley| 1153 | 
| Kurt Kitayama | 1040 | 
| Chris Kirk | 1024 | 
| Sam Burns | 984 | 


Top 10 Selections:

| Rank | Name | Wins | Average Score | Putts per Hole | GIR | DDIS |
| --------------- | --------------- | --------------- | --------------- | --------------- | --------------- | --------------- | 
|1| Patrick Cantlay | 6 | 69.10 | 1.7370 | 70.57% | 306.37 | 
|2| Xander Schauffele| 3 | 69.40 | 1.7267 | 69.20% | 305.30 |
|3| Taylor Montgomery | 0 | 69.30 | 1.6460 | 62.80% | 304.90 | 
|6| Tony Finau | 4 | 69.43 | 1.7297 | 69.53% | 304.97 |
|5| Collin Morikawa | 2 | 69.60 | 1.7437 | 69.97% | 296.53 | 
|6| Jordan Spieth| 2 | 69.87 | 1.7287 | 66.13% | 303.30 | 
|7| Andrew Putnam | 0 | 69.50 | 1.7550 | 68.70% | 283.10 | 
|8| Justin Thomas | 0 | 69.60 | 1.7297 | 66.93% | 307.63 | 
|9| Rickie Fowler | 0 | 69.87 | 1.707 | 69.40% | 305.50 | 
|10| Tom Hoge | 1 | 69.43 | 1.7290 | 68.25% | 296.15 | 

# Data Visualization

Other metrics were used, please see [dashboard](https://public.tableau.com/app/profile/joseph.glenn/viz/RyderCupCandidates-CareerStats2020-Present/RyderCup-CaseStudy).

# Conclusion
Based on the analysis, the recommendation is that US Ryder Cup Captain, Zach Johnson, selects the top six ranked players in this list. These players have a combination of the highest performance metrics and wins in the overall pool. However, there are numerous factors that can affect final selection, including movement of the players that automatically qualified, overall team chemistry, experience, and character. 
