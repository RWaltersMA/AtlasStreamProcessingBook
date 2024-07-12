---
description: This section will cover how to use tumbling and hopping windows
---

# Window functions

Unlike traditional batch processing — where data is collected, stored, and then processed in chunks — streaming data is processed as it is produced, allowing for immediate analysis, response, and decision-making. Window operators within MongoDB [Atlas Stream Processing](https://www.mongodb.com/products/platform/atlas-stream-processing) pipelines allow developers to analyze and process specific fixed-sized “windows” of data within a continuous data stream. This bucketing of the data makes it easy to discover patterns and trends. Without window operators, developers have to process every single data point in the stream, and depending on the volume of data, this can be very resource-intensive.

For example, consider the solar farm use case where thousands of sensors across the farm are capturing input wattage every second. Moving every data point across tens of thousands of sensor readings is time-consuming and costly. A better solution is to capture the trend of the data by using a window operator and calculating the average watts over an interval of time. This adds value to the business while conserving storage and network resources.

Currently, Atlas Stream Processing supports two methods for windowing: [tumbling windows](https://www.mongodb.com/docs/atlas/atlas-sp/stream-aggregation/#-tumblingwindow) and [hopping windows](https://www.mongodb.com/docs/atlas/atlas-sp/stream-aggregation/#mongodb-pipeline-pipe.-hoppingWindow). Each of these variations treats the actions within the time window slightly differently. Let’s explore these window operators in more detail.
