# AngularJS Notes

## Table of contents

1. Modules
   1. Module lifecycle
   2. Module creation 
   3. Module retrieval
2. Module components
   1. Component dependancy injection
      1. Inferred syntax
      2. Inline annotation syntax
   2. Constants
   3. Config
   4. Run
   5. Services
   6. Controllers
   7. Filters

## Modules

A module is a logical collection of application components. They can contain nested child modules, services, controllers, filters, directives, constants and simple values.

Modules can be organised based on type (i.e. a module containing only controllers) or by app feature (i.e. a module containing a variety of components related to a particular site section) - pick whichever way works for your project. 

### Module lifecycle

Modules have two distinct phases in their lifecycle:

1. The configuration phase. Constants are set and module components are registered and configured.
2. The run phase. Module components are instantiated and injected with dependancies as needed.

### Module creation

Modules are created by defining the module name and an array of dependancies on other modules:

```
angular.module("app",["services","controllers"]);
```

### Module retrieval 

Once created, modules can be retrieved later on by just passing in a single parameter, the module name:

```
var myModule = angular.module("app");
```

## Module components

These are the various decoupled components that are registered on the module. They include services, controllers, directives and other Angular component types (more on these later on).

Module components are registered during the configuration phase and instantiated and injected with dependancies later on during the run phase, therefore they can be registered in any order.

Each component is defined together with a unique name which is used to resolve dependancy injections. Therefore module component names must be unique across the entire app.

A module component can be registered as follows:

```
myModule.service("name",ConstructorFunction);
```

During component registration the constructor function or "recipe" for creating the component at a later date is passed in but it is not invoked. Components are instantiated by the framework later on lazily (on demand) during the run phase.

The registration syntax also supports method chaining. This is used when defining multiple components in one file. However it's generally a good idea to organise components into folders and individual files.

```
angular.module("app")
    .service("apiService",ConstructorFunction)
    .controller("HomeCtrl",ConstructorFunction)
    .controller("HeaderCtrl",ConstructorFunction);
```

### Component dependancy injection

A key feature of AngularJS is the way that module components are only instantiated as needed with their dependancies injected dynamically during the run phase. This makes writing testable code easier as modules and components can be created in isolation and injected with mock/stub dependancies during a test.

All dependancy names are combined into one app wide namespace. Therefore components registered in one module are available for injection in any other module.

DI occurs when a component is created on demand by the framework. There are two main types of DI syntax.

#### Inferred syntax

This is where the framework infers the dependancies from the function parameter names. This is a terse syntax but it will fail once the code has been minified since this will typically rename function parameters and destroy the mappings:

```
angular.module("app")
    .service("foo",function ($http,API,API_KEY) {
        // The AngularJS Core $http component and the custom constants
        // API and API_KEY will be injected here and available
        // for use in "foo" service.
        return function () {};
    });
```

#### Inline annotation syntax

In this syntax dependancy names are hardcoded as strings into an ordered array along with the component function itself as the last element. This syntax is functionally equivalent to the inferred syntax above and resilient to minification although arguably more verbose and harder to maintain.

```
angular.module("app")
    .service("foo",["$http","API","API_KEY",function ($http,API,API_KEY) {
        return function () {};
    }]);
```
		
### Constants

Module constants can be set via the module `constant` method. These define immutable values that are then available for DI in both the configuration and run module phases.

```
angular.module("app")
	.constant("API","/api/v1/")
	.constant("API_KEY","940e12e5eaeb4e1baf964e45fc946498")
	.constant("ERROR_MESSAGES",{
	    "500": "Oh no, something has broken",
	    "403": "You need to be logged in to do that"
	});
```

### Config

This is code to be run only during the configuration phase. Components can be injected into a config block and configured at will. Constants and providers are the only other sort of component that may be injected during config:

```
angular.module("app")
    .config(function (API,API_KEY,apiService) {
        apiService.base = API;
        apiService.key = API_KEY;
    );
```

### Run

This is code to be run after the configuration phase but before the module at large kicks into gear. This is where you can bootstrap your app if required. It is conceptually similar to an application's `main` function or primary entry point - the difference being that they are run on a per module basis and there can be multiple run blocks.

For example to expose the time at which the app started-up on the root view scope:

```
angular.module("app") 
    .run(function ($rootScope) {
        $rootScope.appStartTime = new Date();
    });
```

### Services

AngularJS services are singleton components that can be used to:

- Persist and share data between components
- Provide an interface for accessing/loading that data
- Containers for reusable chunks of logic (something like the Commands pattern)

Services can be registered on a module in a number of ways.

#### The service method

#### The factory method

#### The provider method

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

## Views

## Building AngularJS Apps

- AngularJS has an in-built module system that isolates app modules from the global scope and handles dependancy resolution and DI automatically so solutions like RequireJS may not be strictly needed. AngularJS app files can just be concatenated together when built, no special processing or ordering of code blocks is required.
- If you use the "inferred" DI style then AngularJS code will break when minified. Either use the "inline annotation" DI style or a tool like `ngmin` in your build process to get around this.