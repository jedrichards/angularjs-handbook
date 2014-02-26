# AngularJS Notes

## Table of contents

1. Modules
2. Module components
3. Scopes
4. Views
5. Directives
6. Controllers
7. Filters
8. HTTP 
9. Forms
10. Routing
11. Building

## Modules

A module is a logical collection of application components. They can contain services, controllers, filters, directives, constants, simple values and further nested sub modules.

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

These are the various decoupled components that are registered on the module.

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
		
### Constant components

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

### Config components

This is code to be run only during the configuration phase. Components can be injected into a config block and configured at will. Constants (and "provider" services, more on these later) are the only sort of component that may be injected during config:

```
angular.module("app")
    .config(function (API,API_KEY,apiService) {
        // Here the apiService provider is configured
        // with some constant values during the config phase
        apiService.base = API;
        apiService.key = API_KEY;
    );
```

### Run components

This is code to be run after the configuration phase but before the module at large kicks into gear. This is where you can bootstrap your app if required. It is conceptually similar to an application's `main` function or primary entry point - the difference being that they are run on a per module basis and there can be multiple run blocks.

For example to expose the time at which the app started-up on the root view scope:

```
angular.module("app") 
    .run(function ($rootScope) {
        $rootScope.appStartTime = new Date();
    });
```

### Service components

AngularJS services are singleton components. Only one instance is created by the framework and then supplied for every DI request. Service uses are typically:

- Persist and share data between components
- Provide an interface for loading and accessing that data sync/aysnc
- Containers for reusable chunks of app logic and functionality

Services can receive DI just like any other component and can be registered on a module in a number of ways.

#### module.service

This is the simplest pattern (although naming one of a set of module service creation methods `service` itself was a confusing naming mistake). It allows registration of a service via a constructor function:

```
angular.module("app")
    .service("myService",function () {
        this.foo = function () {
        };
        this.bar = function () {
        };
    });
```

#### module.factory

This is a slightly more flexible pattern. Register an arbitrary object as a service. Object creation logic can be more complex and private fields can be simulated:

```
angular.module("app")
    .factory("myServiceFactory",function () {
        var privateVal = "baz";
        return {
            foo: function () {
            },
            bar: function () {
            }
        }
    });
```

#### module.provider

The most complex pattern. It allows an arbitrary object to be registered just like the factory pattern but also allows that object to be configured during the configuration phase before it's used for DI. Usually overkill for most services, and most useful when a service needs to be re-used across applications with configurable changes to behaviour.

To register a provider service:

```
angular.module("app")
    .provider("myServiceProvider",function () {
        var configurableVal = "foo";
        this.setConfigurableVal = function (val) {
            configurableVal = val;
        }
        this.$get = function () {
            return {
                // Provider service body is defined here as the return
                // value of the special $get method and has access to
                // any configured values
            }
        }
    });
```

And to configure it:

```
angular.module("app")
    .config(function (myServiceProvider) {
        myServiceProvider.setConfigurableVal("foo");
    });
```

Note that providers injected during the configuration phase have not been fully instantiated via their `$get` method yet, only the configuration API is exposed to the config code.

## Scopes

Scopes are the application view model, that is, the model as it pertains to a section of the DOM. They provide a model data context for "scope-creating" directives (i.e. `ng-controller`) and view templates.

Each scope is an instance of the `Scope` class. The `Scope` class doesn't contain any special data model related functionality beyond some framework utility methods, and therefore such data models can be thought of as POJOs (plain old JavaScript objects) as least in terms of data access and assignment.

Any value assigned to a `$scope` instance will become available for evaluation in a view template and will be automatically watched for changes - this is the live two way data binding between template and model that Angular is well known for.

Example of automatic scope creation and injection in a controller:

```
angular.controller("MyController",function ($scope) {
    // A new Scope instance has been instantiated and injected
    // ready for use in the controller and its view
    $scope.foo = "bar";
});
```

### Scope inheritance

All `$scope` objects inherit from the application's `$rootScope` instance via normal prototypical inheritance. Therefore properties defined in a parent scope are visible in all child scopes (unless a child scope redefines an identically named property). Since new scopes are created by directives/controllers they will tend to create a hierarchical, inheriting tree-like structure that partially reflects the DOM structure.

Note that inherited primitive properties such as the `string` and `number` types will prototypically propagate into child scopes by initial value only. That is, if their value is changed in the child scope then that change won't be reflected back into any parent scopes. Therefore if it is required that scope properties are fully shared throughout the scope inheritance hierarchy then define them as properties on an object reference, e.g. `obj:{prop:value}`.

### Scope events

Scopes can broadcast events down to child scopes, and emit events back up to parent scopes.

```
$scope.$broadcast("event",data);

$scope.$emit("event",data);

$scope.$on("event",handlerFunction);
```

In general scope events should be used sparingly, the auto two-way data binding to model values can go a long way towards coordinating state changes across an application without the need for events. Events are best suited for notifying app actors of global async state changes that don't relate to state held in scope data models.

## Views

Views are live portions of the DOM, defined by valid HTML templates and associated with Angular code (directives, controllers etc.) via special attributes and expressions in the markup.

Views are glued to their respective controller via their shared `$scope` instance and the two way data binding that Angular sets up between the view's DOM and the model values.

Importantly views are declarative. They are focussed on describing *what* should happen rather *how* it should happen. The data model manipulation and mutation logic for UI behaviour should live in the view's controller, and any DOM manipulation code should live in directives.

More on directives and controllers below.

### View expressions

Expressions are simple JavaScript-like code snippets inside HTML templates that operate within the data context provided by the current scope. They can access scope values, call scope methods or perform simple operations.

