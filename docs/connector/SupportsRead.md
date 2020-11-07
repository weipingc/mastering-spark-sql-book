# SupportsRead

`SupportsRead` is an [extension](#contract) of the [Table](Table.md) abstraction for [readable tables](#implementations).

## Contract

### <span id="newScanBuilder"> newScanBuilder

```java
ScanBuilder newScanBuilder(
  CaseInsensitiveStringMap options)
```

Creates a [ScanBuilder](ScanBuilder.md)

Used when:

* `DataSourceV2Relation` logical operator is requested to [computeStats](../logical-operators/DataSourceV2Relation.md#computeStats)
* [V2ScanRelationPushDown](../logical-optimizations/V2ScanRelationPushDown.md) logical optimization is executed
* `MicroBatchExecution` (Spark Structured Streaming) is requested for a logical query plan
* `ContinuousExecution` (Spark Structured Streaming) is created (and initializes a logical query plan)

## Implementations

* [FileTable](FileTable.md)
* [KafkaTable](KafkaTable.md)
* [MemoryStreamTable](MemoryStreamTable.md)
* [RateStreamTable](RateStreamTable.md)
* [TextSocketTable](TextSocketTable.md)