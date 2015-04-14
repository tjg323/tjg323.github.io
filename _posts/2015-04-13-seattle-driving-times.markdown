---
layout: post
title:  "Seattle's Bad Weather Driving"
date:   2015-04-13 19:47:51
categories: data
---

How much does a little rain actually affect Seattle drivers' abilities? Every time it rains, it seems like everyone moves a bit slower and it takes twice as long to get anywhere. I set out to examine this a bit more closely and over the months November to February, I programattically logged travel times for my commute (Redmond to Ballard) at a 5 minute granularity. Along with just transit times, other current weather conditions have been logged just in case there is some other, better indicator of travel times.

A very quick analysis shows a surprising result: **rain really doesn't affect driving times that much**; it seems to only add a little over a minute to a rush hour commute. The largest factors (obviously) are day of week and time of day, but presence of rain seems to also have a statistically significant affect of ~1 minute. All this is to be taken with a grain of salt, as this is a very quick examination, but the data is there for anyone to pick up and run with.

Other findings include the fact that Thursdays and Fridays compete for the worst commutes and the fact that it only snowed once during this period (11/29) for about 3 hours.

    import pandas as pd
    import numpy as np
    import statsmodels.api as sm
    import seaborn as sns
    import matplotlib.pyplot as plt
    import statsmodels.formula.api as smf
    import datetime
    import time
    from time import mktime
    
    %pylab inline
    
    sns.set_context("notebook")

    # Read in the data
    df = pd.read_csv('www.thomasgaid.us/Resources/TravelTimes/travel.csv')

    # Convert time of day to minutes after midnight
    def convertToMinutesAfterMidnight(date):
        dateT = datetime.datetime.strptime(date,"%Y-%m-%dt%H:%M:%S.000Z")
        midnight = dateT.replace(hour=0, minute=0, second=0, microsecond=0)
        timesince = (dateT - midnight)
        minutes = int(timesince.total_seconds() / 60)
        return minutes
    
    def convertColumnToMinutesAfterMidnight(series):
        return convertToMinutesAfterMidnight(series['time'])
    
    # From the datetime get the day of week
    def getDayOfWeek(date):
        dateT = datetime.datetime.strptime(date,"%Y-%m-%dt%H:%M:%S.000Z")    
        dayNum =  str(dateT.weekday())
        return convertToDay(dayNum)
        
    def convertToDay(date):
        return {
            '0': 'Monday',
            '1': 'Tuesday',
            '2': 'Wednesday',
            '3': 'Thursday',
            '4': 'Friday',
            '5': 'Saturday',
            '6': 'Sunday'
        }[date]
    
    def convertColumnToDayOfWeek(series):
        return getDayOfWeek(series['time'])

    # Also convert travel time from seconds to minutes
    df['minutesMidnight'] = df.apply(convertColumnToMinutesAfterMidnight,axis=1)
    df['dayOfWeek'] = df.apply(convertColumnToDayOfWeek,axis=1)
    df['WorkToHomeMinutes'] = df.WorkToHome.apply(lambda x: x/60)

    # Plot the travel times by day, shaded by precip type
    sns.lmplot("minutesMidnight", "WorkToHomeMinutes", col="dayOfWeek", data=df, hue = "precipType", col_order = ['Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday']);

![png](/Resources/TravelTimes/TravelTimes1.png)

    # Get just from 4:00 to 7:00 on weekdays where it is raining or not raining
    dfrush = df[df['minutesMidnight'].isin(range(960,1140))]
    dfworkweekrush = dfrush[dfrush['dayOfWeek'].isin(['Monday','Tuesday','Wednesday','Thursday','Friday'])]
    dfcleaned = dfworkweekrush[dfworkweekrush['precipType'].isin(['null','rain'])]

    # Group the rush hour times by precip type
    groupedday = dfcleaned['WorkToHomeMinutes'].groupby([dfcleaned['dayOfWeek'],dfcleaned['precipType']])
    groupedday.mean()

    dayOfWeek  precipType
    Friday     null          42.566265
               rain          43.046512
    Monday     null          39.443820
               rain          40.216216
    Thursday   null          40.876712
               rain          43.358491
    Tuesday    null          40.649573
               rain          41.529412
    Wednesday  null          40.943820
               rain          41.930556
    Name: WorkToHomeMinutes, dtype: float64

    # Linear regression of travel times with day of week, precip type, and minutes after midnight
    est = smf.ols(formula='WorkToHomeMinutes ~ dayOfWeek + precipType + minutesMidnight', data=dfcleaned).fit()
    est.summary()

