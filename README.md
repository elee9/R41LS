# R41LS

R41LS is a web-application framework that is inspired by [Rails](github.com/rails/rails).  R41LS includes controller, route, router, and session classes to create database-backed web applications according to the Model-View-Controller (MVC) Pattern.

## Technical Features

### Controller Base
* Responsible for receiving incoming HTTP requests and parameters, and providing a response based on specified methods of loading and manipulating models
* Renders view templates to generate desired HTTP response. Evaluation and interpolation of Ruby code using in-scope variables is achieved through the use of ```binding```, in conjunction with ```#result``` in Ruby's ERB object:
```ruby
require 'active_support/inflector'

def render(template_name)
  # note: this method requires that naming conventions be followed
  controller_name = self.class.to_s.underscore[0...-11]
  template = File.read("views/#{controller_name}/#{template_name}.html.erb")
  content = ERB.new(template).result(binding)

  render_content(content, 'text/html')
end
```

* Through #invoke_action, ControllerBase opens up all methods defined on its descendants to the router/route to call with this single line: ```self.send(name)```

### Router/Routes
* Given an HTTP request, the ```router``` will match it to the correct ```route``` and run the appropriate method from the specified controller.
* A ```Route``` object consists of a URL pattern, HTTP method, associated controller, and the name of an action. Running a route (matched by the router) will use the ```regexp``` object's matching function to parse parameters, which are then passed to the proper method within the controller with #invoke_action:
```ruby
def run(req, res)
  match_data = @pattern.match(req.path)
  route_params = Hash[ match_data.names.zip(match_data.captures) ]

  controller = @controller_class.new(req, res, route_params)
  controller.invoke_action(@action_name)
end
```

* Meta-programming is used to allow a router to add routes as requested through its ```#draw``` function.
```ruby
def draw(&proc)
  self.instance_eval(&proc)
end

# expose a set of HTTP methods that can be created when called for in #draw:
[:get, :post, :put, :delete].each do |http_method|
  define_method(http_method) do |pattern, controller_class, action_name|
    add_route(pattern, http_method, controller_class, action_name)
  end
end
```

### Session
* Creates a cookie for the client and provides access to it to store information to be saved between visits
* Because the stored value of a cookie is a string, JSON parsing is used to store and extract objects from cookies
```ruby
class Session
  # if a previous session exists in cookie, extract it (under "_railz") with JSON.parse
  def initialize(req)
    cookie_json = req.cookies["_railz"]
    @cookie_hash = cookie_json.nil? ? {} : JSON.parse(cookie_json)
  end

  # ... methods get/set cookie values

  # repackage cookie (again, under "_railz") and send to client
  def store_session(res)
    res.set_cookie("_railz", { path: "/" , value: @cookie_hash.to_json })
  end
end
```
