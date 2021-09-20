![](logo.png)

# BLOCKS FILTERS

## A Technical help for using blocks filters
###     Vortex FdC

## 1. Introduction

With Vortex BLOCKS, the full time series for the whole region of the site are saved. The simulation output can be processed after the run to obtain averages, histograms, WRG files... This allows for a pre-selection of timestamps prior to generating such deliverables. This is called **filtering**.

The additional filtering option for the deliverables of a BLOCKS run uses a custom configuration file which defines the filter to be applied. We are presenting in this document a tutorial to show how to use and customize the **filter configuration file** within the different possibilities we have implemented.

### Where do I see it?

In the «Download Layer» menu of a BLOCKS run, one of the options allows the user to choose a filter for the deliverable. The menu options are:

* **GIS or WRG-WRB files:** you will select the desired deliverable format.
* **Origin:** if you requested a calibration in the run submit, you can select here whether you want to use the *calibrated* or *non-calibrated* simulation.
* **Filter By:** In this menu you can choose the **filter configuration file** you want to use or select 'No-Filter' to use all available timestamps for this deliverable. How to create, modify and reuse **filter configuration files** is explained in the following section.

There are some «Download Layer» differences depending on the type of files you are selecting:

* __GIS:__

    * **Variable:** You can select which variable you want for the GIS file (Wind Speed, TI (15m/s), Mean TI , Temperature, Density, Pressure, Richardson Number and Inverse Monin Obukhov Length).
      
    * **Metric:** you can select here how to process all the timestamps for a GIS file from the metric (Mean, Median, Maximum, Minimum, Standard deviation, P10 and P90).
      
    * **Height:** you can choose from 50 to 300 meters height.


* __WRG-WRB:__

    * **Height(s):** one height (WRG) or a list of multiple heights (WRB).
    * **Sectors Number:** the default number of wind direction sectors is 16 but other values can be chosen  (12, 16, 24, 36)


### What does filtering mean?

Imagine from sequence of timestamps that we have a meteorological set of variables computed for the domain. Let’s say we want to compute the average. We would typically process the average for all the timestamps representing the long term to generate a typical mean wind speed MAP. In a FARM we would compute A, k, mean wind speed for each grid point in different sectors for all the timestamps.

The difference now is that we can select the desired subset of timestamps to compute the same deliverables - GIS & WRG files. To select a subset we use a custom configuration file which defines how to subset the simulation timestamps. The format and options of this file are explained below.

For example, you can select timestamps by defining a list or a range of hours, days, months or/and years. This can be useful in situations like:

* **Day and night** study for your wind farm. You know exactly which hours of the day you want to select.

* **Seasonal changes** in wind flow is affecting your production estimations. You want to select specific months of the year.



## 2. Filter Configuration Files

A filter configuration file is a text file that can be written by the user where several *sections* can be defined:

1. `[TIMESTAMPS]`
2. `[DATES]`
3. `[PERIODS]`
4. `[REFERENCE POINT]`

After defining the section initiation `[SECTION]`, specific keywords allow the user to define the desired filtering. Note that a filtering file *can contain more than one section*, which allows the filtering of an intersection of conditions. For example, the following is a "summer nights" filter aware of the local UTC and season obtained by combining hours and season filters:

```
[TIMESTAMPS]
utc = 'auto'
hour_range = 18,6

[PERIODS]
season = 'summer'
start_day = 21
```


### 2.1 Timestamps

To select particular hours the _TIMESTAMPS_ method is used. We initialize a section definition with this tag:

```
[TIMESTAMPS]
```

and it is mandatory to define either a `hours` or an `hour_range` keyword. 

* `hours`: A list of hours or timestamps separated by commas.
    * Hours: The format of the elements of the list is `H`. All timestamps of the hours requested are included. For example, since BLOCKS data are 30-minutal, `hours = 1, 2` will consider the timestamps 01:00, 01:30, 02:00 and 02:30.
    * Timestamps: the format of the elements of the list is `HH:MM`. Only the timestamps included in this list are considered. For example, `hours = 13:00, 13:30, 14:00` will consider the timestamps 13:00, 13:30 and 14:00.
* `hour_range`: Two hours or timestamps separated by commas. All timestamps between them are selected. **The last timestamp is not included**. For example, `hour_range = 18:30, 6:30` will select the timestamps 18:30, 19:00, ..., 5:30 and 6:00 (**not 6:30**).

It is very important to keep in mind that by default the timestamps are considered in universal time: UTC+0. The `utc` keyword can be set to   a specific time offset (a float value like 3 or 5.25 corresponding to the UTC of the region) or to `'auto'`. In the automatic case, the offset will be deduced from the location of the site (but ignoring daylight saving). In case a `utc` value is defined, the timestamps written in the configuration file are assumed to be requested in local time. 

