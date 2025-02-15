webpack-imported
======
📝stats-webpack-plugin and 💩webpack-flush-chunks had a baby!


# Server side API
## Webpack plugin
```js
const {ImportedPlugin} = require('webpack-imported/server');

module.exports = {
  plugins: [
    new ImportedPlugin('imported.json')
  ]
};
```
This will output `imported.json` as a one of emitted assets, with all the information carefully sorted.

## SSR API
- `importedAssets(stats, chunks, [tracker])` - return all assets accosiated with provided chunks.
Could be provided a `tracker` to prevent duplications between runs.
- `createImportedTracker()` - creates a duplication prevention tracker

```js
import {importedAssets} from "webpack-imported";
import importedStat from "build/imported.json"; // this file has to be generated

const relatedAssets = importedAssets(importedStat, ['main']); // main is your "main" bundle

relatedAssets.scripts.load // list scripts to load
relatedAssets.scripts.preload // list scripts to preload
relatedAssets.styles.load // list styles to load
relatedAssets.styles.preload // list styles to preload

importedStat.config.publicPath // public path used on build time
```

with tracking
```js
import {importedAssets, createImportedTracker} from "webpack-imported";
import importedStat from "build/imported.json"; // this file has to be generated

const tracker = createImportedTracker();
const relatedAssets1 = importedAssets(importedStat, ['main'], tracker);
// use scripts and styles

const relatedAssets2 = importedAssets(importedStat, ['home'], tracker);
// render only new scripts and styles
```

# Client side API

## React bindings (better at SSR)
- `createImportedTracker()` - creates a duplication prevention tracker
- `WebpackImportedProvider` - wires tracker down to React context
- `WebpackImport` - chunk importer
- `processImportedStyles` - helper for critical styles.
```js
import {createImportedTracker, WebpackImportedProvider, WebpackPrefetch} from "webpack-imported/react";
import importedStat from "build/imported.json";

const tracker = createImportedTracker();// this is optional, only needed is you render is multipart(head/body)

<WebpackImportedProvider tracker={tracker}>
  <WebpackImport stats={importedStat} chunks={['main']} />
</WebpackImportedProvider>  
```

`WebpackImport` has many props:
- [`preload`=false] - only preloads resources. If preload is set resources would be loaded via network, but not executed. 
Never use this option for the main chunk.
- [`async`]=true] - loads scripts with `async` tag, use `deferred` in other case.
- [`critical-styles`=false] - enabled critical styles handling. No styles would be loaded or prefetched,
but system would leave extra markup to prevent `MiniCssExtractPlugin` from adding them by itself.
With this option enabled __you have to call__ `processImportedStyles` after application start to load the missing styles. 



# Related
### Get stats from webpack
- [stats-webpack-plugin](https://github.com/unindented/stats-webpack-plugin)
- [loadable components webpack plugin](https://github.com/smooth-code/loadable-components/tree/master/packages/webpack-plugin)

### Handle chunks dependencies
- [webpack-flush-chunks](https://github.com/faceyspacey/webpack-flush-chunks)


# Licence 
MIT