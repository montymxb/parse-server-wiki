##### The development wiki for the parse-server module.

## Files

* [[index.js|index.js]] - exposes the ParseServer constructor and mutates Parse.Cloud
* [[analytics.js|analytics.js]] - handle the /events routes
* [[Auth.js|Auth.js]] - Auth object, created to hold config/master/user information for requests
* [[batch.js|batch.js]] - batch handling implemented for PromiseRouter
* [[cache.js|cache.js]] - simple caching for the app and user sessions
* [[classes.js|classes.js]] - handle the /classes routes
* [[Config.js|Config.js]] - Config object, storage for the application configuration and some router information
* [[crypto.js|crypto.js]] - uses bcrypt for password hashing and comparison
* [[DatabaseAdapter.js|DatabaseAdapter.js]] - Interface for allowing the underlying database to be changed
* [[ExportAdapter.js|ExportAdapter.js]] - DatabaseAdapter for MongoDB (default)
* [[facebook.js|facebook.js]] - helper functions for accessing the Graph API
* [[files.js|files.js]] - handle the /files routes
* [[FilesAdapter.js|FilesAdapter.js]] - Interface for allowing the underlying file storage to be changed
* [[FileLoggerAdapter.js|FileLoggerAdapter.js]] - LoggerAdapter for logging info and error messages into local files (default)
* [[functions.js|functions.js]] - handle the /functions routes
* [[GridStoreAdapter.js|GridStoreAdapter.js]] - FilesAdapter for storing uploaded files in GridStore/MongoDB (default)
* [[installations.js|installations.js]] - handle the /installations routes
* [[LoggerAdapter.js|LoggerAdapter.js]] - Interface for allowing the underlying logging transport to be changed
* [[middlewares.js|middlewares.js]] - Express middleware used during request processing
* [[PromiseRouter.js|PromiseRouter.js]] - PromiseRouter uses promises instead of req/res/next middleware conventions
* [[push.js|push.js]] - handle the /push route
* [[rest.js|rest.js]] - main interface for REST operations
* [[RestQuery.js|RestQuery.js]] - RestQuery encapsulates everything needed for a 'find' operation from REST API format
* [[RestWrite.js|RestWrite.js]] - RestWrite encapsulates everything needed for 'create' and 'update' operations from REST API format
* [[roles.js|roles.js]] - handle the /roles routes
* [[Schema.js|Schema.js]] - Schema handles schema validation, persistence, and modification.
* [[sessions.js|sessions.js]] - handle the /sessions and /logout routes
* [[testing-routes.js|testing-routes.js]] - used by internal Parse integration tests
* [[transform.js|transform.js]] - transforms keys/values between Mongo and Rest API formats.
* [[triggers.js|triggers.js]] - cloud code methods for handling database trigger events
* [[users.js|users.js]] - handle the /users and /login routes