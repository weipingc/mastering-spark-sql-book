title: TimeWindow

# TimeWindow Unevaluable Unary Expression

`TimeWindow` is an [unevaluable](Unevaluable.md) and Expression.md#NonSQLExpression[non-SQL] unary expression that represents spark-sql-functions.md#window[window] function.

[source, scala]
----
import org.apache.spark.sql.functions.window
scala> val timeColumn = window('time, "5 seconds")
timeColumn: org.apache.spark.sql.Column = timewindow(time, 5000000, 5000000, 0) AS `window`

scala> val timeWindowExpr = timeColumn.expr
timeWindowExpr: org.apache.spark.sql.catalyst.expressions.Expression = timewindow('time, 5000000, 5000000, 0) AS window#3

scala> println(timeWindowExpr.numberedTreeString)
00 timewindow('time, 5000000, 5000000, 0) AS window#3
01 +- timewindow('time, 5000000, 5000000, 0)
02    +- 'time

import org.apache.spark.sql.catalyst.expressions.TimeWindow
scala> val timeWindow = timeColumn.expr.children.head.asInstanceOf[TimeWindow]
timeWindow: org.apache.spark.sql.catalyst.expressions.TimeWindow = timewindow('time, 5000000, 5000000, 0)
----

[[units]]
`interval` can include the following units:

* year(s)
* month(s)
* week(s)
* day(s)
* hour(s)
* minute(s)
* second(s)
* millisecond(s)
* microsecond(s)

[source, scala]
----
// the most elaborate interval with all the units
interval 0 years 0 months 1 week 0 days 0 hours 1 minute 20 seconds 0 milliseconds 0 microseconds

interval -5 seconds
----

NOTE: The number of months greater than `0` <<getIntervalInMicroSeconds, are not supported>> for the interval.

[[resolved]]
`TimeWindow` can never be resolved as it is converted to `Filter` with `Expand` logical operators at <<analyzer, analysis phase>>.

=== [[parseExpression]] `parseExpression` Internal Method

[source, scala]
----
parseExpression(expr: Expression): Long
----

CAUTION: FIXME

=== [[analyzer]] Analysis Phase

`TimeWindow` is resolved to Expand.md[Expand] logical operator when [TimeWindowing](../logical-analysis-rules/TimeWindowing.md) logical evaluation rule is executed.

```
// https://docs.oracle.com/javase/8/docs/api/java/time/LocalDateTime.html
import java.time.LocalDateTime
// https://docs.oracle.com/javase/8/docs/api/java/sql/Timestamp.html
import java.sql.Timestamp
val levels = Seq(
  // (year, month, dayOfMonth, hour, minute, second)
  ((2012, 12, 12, 12, 12, 12), 5),
  ((2012, 12, 12, 12, 12, 14), 9),
  ((2012, 12, 12, 13, 13, 14), 4),
  ((2016, 8,  13, 0, 0, 0), 10),
  ((2017, 5,  27, 0, 0, 0), 15)).
  map { case ((yy, mm, dd, h, m, s), a) => (LocalDateTime.of(yy, mm, dd, h, m, s), a) }.
  map { case (ts, a) => (Timestamp.valueOf(ts), a) }.
  toDF("time", "level")
scala> levels.show
+-------------------+-----+
|               time|level|
+-------------------+-----+
|2012-12-12 12:12:12|    5|
|2012-12-12 12:12:14|    9|
|2012-12-12 13:13:14|    4|
|2016-08-13 00:00:00|   10|
|2017-05-27 00:00:00|   15|
+-------------------+-----+

val q = levels.select(window($"time", "5 seconds"))

// Before Analyzer
scala> println(q.queryExecution.logical.numberedTreeString)
00 'Project [timewindow('time, 5000000, 5000000, 0) AS window#18]
01 +- Project [_1#6 AS time#9, _2#7 AS level#10]
02    +- LocalRelation [_1#6, _2#7]

// After Analyzer
scala> println(q.queryExecution.analyzed.numberedTreeString)
00 Project [window#19 AS window#18]
01 +- Filter ((time#9 >= window#19.start) && (time#9 < window#19.end))
02    +- Expand [List(named_struct(start, ((((CEIL((cast((precisetimestamp(time#9) - 0) as double) / cast(5000000 as double))) + cast(0 as bigint)) - cast(1 as bigint)) * 5000000) + 0), end, (((((CEIL((cast((precisetimestamp(time#9) - 0) as double) / cast(5000000 as double))) + cast(0 as bigint)) - cast(1 as bigint)) * 5000000) + 0) + 5000000)), time#9, level#10), List(named_struct(start, ((((CEIL((cast((precisetimestamp(time#9) - 0) as double) / cast(5000000 as double))) + cast(1 as bigint)) - cast(1 as bigint)) * 5000000) + 0), end, (((((CEIL((cast((precisetimestamp(time#9) - 0) as double) / cast(5000000 as double))) + cast(1 as bigint)) - cast(1 as bigint)) * 5000000) + 0) + 5000000)), time#9, level#10)], [window#19, time#9, level#10]
03       +- Project [_1#6 AS time#9, _2#7 AS level#10]
04          +- LocalRelation [_1#6, _2#7]
```

=== [[apply]] `apply` Factory Method

[source, scala]
----
apply(
  timeColumn: Expression,
  windowDuration: String,
  slideDuration: String,
  startTime: String): TimeWindow
----

`apply` creates a <<TimeWindow, TimeWindow>> with `timeColumn` Expression.md[expression] and `windowDuration`, `slideDuration`, `startTime` <<getIntervalInMicroSeconds, microseconds>>.

NOTE: `apply` is used exclusively in spark-sql-functions-datetime.md#window[window] function.

=== [[getIntervalInMicroSeconds]] Parsing Time Interval to Microseconds -- `getIntervalInMicroSeconds` Internal Method

[source, scala]
----
getIntervalInMicroSeconds(interval: String): Long
----

`getIntervalInMicroSeconds` parses `interval` string to microseconds.

Internally, `getIntervalInMicroSeconds` adds *interval* prefix to the input `interval` unless it is already available.

`getIntervalInMicroSeconds` creates `CalendarInterval` from the input `interval`.

`getIntervalInMicroSeconds` reports `IllegalArgumentException` when the number of months is greater than `0`.

[NOTE]
====
`getIntervalInMicroSeconds` is used when:

* `TimeWindow` is <<apply, created>>
* `TimeWindow` does <<parseExpression, parseExpression>>
====
