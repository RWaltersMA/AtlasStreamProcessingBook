# Tumbling Windows

### Tumbling windows <a href="#tumbling-windows" id="tumbling-windows"></a>

Tumbling windows segment the data into non-overlapping, fixed-size windows. This can be helpful for batch-style analysis on discrete chunks of data. For example, if we want to calculate the average price per minute, we can define a tumbling window with an interval of one minute, as shown in the following figure.

[![Graphic showing non-overlapping, fixed-size tumbling windows.](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9a6rv2t2ixtdymuu0roj.png)](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9a6rv2t2ixtdymuu0roj.png)

For every minute, the average price for the LEAF stock will be calculated, and that value will be moved on to the next stage in the Atlas Stream Processing aggregation pipeline.

Using Atlas Stream Processing, we use the [$tumblingWindow](https://www.mongodb.com/docs/atlas/atlas-sp/stream-aggregation/#-tumblingwindow) pipeline operator to define the interval, and aggregate operations we seek to perform within the time window. For example, consider a stream of stock data in JSON format:\


```
 {
    company_symbol: 'SSC',
    company_name: 'SUPERIOR SAFFRON CORPORATION',
    exchange: 'NASDAQ',
    price: 70.14,
    tx_time: ISODate("2023-08-25T06:56:11.129Z")
  },
  {
    company_symbol: 'GTH',
    company_name: 'GRACEFUL TRAINER HOLDINGS',
    exchange: 'NYSE',
    price: 66.78,
    tx_time: ISODate("2023-08-25T06:56:11.129Z")
  },
  {
    company_symbol: 'FIL',
    company_name: 'FRUSTRATING INK LLC',
    exchange: 'NASDAQ',
    price: 83.92,
    tx_time: ISODate("2023-08-25T06:56:11.129Z")
  },
…
```

Here the data from multiple securities is being streamed once per second. To configure Atlas Stream Processing to perform an average price value over one-minute grouping by the company symbol, you configure the $tumblingWindow pipeline operator as follows:\


```
$tumblingWindow: 
{
            interval: {size: NumberInt(1), unit: "minute"},
            pipeline: [
                {
                    $group: {
                         _id: "$fullDocument.company_symbol",
                        max: {$max: "$fullDocument.price"},
                        avg: {$avg: "$fullDocument.price"}
                    }
                }
            ]
}
```

Note: The use of `$fullDocument` is needed here since Atlas Stream Processing is reading from a MongoDB change stream on the collection. The event that comes from the change stream contains metadata about the event and a field called “fullDocument” that includes the data we are interested in. For more information on the change stream event format, check out the [Change Events documentation](https://www.mongodb.com/docs/manual/reference/change-events/#change-events).

The interval units can be: "year", "month", "day", "hour", "minute", “second”, and “ms”. Inside the pipeline, you define aggregation operations on the time interval. In this case, we want to use `$group` to group the data by company symbol and return the maximum value and the average of the price value during the one-minute interval.

When this pipeline is run, the following results are obtained:\


```
{
  _id: 'IST',
  max: 89.77,
  avg: 89.50254545454546,
  _stream_meta: {
    sourceType: 'atlas',
    windowStartTimestamp: 2023-08-25T11:02:00.000Z,
    windowEndTimestamp: 2023-08-25T11:03:00.000Z
  }
}
{
  _id: 'DPP',
  max: 51.38,
  avg: 51.23148148148148,
  _stream_meta: {
    sourceType: 'atlas',
    windowStartTimestamp: 2023-08-25T11:02:00.000Z,
    windowEndTimestamp: 2023-08-25T11:03:00.000Z
  }
}
{
  _id: 'RAC',
  max: 60.63,
  avg: 60.47611111111111,
  _stream_meta: {
    sourceType: 'atlas',
    windowStartTimestamp: 2023-08-25T11:02:00.000Z,
    windowEndTimestamp: 2023-08-25T11:03:00.000Z
  }
}
```

Notice there is an extra field, “\_stream\_meta,” included as part of the result. This data describes the time interval for the aggregation. This output is explained in the section “Window output” later in this post since it applies to the other supported window in Atlas Stream Processing, a hopping window.
