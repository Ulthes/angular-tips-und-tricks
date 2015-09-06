# angular-tips-und-tricks
Tips and Tricks which helped me during my RIA development with AngularJS..
## Credits where credits due
This documentation would not exist if not the great [AngularJS Design by John Papa](https://github.com/johnpapa/angular-styleguide). So, read his Style Guide before going through here.

## Table of Contents

1. [angular.isDefined vs JavaScript's '!!'](#angularisdefined-vs-javascripts-)
1. [Distributing reference among directives via service](#distributing-reference-among-directives-via-service)
1. [Disposing watchers & events' bindings](#disposing-watchers-&-events'-bindings)
1. [Calling parent directive's function from child directive](#calling-parent-directive's-function-from-child-directive)
1. [Directives and isolated scope](#directives-and-isolated-scope)
1. [Clean up array without losing reference](#clean-up-array-without-losing-reference)

### Angular.isDefined vs JavaScript's '!!'

Let's look at scenario where you have and object with properties and you need some property from that object, but you don't know whether this object (and property!) is null/undefined or not.
If you use `angular.isDefined` you will only check if variable is different from undefined ONLY, whereas `!!` notation will check if variable is different from `null` or `undefined`.

[Example](http://codepen.io/Ulthes/pen/jbOBdb?editors=101)

If you want to check if object is `undefined`/`null` just use `!` (like `!someVariable`)

### Distributing reference among directives via service

Normally, when we pass reference between directives we'd normally declare a variable inside controller, and then pass it through view by `ng-model`. However, this creates few problems:
- We're polluting controller, which only creates an object and (mostly) nothing else.
- We're polluting the view.
- Finally, we're creating two watchers (ng-model are watchers) for sending a single reference.

Instead we could have done something like this:

Create a service which exposes just one field:
```javascript
(function(){
	angular
		.module("app")
		.service("someService", someService);

	someService.$inject = [];


	function someService(){
		var service = {
			reference : null
		}

		return service;
	}
})();
```

This will be service which only exposes a reference to whatever entity will inject it.
Now, let's make a directive which injects this service and assigns some function to `reference`:

```javascript
(function(){
	angular.
		.module("app")
		.directive("firstDirective", firstDirective);

	firstDirective.$inject = ["someService"];

	function firstDirective(someService){
		var directive = {
			controller: MainController,
			scope: {},
			restrict: 'EA',
			template: '',
			controllerAs: 'vm',
			bindToController: true
		}

		return directive;

		function MainController(){
			var vm = this;

			function showAlertBox(){
				alert("I'm from first directive!");
			}		
			
			someService.reference = showAlertBox;
		}
	}
})();
```
And now we're going to create a second directive which will activate function from first directive on button click:
```javascript
(function(){
	angular.
		.module("app")
		.directive("secondDirective", secondDirective);

	secondDirective.$inject = ["someService"];

	function secondDirective(someService){
		var directive = {
			controller: MainController,
			scope: {},
			restrict: 'EA',
			template: '<div><button type="button" ng-click="vm.OnButtonClick()">Click Me!</button></div>',
			controllerAs: 'vm',
			bindToController: true
		}

		return directive;

		function MainController(){
			var vm = this;
			vm.OnButtonClick = OnButtonClick;

			function OnButtonClick(){
				someService.reference();
			}
		}
	}
})();
```
This way, when you click a button, function from `firstDirective` will be called. Of course, it's always better to use functions like that instead of watchers.
Sure, you can use a `watcher` on a referenced object, you still will avoid polluting the view, so this way, you are omitting the Angular's `$scope` and the `$digest` cycle.

[Example](http://codepen.io/Ulthes/pen/VvLZjd)





