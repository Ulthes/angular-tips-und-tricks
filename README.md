# angular-tips-und-tricks
Tips and Tricks which helped me during my RIA development with AngularJS..
## Credits where credits due
This documentation would not exist if not the great [AnuglarJS Design by John Papa](https://github.com/johnpapa/angular-styleguide). So, read his Style Guide before going through here.

## Table of Contents

1. [angular.isDefined vs JavaScript's '!!'](#angularisdefined-vs-javascripts-)
1. [Distributing reference among directives via service](#distributing-reference-among-directives-via-service)
1. [Disposing watchers & events' bindings](#disposing-watchers-&-events'-bindings)
1. [Calling parent directive's function from child directive](#calling-parent-directive's-function-from-child-directive)
1. [Directives and isolated scope](#directives-and-isolated-scope)
1. [Clean up array without losing reference](#clean-up-array-without-losing-reference)

### Angular.isDefined vs JavaScript's '!!'

Let's look at scenario where you have and object with properties and you need some property from that object, but you don't know whether this object (and property!) is null/undefined or not.
If you use angular.isDefined you will only check if variable is different from undefined ONLY, whereas !! notation will check if variable is different from null or undefined.

[Example](http://codepen.io/Ulthes/pen/jbOBdb?editors=101)

If you want to check if object is undefined/null just use ! (like !someVariable)


