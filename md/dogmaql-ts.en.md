# Time series

*Reading time: 15min*

**DogmaQL/TS** is the **DogmaQL** spec for time-series engines.

A **time series** is a series of data objects that, as a main aspect, are generated at certain times.
Its data are organized from the moment when they were generated, which is known as **time** or **timestamp**.
Every data object represents a record known formally as **sample**.

**DogmaQL** provides the **DogmaQL/TS** spec for working with tables saving time series.

A **time-series database** optimizes the storage and the query of time-based data.
As any data model, this kind of database sets how to save and to access the data by the users.
Due to the data usage, generally the following is complied:

- The writes are usually insert ops, not being the updates frequent.
  Moreover, generally the update op isn't supported.
  The insertions are very frequent.
- The reads are usually very frequent too, and these consist in searching data from the primary key or a time interval.
- The deletions aren't frequent, but are supported for purging expired and non-useful data.

Nowadays, the time series are very used.
Mainly in:

- Statistics.
- Pattern or trend recognition of prices, actions, events, diseases, contamination, traffic, etc.
- Forecast of weather, sales, actions, events, traffic, etc.
- Monitoring of prices, sales, actions, sent messages, etc.

Also is used in multiple areas as for example:

- Banking.
- Science.
- Trade.
- Government.
- IoT.
- Robotics.
- Health.
- Telecommunications.

Some organizations using time-series database:
`Airbnb`, `Alibaba`, `bet365`, `Cisco`, `eBay`, `Facebook`, `IBM`, `Mozilla`, `Nasa`, `Netflix`, `OVH`, `PayPal`, `Telefónica`, `The Weather Company`, `Twitter`, `Wikipedia Foundation` and `Xiaomi`.

## Time-series tables

A **time-series table** saves objects or samples related to some particular aspect.
For example, suppose that we need to store a record (object or sample) with the daily average temperature of the Spanish cities.
This can be done via a time-series table, where the series make reference to the average temperature of each city.
Also we can save the price evolution of multiple quotes, of some city traffic, diseases, CO2 levels, etc.

In **DogmaQL**, this type of table is created in the `ts` name space.
All table created in this name space is a time-series table.

### Types of field

A time-series table has the following types of field:

- One **time field**, which name is always `time`.
- One **key field**, which identifies the sample owner as, for example, a city name, a sensor id, etc.
- One or more **measure fields**.
  They are numeric and contain the values of the sample.
  As for example the the temperature, the humidity, the sales, etc.
- Zero, one or multiple additional fields which are providing supplementary info.

In the time-series tables, the primary key is composite and this is formed by the key field and the time field.
At the moment of creating the table, we only have to indicate the key field, the `time` field is implicit.

Let's see an illustrative example for inserting a sample:

```
insert into ts.weather({
  city = "valencia"
  time = 2018011210   #12 January 2018 10AM
  temp = 8
  hum = 63
})
```

The `city` field is the key field.
`time` is the time field.
And `temp` (temperature) and `hum` (humidity) are measure fields.
Note a same sample can have multiple measure fields, as many as needed, all of them related with the sample and the series.

### Quantum

The **quantum** represents the frequency to generate the samples, records or objects.
As, for example, every hour, day, month, etc.

Usually, all the samples of the same time-series table present the same quantum.

### Creating time-series tables

Here's an example for creating a table:

```
insert into db.tables({
  table = "ts.temps"
  key = "city"
})
```

Although the primary key is composite, we only have to indicate the key field, the time field is always `time` and this is implicit.

## Time field

Remember all object must have the `time` field, which contains when the sample was taken.
Also remember the quantum is the frequency which the sample is taken, in our case, it can be represented with the following numeric formats:

- `yyyy`, used in tables where the quantum is the year.
  Example: `2018`.
- `yyyymm`, where the quantum is the month and year.
  Example: `201802`.
- `yyyymmdd`, day, month and year.
  Example: `20180201` (1 February 2018).
- `yyyymmddhh`, year, month, day and hour.
  Example: `2018020119` (1 February 2018 7PM).
- `yyyymmddhhmm`, same as the previous one, but with minutes too.
  Example: `201802011935` (1 February 2018 19:35).
- `yyyymmddhhmmss`, the full format.
  Example: `20180201193542` (1 February 2018 19:35:42).

When inserting a sample in a table, although not mandatory, recommended all the samples have the same quantum format.

## insert statement

Using the **DogmaQL** `insert` statement, we can insert new samples in a time-series table.
Don't forget to indicate the `ts` name space, setting so we are working with the time-series engine.
Example:

```
insert into ts.weather(
  {city="Valencia", time=2018011001, temp=4}
  {city="Valencia", time=2018011002, temp=4}
  {city="Valencia", time=2018011003, temp=5}
  ...
)
```

**Important note**. The first field must be the key field; and the second one, the time field.
Please, don't forget it.

## set statement

This is similar to `insert`, but when the sample already exists, it is replaced.
Example:

