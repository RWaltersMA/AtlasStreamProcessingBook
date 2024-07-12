# Blocking vs Non-blocking stages

### Blocking versus non-blocking stages <a href="#blocking-versus-nonblocking-stages" id="blocking-versus-nonblocking-stages"></a>

Keep in mind that the window operations mentioned previously are used within the context of a stream processing query. This query is a MongoDB Aggregation pipeline that can contain other operators. For example, a typical stream processing query includes a `$source`, indicating the source of the data stream, and an `$emit` or `$merge` stage that describes where to write the stream data. To make it easy to build pipelines, we can define variables for these stages. When ready to process the query, we simply pass the stages in an array. To illustrate, consider the following variables:

A source:\


```
sourceStocks={$source: { 
connectionName: "stockdb",
db: "StockData",
coll:"Stocks", 
allowedLateness: { unit: 'minute', size: 1 },
timeField: { $dateFromString:{"dateString":"fullDocument.$tx_time"}}}
```

Our hopping window from earlier:\


```
hoppingwindow=
    { $hoppingWindow: {
      interval: {size: 1, unit: "minute"}, 
      hopSize: {size: 30, unit: "second"},
      pipeline: 
      [
        { $group: {
            _id: "$fullDocument.company_symbol",
            max: { $max: "$fullDocument.price" },
            min: { $min: "$fullDocument.price" },
            avg: { $avg: "$fullDocument.price" }
        }},
        { $sort: { _id: 1 }}
      ]
    }}
```

And a $merge stage:\


```
mergeStocks={$merge: {      
            into: {
                connectionName: "stockdb",
                db: "StockData",
                coll: "stocksummary"            }
            }}
```

Now, when we create the stream processor, we simply issue the:\


```
sp.createStreamProcessor("StockSummary",[sourceStocks,hoppingwindow,mergeStocks])
```

When building pipelines for Atlas Stream Processing, there are certain operators— such as `$group`, `$sort`, `$count`, and `$limit` — that are considered blocking stages of a pipeline. This means that the process waits for all of the input data set to arrive and accumulate before processing the data together. In the context of a data stream, these blocking operations do not make sense to be executed on individual data points that arrive in the stream, since data is flowing continuously and value is obtained from more than one data point.

Other aggregation pipeline operators are not blocking, such as `$addFields`, `$match`, `$project`, `$redact`, `$replaceRoot`, `$replaceWith`, `$set`, `$unset`, and `$unwind`, to name a few. These non-blocking operators can be used anywhere within the stream processing pipeline. But blocking stages, such as `$avg`, must be used within the tumbling or hopping window pipeline operators.

Defining a window gives the stream processor the bounded context it needs (the [interval](https://www.mongodb.com/docs/atlas/atlas-sp/stream-aggregation/#fields-3) and [hop size](https://www.mongodb.com/docs/atlas/atlas-sp/stream-aggregation/#fields-2), if applicable) to appropriately process your data. Thus, the following `$group` would not be valid if it was outside of a tumbling or hopping window operator:\


```
Var g={$group: {
            _id: "$fullDocument.company_symbol",
            max: { $max: "$fullDocument.price" },
            min: { $min: "$fullDocument.price" },
            avg: { $avg: "$fullDocument.price" }
        }}
```

Also, note that Atlas Stream Processing does not support the $setWindowFields operator. Behind the scenes, `$setWindowFields` produces a different output document schema and uses different window boundary semantics. The window operators `$tumblingWindow` and `$hoppingWindow` used within Atlas Stream Processing are purpose-built to handle streams of data and common issues such as out-of-order and late-arriving data.
