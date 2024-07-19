# Atlas Stream Processing Architecture

### A primer on Atlas Stream Processing’s architecture

Atlas Stream Processing does not exist within an Atlas cluster. Instead, it is contained within an Atlas project and does not depend on an Atlas cluster. Consider the Atlas Stream Processing architecture diagram below:

<figure><img src="https://www.mongodb.com/developer/_next/image/?url=https%3A%2F%2Fimages.contentstack.io%2Fv3%2Fassets%2Fblt39790b633ee0d5a7%2Fblt5188a9aabfeae08c%2F66292291b0ec775eb8d6e747%2F4.png&#x26;w=3840&#x26;q=75" alt=""><figcaption><p>Atlas Stream Processing Architecture</p></figcaption></figure>

A Stream Processing Instance is a logical grouping of zero or more stream processors (SP). These processors are created within the SPI and operate within the cloud provider and region specified within the SPI. SPs leverage data sources that are defined within the Connection Registry. Each Connection Registry is mapped directly to an SPI and contains connection definitions, such as connections to other Atlas clusters, within the same Atlas project or to external Apache Kafka systems such as Confluent Cloud, AWS MSK, or a self-hosted Kafka deployment, to name a few. Now that you have a basic understanding of Atlas security and the architecture of Atlas Stream Processing, let’s take a look at some specific security features and walk through an example of implementing least privileges.
