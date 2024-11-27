---
title: "Time_Series"
date: 2024-11-22T10:42:35+08:00
lastmod: 2024-11-22T10:42:35+08:00
categories: [""]
summary: "Time Series"
---

## Summary

填充此处
## To be translated

Oh Sorry!

This blog has't been translated to English, please wait for a little while...

## Date and Time Data Types

## Basic operations

## DateTime Formats
**fromisoformat()** ： 将ISO形式的string转化为datetime类型。
```python

```
**isoformat()**：
```python

```
**ctime()**：
```python

```
**__format__()**:
```python

```
**strptime()**：
```python
```
**stftime()**：
## DateTime in Pandas
**to_datetime()**
注意type是timestamps
**to_timedelta()**
```python
```
**date_range()**：
```python
```
**shift()**


TODO


**.tz_localize()**
```python

```
pay attention, number doesn't change.

**tz.convert()**


**pytz.country_timezones**

**pytz.country_names**

## operations between different time zones

## Periods

**asfreq**: 

p = pd.Period('Aug-2020', 'M')
p.asfreq('A-JUN')
为什么p是Period('2011','A-JUN')