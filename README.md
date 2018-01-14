<p align="center">
  <img src="https://github.com/amberframework/site-assets/raw/master/images/amber.png" width="200">
  <p align="center"><strong>Introduction to the Amber Web Framework</strong><br>
  And its Out-of-the-Box Features<p>
  <p align="center">
    <sup>
      <i>
        Amber is Rails-like to an extent, but simpler, reasonable, and easy to understand and use.
      </i>
    </sup>
  </p>
  <p align="center">
  </p>
</p>

# Introduction

**Amber** is a web application framework written in [Crystal](http://www.crystal-lang.org). Homepage is at [amberframework.org](https://amberframework.org/), docs are on [Amber Docs](https://docs.amberframework.org), Github repository is at [amberframework/amber](https://github.com/amberframework/amber), and the chat is on FreeNode IRC (channel #amber) or on [Gitter](https://gitter.im/amberframework/amber).

It is inspired by Kemal, Rails, Phoenix and other frameworks. It is simple to get used to, and much more intuitive than frameworks like Rails. (But it does inherit some concepts from Rails that are good.)

This document is here to describe everything that Amber offers out of the box, sorted in a logical order and easy to consult repeatedly over time. The Crystal level is not described; it is expected that the readers coming here have a formed understanding of Crystal and its features.

# Installation

```shell
git clone https://github.com/amberframework/amber
cd amber
make # The result of 'make' is one file -- command line tool bin/amber

# To install the file, or to symlink the system-wide executable to current directory, run one of:
make install # default PREFIX is /usr/local
make install PREFIX=/usr/local/stow/
make force_link # can also specify PREFIX=...
```

# Creating New Amber App

```shell
amber new <app_name> [-d DATABASE] [-t TEMPLATE_LANG] [-m ORM_MODEL]
```

Supported databases are [PostgreSQL](https://www.postgresql.org/) (pg, default), [MySQL](https://www.mysql.com/) (mysql), and [SQLite](https://sqlite.org/) (sqlite).

Supported template languages are [slang](https://github.com/jeromegn/slang) (default) and [ecr](https://crystal-lang.org/api/0.21.1/ECR.html). (ecr is very similar to Ruby's erb.)

Slang is extremely elegant, but very different from the traditional perception of HTML.
ECR is HTML-like and more then mediocre when compared to slang, but may be the best choice for your application if you intend to use some HTML site template (from e.g. [themeforest](https://themeforest.net/)) whose pages are in HTML + CSS or SCSS.

In any case, you can even combine templates in various languages in a project, and regardless of the language, have in mind that the templates are compiled into the application. There is no lookup on disk or choosing between available templates during runtime. This makes them extremely fast, as well as read-only which is a very welcome side-benefit!

Supported ORM models are [granite](https://github.com/amberframework/granite-orm) (default) and [crecto](https://github.com/Crecto/crecto).

Granite is a very nice and simple, effective ORM model, where you mostly write your own SQL (i.e. all search queries typically look like YourModel.all("WHERE field1 = ? AND field2 = ?", [value1, value2])). But it also has belongs/has relations, and some other little things. (If you have by chance known and loved [Class::DBI](http://search.cpan.org/~tmtm/Class-DBI-v3.0.17/lib/Class/DBI.pm) for Perl, it might remind you of it in some ways.)

Supported migrations engines are [micrate](https://github.com/juanedi/micrate). Micrate is very, very simple and you basically write raw SQL in your migrations. There are just two keywords in the migration file which give instructions whether the SQLs that follow pertain to migrating up or down. These keywords are "-- +micrate Up" and "-- +micrate Down".

# Running the App

The app can be started as soon as you have created it and ran `crystal deps` in the app directory.
(It is not necessary to run deps if you have invoked `amber new` with argument --deps.)

To run it, you can use a couple different approaches. Some are of course suitable for development, some for production, etc.:

```shell
# For development, clean and simple - compiles and runs your app:
crystal src/<app_name>.cr

# For development, clean and simple - compiles and runs your app, but
# also watches for changes in files and rebuilds/re-runs automatically.
amber watch

# For production, compiles app with optimizations and places it in bin/app.
# Crystal by default compiles using 8 threads (tune with --threads NUM)
crystal build --no-debug --release --verbose -t -s -p -o bin/app src/app.cr
```

The watch command currently has some issues in edge cases. For example, it may try to run things even if some steps fail ([#499](https://github.com/amberframework/amber/issues/499)) or start re-building the application twice concurrently ([#507](https://github.com/amberframework/amber/issues/507)), and it is generally non-configurable ([#476](https://github.com/amberframework/amber/issues/476)).

Amber itself also currently has problems in edge cases. For example, if you create a new model but do not specify any fields for it, then until you add at least one field, Amber won't start due to a compile error in Granite ([#112](https://github.com/amberframework/granite-orm/issues/112)).

Please ignore these temporary problems until they are solved.

Amber by default uses a feature called "port reuse" available in newer Linux kernels. If you get an error "setsockopt: Protocol not available", it means your kernel does not have it. Please edit `config/environments/development.yml` and set "port_reuse" to false.

# Building the App and Troubleshooting

The application is always built, regardless of whether one is using the Crystal command 'run' (the default) or 'build'. It is just that in run mode, the resulting binary won't be saved to a file, but will be executed and later discarded.

For faster build speed, development versions are compiled without the --release flag. With the --release flag, the compilation takes noticeably longer, but the resulting binary has incredible performance.

Crystal caches partial results of the compilation (*.o files etc.) under `~/.cache/crystal/` for faster subsequent builds.

Sometimes building the App will fail on the C level because of missing header files or libraries. If Crystal doesn't print the actual C error, it will at least print the compiler line that caused it.

The best way to see the actual error from there is to copy-paste the command reported and run it manually in the terminal. The error will be shown and from there the cause will be determined easily.

There are some issues with the `libgc` library here and there. Crystal comes with built-in `libgc`, but it may conflict with the system one. In my case the solution was to install and then remove package `libgc-dev`.

# REPL

Often times, it is very useful to enter an interactive console (think of IRB shell) with all applications classes initialized etc. In Ruby this would be done with IRB or with a command like `rails console`.

Due to its nature, Crystal does not have a free-form [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop), but you can save and execute scripts in the context of the application. One way to do it is via command `amber x [filename]`. This command will allow you to type or edit the contents, and then execute the script.

Another, more professional way to do it is via REPL-like script tools [cry](https://github.com/elorest/cry) and [icr](https://github.com/crystal-community/icr). `cry` began as an experiment and a predecessor to `amber x`, but now offers additional functionality such as repeatedly editing and running the script if `cry -r` is invoked.

In any case, running a script "in application context" simply means requiring `config/application.cr` (or more generally, `config/**`), Therefore, be sure to list all your requires in `config/application.cr` so that everything works as expected.

# File Structure

So, at this point you might be wanting to know what's placed where in an Amber application. The default structure looks like this:

```
./spec                     - Tests (named *_spec.cr)
./config                   - All configuration
./config/application.cr    - Main configuration file
./config/routes.cr         - All routes
./config/environments      - Environment-specific YAML configurations
./config/webpack           - Webpack (asset bundler) configuration
./config/initializers      - Initializers
./db/migrations            - All DB migration files (created with 'amber g migration ...')
./src                      - Main source directory, with <app_name>.cr being the main/entry file
./src/controllers          - All controllers
./src/models               - All models
./src/views                - All views
./src/views/layouts        - All layouts
./src/views/home           - Views for HomeController (path "/")
./src/assets               - Static assets which will be bundled and placed into ./public/dist/
./src/assets/stylesheets
./src/assets/fonts
./src/assets/images
./src/assets/javascripts
./public                   - The "public" directory for static files
./public/dist              - Directory inside "public" for generated files and bundles
./public/dist/images
```

I prefer to have some of these directories accessible directly in the root directory of the application and to have the config directory named `etc`, so I run:

```
ln -sf config etc
ln -sf src/assets
ln -sf src/controllers
ln -sf src/models
ln -sf src/views
ln -sf src/views/layouts
```

# Database Commands

Amber provides a group of commands under the 'db' group to allow working with the database. The simple commands you will most probably want to run just to see basic things working are:

```shell
amber db create
amber db status
amber db version
```

But, beware:

Before these commands will work, you will need to take care of a couple things.

First, create a user to access the database. For PostgreSQL, this is done by invoking something like:

```shell
$ sudo su - postgres
$ createuser -dElPRS myuser
Enter password for new role: 
Enter it again: 
```

Then, edit `config/environments/development.yml` and configure "database_url:" to match your settings. If nothing else, the part that says "postgres:@" should be replaced with "yourusername:yourpassword@".

Then, to avoid seeing a minor/harmless error, please run `mkdir -p db/migrations` ([#522](https://github.com/amberframework/amber/issues/522)).

And then try the three database commands from the beginning of this section.

Please note that for the database connection to succeed, all parameters must be correct (hostname, port, username, password, database name), database server must be accessible, and the database must actually exist (unless you are invoking 'amber db create' to create it). In case of *any error in any of the stages* of connecting to the database, the error message will be very terse and just say "Connection unsuccessful: <database_url>". The solution is simple, though - simply use the printed database_url to manually attempt a connection to the database, and the problem will most likely quickly reveal itself.

Please note that the environment files for non-production environment are given in plain text. Environment file for the production environment is encrypted for additional security and can be seen or edited by invoking `amber encrypt`.

# Routes

Routes are very easy to understand. Routes connect HTTP methods (and the paths with which they were invoked) to controllers and methods on the Amber side.

Amber includes a wonderful command `amber routes` to display current routes. By default, the routes table looks like the following:

```shell
$ amber routes

╔══════╦═══════════════════════════╦════════╦══════════╦═══════╦═════════════╗
║ Verb | Controller                | Action | Pipeline | Scope | URI Pattern ║
╠──────┼───────────────────────────┼────────┼──────────┼───────┼─────────────╣
║ get  | Amber::Controller::Static | index  | static   |       | /*          ║
╠──────┼───────────────────────────┼────────┼──────────┼───────┼─────────────╣
║ get  | HomeController            | index  | web      |       | /           ║
╚══════╩═══════════════════════════╩════════╩══════════╩═══════╩═════════════╝
```

Here's an example of an actual route definition that routes HTTP POST requests to "/registration" to the method create() in class RegistrationController:

```
post "/registration", RegistrationController, :create
```

Standard HTTP verbs (GET, HEAD, POST, PUT, PATCH, DELETE) by convention go to standard methods on the controllers (show, new, create, edit, update, destroy). However, there is nothing preventing you from routing URLs to any methods you want in the controllers.

Websocket routes are supported too.

The DSL language specific to `config/routes.cr` file is defined in [dsl/router.cr](https://github.com/amberframework/amber/blob/master/src/amber/dsl/router.cr) and [dsl/server.cr](https://github.com/amberframework/amber/blob/master/src/amber/dsl/server.cr).

It gives you the following top-level commands/blocks:

```
# Define a pipeline
pipeline :name do
  ...
end

# Group a set of routes
routes :name, "path" do
  ...
end
```

Such as:

```crystal
Amber::Server.configure do |app|
  pipeline :web do
    # Plug is the method to use connect a pipe (middleware)
    # A plug accepts an instance of HTTP::Handler
    plug Amber::Pipe::Logger.new
  end

  routes :web do
    get "/", HomeController, :index    # Routes to HomeController::index()
    get "/test", PageController, :test # Routes to PageController::test()
  end
end
```

Within 'routes', the following commands are available:

```crystal
get, post, put, patch, delete, options, head, trace, connect, websocket, resources
```

`resources` is a macro defined as:

```crystal
    macro resources(resource, controller, only = nil, except = nil)
```

And unless it is confined with arguments `only` or `except`, it will automatically define get, post, put, patch, and delete routes for your resource and route them to the following methods in the controller:

```crystal
index, new, create, show, edit, update, destroy
```

Please note that it is not currently possible to define a different behavior for HEAD and GET methods ont he same path, because if a GET is defined it will also automatically add the matching HEAD route. That will result in two HEAD routes existing for the same path and trigger error `Amber::Exceptions::DuplicateRouteError`.

# Views

Information about views can be summarized in bullet points:

- Views in Amber are located in `src/views/`
- They are rendered using `render()`
- The first argument given to `render()` is the template name (e.g. `render("index.slang")`)
- If we are in the context of a controller, `render("index.slang")` will look for view using the path `src/views/<controller_name>/index.slang`
- If we are not rendering a partial, by default the template will be wrapped in a layout
- If the layout name isn't given, the default layout will be `views/layouts/application.slang`
- There is no unnecessary magic applied to template names &mdash; name given is the name that is looked up on disk
- Partials begin with "_" by convention, but that is not required
- To render a partial, use `render( partial: "_name.ext")`

# Variables in Views

In Amber, templates are compiled in the same scope as controller methods. This means you do not need instance variables for passing the information from controllers to views. Any variable you define in the controller method is automagically visible in the template.

For example, let's add the current date and time display to our /about page:

```shell
$ vi src/controllers/page_controller.cr

def about
	time = Time.now
	render "about.ecr"
end

$ vi src/views/page/about.ecr

Hello, World! The time is now <%= time %>.
```

# Static Pages

It can be pretty much expected that a website will need a set of simple, "static" pages. Those pages are served by the application, but do not come from a database nor typically use any complex code. Such pages might include About and Contact pages, Terms of Conditions, etc. Making this work is trivial.

Let's say that, for simplicity and grouping, we want all "static" pages to be served by PageController. We will group all these pages under a common web-accessible prefix of /page/, and finally we will route page requests to controller methods. (Because these pages will not be powered by a database or have any methods really, we won't need a model nor more than a single view file per page.)

Let's start by creating a controller:

```shell
amber g controller page
```

Afterwards, we edit `config/routes.cr` to link URL "/about" to method about() in PageController. We do this inside the "routes :web" block:

```
routes :web do 
  ...
  get "/about", PageController, :about
  ...
end
```

Then, we edit the controller and actually add method about(). This method can just directly return some string in response, or it can render a view, and then the expanded view contents will be returned as the response.

```shell
$ vi src/controllers/page_controller.cr

# Inside the file, we add:

def about
  # "return" can be omitted here. It is included only for clarity.
  return render "about.ecr"
end
```

Since this is happening in the "page" controller, the view directory for finding the templates defaults to `src/views/page/`. We will create the directory and the file "about.ecr" in it:

```shell
$ mkdir -p src/views/page/
$ vi src/views/page/about.ecr

# Inside the file, we add:

Hello, World!
```

Because we have called render() without additional arguments, the template will default to being rendered within the default application layout, `views/layouts/application.cr`.

And that's it! Visiting `/about` will go to the router, router will invoke `PageController::about()`, that method will render template `src/views/page/about.ecr` in the context of layout `views/layouts/application.cr`, the result of rendering will be a full page with content `Hello, World!` in the body, that result will be returned to the controller, and from there it will be returned to the client.

# Assets Pipeline

In an Amber project, raw assets are in `src/assets/`:

```shell
src/assets/
src/assets/fonts
src/assets/images
src/assets/javascripts
src/assets/javascripts/main.js
src/assets/stylesheets/main.scss
```

At build time, all these are processed and placed under `public/dist/`.
The JS resources are bundled to `main.bundle.js` and CSS resources are bundled to `main.bundle.css`.

Currently, webpack is being used for asset management. It is terrible and I recommend replacing it with at least [Parcel](https://parceljs.org/). Finding a non-js/non-node/non-npm application for this purpose would be even better; please let me know if you know one.

This section will be expanded to include a full replacement procedure. (In general it seems it shouldn't be much more complex than replacing the command and development dependencies in project's `package.json` file.)

# Default Shards

By default, Amber project depends on just a few shards:

```
amberframework/amber          - Obviously, must depend on Amber
amberframework/granite-orm    - Database ORM
amberframework/quartz-mailer  - Sending and receiving emails
amberframework/jasper-helpers - Helpers for working with HTML in Amber/Crystal
will/crystal-pg               - PostgreSQL connector
amberframework/garnet-spec    - Extended Crystal specs for testing web applications
```

In turn, these depend on:

```
luislavena/radix                      - Radix Tree implementation
jeromegn/kilt                         - Generic template interface
jeromegn/slang                        - Slang template language
stefanwille/crystal-redis             - 
amberframework/cli                    - Building cmdline apps (based on mosop)
mosop/optarg                          - Parsing cmdline args
mosop/callback                        - Defining and invoking callbacks
mosop/string_inflection               - Word plurals, counts, etc.
amberframework/teeplate               - Rendering multiple template files
juanedi/micrate                       - Database migration tool
crystal-lang/crystal-db               - Common DB API
jwaldrip/shell-table.cr               - Creates textual tables in shell
askn/spinner                          - Spinner for the shell
crystal-lang/crystal-mysql            - 
crystal-lang/crystal-sqlite3          - 
amberframework/smtp.cr                - SMTP client (to be replaced with arcage/crystal-email)
ysbaddaden/selenium-webdriver-crystal - Selenium Webdriver client
```

And basic Crystal's build-in shards:

```
http
logger
json
colorize
random/secure
```

Only the parts that are used end up in the compiled project.

Let's take a tour of all the important classes that exist in the Amber application and are useful for understanding the flow.

# Extensions

Amber adds some very convenient extensions to existing String and Number classes. The extensions are in the [extensions/](https://github.com/amberframework/amber/tree/master/src/amber/extensions) directory, but here's a listing of the current ones:

For String:

```crystal
      def str?
      def email?
      def domain?
      def url?
      def ipv4?
      def ipv6?
      def mac_address?
      def hex_color?
      def hex?
      def alpha?(locale = "en-US")
      def numeric?
      def alphanum?(locale = "en-US")
      def md5?
      def base64?
      def slug?
      def lower?
      def upper?
      def credit_card?
      def phone?(locale = "en-US")
      def excludes?(value)
      def time_string?
```

For Number:

```crystal
      def positive?
      def negative?
      def zero?
      def div?(n)
      def above?(n)
      def below?(n)
      def lt?(num)
      def self?(num)
      def lteq?(num)
      def between?(range)
      def gteq?(num)
```

# Support Routines

In [support/](https://github.com/amberframework/amber/tree/master/src/amber/support} directory there is a number of various support files that provide additional, ready made routines.

Currently, the following can be found there:

```
client_reload.cr      - Support for reloading developer's browser

file_encryptor.cr     - Support for storing/reading encrypted versions of files
message_encryptor.cr
message_verifier.cr

locale_formats.cr     - Very basic locate data for various, manually-added locales

mime_types.cr         - List of MIME types and helper methods for working with them:
      def self.mime_type(format, fallback = DEFAULT_MIME_TYPE)
      def self.zip_types(path)
      def self.format(accepts)
      def self.default
      def self.get_request_format(request)
```

# Amber::Controller::Base

This is the base controller from which all other controllers inherit. Source file is in [src/amber/controller/base.cr](https://github.com/amberframework/amber/blob/master/src/amber/controller/base.cr).

On every request, the appropriate controller is instantiated and its initialize() runs. Since this is the base controller, this code runs on every request so you can understand what is available in the context of every controller.

The content of this controller and the methods it gets from including other modules are intuitive enough to be copied here and commented where necessary:

```crystal
module Amber::Controller
  class Base
    include Helpers::CSRF
    include Helpers::Redirect
    include Helpers::Render
    include Helpers::Responders
    include Helpers::Route
    include Callbacks

    protected getter context : HTTP::Server::Context
    protected getter params : Amber::Validators::Params

    delegate :cookies, :format, :flash, :port, :requested_url, :session, :valve,
      :request_handler, :route, :websocket?, :get?, :post?, :patch?,
      :put?, :delete?, :head?, :client_ip, :request, :response, :halt!, to: context

    def initialize(@context : HTTP::Server::Context)
      @params = Amber::Validators::Params.new(context.params)
    end
  end
end
```

[Helpers::CSRF](https://github.com/amberframework/amber/blob/master/src/amber/controller/helpers/csrf.cr) provides these methods:

```crystal
    def csrf_token
    def csrf_tag
    def csrf_metatag
```

[Helpers::Redirect](https://github.com/amberframework/amber/blob/master/src/amber/controller/helpers/redirect.cr) provides:

```crystal
    def redirect_to(location : String, **args)
    def redirect_to(action : Symbol, **args)
    def redirect_to(controller : Symbol | Class, action : Symbol, **args)
    def redirect_back(**args)
```

[Helpers::Render](https://github.com/amberframework/amber/blob/master/src/amber/controller/helpers/render.cr) provides:

```crystal
    LAYOUT = "application.slang"
    macro render(template = nil, layout = true, partial = nil, path = "src/views", folder = __FILE__)
```

[Helpers::Responders](https://github.com/amberframework/amber/blob/master/src/amber/controller/helpers/responders.cr) helps control what final status code, body, and content-type will be returned to the client.

[Helpers::Route](https://github.com/amberframework/amber/blob/master/src/amber/controller/helpers/route.cr) provides:

```crystal
    def action_name
    def route_resource
    def route_scope
    def controller_name
```

[Callbacks](https://github.com/amberframework/amber/blob/master/src/amber/dsl/callbacks.cr) provide:

```crystal
    macro before_action
    macro after_action
```
# Other Classes

Here follow notes on some other/remaining Amber classes and what they are for.

The classes mentioned here are mostly supporting elements, and while they do help understand the big picture, they are not particularly crucial.

## Amber::Exceptions

Defines error conditions that Amber may encounter, such as DuplicateRouteError, RouteNotFound, ForbiddenCookieOverflow, ValidationFailed, InvalidParam etc.

Source file is in [exceptions.cr](https://github.com/amberframework/amber/blob/master/src/amber/exceptions/exceptions.cr).

## Amber::Environment

Once this module is included into your app, it creates variable `@@settings : Settings?`. The settings are loaded from `config/environments/` and a default configuration exists as well.

The config keys are pre-determined. The best way to check existing keys is to look up `config/environments/*.yml` (or run `amber encrypt` to see the contents of the `production.yml` file).

It adds the following accessors (relevant excerpts shown):

```
def self.settings; @@settings ||= Loader.new(env.to_s, path).settings
def self.logger; settings.logger
def self.env=(env : EnvType) @@env = Env.new(env.to_s); @@settings = Loader.new(...)
def self.env; @@env ||= Env.new
```

Example of using it:

```crystal
require "./src/amber/exceptions/exceptions.cr"
require "./src/amber/environment.cr"

class My
  include Amber::Environment
end

p My.settings, My.logger
```

## Amber::Environment::Settings

This is a more low-level class and it won't make changes to your module/class like Amber::Environment does.

Example of using it:

```crystal
require "./src/amber/environment/settings.cr"

settings= Amber::Environment::Settings.from_yaml("host: my.host.com")

p settings
```

Note that this always returns standard Amber settings, and you can use YAML content only to re-define default values, not to create your own keys.

[//]: # (controller/filters, process/thread_count in server/server.cr, file upload)
