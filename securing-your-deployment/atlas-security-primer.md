# Atlas Security Primer

### MongoDB Atlas Organizations, Projects and Clusters

Recall that in MongoDB Atlas, organizations, projects, and clusters are hierarchical components that facilitate the organization and management of MongoDB resources. An organization is a top-level entity representing an independent deployment of MongoDB Atlas, containing one or more projects.  A project is a logical container within an organization, grouping related resources and serving as a unit for access control and billing. Within a project, MongoDB clusters are deployed. Clusters are instances of MongoDB databases, each with its own configurations, performance characteristics, and data. Clusters can span multiple cloud regions and availability zones for high availability and disaster recovery.  This hierarchy allows for the efficient management of MongoDB deployments, access control, and resource isolation within MongoDB Atlas.

<figure><img src="https://www.mongodb.com/developer/_next/image/?url=https%3A%2F%2Fimages.contentstack.io%2Fv3%2Fassets%2Fblt39790b633ee0d5a7%2Fblt493531d0261fd667%2F6629225351b16f1ecac4e6cd%2F1.png&#x26;w=3840&#x26;q=75" alt=""><figcaption><p>Atlas Organization, Project and Clusters</p></figcaption></figure>

With respect to security, Atlas has two separate user entities: **Atlas users** and **Atlas database users**. They are defined at different scopes, with Atlas users being used within Atlas organizations and Atlas projects, and Atlas database users being used within Atlas clusters. Understanding the differences and how your end users will authenticate to Atlas Stream Processing is important. Let’s dig deeper into these two types of users.

#### Atlas users (the control plane)

[Atlas users](https://www.mongodb.com/docs/atlas/reference/user-roles/) authenticate with Atlas UI, API, or CLI only (a.k.a the control plane). Authorization includes access to an Atlas organization and the Atlas projects within the organization.

<figure><img src="https://www.mongodb.com/developer/_next/image/?url=https%3A%2F%2Fimages.contentstack.io%2Fv3%2Fassets%2Fblt39790b633ee0d5a7%2Fblt5a6ba7eaf44c67a2%2F662922674da2a996e6ff2ea8%2F2.png&#x26;w=3840&#x26;q=75" alt=""><figcaption><p>Atlas Organization Access Manager</p></figcaption></figure>

Atlas users' access to Atlas clusters is determined by membership in one or more of the fixed organizational or project roles. Here is a sample of some of these roles and their associated permissive capabilities:

* **Organization Owner** — root access to the entire organization and projects contained within it
* **Organization Project Owner** — can create Atlas projects
* **Organization Read Only** — read-only access to the settings, users, and projects in the organization
* **Project Owner** — has full administrative access
* **Project Cluster Manager** — can update clusters
* **Project Data Access Admin** — can access and modify a cluster's data and indexes and kill operations
* **Project Data Access Read/Write** — can access a cluster's data and indexes and modify data
* **Project Data Access Read Only** — can access a cluster's data and indexes
* **Project Read Only** — may only modify personal preferences

While Atlas users may have access to an Atlas cluster through high-level permission like Project Owner, **they can only access the cluster through the Atlas UI, the Atlas API, or Atlas CLI**. Users who wish to connect to the Atlas cluster through a client driver like Java or a tool like mongosh cannot as these Atlas users do not exist within the Atlas database. This is where Atlas database users come into play.

#### Atlas database user (the data plane)

Atlas database users authenticate with an Atlas cluster directly and have no access to the Atlas UI, Atlas API, or Atlas CLI. These users authenticate using a client tool such as [mongosh](https://www.mongodb.com/docs/mongodb-shell/) or via a [MongoDB driver](https://www.mongodb.com/docs/drivers/) like the MongoDB Java driver. If you have previously used a self-hosted MongoDB server, Atlas database users are the equivalent of the [MongoDB user](https://www.mongodb.com/docs/manual/core/security-users/). MongoDB Atlas supports a variety of [authentication methods](https://www.mongodb.com/docs/manual/core/authentication/?\_ga=2.227360477.1370057921.1711395348-258656862.1711395348#authentication-mechanisms) such as SCRAM (username and password), LDAP Proxy Authentication, OpenID Connect, Kerberos, and x.509 Certificates. While clients use any one of these methods to authenticate, Atlas services, such as Atlas Data Federation, access other Atlas services like Atlas clusters via temporary x.509 certificates. This same concept is used within Atlas Stream Processing and will be discussed later in this post.**Note:** Unless otherwise specified, a “user” in this article refers to an Atlas database user.

<figure><img src="https://www.mongodb.com/developer/_next/image/?url=https%3A%2F%2Fimages.contentstack.io%2Fv3%2Fassets%2Fblt39790b633ee0d5a7%2Fbltcdd67839c2a52fa2%2F6629227fb0ec7701ffd6e743%2F3.png&#x26;w=3840&#x26;q=75" alt=""><figcaption><p>Atlas Control Plane vs Atlas Data Plane</p></figcaption></figure>

Understanding the concepts of the data plane and control plane will make it easy to understand and configure Atlas Stream Processing. In summary, you create the Stream Processing Instance (SPI) and define Connection Registry entries using the Atlas Control plane and an **Atlas user**, and you connect to an SPI and create stream processors using an **Atlas database user** account. If you are not already familiar with the components that make up Atlas Stream Processing, we’ll dive deeper in the next section.
