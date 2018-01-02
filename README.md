# esri-loader
A tiny library to help load modules from either the [4.x](https://developers.arcgis.com/javascript/) or [3.x](https://developers.arcgis.com/javascript/3/) versions of the [ArcGIS API for JavaScript](https://developers.arcgis.com/javascript/) in non-Dojo applications.

![ArcGIS logo, mended broken heart, Angular logo, Ember logo, Rreact logo, Vue logo](https://docs.google.com/drawings/d/e/2PACX-1vSUEfgaupMLz6FXBX65X-nm7cqA0r9ed3rJ_KNISeqzwDDkd8LsubLhQ_hCWwO3zjS41cD5eG7QUBHl/pub?w=888&h=222)

See below for more information on [why this library is needed](#why-is-this-needed) and how it can help improve application load performance and allow using the ArcGIS API in isomorphic/universal applications.

**NOTE**: If you want to use the ArcGIS API in an [Ember](#ember), or Angular 1 application, you should use one of these libraries instead:
 - [ember-esri-loader](https://github.com/Esri/ember-esri-loader) - An Ember addon that wraps this library
 - [angular-esri-map](https://github.com/Esri/angular-esri-map), which is actually where the code in this library was originally extracted from

Otherwise you'll want to follow the [Install](#install) and [Usage](#usage) instructions below to use this library directly in your application.

See the [Examples](#examples) section below for links to applications that use this library.

## Install
```bash
npm install --save esri-loader
```

## Usage
The code snippets below show how to load the ArcGIS API and it's modules and then use them to create a map. Where you would place similar code in your application will depend on which application framework you are using. See below for [example applications](#examples).

### Loading Styles

Before you can use the ArcGIS API in your app, you'll need to load the styles. For example, if you're using the latest 4.x version (the default):

```css
/* esri styles */
@import url('https://js.arcgis.com/4.6/esri/css/main.css');
```

If you're using a specific version other than the latest 4.x:

```css
/* esri styles */
@import url('https://js.arcgis.com/3.23/esri/css/esri.css');
```

### Loading Modules from the ArcGIS API for JavaScript

Here's an example of how you could load and use the 4.x `Map` and `MapView` classes in a component to create a map (based on [this sample](https://developers.arcgis.com/javascript/latest/sample-code/sandbox/index.html?sample=webmap-basic)):

```js
// first, we use Dojo's loader to require the map class
esriLoader.loadModules(['esri/views/MapView', 'esri/WebMap'])
.then(([MapView, WebMap]) => {
  // then we load a web map from an id
  var webmap = new WebMap({
    portalItem: { // autocasts as new PortalItem()
      id: 'f2e9b762544945f390ca4ac3671cfa72'
    }
  });
  // and we show that map in a container w/ id #viewDiv
  var view = new MapView({
    map: webmap,
    container: 'viewDiv'
  });
})
.catch(err => {
  // handle any errors
  console.error(err);
});
```

#### Lazy Loading the ArcGIS API for JavaScript

If users may never end up visiting any map routes, you can lazy load the ArcGIS API for JavaScript the first time a user visits a route with a map. 

In the above snippet, the first time `loadModules()` is called, it will attempt to lazy load the most recent 4.x version of the ArcGIS API by calling `loadScript()` for you if the API has not already been loaded. The snippet below uses version 3.x of the ArcGIS API to create a map.

```js
// if the API hasn't already been loaded (i.e. the frist time this is run)
// loadModules() will call loadScript() and pass these options, which, 
// in this case are only needed b/c we're using v3.x instead of the latest 4.x
const options = {
  url: 'https://js.arcgis.com/3.23/'
};
esriLoader.loadModules(['esri/map'], options)
.then(([Map]) => {
  // create map with the given options at a DOM node w/ id 'mapNode'
  let map = new Map('mapNode', {
    center: [-118, 34.5],
    zoom: 8,
    basemap: 'dark-gray'
  });
})
.catch(err => {
  // handle any script or module loading errors
  console.error(err);
});
```

See the [Advanced Usage](#advanced-usage) section below for more advanced techniques such as [pre-loading the ArcGIS API](#pre-loading-the-arcgis-api-for-javascript), [using in isomorphic/universal applications](#isomorphicuniversal-applications) and [configuring Dojo](#configuring-dojo).

## Why is this needed?

Unfortunately, you can't simply `npm install` the ArcGIS API and then `import` ArcGIS modules directly from the modules in a non-Dojo application. The only reliable way to load ArcGIS API for JavaScript modules is using Dojo's AMD loader. However, when using the ArcGIS API in an application built with another framework, you typically want to use the tooling and conventions of that framework rather than the Dojo build system. This library lets you do that by providing an ES module that you can `import` and use to dynamically inject an ArcGIS API script tag in the page and then use its Dojo loader to load only the ArcGIS API modules as needed.

[This blog post](http://tomwayson.com/2016/11/27/using-the-arcgis-api-for-javascript-in-applications-built-with-webpack/) explains in more detail how libraries like this provide a workaround to the challenges of loading ArcGIS API for JavaScript modules from bundlers like [webpack](http://webpack.github.io/) and [rollup.js](https://rollupjs.org/).

Unlike the other techniques and boilerplates discussed in that article, esri-loader is the only solution that also helps:
- improve the performance of initial application load
- allows you to use the ArcGIS API in [isomorphic/universal applications]((https://medium.com/airbnb-engineering/isomorphic-javascript-the-future-of-web-apps-10882b7a2ebc#.4nyzv6jea))

This is because esri-loader let's you control when the ArcGIS API and its modules are loaded and used. You can [lazy load the API and modules](#lazy-loading-the-arcgis-api-for-javascript) only on routes that require them to render a map, or you can [pre-load the API](#pre-loading-the-arcgis-api-for-javascript) without blocking rendering. You can [ensure the API and its modules are not loaded/used when rendering on the server](#isomorphicuniversal-applications) where they will cause errors.

## Examples

Here are some applications and framework-specific wrapper libraries that use this library (presented by framework in alphabetical order - not picking any favories here :stuck_out_tongue_winking_eye:):

### [Angular](https://angular.io/)
- [esri-angular-cli-example](https://github.com/tomwayson/esri-angular-cli-example) - Example of how to to use the ArcGIS API for JavaScript in an Angular CLI app, which uses the [angular-esri-components](https://github.com/TheKeithStewart/angular-esri-components) library.

### [Electron](https://electron.atom.io/)
- [ng-cli-electron-esri](https://github.com/TheKeithStewart/ng-cli-electron-esri) - This project is meant to demonstrate how to run a mapping application using the ArcGIS API for JavaScript inside of Electron.

### [Ember](https://www.emberjs.com/)
- [ember-esri-loader Dummy App](http://ember-esri-loader.surge.sh/) - The dummy application for the ember-esri-loader addon

### [Glimmer.js](https://glimmerjs.com/)
 - [esri-glimmer-example](https://github.com/tomwayson/esri-glimmer-example) - An example of how to use the ArcGIS API for JavaScript in a https://glimmerjs.com/ application
 
### [Preact](https://github.com/developit/preact)
- [esri-preact-pwa](https://github.com/tomwayson/esri-preact-pwa) - An example progressive web app (PWA) using the ArcGIS API for JavaScript built with Preact

### [React](https://facebook.github.io/react/)
- [esri-loader-react-starter-kit](https://github.com/tomwayson/esri-loader-react-starter-kit) - A fork of the [react-starter-kit](https://github.com/kriasoft/react-starter-kit) showing how to use esri-loader in an isomorphic/universal React application
- [esri-loader-react](https://github.com/davetimmins/esri-loader-react) - A React component wrapper around esri-loader
- [esri-react-router-example](https://github.com/tomwayson/esri-react-router-example) - An example react-router application that uses [esri-loader-react](https://github.com/davetimmins/esri-loader-react) to preload the ArcGIS API
 - [create-react-app-esri-loader](https://github.com/davetimmins/create-react-app-esri-loader/) - An example create-react-app application that uses [esri-loader-react](https://github.com/davetimmins/esri-loader-react) to load the ArcGIS API
- [React-Typescript-App-with-ArcGIS-JSAPI](https://github.com/guzhongren/React-Typescript-App-with-ArcGIS-JSAPI) - An example create-react-app application that uses[esri-loader](https://github.com/Esri/esri-loader) ,[esri-loader-react](https://github.com/davetimmins/esri-loader-react),[Typescript](http://www.typescriptlang.org/),[Webpack3](https://webpack.js.org/) to create MapView

### [Vue.js](https://vuejs.org/)
- [CreateMap](https://github.com/oppoudel/CreateMap) - Create Map: City of Baltimore - https://gis.baltimorecity.gov/createmap/#/
- [City of Baltimore: Map Gallery](https://github.com/oppoudel/MapGallery_Vue) - Map Gallery built with Vue.js that uses this library to load the ArcGIS API

## Advanced Usage

### Pre-loading the ArcGIS API for JavaScript

If you have good reason to believe that the user is going to transition to a map route, you may want to start pre-loading the ArcGIS API as soon as possible w/o blocking rendering, for example:

```js
// preload the ArcGIS API
// NOTE: in this case, we're not passing any options to loadScript()
// so it will default to loading the latest 4.x version of the API from the CDN
this.loadScriptPromise = esriLoader.loadScript();

// later, for example once a component has been rendered,
// you can wait for the above promise to resolve (if it hasn't already)
this.loadScriptPromise
.then(() => {
  // you can now load the map modules and create the map
})
.catch(err => {
  // handle any script loading errors
  console.error(err);
});
```

### Isomorphic/universal applications

This library also allows you to use the ArcGIS API in [isomorphic or universal](https://medium.com/airbnb-engineering/isomorphic-javascript-the-future-of-web-apps-10882b7a2ebc#.4nyzv6jea) applications that are rendered on the server. There's really no difference in how you invoke the functions exposed by this libary, however you should avoid trying to call them from any code that runs on the server. The easiest way to do this is to use them in component lifecyle hooks that are only invoked in a browser, for example, React's [`componentDidMount`](https://reactjs.org/docs/react-component.html#componentdidmount). See [tomwayson/esri-loader-react-starter-kit](https://github.com/tomwayson/esri-loader-react-starter-kit/) for [an example of a component that lazy loads the ArcGIS API and renders a map only once a specific route is loaded in a browser](https://github.com/tomwayson/esri-loader-react-starter-kit/commit/a513b7fe207a809105fcb621a26a687cc47918b4).

Alternatively, you could use checks like the following to ensure these functions aren't invoked on the server:

```js
if (typeof window !== 'undefined') {
  // this is running in a browser, 
  // pre-load the ArcGIS API for later use in components
  this.loadScriptPromise = esriLoader.loadScript();
}
```

### Configuring Dojo

You can pass a [`dojoConfig`](https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config/) option to `loadScript()` or `loadModules()` to configure Dojo before the script tag is loaded. This is useful if you want to use esri-loader to load Dojo packages that are not included in the ArcGIS API for JavaScript such as [FlareClusterLayer](https://github.com/nickcam/FlareClusterLayer).

```js
// in this case options are only needed so we can configure dojo before loading the API
const options = {
  // tell Dojo where to load other packages
  dojoConfig: {
    async: true,
    packages: [
      {
        location: '/path/to/fcl',
        name: 'fcl'
      }
    ]
  }
};
esriLoader.loadModules(['esri/map', 'fcl/FlareClusterLayer_v3'], options)
.then(([Map, FlareClusterLayer]) => {
  // you can now create a new FlareClusterLayer and add it to a new Map
})
.catch(err => {
  // handle any errors
  console.error(err);
});
```

### Using your own script tag

It is possible to use this library only to load modules (i.e. not to pre-load or lazy load the ArcGIS API). In this case you will need to add a `data-esri-loader` attribute to the script tag you use to load the ArcGIS API for JavaScript. Example:

```html
<!-- index.html -->
<script src="https://js.arcgis.com/3.23/" data-esri-loader="loaded"></script>
```

### ArcGIS Types
This library doesn't make any assumptions about which version of the ArcGIS API you are using, so you will have to manually install the appropriate types.

#### 4.x Types
Follow [these instructions](https://github.com/Esri/jsapi-resources/tree/master/4.x/typescript) to install the 4.x types.

NOTE: For Angular CLI applications, you will also need to add "arcgis-js-api" to `compilerOptions.types` in src/tsconfig.app.json and src/tsconfig.spec.json [as shown here](https://gist.github.com/tomwayson/e6260adfd56c2529313936528b8adacd#adding-the-arcgis-api-for-javascript-types).

Then you can use the `__esri` namespace for the types as seen in [this example](https://github.com/kgs916/angular2-esri4-components/blob/68861b286fd3a4814c495c2bd723e336e917ced2/src/lib/esri4-map/esri4-map.component.ts#L20-L26).

#### 3.x Types
Unfortunately the `__esri` namespace is not defined for 3.x types. You can use [these instructions](https://github.com/Esri/jsapi-resources/tree/master/3.x/typescript) to install the 3.x types, but then [you will still need to use `import` statements to get the types](https://github.com/Esri/jsapi-resources/issues/60). This may cause build errors that may or may not result in actual runtime errors depending on your environment.

## Dependencies

### Browsers

This library doesn't have any external dependencies, but the functions it exposes to load the ArcGIS API and it's modules expect to be run in a browser. You cannot run the ArcGIS API for JavaScript in Node, but [you _can_ use this library to isomorphic/universal applications](#isomorphicuniversal-applications).

This library officially supports [the same browers that are supported by the latest version of the ArcGIS API for JavaScript](https://developers.arcgis.com/javascript/latest/guide/system-requirements/index.html#supported-browsers). Since this library also works with [v3.x of the ArcGIS API](https://developers.arcgis.com/javascript/3/), the community [has made some effort](https://github.com/Esri/esri-loader/pull/67) to get it to work with [some of the older browsers supported by 3.x](https://developers.arcgis.com/javascript/3/jshelp/supported_browsers.html) like IE < 11.

### Promises

Since v1.5 asynchronous functions like `loadScript()` and `loadModules()` return [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)s, so if your application has to support [browers that don't support Promise (i.e. IE)](https://caniuse.com/#search=promise) you have a few options.

If there's already a Promise implementation loaded on the page you can configure esri-loader to use that implementation. For example, in [ember-esri-loader](https://github.com/Esri/ember-esri-loader), we configure esri-loader to use the RSVP Promise implementation included with Ember.js.

```js
  init () {
    this._super(...arguments);
    // have esriLoader use Ember's RSVP promise
    esriLoader.utils.Promise = Ember.RSVP.Promise;
  },
```

Otherwise, you should consider using a [Promise polyfill](https://www.google.com/search?q=promise+polyfill), ideally [only when needed](https://philipwalton.com/articles/loading-polyfills-only-when-needed/).

Finally, for now you can still use `bootstrap()` and `dojoRequire()` which are the callback-based equivalents of the above functions. See the [v1.4.0 documentation](https://github.com/Esri/esri-loader/blob/v1.4.0/README.md#usage) for how to use the callback-based API, but _keep in mind that these functions have been deprecated and will be removed at the next major release_.

## Issues

Find a bug or want to request a new feature?  Please let us know by [submitting an issue](https://github.com/Esri/esri-loader/issues/).

## Contributing

Esri welcomes contributions from anyone and everyone. Please see our [guidelines for contributing](https://github.com/esri/contributing).

## Licensing
Copyright 2017 Esri

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

A copy of the license is available in the repository's [license.txt]( ./license.txt) file.
