# Migrating

**The Parse hosted backend will be fully retired on January 28, 2017. If you are planning to migrate an app, you need to begin work as soon as possible. You will need to go through the [migration guide](https://parse.com/migration) or your app will stop working after the retirement date.**

---

* [Overview](/ParsePlatform/parse-server/wiki/Parse-Server-Guide#overview)
* [Getting Started](/ParsePlatform/parse-server/wiki/Parse-Server-Guide#getting-started)
* [Usage](/ParsePlatform/parse-server/wiki/Parse-Server-Guide#usage)
* [Database](/ParsePlatform/parse-server/wiki/Parse-Server-Guide#database)
  * [Using RocksDB](/ParsePlatform/parse-server/wiki/MongoRocks)
* [Keys](/ParsePlatform/parse-server/wiki/Parse-Server-Guide#keys)
* [Using Parse SDKs with Parse Server](/ParsePlatform/parse-server/wiki/Parse-Server-Guide#using-parse-sdks-with-parse-server)
* [Deploying Parse Server](/ParsePlatform/parse-server/wiki/Parse-Server-Guide#deploying-parse-server)
* [Configuring Third Party Authentication](/ParsePlatform/parse-server/wiki/OAuth)
* [Configuring File Adapters](/ParsePlatform/parse-server/wiki/Configuring-File-Adapters)
* [Setting up Push Notifications](/ParsePlatform/parse-server/wiki/Push)
* [Using LiveQuery](/ParsePlatform/parse-server/wiki/Parse-LiveQuery)
  * [LiveQuery Protocol](/ParsePlatform/parse-server/wiki/Parse-LiveQuery-Protocol-Specification)
* [Migrating an Existing App](https://parse.com/migration)

# Overview

Parse Server is an open source version of the Parse backend that can be deployed to any infrastructure that can run Node.js. You can find the source on the [GitHub repo](/ParsePlatform/parse-server).

* Parse Server is not dependent on the hosted Parse backend.
* Parse Server uses MongoDB directly, and is not dependent on the Parse hosted database.
* You can migrate an existing app to your own infrastructure.
* You can develop and test your app locally using Node.

## Prerequisites

* Node 4.3
* MongoDB version 2.6.X, 3.0.X or 3.2.6
* Python 2.x (For Windows users, 2.7.1 is the required version)
* For deployment, an infrastructure provider like Heroku or AWS

## Compatibility with hosted Parse

There are a few areas where Parse Server does not provide compatibility with the Parse hosted backend. If you're migrating a hosted Parse.com app to Parse Server, please take some time to carefully review the [compatibility issues](https://github.com/ParsePlatform/parse-server/wiki/Compatibility-with-Hosted-Parse) document.

# Getting Started

The fastest and easiest way to get started is to follow our [Quick Start guide](/ParsePlatform/parse-server/wiki/Quick-Start) to run MongoDB and Parse Server locally.

## Installation

Start using Parse Server by grabbing the npm module:

```bash
npm install -g parse-server
```

Or, you can specify "parse-server" in your `packages.json` file.

## Installation from source

If you wish to run the latest version of Parse Server from `master`:

If your project is not under version control or not configured to use npm:

```bash
git init
npm init
```

Add the "parse-"server submodule and link.

```bash
# in your root folder of your project
git submodule add  git@github.com:ParsePlatform/parse-server.git
npm link parse-server ./parse-server
```

### Update to latest version from source

```bash
cd parse-server
git checkout master
git pull
cd ..
git commit -am 'Updates parse-server to latest version'
```

# Usage

Parse Server is meant to be mounted on an [Express](http://expressjs.com/) app. Express is a web framework for Node.js. The fastest way to get started is to clone the [Parse Server repo](/ParsePlatform/parse-server), which at its root contains a sample Express app with the Parse API mounted.

The constructor returns an API object that conforms to an [Express Middleware](http://expressjs.com/en/api.html#app.use). This object provides the REST endpoints for a Parse app. Create an instance like so:

```js
var api = new ParseServer({
  databaseURI: 'mongodb://your.mongo.uri',
  cloud: './cloud/main.js',
  appId: 'myAppId',
  fileKey: 'myFileKey',
  masterKey: 'mySecretMasterKey',
  push: { ... }, // See the Push wiki page
  filesAdapter: ...,
});
```

The parameters are as follows:

* `databaseURI`: Connection string URI for your MongoDB.
* `cloud`: Path to your app’s Cloud Code.
* `appId`: A unique identifier for your app.
* `fileKey`: A key that specifies a prefix used for file storage. For migrated apps, this is necessary to provide access to files already hosted on Parse.
* `masterKey`: A key that overrides all permissions. Keep this secret.
* `clientKey`: The client key for your app. (optional)
* `restAPIKey`: The REST API key for your app. (optional)
* `javascriptKey`: The JavaScript key for your app. (optional)
* `dotNetKey`: The .NET key for your app. (optional)
* `push`: An object containing push configuration. See [Push](/ParsePlatform/parse-server/wiki/Push)
* `filesAdapter`: An object that implements the [FilesAdapter](/ParsePlatform/parse-server/blob/master/src/Adapters/Files/FilesAdapter.js) interface. For example, [the S3 files adapter](/ParsePlatform/parse-server/wiki/Configuring-File-Adapters)
* `oauth`: Configure support for [3rd party authentication](/ParsePlatform/parse-server/wiki/OAuth).

The Parse Server object was built to be passed directly into `app.use`, which will mount the Parse API at a specified path in your Express app:

```js
var express = require('express');
var ParseServer = require('parse-server').ParseServer;

var app = express();
var api = new ParseServer({ ... });

// Serve the Parse API at /parse URL prefix
app.use('/parse', api);

var port = 1337;
app.listen(port, function() {
  console.log('parse-server-example running on port ' + port + '.');
});
```

And with that, you will have a Parse Server running on port 1337, serving the Parse API at `/parse`.

# Database

Parse Server uses [MongoDB](https://www.mongodb.org/) as the database for your application. If you have not used MongoDB before, we highly recommend familiarizing yourself with it first before proceeding.

The Mongo requirements for Parse Server are:

* MongoDB version 2.6.X or 3.0.X or 3.2.6
* The [failIndexKeyTooLong](https://docs.mongodb.org/manual/reference/parameters/#param.failIndexKeyTooLong) parameter must be set to `false`.
* An SSL connection is recommended (but not required).
* We strongly recommend that your MongoDB servers be hosted in the US-East region for minimal lantecy.

If this is your first time setting up a production MongoDB instance, we recommend using either [mLab](http://www.mLab.com) or [ObjectRocket](https://objectrocket.com/). These are database-as-a-service companies which provide fully managed MongoDB instances, and can help you scale up as needed.

If you are migrating an existing Parse app to a MongoDB instance that isn't backed by WiredTiger or RocksDB, a good rule of thumb is to assume you will need 10X the space you currently are using with Parse.

When using MongoDB with your Parse app, there are some differences with the hosted Parse database:

* You need to manage your indexes yourself. Hosted Parse automatically adds indexes based on the incoming query stream.
* You need to size up your database as your data grows.

If you are planning to run MongoDB on your own infrastructure, we highly recommend using the [RocksDB Storage Engine](/ParsePlatform/parse-server/wiki/MongoRocks).


## Why do I need to set failIndexKeyTooLong=false?

MongoDB only allows index keys that are 1024 bytes or smaller. If a write operation attempts to store a value greater than 1024 bytes in size to a field that has been indexed, it will fail with an error. Due to how Parse dynamically indexes collections based on query traffic, we inevitably have indexed some fields with values larger than 1024 bytes. To avoid random write errors, we configured "failIndexKeyTooLong=false" on our databases, and accept the write even if the field is indexed. A side effect of this is that data with fields larger than 1024 bytes will appear to be "missing" depending on which index is selected by the MongoDB query planner.

Customers migrating their data only need to configure this parameter if they have indexed fields larger than 1024 bytes in size *and* they have collections larger than 1 million documents. For smaller apps, we will automatically clean up offending indexes during the migration. Larger apps should follow these steps as a best practice:

1. Configure `failIndexKeyTooLong=false` on the destination database
2. Migrate all data per the [migration](https://parse.com/migration) guide.
3. Evaluate all existing indexes and drop indexes for fields with data larger than 1024 bytes. The number of fields and indexes will depend entirely on the nature of your application and its data.
4. Configure `failIndexKeyTooLong=true` on the database

# Keys

Parse Server does not require the use of client-side keys. This includes the client key, JavaScript key, .NET key, and REST API key. The Application ID is sufficient to secure your app.

However, you have the option to specify any of these four keys upon initialization. Upon doing so, Parse Server will enforce that any clients passing a key matches. The behavior is consistent with hosted Parse.

# Using Parse SDKs with Parse Server

To use a Parse SDK with Parse Server, change the server URL to your Parse API URL (make sure you have the [latest version of the SDKs](https://parse.com/docs/downloads)). For example, if you have Parse Server running locally mounted at /parse:

**iOS / OS X / watchOS / tvOS**

_Swift_
```swift
let configuration = ParseClientConfiguration {
    $0.applicationId = "YOUR_APP_ID"
    $0.clientKey = ""
    $0.server = "http://localhost:1337/parse"
}
Parse.initializeWithConfiguration(configuration)
```

_Objective-C_
```objc
[Parse initializeWithConfiguration:[ParseClientConfiguration configurationWithBlock:^(id<ParseMutableClientConfiguration> configuration) {
   configuration.applicationId = @"YOUR_APP_ID";
   configuration.clientKey = @"";
   configuration.server = @"http://localhost:1337/parse";
}]];
```

**Android**

```java
Parse.initialize(new Parse.Configuration.Builder(myContext)
    .applicationId("YOUR_APP_ID")
    .clientKey(null)
    .server("http://localhost:1337/parse/") // The trailing slash is important.

    ...

    .build()
);
```

**JavaScript**

```js
Parse.initialize("YOUR_APP_ID");
Parse.serverURL = 'http://localhost:1337/parse'
```

**.NET**

```csharp
ParseClient.initialize(new ParseClient.Configuration {
    ApplicationId = "YOUR_APP_ID",
    Server = "http://localhost:1337/parse/"
});
```

**PHP**

```php
ParseClient::initialize('YOUR_APP_ID', 'YOUR_CLIENT_KEY', 'YOUR_MASTER_KEY');
ParseClient::setServerURL('http://localhost:1337/parse');
```

# Deploying Parse Server

The fastest and easiest way to start using Parse Server is to run MongoDB and Parse Server locally. Once you have a better understanding of how the project works, read on to learn how to deploy Parse Server to major infrastructure providers. If your provider is not listed here, please take a look at the [list of articles from the community](https://github.com/ParsePlatform/parse-server/wiki#community-links) as someone may have already written a guide for it.

## Deploying to Heroku and mLab

Heroku and mLab provide an easy way to deploy Parse Server, especially if you're new to managing your own backend infrastructure.

Here are the steps:

1. Create a repo for your Express app with the Parse Server middleware mounted (you can use our [sample project](https://github.com/ParsePlatform/parse-server-example), or start your own).
2. Create a Heroku account (if you don’t have one already) and use the Heroku Toolbelt to log in and prepare a new app in the same directory as your Express app. Take a look at Heroku's [Getting Started with Node.js guide](https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction) for more details.
3. Use the mLab addon: `heroku addons:create mongolab:sandbox` (or, you can create a Mongo instance yourself, either directly with mLab or your own box)
4. Use heroku config and note the URI provided by mLab under the var MONGOLAB_URI
5. Copy this URI and set it as a new config variable: heroku config:set `DATABASE_URI=mongodb://...`
6. Deploy it: `git push heroku master`

You may also refer to the Heroku Dev Center article on [Deploying a Parse Server to Heroku](https://devcenter.heroku.com/articles/deploying-a-parse-server-to-heroku).

# Setting up Push Notifications

* See the [Push Notification guide](/ParsePlatform/parse-server/wiki/Push).
* [PPNS Protocol Specification (for Parse IoT devices)](/ParsePlatform/parse-server/wiki/PPNS-Protocol-Specification)

# Configuring File Adapters

Parse Server allows developers to choose from several options when hosting files (GridStore, S3, Google Cloud Storage). GridStore is used by default and requires no setup, but if you're interested in using S3 or Google Cloud Storage, [additional configuration information is available](/ParsePlatform/parse-server/wiki/Configuring-File-Adapters).

# Using LiveQuery

LiveQuery provides real-time subscriptions to Parse Queries. To learn more, check out the [LiveQuery guide](/ParsePlatform/parse-server/wiki/Parse-LiveQuery).