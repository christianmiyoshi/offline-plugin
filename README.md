<div align="center">
# `offline-plugin` for webpack

[![npm](https://img.shields.io/npm/v/offline-plugin.svg?maxAge=3600&v4)](https://www.npmjs.com/package/offline-plugin)
[![npm](https://img.shields.io/npm/dm/offline-plugin.svg?maxAge=3600)](https://www.npmjs.com/package/offline-plugin)
[![Join the chat at https://gitter.im/NekR/offline-plugin](https://badges.gitter.im/NekR/offline-plugin.svg)](https://gitter.im/NekR/offline-plugin?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


[![offline-plugin](https://rawgit.com/NekR/offline-plugin/v4/logo/logo.svg)](https://github.com/NekR/offline-plugin)
</div>

This plugin is intended to provide an offline experience for **webpack** projects. It uses **ServiceWorker**, and **AppCache** as a fallback under the hood. Simply include this plugin in your ``webpack.config``, and the accompanying runtime in your client script, and your project will become offline ready by caching all (or some) of the webpack output assets.

## Install

`npm install offline-plugin [--save-dev]`

## Setup

First, instantiate the plugin with [options](#options) in your `webpack.config`:

```js
// webpack.config.js example

var OfflinePlugin = require('offline-plugin');

module.exports = {
  // ...

  plugins: [
    // ... other plugins
    // it always better if OfflinePlugin is the last plugin added
    new OfflinePlugin()
  ]
  // ...
}

```

Then, add the [runtime](docs/runtime.md) into your entry file (typically main entry):

```js
require('offline-plugin/runtime').install();
```

## Docs

* [Caches](docs/caches.md)
* [Update process](docs/updates.md)
* [FAQ](FAQ.md)

## Options

**All options are optional and `offline-plugin` can be used without specifying them.** Also see full list of default options [here](https://github.com/NekR/offline-plugin/blob/master/src/index.js#L9).

#### `caches: 'all' | Object`

Allows you to define what to cache and how.

* `'all'`: means that everything (all the webpack output assets) and URLs listed in `externals` option will be cached on install.
* `Object`: Object with 3 possible `Array<string>` sections (properties): `main`, `additional`, `optional`. All sections are optional and by default are empty (no assets added).

[More details about `caches`](docs/caches.md)

> Default: `'all'`.

#### `publicPath: string`

Same as `webpack`'s `output.publicPath` option. Useful to specify or override `publicPath` specifically for `offline-plugin`. When not specified, `webpack`'s `output.publicPath` value is used. When `webpack`'s `output.publicPath` value isn't specified, relative paths are used (see `relativePaths` option).

> __Examples:__  
`publicPath: '/project/'`  
`publicPath: 'https://example.com/project'`  

#### `responseStrategy: 'cache-first' | 'network-first'`
Response strategy. Whether to use a cache or network first for responses.
> Default: `'cache-first'`.

#### `updateStrategy: 'changed' | 'all'`
Cache update strategy. [More details about `updateStrategy`](docs/update-strategies.md)  
> Default: `'changed'`.

#### `externals: Array<string>`
Allows you to specify _external_ assets (assets which aren't generated by webpack) that should be included in the cache. If you don't change the `caches` configuration option then it should be enough to simply add assets to this (`externals`) option. For other details and more advanced use cases see [`caches` docs](docs/caches.md).

> Default: `null`  
> **Example value:** `['fonts/roboto.woff']`

#### `excludes: Array<string | globs_pattern>`
Excludes matched assets from being added to the [caches](https://github.com/NekR/offline-plugin#caches-all--object). Exclusion is performed before _rewrite_ happens.  
[Learn more about assets _rewrite_](docs/rewrite.md)

> Default: `['**/.*', '**/*.map']`  
> _Excludes all files which start with `.` or end with `.map`_

#### `relativePaths: boolean`
When set to `true`, all the asset paths generated in the cache will be relative to the `ServiceWorker` file or the `AppCache` folder location respectively.  
`publicPath` option is ignored when this is **explicitly** set to `true`.
> **Default:** `true`

#### `version: string | (plugin: OfflinePlugin) => void`
Version of the cache. Can be a function, which is useful in _watch-mode_ when you need to apply dynamic value.

* `Function` is called with the plugin instance as the first argument
* `string` which can be interpolated with `[hash]` token

> Default: _Current date_

#### `rewrites: Function | Object`

Rewrite function or rewrite map (`Object`). Useful when assets are served in a different way from the client perspective, e.g. usually `index.html` is served as `/`.

[See more about `rewrites` option and default function](docs/rewrite.md)

#### `cacheMaps: Array<Object>`

See [documentation of `cacheMaps`](docs/cache-maps.md) for syntax and usage examples

#### `autoUpdate: true | number`

Enables automatic updates of ServiceWorker and AppCache. If set to `true`, it uses default interval of _1 hour_. Set a `number` value to have provide custom update interval.

_**Note:** Please not that if user has multiple opened tabs of your website then update may happen more often because each opened tab will have its own interval for updates._

> Default: `false`  
> **Example:** `true`  
> **Example:** `1000 * 60 * 60 * 5` (five hours)

#### `ServiceWorker: Object | null | false`

Settings for the `ServiceWorker` cache. Use `null` or `false` to disable `ServiceWorker` generation.

* `output`: `string`. Relative (from the _webpack_'s config `output.path`) output path for emitted script.  
_Default:_ `'sw.js'`

* `entry`: `string`. Relative or absolute path to the file which will be used as the `ServiceWorker` entry/bootstrapping. Useful to implement additional features or handlers for Service Worker events such as `push`, `sync`, etc.  
_Default:_ _empty file_

* `scope`: `string`. Reflects [ServiceWorker.register](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerContainer/register)'s `scope` option.  
_Default:_ `null`

* `cacheName`: `string`. **This option is very dangerous. Touching it you must realize that you should **not** change it after you go production. Changing it may corrupt the cache and leave old caches on users' devices. This option is useful when you need to run more than one project on _the same domain_.  
_Default:_ _`''`_ (empty string)
_Example:_ `'my-project'`

* `navigateFallbackURL`: `string`. The URL that should be returned from the cache when a requested navigation URL isn't available on the cache or network. Similar to the `AppCache.FALLBACK` option.  
_Example:_ `navigateFallbackURL: '/'`

* `events`: `boolean`. Enables runtime events for the ServiceWorker. For supported events see `Runtime`'s `install()` options.
_Default:_ `false`

* `publicPath`: `string`. Provides a way to override `ServiceWorker`'s script file location on the server. Should be an exact path to the generated `ServiceWorker` file.
_Default:_ `null`
_Example:_ `'my/new/path/sw.js'`

* `prefetchRequest`: `Object`. Provides a way to specify [request init options](https://developer.mozilla.org/en-US/docs/Web/API/Request/Request) for pre-fetch requests (pre-cache requests on `install` event). Allowed options: `credentials`, `headers`, `mode`, `cache`.  
_Default:_ `{ credentials: 'omit', mode: 'cors' }`  
_Example:_ `{ credentials: 'same-origin' }`  

#### `AppCache: Object | null | false`

Settings for the `AppCache` cache. Use `null` or `false` to disable `AppCache` generation.

 > _**Warning**_: Officially the AppCache feature [has been deprecated](https://developer.mozilla.org/en-US/docs/Web/HTML/Using_the_application_cache) in favour of Service Workers.  However, Service Workers are still being implemented across all browsers (you can track progress [here](https://jakearchibald.github.io/isserviceworkerready/)) so AppCache is unlikely to suddenly disappear.  Therefore please don't be afraid to use the AppCache feature if you have a need to provide offline support to browsers that do not support Service Workers, but it is good to be aware of this fact and make a deliberate decision on your configuration.

* `directory`: `string`. Relative (from the _webpack_'s config `output.path`) output directly path for the `AppCache` emitted files.  
_Default:_ `'appcache/'`

* `NETWORK`: `string`. Reflects `AppCache`'s `NETWORK` section.  
_Default:_ `'*'`

* `FALLBACK`: `Object`. Reflects `AppCache`'s `FALLBACK` section. Useful for single page applications making use of HTML5 routing or for displaying custom _Offline page_.  
_Example 1:_ `{ '/blog': '/' }` will map all requests starting with `/blog` to the domain roboto when request fails.  
_Example 2:_ `{ '/': '/offline-page.html' }` will return contents of `/offline-page.html` for any failed request.  
_Default:_ `null`

* `events`: `boolean`. Enables runtime events for AppCache. For supported events see `Runtime`'s `install()` options.  
_Default:_ `false`

* `publicPath`: `string`. Provides a way to override `AppCache`'s folder location on the server. Should be exact path to the generated `AppCache` folder.  
_Default:_ `null`  
_Example:_ `'my/new/path/appcache'`

* `disableInstall` :`boolean`. Disable the automatic installation of the `AppCache` when calling to `runtime.install()`. Useful when you to specify `<html manifest="...">` attribute manually (to cache every page user visits).  
_Default:_ `false`

* `includeCrossOrigin` :`boolean`. Outputs cross-origin URLs into `AppCache`'s manifest file. **Cross-origin URLs aren't supported in `AppCache` when used on HTTPS.**  
_Default:_ `false`

## Who is using `offline-plugin`

### Projects

* [React Boilerplate](https://github.com/mxstbr/react-boilerplate)
* [Phenomic](https://phenomic.io)
* [Gatsby](https://github.com/gatsbyjs/gatsby)
* [Angular CLI](https://github.com/angular/angular-cli)
* [React, Universally](https://github.com/ctrlplusb/react-universally)

### PWAs

* [Offline Kanban](https://offline-kanban.herokuapp.com) ([source](https://github.com/sarmadsangi/offline-kanban))
* [Preact](https://preactjs.com/) ([source](https://github.com/developit/preact-www))
* [Omroep West (_Proof of Concept_)](https://omroep-west.now.sh/)


_If you are using `offline-plugin`, feel free to submit a PR to add your project to this list._

## Like `offline-plugin`?

Support it by giving [feedback](https://github.com/NekR/offline-plugin/issues), contributing or just by 🌟 starring the project!


## License

[MIT](LICENSE.md)


## CHANGELOG

[CHANGELOG](CHANGELOG.md)