```
set into ts.co2(
  {station="Valencia, Avd. Francia", time=2018011210, aqi=28, o3=5, no2=28, so2=4, pm25=25, pm10=12}
  {station="Valencia, Politècnic", time=2018011210, aqi=50, 03=0, no2=32, so2=2, pm25=50, pm10=22}
  {station="Valencia, Molí del Sol", time=2018011210, aqi=53, o3=2, no2=31, so2=3, pm25=53, pm10=14}
  ...
)
```

**Important note**. Like the `insert` statement, the first field must be the key field; and the second one, `time`.

## update statement

Not supported by the time-series engines.

## delete statement

For removing samples, have to use the **DogmaQL** `delete` statement, indicating the samples to remove in the `where` clause.

The `if` clause can't be used. Although `return old` does.

Example:

```
delete from ts.quotes
where quote == "Apple Inc"
return old
```

## select statement

For finding samples, have to use the `select` statement with a `where` clause indicating the samples to get.
Next, let's show an example for getting the samples from the last 7 days:

```
select from ts.visits
where hospital == "Valencia, La Fe" and time == "last 7d"
```

Unlike the `delete` statement, we can skip the `where` clause.
In which case, all samples are got.
Example:

```
select from ts.visits
```

The `if` and `set` clauses can't be used with time-series tables.

## Fields comparisons

The databases optimized for storing time series usually only support the access to the samples through the primary key.
No other field possible, it doesn't make sense.
When the samples are saved in stores of this type, it is due to they are accessing via the time field and/or the key field.

When the key field used, only the `==` operator can be used.

While the `time` field can be compared as follows:

- `time == value`
- `time < value`
- `time <= value`
- `time > value`
- `time >= value`
- `time between start and end`

Note that a concrete time or an interval is specified.
The value must be numeric o textual.

Let's see some illustrative examples:

```
#remove Valencia's samples
delete from ts.weather
where city == "Valencia"

#remove the samples of a given time
delete from ts.weather
where time == 2018011202

#remove the samples between two times
delete from ts.weather
where time between 20170101 and 20171231

#remove the Valencia's samples between two times
delete from ts.weather
where city == "Valencia" and time between 20170101 and 20171231
```

When determining the samples from a numerical comparison, the value is rounded.
Suppose we are saving the temperatures and have a sample for every hour.
This means that we are having a `time` field as follows: `yyyymmddhh`.
Let's see some usage examples keeping in mind the previous format:

- `time == 2018` makes reference to all the 2018 year's samples.
- `time == 201801` refers to all the samples of the January 2018.
- `time == 20180102` makes reference to all the samples of 2 January 2018.
- `time == 2018010203` refers to all the samples of 2 January 2018 3AM.

Note during the find, the format is completed to an interval.
The interval start is rounded to the start of year, month, day, hour...
While the end is rounded to the end of year, month, day, hour...
As appropriate.
So well, `time == 2018` is same as `time between 20180101000000 and 20181231235959`.
And `time == 201801` is similar to `time between 20180101000000 and 20180131235959`.

The `time` field accepts a numeric value but also a textual value:

- `today` is today.
- `yesterday` is yesterday.
- `last Xm` refers to the last *X* minutes.
  Example: `last 60m`.
- `last Xh` refers to the last *X* hours, skipping the current one.
  Also possible `last Xd`, `last Xw`, `last XM` and `last Xy`.
- `[last Xh]` refers to the last *X* hours, including the current one.
  Also possible `[last Xd]`, `[last Xw]`, `[last XM]` and `[last Xy]`.
- `Xm ago` refers to *X* minutes ago.
  Also possible `Xh ago`, `Xd ago`, `Xw ago`, `Xd ago`, `XM ago` and `Xy ago`.

The usage of the `last XhdwMy` and `[last XhdwMy]` is illustrated better with an example.
Suppose that today is 3 January 2018 and want to get the data from the last 3 months.
The answer to response is: do we want to get the last year's quarter skipping the first three days of this year or we want the last 3 months, that is, from 3 October 2017 until 3 January 2018?
If the answer is the last year's quarter, we can use `last 3M`.
However when the answer is from 3 October 2017, we need to use `[last 3M]`.

Other illustrative and interesting examples:

```
#from 30 days ago and yesterday
select from ts.weather
where time between "30d ago" and "yesterday"

#the samples with time field from 180 days ago
select from ts.weather
where time == "180d ago"

#the last 180 days, including today
select from ts.weather
where time == "[last 180d]"
```

### where clause

As indicated, remember that we only can access to the samples or data objects using `where` clauses as follows:

- Only one field.
  In this case, the field must be the key field or the `time` field.
  When the key field used, this only can be compared with the `==` operator.
  But if the `time` field used, then we can use `==`, `<`, `<=`, `>`, `>=` or `between`.

- Two fields: the key field and the `time` field.
  In this case, the syntax to use is as follows:
  `key == value and time operator value`, where the operator can be `==`, `<`, `<=`, `>`, `>=` or `between`.
  But only two predicates possible: the 1st one is for the key field and the 2nd one for the `time` field.
  Example: `city == "Valencia" and time == 2015`.