When possible expressions are automatically re-evaluated and re-rendered when the related scope value changes.

Render a scope value in a template:

```
<span>{{user.firstname}}</span>
```

Render the result of a scope method call in a template:

```
<span>Remaining issues: {{remainingIssues()}}
```

Perform a simple operation:

```
<span>{{2+2}}</span>
```

## Directives

Directives are reusable blocks of Angular code/behaviour that can be attached to the live DOM via markers in the HTML. When Angular "compiles" the template HTML it will attempt to match directives with DOM elements based on these markers.

Directives are commonly used to attach specific behaviour to a DOM element or transform a DOM element's (and potentially its children's) structure.

Angular provides are large library of useful directives as part of the core framework, in addition you can define more application specific custom directives yourself.

Directive markers can be placed in HTML using a number of methods (comments, tag names, classes and attributes). The recommended approach however is to use attributes in the dash delimited format `ng-<directive-name>`.

### Simple core directive examples

#### `ngClick`

Evaluates an expression when an element is clicked.

```
<button ng-click="onBtnClicked()">Click me</button>
```

#### `ngRepeat`

Repeats the current element based on an expression.

```
<li ng-repeat="user in users">
    {{user.name}}
</li>
```

#### `ngClass`

Adds class names to an element if an expression in an object hash of potential class names evaluates to true.

```
<span ng-class="{'warning':shouldWarn()}"></span>
```

### Custom directives

Custom directives can be defined on a module just like other components. They are defined via a factory function that can either return a more declarative-style directive definition object, or a directive function:

```
angular.module("app")
    .directive("my-directive",function () {
        return {
            // Directive options go here
        };
    });
```

```
angular.module("app")
    .directive("my-directive",function () {
        return function (scope,element,attr) {
            // Directive init code here
        };
    });
```

#### Element vs. attribute directives

#### Isolated scope

#### Template expansion

Custom directives can be used much like partials to avoid repeating often used chunks of HTML. For example the following directive will operate on a custom element `<customer-details>` and populate it with the contents of an external template.

The custom directive definition:

```
angular.module("app")
    .directive("customer-details",function () {
        return {
            templateUrl: "customer-details.html",
            restrict: "E"
        }
    });
```

The controller definition:

```
angular.module("app")
    .controller("CustomerCtrl",function ($scope) {
        $scope.customer = {
            name: "foo",
            address: "bar"
        }
    });
```

The application markup:

```
<customer-details></customer-details>
```

And the `customer-details.html` template:

```
Name: {{customer.name}} Address: {{customer.address}}
```

#### DOM manipulation

#### Wrappers

#### Event listeners

#### Composing

## Controllers

Controllers are constructor functions for JavaScript classes that are used to augment a view's scope with behaviour related to its particular data model.

Controllers are implemented by first preparing them for DI as a module component:

```
angular.module("app")
    .controller("MyCtrl",function ($scope) {
        $scope.salutation = "Hello";
        $scope.salutationTarget = "world";
        $scope.num = 2;
        $scope.double = function (value) {
            return value*2;
        }
    });
```

And then referencing them in a view via the `ng-controller` directive:

    <div ng-controller="MyCtrl">
        <p>{{salutation}} {{salutationTarget}}</p>
        <p>{{num}} doubled is {{double(num)}}</p>
    </div>

### Controller dos and don'ts

- **DO** make them slim. They should only contain simple, testable logic related only to the model data context for a single view.
- **DO** inject controllers with services if they need access to any shared state or functionality.
- **DON'T** manipulate the DOM in a controller, use the automatic data binding or core Angular directives library instead, or if you need to go further custom directives.
- **DON'T** filter output directly, use Angular filters.
- **DON'T** format input, use Angular form controls.

## Filters

Create data transformation "pipelines" either in template expressions or other app components. A set of basic filters are provided by the framework but additional custom ones can be defined.

### In views

For example in a template expression to limit a string to 80 chars and then transform it to all lowercase:

	{{myString | limitTo:80 | lowercase}}

Filters can also be applied to arrays, for example when reducing the elements outputted by the `ng-repeat` directive. In the code below only model items that contain an `active` property equal to `true` will be rendered.

	<li ng-repeat="item in items | filter{active:true}">
		{{item.firstName}} {{item.lastName}}
	</li>

### In controllers

When using a filter directly in a controller then its DI name is formed from the pattern `<filter-name>Filter`, for example to inject a filter called `bar`:

	angular.module("foo")
		.controller("FooCtrl",function ($scope,$barFilter) {
			$scope.filteredValue = $barFilter(valueToFilter);
		});

As a rule its better to define complex filtering logic in a controller since this can be tested in isolation away from the DOM.

### Custom filters

Custom filters can also be defined via the `filter` method. The below is a simple custom filter for paginating an array:

	angular.module("app")
		.filter("paginate",function () {
			return function (array,page,length) {
				var start = page*length;
				return array.slice(start,start+length);
			};
		});

### Filter dos and don'ts

- **DO** make them fast and efficient, they are typically "hot" code paths
- **DO** make them idempotent. Calling them multiple times shouldn't have side effects and they shouldn't modify passed-in data directly
- **DON'T** return HTML markup from filters, they should just operate abstractly on data

## HTTP

## Forms

## Routing

## Build

- AngularJS has an in-built module system that isolates app modules from the global scope and handles dependancy resolution and DI automatically so solutions like RequireJS may not be strictly needed. AngularJS app files can just be concatenated together when built, no special processing or ordering of code blocks is required.
- If you use the "inferred" DI style then AngularJS code will break when minified. Either use the "inline annotation" DI style or a tool like `ngmin` in your build process to get around this.