
### Typescript Guidelines

- <a href="#services"> Services </a>
- <a href="#controllers"> Controllers </a>
- <a href="#directives"> Directives </a>
- <a href="#classinterface"> Class or Interface? </a>
- <a href="#namespacing"> Namespacing </a>
- <a href="#naming"> Naming </a>
- <a href="#any"> Any </a>
- <a href="#letvar"> Let & var </a>
- <a href="#scope"> Extending the Scope object </a>
- <a href="#extending"> Extending everything! </a>
- <a href="#style"> File Style </a>

<br />
<br />

#### <a name="services"> Services </a>

File's name should be `ServiceName.service.ts`

````javascript

module ModuleName {

    // Export service class, a class should always start with capital letter
    export class ServiceName {

        // variables before constructor
        private _privateVariable: string = 'foo';

        // always use the 'private' keyword when injecting
        constructor (
            // angular specific elements ($) injected first
            private $http: ng.IHttpService,
            // our items injected after
            private someTripService: TR.ISomeTripService
        ) {}

        // Always use generics when returning a promise value
        exposedFunction (foo: string, bar: number[]): ng.IPromise<Models.TypeReturned> {
            this.$http.get('url');
        }

        // use the underscore notation for private items
        // put private functions at the end of the file
        private _privateFunction: number (foo: number) {
            return foo++;
        }
    }

    angular
        .module('moduleName')
        .service('serviceName', ServiceName);
}
````

<br />
<br />

#### <a name="services"> Controllers </a>

File's name should be `ControllerName.controller.ts`

````javascript

module ModuleName {

    // most of the time there is no need to export the controller class
    class ControllerName {

        private _privateVariable: string = 'foo';

        // exposed on scope
        someValue: string = 'bar';

        constructor (
            // use `ng` keyword when injecting angular elements
            private $rootScope: ng.IRootScopeService
            private someTripService: TR.ISomeTripService
        ) {}

        // pass all the external parameters to the function via template, if
        // possible avoid using closures
        exposedOnScope (foo: number, bar: string) : void {
            this.someTripService.someExposedFunction(foo, bar);
        }

        // a function should always return a type
        private _someCalculation (firstVal: number, secondVal: number) : void {
            console.log(firstVal + secondVal);
        }

        // Use the `angular` keyword when creating controllers/services/...
        angular
            .module('moduleName')
            .controller('controllerName', ControllerName);
    }

}

````

<br />
<br />

#### <a name="directives"> Directives </a>

