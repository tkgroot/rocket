# Configuration >> Overview || 10

The configuration file is `rocket.config.js` or `rocket.config.mjs`.

The config files consist of the following parts:

<!-- prettier-ignore-start -->
```js
import { rocketLaunch } from '@rocket/launch';

/** @type {import('rocket/cli').RocketCliConfig} */
export default ({
  presets: [rocketLaunch()],
  emptyOutputDir: true,
  pathPrefix: 'subfolder-only-for-build',
});
```
<!-- prettier-ignore-end -->

Rocket is primarily build around plugins for each of its systems.

New plugins can be added and all default plugins can be adjusted or even removed by using the following functions.

<!-- prettier-ignore-start -->
```js
/** @type {import('rocket/cli').RocketCliConfig} */
export default ({
  // add remark/unified plugin to the Markdown processing (e.g. enable special code blocks)
  setupUnifiedPlugins: [],

  // add a rollup plugins to the web dev server (will be wrapped with @web/dev-server-rollup) AND the rollup build (e.g. enable json importing)
  setupDevAndBuildPlugins: [],

  // add a plugin to the web dev server (will not be wrapped) (e.g. esbuild for TypeScript)
  setupDevPlugins: [],

  // add a plugin to the rollup build (e.g. optimization steps)
  setupBuildPlugins: [],

  // add a plugin to Eleventy (e.g. a filter packs)
  setupEleventyPlugins: [],

  // add a computedConfig to Eleventy (e.g. site wide default variables like socialMediaImage)
  setupEleventyComputedConfig: [],

  // add a plugin to the cli (e.g. a new command like "rocket my-command")
  setupCliPlugins: [],
});
```
<!-- prettier-ignore-end -->

## Adding Rollup Plugins

For some projects you might want to enable non-standard behaviors like importing JSON files as JavaScript.

```js
import data from './data.json';
```

You can accomplish this with Rollup and dev server plugins. Make sure to add both the dev-server plugin as well as the Rollup plugin, so that the behaviors is the same during development as it is in the production build.

For these cases you can use `setupDevAndBuildPlugins`, which will automatically add the plugin for you to both Rollup and dev-server:

<!-- prettier-ignore-start -->
```js
import json from '@rollup/plugin-json';
import { addPlugin } from 'plugins-manager';

/** @type {import('@rocket/cli').RocketCliOptions} */
export default ({
  setupDevAndBuildPlugins: [
    addPlugin({ name: 'json', plugin: json, location: 'top', options: { my: 'settings' } }),
  ],
});
```
<!-- prettier-ignore-end -->

This will add the Rollup plugin `json` with the id `json` at the top of the plugin list of Rollup and the dev server. It needs to be at the top so further plugins down the line can work with JSON imports.
For the Dev Server the plugins are automatically wrapped by `@web/dev-server-rollup`. Note that [not all Rollup plugins](https://modern-web.dev/docs/dev-server/plugins/rollup/#compatibility-with-rollup-plugins) will work with the dev-server.

## Modifying Options of Plugins

All plugins which are either default or are added via a preset can still be adjusted by using `adjustPluginOptions`.

<!-- prettier-ignore-start -->
```js
import { adjustPluginOptions } from 'plugins-manager';

/** @type {import('@rocket/cli').RocketCliOptions} */
export default ({
  setupDevAndBuildPlugins: [adjustPluginOptions('json', { my: 'overwrite settings' })],
});
```
<!-- prettier-ignore-end -->

## Lifecycle

You can hook into the rocket lifecycle by specifying a function for `before11ty`. This function runs before 11ty calls it's write method. If it is an async function, Rocket will await it's promise.

<!-- prettier-ignore-start -->
```js
/** @type {import('rocket/cli').RocketCliConfig} */
export default ({
  async before11ty() {
    await copyDataFiles();
  },
});
```
<!-- prettier-ignore-end -->

## Advanced

Sometimes you need even more control over specific settings.

### Rollup

For example if you wanna add an `acron` plugin to rollup

<!-- prettier-ignore-start -->
```js
import { importAssertions } from 'acorn-import-assertions';

/** @type {import('rocket/cli').RocketCliConfig} */
export default ({
  rollup: config => ({
    ...config,
    acornInjectPlugins: [importAssertions],
  }),
});
```
<!-- prettier-ignore-end -->

### Eleventy

For example to add custom filter you can access the eleventy config directly

<!-- prettier-ignore-start -->
```js
/** @type {import('rocket/cli').RocketCliConfig} */
export default ({
  eleventy: eleventyConfig => {
    eleventyConfig.addFilter('value', value => `prefix${value}`);
  },
});
```
<!-- prettier-ignore-end -->

You even have access to the full rocketConfig if you for example want to create filters that behave differently during start/build.

<!-- prettier-ignore-start -->
```js
/** @type {import('rocket/cli').RocketCliConfig} */
export default ({
  eleventy: (config, rocketConfig) => {
    config.addFilter('conditional-resolve', value => {
      if (rocketConfig.command === 'build') {
        return `build:${value}`;
      }
      if (rocketConfig.command === 'start') {
        return `start:${value}`;
      }
    });
  },
});
```
<!-- prettier-ignore-end -->
