# Hopping Windows

### Hopping windows <a href="#hopping-windows" id="hopping-windows"></a>

A hopping window, sometimes referred to as a sliding window, continuously moves over the data stream in a fixed size as new data arrives. This is useful for ongoing analysis and determining trends — for example, if we want to calculate the average price over the past hour in 30-minute increments as shown in the following figure. Stated another way, at time one hour, you get an average over the past hour. Then at time one hour and 30 minutes, you get an average over the past one hour (which is the calculation of time between 30 minutes and the current time of 90 minutes).

[![Graphic showing overlapping, fixed-size hopping windows.](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5plv5hl8fzaivybd2hjv.png)](https://media.dev.to/cdn-cgi/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5plv5hl8fzaivybd2hjv.png)

Given our stock example, we can create a hopping window pipeline operator that averages over the past minute every 30 seconds as follows:\


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

As with the tumbling window, we start by specifying an interval. But unique to the hopping window, we also specify “hopSize” to define the time segment the pipeline will be evaluated over. An example output of the hopping window defined above is as follows:\


```
{
  _id: 'IST',
  max: 89.69,
  min: 89.44,
  avg: 89.59533333333334,
  _stream_meta: {
    sourceType: 'atlas',
    windowStartTimestamp: 2023-08-25T10:43:00.000Z,
    windowEndTimestamp: 2023-08-25T10:44:00.000Z
  }
}
{
  _id: 'IST',
  max: 89.69,
  min: 89.27,
  avg: 89.53133333333334,
  _stream_meta: {
    sourceType: 'atlas',
    windowStartTimestamp: 2023-08-25T10:43:30.000Z,
    windowEndTimestamp: 2023-08-25T10:44:30.000Z
  }
}
{
  _id: 'IST',
  max: 89.8,
  min: 89.27,
  avg: 89.53566666666667,
  _stream_meta: {
    sourceType: 'atlas',
    windowStartTimestamp: 2023-08-25T10:44:00.000Z,
    windowEndTimestamp: 2023-08-25T10:45:00.000Z
  }
}
```

The above result set was filtered to only show one of the stock symbols, “IST”, so you can observe the data as it is returned per the “hopSize” defined in the query. The first result was from the interval 43:00 to 44:00, then the minute 43:30 to 44:30, then the minute from 44:00 to 45:00. Note these computations are in 30-second “hops.”

### Window output <a href="#window-output" id="window-output"></a>

For every message being emitted from a window, some implicit projections are made. This allows the developer to understand the bounds of the window being emitted. Output is automatically projected into a "\_stream\_meta" field for the message.

For example, the output of a single tumblingWindow from the earlier example is as follows:\


```
{
  _id: 'TRF',
  max: 29.64,
  avg: 29.541632653061225,
  _stream_meta: {
    sourceType: 'atlas',
    windowStartTimestamp: 2023-08-25T09:50:00.000Z,
    windowEndTimestamp: 2023-08-25T09:51:00.000Z
  }
}
{
  _id: 'DPP',
  max: 51.28,
  avg: 51.13448979591837,
  _stream_meta: {
    sourceType: 'atlas',
    windowStartTimestamp: 2023-08-25T09:50:00.000Z,
    windowEndTimestamp: 2023-08-25T09:51:00.000Z
  }
}
{
  _id: 'GCC',
  max: 60.41,
  avg: 60.30142857142857,
  _stream_meta: {
    sourceType: 'atlas',
    windowStartTimestamp: 2023-08-25T09:50:00.000Z,
    windowEndTimestamp: 2023-08-25T09:51:00.000Z
  }
}
```

The windowStartTimestamp is the first timestamp of the window, and its data is inclusive in the calculation. The windowEndTimestamp is the last timestamp of the window, and its data is exclusive in the calculation.
