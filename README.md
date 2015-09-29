# angular-tips-und-tricks
Tips and Tricks which helped me during my RIA development with AngularJS..
## Credits where credits due
This documentation would not exist if not the great [AngularJS Design by John Papa](https://github.com/johnpapa/angular-styleguide). So, read his Style Guide before going through here.

## Table of Contents

1. [angular.isDefined vs JavaScript's '!!'](#angularisdefined-vs-javascripts-)
1. [Distributing reference among directives via service](#distributing-reference-among-directives-via-service)
1. [Disposing watchers](#disposing-watchers)
1. [Calling parent directive's function from child directive](#calling-parent-directive's-function-from-child-directive)
1. [Events broadcasts](#events-broadcasts)
1. [Clean up array without losing reference](#clean-up-array-without-losing-reference)

### Angular.isDefined vs JavaScript's '!!'

Let's look at scenario where you have and object with properties and you need some property from that object, but you don't know whether this object (and property!) is `null`/`undefined` or not.
If you use `angular.isDefined` you will only check if variable is different from undefined ONLY, whereas `!!` notation will check if variable is different from `null` or `undefined`.

#### [Live Example](http://codepen.io/Ulthes/pen/jbOBdb?editors=101)

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
        angular
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
        angular
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

#### [Live Example](http://codepen.io/Ulthes/pen/VvLZjd)

### Disposing watchers

Sometimes, when you really have to use watchers or bindings in directive or controller, it's good to dispose them once `$destroy` event shots. But then again, when there is a lot of code you might forget about them. It's good place to put them in object so afterwards you can clean it up just by using simple loop on it's properties.

For instance, let's say we have 4 different watchers:
```javascript
    $scope.$on("rootScopeEventOne", doSomethingOne);
    $scope.$on("rootScopeEventTwo", doSomethingTwo);
    $scope.$on("rootScopeEventThree", doSomethingThree);
    $scope.$on("rootScopeEventFour", doSomethingFour);
```
If we put them all in object, like this:
```javascript
    var Listeners = {};
    Listeners.firstListener = $scope.$on("rootScopeEventOne", doSomethingOne);
    Listeners.secondListener = $scope.$on("rootScopeEventTwo", doSomethingTwo);
    Listeners.thirdListener = $scope.$on("rootScopeEventThree", doSomethingThree);
    Listeners.fourthListener = $scope.$on("rootScopeEventFour", doSomethingFour);
```
We can clear all watchers by doing this, once `$destroy` event fires up:
```javascript
    function onDestroy(){
        for (var key in Listeners){
            Listeners[key]();
        }		
    }

    $scope.$on("$destroy", onDestroy);
```
#### [Live Example](http://codepen.io/Ulthes/pen/Lpbpzb)

### Calling parent directive's function from child directive

In progress...

#### Live Example Coming Soon

### Events broadcasts

#### __Remember:__
```javascript
    $rootScope.$broadcast(..);
```
#### __is a no-go!__ 
When you use it, you send that event __EVERYWHERE__ (and you probably just wanted to send event from/to controller/directive to/from directive). When you use that kind of broadcast you can slow down your application pretty much and cause lots of problems with synchonization between components.

If you __REALLY__ need to send an event, better use:
```javascript
    $scope.$broadcast(..);
```
to send event from parent to children OR: 
```javascript
    $scope.$emit(..);
```
to send event from the child to parent. Either way, you keep it in the gean... geanola...you keep it in the family tree of the view.

#### Live Example Coming Soon

### Clean up array without losing reference

Let's say, you have an array, which is bound to the view (or it is referenced to some service endpoint) and you need to clean it up completely. You probably would use this:
```javascript
    someArray = [];
```
What does that do? It cleans up the array, sure, but at the same time you set a new instance of that array and completely loose any references of that array in other places.

But, if you use it like this:
```javascript
    someArray.length = 0;
```
You clean up array and you keep reference.

#### Live Example Coming Soon.