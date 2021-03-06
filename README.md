angulate2
===========
[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/angulate2/Lobby)
[![Scala.js](https://www.scala-js.org/assets/badges/scalajs-0.6.13.svg)](https://www.scala-js.org)
<!--[![Build Status](https://travis-ci.org/jokade/angulate2.svg?branch=master)](https://travis-ci.org/jokade/angulate2)-->

[Scala.js](http://www.scala-js.org/) bindings for [Angular 2](http://www.angular.io). The goal is to provide an API/ experience very similar to the [TypeScript API](https://angular.io/docs/ts/latest/guide/cheatsheet.html) of Angular 2.

**WARNING: This is work in progress!**  
Many of the features of Angular 2 are still missing and the angulate API may still change significantly.

**IMPORTANT: angulate2 now uses the CommonJS module format introduced with Scala.js 0.6.13 instead of global scope.
  The entire code basis is currently being refactored to facilitate this.**

A basic [Quickstart Example](https://github.com/jokade/angulate2-quickstart) that may serve as template is available, as well as set of **[extended examples](https://github.com/jokade/angulate2-examples)**.

**[Release Notes](https://github.com/jokade/angulate2/wiki/Release-Notes)**

Getting Started
---------------
### SBT Settings
Add the following lines to your `project/plugins.sbt`:
```scala
addSbtPlugin("org.scala-js" % "sbt-scalajs" % "0.6.13")

addSbtPlugin("de.surfice" % "sbt-angulate2" % "0.0.5")
```
and this to your `build.sbt`:
```scala
enablePlugins(ScalaJSPlugin)
enablePlugins(Angulate2Plugin)

ngBootstrap := Some("AppModule") //qualified name (including packages) of Scala class called NAME_OF_THE_MODULE_TO_BOOTSTRAP
```
The current version of angulate2 is built for Angular-2.4+ and Scala.js 0.6.13+.

### Build and run with System.js
With the above configuration, a separate JS file `PROJECT-sjsx.js` is written to `target/scala-2.11/` every time you run `fastOptJS` or `fullOptJS`. This file contains the class decorators generated from Angular2 annotations (@Component, ...) and represents the entry module of your Angular2 application.

If you use System.js to load your application, your `systemjs.config.js` must contain a `map` entry for `scalaModule` which points to the actual JS file generated by Scala.js, e.g.:
```javascript
/**
 * Basic System.js configuration for angulate2.
 * Adjust as necessary for your application needs.
 */
(function (global) {
  System.config({
    paths: {
      // paths serve as alias
     'npm:': 'node_modules/'
    },
    // map tells the System loader where to look for things
    map: {
      // our app is within the target/scala-2.11 folder
      app: 'target/scala-2.11',
      // angular bundles
      '@angular/core': 'npm:@angular/core/bundles/core.umd.js',
      '@angular/common': 'npm:@angular/common/bundles/common.umd.js',
      '@angular/compiler': 'npm:@angular/compiler/bundles/compiler.umd.js',
      '@angular/platform-browser': 'npm:@angular/platform-browser/bundles/platform-browser.umd.js',
      '@angular/platform-browser-dynamic': 'npm:@angular/platform-browser-dynamic/bundles/platform-browser-dynamic.umd.js',
      '@angular/http': 'npm:@angular/http/bundles/http.umd.js',
      '@angular/router': 'npm:@angular/router/bundles/router.umd.js',
      '@angular/forms': 'npm:@angular/forms/bundles/forms.umd.js',
      '@angular/upgrade': 'npm:@angular/upgrade/bundles/upgrade.umd.js',
      // other libraries
      'rxjs':                      'npm:rxjs',
      'angular-in-memory-web-api': 'npm:angular-in-memory-web-api/bundles/in-memory-web-api.umd.js'
    },
    // packages tells the System loader how to load when no filename and/or no extension
    packages: {
      app: {
        // the main script to be loaded from target/scala-2.11
        main: './PROJECT-sjsx.js',
        map: {
          // the name of the Scala.js module to be loaded
          scalaModule: './PROJECT-fastopt.js'
        },
        format: 'cjs',
        defaultExtension: 'js'
      },
      rxjs: {
        defaultExtension: 'js'
      }
    }
  });
})(this);
```
This configuration assumes that the Angular dependecies have been installed with `npm` in the root folder of your app. A simple package.json example would be:
```json
{
  "name": "PROJECT",
  "version": "0.0.1",
  "scripts": {
    "start": "lite-server"
  },
  "dependencies": {
    "@angular/common": "~2.4.0",
    "@angular/compiler": "~2.4.0",
    "@angular/core": "~2.4.0",
    "@angular/forms": "~2.4.0",
    "@angular/http": "~2.4.0",
    "@angular/platform-browser": "~2.4.0",
    "@angular/platform-browser-dynamic": "~2.4.0",
    "@angular/router": "~3.4.0",
    "@angular/upgrade": "~2.4.0",
    "angular-in-memory-web-api": "~0.2.2",
    "core-js": "^2.4.1",
    "reflect-metadata": "^0.1.8",
    "rxjs": "5.0.1",
    "systemjs": "0.19.40",
    "zone.js": "^0.7.4"
  },
  "devDependencies": {
    "lite-server": "^2.2.2"
  }
}
```

Then you only need to import the `app` module from within your `index.html` to bootstrap your Angular application (using the NgModule specified by the `ngBootstrap` property in your `build.sbt`):
```html
<html>
  <head>
    <title>Angulate2 App</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- <link rel="stylesheet" href="styles.css"> -->
    <!-- 1. Load libraries -->
    <!-- Polyfill for older browsers -->
    <script src="node_modules/core-js/client/shim.min.js"></script>
    <script src="node_modules/zone.js/dist/zone.js"></script>
    <script src="node_modules/reflect-metadata/Reflect.js"></script>
    <script src="node_modules/systemjs/dist/system.src.js"></script>
    <!-- 2. Configure System.js -->
    <script src="systemjs.config.js"></script>
    <script>
      System.import('app').catch(function(err){ console.error(err); });
    </script>
  </head>
  <!-- 3. Display the application -->
  <body>
    <my-app>Loading...</my-app>
  </body>
</html>
``` 

### Create root NgModule and Component
```scala
import angulate2.std._
import angulate2.platformBrowser.BrowserModule

@NgModule(
  imports = @@[BrowserModule],
  declarations = @@[AppComponent],
  bootstrap = @@[AppComponent]
)
class AppModule {

}

@Component(
  selector = "my-app",
  template = "<h1>Hello Angular!<h1>"
)
class AppComponent {

}
``` 
Execute `sbt fastOptJS` to build your code and execute a simple server with `npm start` (or open the `index.html` manually in your browser). 

Keeping up one terminal with `npm start` running and the other running `~fastOptJS` within an interactive `sbt` shell will immediately show any changes within the code in your browser.

License
-------
This code is open source software licensed under the [MIT License](http://opensource.org/licenses/MIT).
