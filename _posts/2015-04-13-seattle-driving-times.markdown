---
layout: post
title:  "Seattle Drivers In the Rain"
date:   2015-04-13 19:47:51
categories: data
---

How much does a little rain actually affect Seattle drivers' abilities? Every time it rains, it seems like everyone moves a bit slower and it takes twice as long to get anywhere. I set out to examine this a bit more analytically and over the months November to February, I programattically logged travel times for my commute (Redmond to Ballard) at a 5 minute granularity. Along with just transit times, other current weather conditions have been logged just in case there is some other, better indicator of travel times.

A very quick analysis shows a surprising result: **rain really doesn't affect driving times that much**; it seems to only add a little over a minute to a rush hour commute. The largest factors (obviously) are day of week and time of day, but presence of rain seems to also have a statistically significant affect of ~1 minute. All this is to be taken with a grain of salt, as this is a very quick examination, but the data is there for anyone to pick up and run with.

![png](/Resources/TravelTimes/TravelTimes3.png)

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

Other findings include the fact that commutes generally get worse as the week goes on and the fact that it only snowed once during this period (11/29) for about 3 hours.

![png](/Resources/TravelTimes/TravelTimes2.png)


    
![png](/Resources/TravelTimes/TravelTimes1.png)

The code to log my travel times can be found [here](https://github.com/tjg323/TravelTimes), the full ipython notebook file can be found [here](/Resources/TravelTimes/TravelTimes.ipynb), and the original data can be found [here](/Resources/TravelTimes/travel.csv).