<table class="simpletable">
<caption>OLS Regression Results</caption>
<tr>
  <th>Dep. Variable:</th>    <td>WorkToHomeMinutes</td> <th>  R-squared:         </th> <td>   0.265</td>
</tr>
<tr>
  <th>Model:</th>                   <td>OLS</td>        <th>  Adj. R-squared:    </th> <td>   0.262</td>
</tr>
<tr>
  <th>Method:</th>             <td>Least Squares</td>   <th>  F-statistic:       </th> <td>   75.22</td>
</tr>
<tr>
  <th>Date:</th>             <td>Mon, 13 Apr 2015</td>  <th>  Prob (F-statistic):</th> <td>2.95e-80</td>
</tr>
<tr>
  <th>Time:</th>                 <td>19:37:14</td>      <th>  Log-Likelihood:    </th> <td> -2920.5</td>
</tr>
<tr>
  <th>No. Observations:</th>      <td>  1257</td>       <th>  AIC:               </th> <td>   5855.</td>
</tr>
<tr>
  <th>Df Residuals:</th>          <td>  1250</td>       <th>  BIC:               </th> <td>   5891.</td>
</tr>
<tr>
  <th>Df Model:</th>              <td>     6</td>       <th>                     </th>     <td> </td>   
</tr>
<tr>
  <th>Covariance Type:</th>      <td>nonrobust</td>     <th>                     </th>     <td> </td>   
</tr>
</table>
<table class="simpletable">
<tr>
             <td></td>               <th>coef</th>     <th>std err</th>      <th>t</th>      <th>P>|t|</th> <th>[95.0% Conf. Int.]</th> 
</tr>
<tr>
  <th>Intercept</th>              <td>   23.7808</td> <td>    1.419</td> <td>   16.755</td> <td> 0.000</td> <td>   20.996    26.565</td>
</tr>
<tr>
  <th>dayOfWeek[T.Monday]</th>    <td>   -3.0066</td> <td>    0.221</td> <td>  -13.614</td> <td> 0.000</td> <td>   -3.440    -2.573</td>
</tr>
<tr>
  <th>dayOfWeek[T.Thursday]</th>  <td>   -0.8977</td> <td>    0.221</td> <td>   -4.060</td> <td> 0.000</td> <td>   -1.331    -0.464</td>
</tr>
<tr>
  <th>dayOfWeek[T.Tuesday]</th>   <td>   -1.7230</td> <td>    0.225</td> <td>   -7.648</td> <td> 0.000</td> <td>   -2.165    -1.281</td>
</tr>
<tr>
  <th>dayOfWeek[T.Wednesday]</th> <td>   -1.4476</td> <td>    0.221</td> <td>   -6.541</td> <td> 0.000</td> <td>   -1.882    -1.013</td>
</tr>
<tr>
  <th>precipType[T.rain]</th>     <td>    1.1107</td> <td>    0.161</td> <td>    6.902</td> <td> 0.000</td> <td>    0.795     1.426</td>
</tr>
<tr>
  <th>minutesMidnight</th>        <td>    0.0177</td> <td>    0.001</td> <td>   13.153</td> <td> 0.000</td> <td>    0.015     0.020</td>
</tr>
</table>
<table class="simpletable">
<tr>
  <th>Omnibus:</th>       <td>386.844</td> <th>  Durbin-Watson:     </th> <td>   0.508</td> 
</tr>
<tr>
  <th>Prob(Omnibus):</th> <td> 0.000</td>  <th>  Jarque-Bera (JB):  </th> <td>1198.135</td> 
</tr>
<tr>
  <th>Skew:</th>          <td> 1.534</td>  <th>  Prob(JB):          </th> <td>6.74e-261</td>
</tr>
<tr>
  <th>Kurtosis:</th>      <td> 6.669</td>  <th>  Cond. No.          </th> <td>2.13e+04</td> 
</tr>
</table>

    # Plot each day's rush hour
    sns.boxplot(dfcleaned["WorkToHomeMinutes"], dfcleaned["dayOfWeek"],order = ['Monday','Tuesday','Wednesday','Thursday','Friday'])

    <matplotlib.axes._subplots.AxesSubplot at 0x20cfbda0>

![png](/Resources/TravelTimes/TravelTimes2.png)


The code to log my travel times can be found [here](https://github.com/tjg323/TravelTimes), the full ipython notebook file can be found [here](/Resources/TravelTimes/TravelTimes.ipynb), and the original data can be found [here](/Resources/TravelTimes/travel.csv).