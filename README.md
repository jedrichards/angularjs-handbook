# AngularJS Notes

## Table of contents

1. Modules
   1. Module lifecycle
   2. Module creation 
   3. Module retrieval
2. Module objects
   1. Dependancy injection
      1. Inferred dependancy syntax
      2. Inline annotation dependancy syntax
   2. Constants
   3. Config
   4. Run
   5. Services
   6. Controllers
   7. Filters

## Modules

A module is a logical collection of application objects. They can contain nested child modules, services, controllers, filters, directives, constants and simple values.

Modules can be organised based on type (i.e. a module containing only controllers) or by app feature (i.e. a module containing a variety objects related to a particular site section) - pick whichever way works for your project. 

### Module lifecycle

Modules have two distinct phases in their lifecycle:

1. The configuration phase. Constants are set and module objects are registered and configured.
2. The run phase. Module objects are instantiated and injected with dependancies as needed.

### Module creation

Modules are created by defining the module name and an array of dependancies on other modules:

	angular.module("app",["services","controllers"]);

### Module retrieval 

Once created, modules can be retrieved later on by just passing in a single parameter, the module name:

	var myModule = angular.module("app");

## Module objects

These are the various decoupled objects that are defined on the module. They include services, controllers, directives and other Angular module object types. More on these later on.

Module objects are defined on the module during the configuration phase and their dependencies are created and injected later on during the run phase, therefore they can be defined in any order.

Each module object is defined together with a unique name which is used to resolve dependancy injections. Therefore module object names must be unique across the entire app.

A module object can be created as follows:

    myModule.service("name",ConstructorFunction);

The module creation syntax also supports method chaining. This is used when defining multiple module objects in one file. However it's generally a good idea to organise module objects into folders and individual files.

Creating multiple module objects with chaining:

	angular.module("app")
		
		.service("apiService",ConstructorFunction)
		
		.controller("HomeCtrl",ConstructorFunction);

### Dependancy injection

A key feature of AngularJS is the way that module objects are only instantiated as needed with their dependancies injected dynamically at runtime. This makes writing testable code easier as modules and module objects can be created in isolation and injected with mock/stub collaborators during a test.

All dependancy names are combined into one app wide namespace. Therefore objects defined in one module are available for injection in any other module.

DI occurs when a module object is created on demand by the framework. There are two main types of DI syntax.

#### Inferred dependancy syntax

This is where the framework infers the dependancies from the function parameter names. This is an efficient syntax but it will fail once the code has been minified since this will typically rename function parameters and destroy the mappings:
    
    angular.module("app")
    
        .service("foo",function ($http) {
            // The Angular core $http object will be supplied
            // to "foo" the service whenever needed
        });

#### Inline annotation dependancy syntax

Dependancy names are hardcoded as strings into an ordered array and are therefore resilient to code minification:

    angular.module("app")
        
        .service("foo",["$http",function ($http) {
            // ...
        }]);
		
### Constants

Module constants can be set via the module `constant` method. These define immutable values that are then available for DI in both the configuration and run module phases.

	angular.module("app")
	
		.constant("API","/api/v1/")
		.constant("API_KEY","940e12e5eaeb4e1baf964e45fc946498")
		.constant("ERROR_MESSAGES",{
		    "500": "Oh no, something has broken",
		    "403": "You need to be logged in to do that"
		});

### Config

This is code to be run only during the configuration phase. Provider objects can be injected into a config module object and configured at will. Constants are the only other sort of object that may be injected during config:

    angular.module("app")
    
        .config(function (API,API_KEY,apiService) {
            apiService.base = API;
            apiService.key = API_KEY;
        );

### Run

This is code to be run after the configuration phase but before the module at large kicks into gear. This is where you can bootstrap your app if required. It is conceptually similar to an application `main` function or primary entry point but on a per module basis and multiple `run` objects can be defined.

For example to expose the time at which the app booted up on the root view scope:

    angular.module("app")
        
        .run(function ($rootScope) {
            $rootScope.appStartTime = new Date();
        });

### Services

### Controllers

### Filters

Create data transformation "pipelines" either in template expressions or app modules. A set of basic filters are provided by the framework but additional custom ones can be defined.

#### In templates

For example in a template expression to limit a string to 80 chars and then transform it to all lowercase:

	{{myString | limitTo:80 | lowercase}}

Filters can also be applied to arrays, for example when reducing the elements outputted by the `ng-repeat` directive. In the code below only model items that contain an `active` property equal to `true` and a `firstName` property equal to the value of the `currentFirstName` property in the current `$scope` will be rendered.

	<li ng-repeat="item in items | filter{active:true,firstName:currentFirstName}">
		{{item.firstName}} {{item.lastName}}
	</li>

#### In controllers

When using a filter directly in a controller then its DI name is formed from the pattern <filter-name>Filter, for example:

	angular.module("foo")
		.controller("FooCtrl",function ($scope,$someFilter) {
			$scope.filteredValue = $someFilter(valueToFilter);
		});

As a rule its better to define complex filtering logic in a controller since this can be tested in isolation away from the DOM.

#### Custom filters

Custom filters can also be defined via the `filter` method. The below is a simple custom filter for paginating an array:

	angular.module("customFilters")
		.filter("paginate",function () {
			return function (array,page,length) {
				var start = page*length;
				return array.slice(start,start+length);
			};
		});

#### Filter dos and don'ts

- DO make them fast and efficient, they are typically "hot" code paths
- DO make them idempotent. Calling them multiple times shouldn't have side effects and they shouldn't modify passed-in data directly
- DON'T return HTML markup from filters, they should just operate abstractly on data

Documentation: http://docs.angularjs.org/guide/filter

## Building AngularJS Apps

- AngularJS has an in-built module system that isolates app modules from the global scope and handles dependancy resolution and DI automatically so solutions like RequireJS may not be strictly needed. AngularJS app files can just be concatenated together when built, no special processing or ordering of code blocks is required.
- If you use the "inferred" DI style then AngularJS code will break when minified. Either use the "inline annotation" DI style or a tool like `ngmin` in your build process to get around this.