For example, to see the night wind flow at a location at UTC+3 we can select **from 22:00 to 5:55 local time** by using this filter:

```
[TIMESTAMPS]
utc = 3
hours = 22,23,0,1,2,3,4,5
```

or, equivalently,

```
[TIMESTAMPS]
utc = 3
hour_range = 22,6
```

### 2.2 Dates

To select full days, months or years or certain dates in general, the `
[DATES]` method is used. 

The format of the dates objects can be:

   *   __yyyy__          we define a year.
   *   __yyyy-mm__       we define a particular month from a year.
   *   __mm__            we define a month for any year.
   *   __yyyy-mm-dd__    we define an specific date
   *   __mm-dd__         we define a date for any year.

It is mandatory to define either a `dates` or a `date_range` keyword.

* `dates`: A list of dates (in any of the valid formats) separated by commas. All timestamps belonging to the requested dates are selected. For example, `dates=2021` will select all timestamps from the year 2021 and `dates=07,09` will select all timestamps that correspond to a July or a September month.
* `date_range`: Two dates separated by commas. All timestamps between them are selected. **The last date is not included**. For example, `date_range=6-15, 9-15` will select all timestamps between June the 15th at 0:00 and September the 14th at 23:59 (regardless of the year).

An example would be:
```
[DATES]
dates = 1,2,3
```
which will select January, February and March of each available year. 

Which is not the same as the filter:
```
[DATES]
dates = 2021-01,2021-02, 2021-03
```
which will select first three months of the year 2021. We can also select those months using a date range using starting and ending date objects and remembering the ending date is not included. So,
```
[DATES]
date_range = 2021-01,2021-04
```
will select the first three months of the year 2021 like the previous filter.


### 2.3 Periods

Since seasons are not defined in the same months at both hemispheres, a filter such as:

```
[DATES]
date_range = 6,9
```

will select June, July and August regardless of the hemisphere and cannot be called a 'summer' filter with full generality. To account for hemisphere-aware season definitions, a new section called `[PERIODS]` can be defined.

The mandatory keyword is `season` and it is a list of one or more seasons, that are called:

* `summer` (by default, JJA in NH and DJF in SH)
* `autumn` (by default, SON in NH and MAM in SH)
* `winter` (by default, DJF in NH and JJA in SH)
* `spring` (by default, MAM in NH and SON in SH)

Optionally, one can set a `start_day` keyword that will tell the day of the month that we consider the change of season to happen. By default, it is `start_day=1` in order to choose full months and follow the convention that summer is JJA (June-July-August).

For example,
```
[PERIODS]
season = summer
```
will select timestamps belonging to June, July and August if the site is in the Northern Hemisphere (positive latitude) and months December, january and February if the site is in the Southern Hemisphere (negative latitude). 

Another option is to manually define the range of dates that should be considered in each hemisphere using the keywords `<season>_NH>` and `<season>_SH`. For example:

```
[PERIODS]
season = summer
summer_NH = 06-20, 09-20
summer_SH = 12-20, 03-20
```

which would be equivalent to

```
[PERIODS]
season = summer
start_day = 20
```


### 2.4 Reference Point: Variable conditioning filters

The previous sections allowed you to select the timestamps you had in mind. That is great. You can now compare between different outputs for all the period and different filters.

But imagine you just do not want certain timestamps, and you are thinking on certain meteorological conditions rather than concrete timestamps or dates. Here is a list of situations that would require this more complex filtering strategy:

* You have changes in your TI and you want specific outputs for certain TI ranges.
* You want to study separately certain sectors affected by wakes generated in nearby wind farms.
* The shear is affected at certain temperature ranges, and you want to perform separated analysis.

To start this method we use the `[REFERENCE_POINT]` section.

The way to implement this solution is to define a set of variable range conditions. To apply this condition we choose a point in the grid to set as reference for that condition. The timestamps at that point that match the condition/s will be used for the whole domain.

There are two options to determine the reference point for the timestamp selection:

1. Set the coordinates of a point in the grid using the `coords` keyword:

    `coords = <LAT>, <LON>, <LEV>`. 

    The height `LEV` can be omitted and in that case the height of the requested deliverable will be used (it is recommended to write `LEV` explicitly or otherwise the filter will not work for multiple-height deliverables like a WRB). For example,