File's name should be `DirectiveName.directive.ts`
While controllers and services are instance of a class a directive expects a
function returning an object, in order to adopt the same 'class approach' we
have on all the other elements we will make use of the `bindToController`
property which let us bound properties to a controller instead of the directive
scope ([full explanation](http://blog.thoughtram.io/angularjs/2015/01/02/exploring-angular-1.3-bindToController.html))

````javascript

module ModuleName {

    class DirectiveName {

        private _privateVariable: string = 'foo';

        // the property is binded to this variable,
        // it needs to be explicitly declared
        propertyFromOutside: string;

        // exposed on scope
        someValue: string = 'bar';

        constructor (
            private $rootScope: ng.IRootScopeService
            private someTripService: TR.ISomeTripService
        ) {}

        exposedOnScope () : void {
            console.log(this.propertyFromOutside);
        }
    }

    angular
        .module('moduleName')
        .directive('directiveName', (): ng.IDirective => {
            return {
                scope: {},
                bindToController: {
                    propertyFromOutside: '='
                },
                controller: DirectiveName,
                controllerAs: 'dn',
                link: function ($scope, $el, $attrs, ctrl) {
                    /*
                        any DOM interaction with the directive element is described
                        here - you have access to all the exposed functions of the
                        controller, avoid using this function for any other reason
                    */
                    $el.on('click', ctrl.exposedOnScope);
                }
            }
        });
}

````

<br />
<br />

#### <a name="classinterface"> Class or Interface? </a>

In Typescript, most of the time, `classes` and `interfaces` are interchangeable (a class can be used
as a type exactly like an interface). Usually the developer should decide on a case by case basis.
One tip though: if we just need the shape of the object (i.e. we get it only
as a promise result) we can prefer an `interface`, on the other side if we expect to need an instance
of that type of object we can just pick a `class`.

*repetita iuvant* - bear in mind that this is not an absolute rule - just a suggestion.

<br />
<br />

#### <a name="letvar"> Let & var </a>

> **DO NOT USE VAR**

`var` hoists the variable at the top of the enclosing function block, this behaviour can be replicated with a
good positioning of `let`.

<br />
<br />

#### <a name="namespacing"> Namespacing </a>

Every module should be self-contained in an internal typescript module:

````javascript
    module ModuleName {
        export class Something {

        }
    }
````

Do not use submodules unless strictly necessary.



<br />
<br />

#### <a name="naming"> Naming </a>

Do not use coding structures inside your names:

````javascript
    module ModuleName {
        // Nono
        export class FunctionalityController {

        }

        // Yup!
        export class Functionality {

        }
    }
````

Remember that a name should be unique only inside the containing module:

````javascript
    module ModuleOne {
        export class Foo {

        }
    }

    module ModuleTwo {
        export class Foo {

        }
    }

    // This is perfectly fine as the full name is descriptive enough.
    module Module {
        class Bar {
            let varOne: ModuleOne.Foo,
                varTwo: ModuleTwo.Foo
        }
    }

````

<br />
<br />

#### <a name="any"> Any </a>

From the [Typescript Handbook](http://www.typescriptlang.org/Handbook#basic-types-any):

> The 'any' type is a powerful way to work with existing JavaScript, allowing you to gradually opt-in and opt-out of type-checking during compilation.

While the use of `any` will be allowed by the compiler (for the moment), you should probably **not** use it.
The whole point of moving to Typescript is improving the tooling and the code quality via a static type
language, using `any` defeats this purpose.

So, if you find yourself thinking that `any` is absolutely needed in the piece of code you're writing,
double check if there is any (ahah) other way to do it. Then triple check.

<br />
<br />

#### <a name="scope"> Extending the Scope object </a>

The Typescript compiler will produce an error with this syntax

````javascript
    class Controller {
        constructor (
            private $scope: ng.IScope
        )

        foo () {
            this.$scope.bar = 'foo';
        }
    }
````

Because there is no variable `bar` defined on the `ng.IScope` interface.
In order to avoid this problem there are two solutions:

Using bracket notations:

````javascript
    class Controller {
        constructor (
            private $scope: ng.IScope
        )

        foo () {
            this.$scope['bar'] = 'foo';
        }
    }
````

Extending the `ng.IScope` interface

````javascript
    interface IMyScope extends ng.IScope {
        bar: string
    }

    class Controller {
        constructor (
            private $scope: IMyScope
        )

        foo () {
            this.$scope.bar = 'foo';
        }
    }
````

<br />
<br />

#### <a name="extending"> Extending everything! </a>

Let's consider this:

````javascript

// Replaces string vars by argument index+1
// i.e. {0} is replaced by second argument
String.format = function(tokenised){
    let args = arguments;
    return tokenised.replace(/{[0-9]}/g, function(matched){
        matched = matched.replace(/[{}]/g, '');
        return args[parseInt(matched)+1];
    });
};

````

Here we are exteding the basic `String` object by adding the function `.format(tokenised)`, while
this in javascript is perfectly valid in a typescript file this code:

````javascript

let world: string = 'World';
let foo = String.format('Hello {0}!', world);

````

would produce a compiler error, because the type `StringConstructor` isn't exposing any function
called `format(tokenised)`.

This is solved by using
[declaration merging](http://www.typescriptlang.org/Handbook#declaration-merging)
in a proper [declaration file](http://www.typescriptlang.org/Handbook#writing-dts-files):

````javascript
interface StringConstructor {
    format(str : string, ...tokens : any[]) : string;
}
````

<br />
<br />

#### <a name="style"> File Style </a>

- Use a 100 columns limit, files are more readable and it's easier to work with two files side by side
- 4 spaces indentation