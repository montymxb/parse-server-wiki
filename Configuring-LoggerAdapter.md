By default parse-server uses a winston logger.  

### Configuration 

Logging configuration options for which there are [defaults](https://github.com/ParsePlatform/parse-server/blob/master/src/defaults.js) can be configured by including a key and value in the top level of your parse-server configuration.  These options include: 
* `level` which defaults to info
* `jsonLogs` which defaults to false
* `logsFolder` which defaults to `./logs`

Alternatively, if you want to pass more [complex options to the winston logger](https://github.com/winstonjs/winston#usage), you can override the default config of the winston logger:

```
loggerAdapter: {
  module: "parse-server/lib/Adapters/Logger/WinstonLoggerAdapter",
    options: {
      logLevel: "error",
    }
  },
...
```

### Custom Adapters

By overriding the [LoggerAdapter](https://github.com/ParsePlatform/parse-server/blob/master/src/Adapters/Logger/LoggerAdapter.js) it is possible to introduce a custom logging facility to parse-server.

Open source third party logging adapters:
* [AWS Firehose Logger](https://www.npmjs.com/package/parse-aws-firehose-logger-adapter)
