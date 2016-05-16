# Rails Lite

Rails Lite is a web-application framework that is inspired by [Rails](github.com/rails/rails).  Rails Lite includes controller, route, router, and session classes to create database-backed web applications according to the Model-View-Controller (MVC) Pattern.

## Technical Features

### Controller Base
* Responsible for handling incoming HTTP requests and providing a response.
* Load and manipulate models
* Render view templates to generate an appropriate HTTP response

### Router/Routes
* Recognizes URLs and dispatches them to a controller's action.
* A Route consists of a URL pattern, HTTP method, associated controller, and an action.
* A Router holds all the Routes and matches them with a request appropriately.

### Session
* Stores data from the Apps associated cookie
* Allows information to be saved between visits