2. Use the _Smart Point Selection_ feature. This sets a condition to select the point that will be used as a reference. The syntax is: 

    `point_selection = <STAT>, <VAR>, <LEV>, <SELECT>` 

    to select the point, the statistic `<STAT>` of variable `<VAR>` at height `<LEV>` will be computed at each point of the grid and the statistic `<SELECT>` will identify the selected point. Four parameters must be specified:

    * `STAT`: temporal statistic at each point. This statistic can be `mean`, `min`, `max`, `median`, a quantile defined starting with a q (for example, the quantile 90% value is represented by `q90`),  and also dispersion statistics like the standard deviation `std` and the inter-quantile range `iqr`. All the timestamps for each grid node will be processed using this.

    * `VAR`: a variable available from `M` (wind speed), `Dir` (Direction), `P` (Pressure), `T`(temperature), `D` (density), `TI` (turbulence intensity) for which the statistic `STAT` will be computed at height `LEV`.

    * `LEV`: which height to compute the statistic of the variable on.

    * `SELECT`, horizontal statistic to select the point. This is again a statistic, but it is used to select a point across all the domain values (`mean`, `min`, `max`, `median`, `qXX`). 
    
    For example, 

    `point_selection = mean,T,100,max` 

    sets as a reference point the location where the mean temperature at 100m height is maximum.

    To select the point with the average maximum temperature at 100m the point selection should be written as: 

    `point_selection = max,T,100,mean`

   This computes the maximum temperature for each point at 100m height. It then computes the average for all points. The point with the closest value to it is selected. This is the location where the variable conditioning in order to select timestamps will be applied.

That’s fine, we have now a point where to apply the meterological condition or conditions, that will be defined using the `filterXX` sections (XX being numbers starting at `00` and increasing). The syntax is:

`filterXX = <VAR>, <MIN>, <MAX>`

and it will select those timestamps where, at the reference point, the variable `<VAR>` has a value between `<MIN>` and `<MAX>`.

As you can see there are three arguments needed:

1. `<VAR>`: the variable to use in the condition: `M` (Wind speed), `Dir` (Direction), `P` (Pressure), `T`(temperature), `D` (density), `TI` (turbulence intensity), `sector` (Wind direction sectors), `shear`, `inflow_angle`...
2. `<MIN>`: the minimum value. This can be a number or a statistic (like `mean`, `median`, `q25`...) that will be computed relative to the reference point. It can also be `None` if we don't want to filter a lower limit.  Of course using statistic `max` here will raise an error.
3. `<MAX>`: the maximum value. It can be a number, a statistic or None (as for `MIN`). Of course using here lower values than `MIN` will raise an error as an empty array of timestamps would be generated. If `MIN` = `MAX`, only timestamps with the exact value of `MIN` and `MAX` will be selected (useful for discrete variables like `sector`).

Let's create a filter which selects those timestamps with wind speeds higher than the quantile 25% of windspeeds (ie, select the 75% of timestamps with higher wind speeds) at a reference point chosen by a smart point selection method:

```
[REFERENCE_POINT]
point_selection: max,T,100,mean
filter00=M,q25,max
```

Now let's combine two filters. In this case we would filter by wind speeds higher than 5 m/s and wind directions between 270 and 90 (clockwise).

```
[REFERENCE_POINT]
point_selection = median,M,100,median
filter00 = M,5.0,None
filter11 = Dir,270,90
```

Filtering can be done for variable `sector`, and the number of sectors can be defined in an optional keyword `num_sectors`. For example, to filter timestamps where the wind at the central point `<LAT_center>, <LON_center>` comes from the North (sector zero, with a total of 24 sectors) the filter would be:

```
[REFERENCE_POINT]
coords = <LAT_center>, <LON_center>
num_sectors = 24
filter00 = sector,0,0
```


## 3. Creating and reusing filters

.. #?# Més explicació d'això!!!

This is all the explanation for setting up filters. No need to close a method. It is automatically closed when another is open or when the file is finished.

a reusable You can select one and use it as is or modify it and save. This will set a new filter in your account. This will appear in the following forms when requesting BLOCK layers. This way you can ensure you are using the same as previous and save you time.

## 4. Default Filter Samples

Following there are a few examples explaining the samples available for default.

__Day UTC__
```
[TIMESTAMPS]
utc = 'auto'
hour_range = 6,18
```

__Night UTC__
```
[TIMESTAMPS]
utc = 'auto'
hour_range = 18,6
```

__Summer__
```
[PERIODS]
season = summer
```

__Winter__
```
[PERIODS]
season = summer
```

__High Wind Speeds__
```
[REFERENCE_POINT]
point_selection: mean,M,100,mean
filter00=M,q75,max
```

__Manual Reference Point & Northern Sector__
```
[REFERENCE POINT]
## Select custom location as reference point
## lat,lon,height
coords = -10.1, -38.7, 100
#If you want automatic point selection coment above and uncomment below:
#point_selection: mean,M,100,mean

####### FILTERS
#select Northern Wind Direction , from 270 to 90 degrees.
filter00=Dir,270,90
```

__North using sectors__
```
[REFERENCE POINT]
#select number of sectors to be used (12,16,24,36)
num_sectors = 16
point_selection = median, M, 100, median

####### FILTERS
filter00 = sector, 0, 0
```
