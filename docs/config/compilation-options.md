# Configuration Reference

By default, Farm reads the configuration from the `farm.config.ts|js|mjs` file in the project root directory, an example configuration file:

```ts title="farm.config.ts"
import type { UserConfig } from "@farmfe/core";

function defineConfig(config: Config) {
  return config;
}

export default defineConfig({
  root: process.cwd(), // compiled root directory
  // compile options
  compilation: {
    //...
  },
  // Dev Server options
  server: {
    hmr: true,
    //...
  },
  // plugin configuration
  plugins: [],
});
```

## Compilation options - compilation

All compilation-related configuration is under the `compilation` field.

### input

- **type**: `Record<string, string>`

The entry point for the project. Input files can be `html`, `ts/js/tsx/jsx`, `css` or other files supported by plugins.

```tsx
import type { UserConfig } from "@farmfe/core";

export default <UserConfig>{
  compilation: {
    input: {
      index: "./index.html",
      about: "./about.html",
    },
  },
  // ..
};
```

### output

- **type**: `OutputOptions`

```ts
interface OutputOptions {
  // After partial bundling, the file name configuration of the resource where the entry file is located
  entryFilename?: string;
  // After partial bundling, other resources except the entry resource input file name configuration
  filename?: string;
  // output directory
  path?: string;
  // public path: resource loading prefix
  publicPath?: string;
  // Static resource file name configuration
  assetsFilename?: string;
  // Target execution environment, browser or Node
  targetEnv?: "browser" | "node";
  // output module format
  format?: "cjs" | "esm";
}
```

:::note
We call the compiled result a `resource`
:::

#### `output.entryFilename`

- **default**: `"[entryName].[ext]"`

Configure the filename of the entry resource: you can use placeholders such as `[entryName]`. All placeholders are as follows:

- `[entryName]`: entry name, for example, for `input: { index: "./index.html" }`, `[entryName]` is `index`
- `[resourceName]`: The name of the resource, usually a unique hash value generated internally by Farm.
- `[contentHash]`: The content hash of the resource.
- `[ext]`: The extension of the resource, `js` for `js/jsx/ts/tsx`, `css` for `css/scss/less`.

#### `output.filename`

- **Default value**: `"[resourceName].[ext]"`

After partial bundling, other resource file names except the resources configured by `entryFilename`. All placeholders are as follows:

- `[resourceName]`: The name of the resource, usually a unique hash value generated internally by Farm.
- `[contentHash]`: The content hash of the resource.
- `[ext]`: The extension of the resource, `js` for `js/jsx/ts/tsx`, `css` for `css/scss/less`.

#### `output.path`

- **default**: `"dist"`

directory of output resources

#### `output.publicPath`

- **Default value**: `"/"`

The resource url load prefix. For example URL `https://xxxx`, or a absolute path `/xxx/`.

#### `output.assetsFileName`

- **Default value**: `"[resourceName].[ext]"`

The filename configuration for static resource output, the placeholder is the same as `output.filename`.

#### `output. targetEnv`

- **default**: `"browser"`

Configure the execution environment of the product, which can be `"browser"` or `"node"`.

#### `output. format`

- **default**: `"esm"`

The format of the configuration product, which can be `"esm"` or `"cjs"`.

:::note
This option is only valid for Js products
:::

### resolve

- **type**: `ResolveOptions`

```ts
interface ResolveOptions {
  extensions?: string[];
  alias?: Record<string, string>;
  mainFields?: string[];
  conditions?: string[];
  symlinks?: boolean;
  strictExports?: boolean;
}
```

#### `resolve. extensions`

- **default**: `["tsx", "ts", "jsx", "js", "mjs", "json", "html", "css"]`

Configure the suffix when parsing dependencies. For example, when parsing `./index`, if it is not resolved, the suffix parsing will be automatically added, such as trying `./index.tsx`, `./index.css`, etc.

#### `resolve.alias`

- **Default value**: `{}`

Configure parsing alias, example:

```ts
export default defineConfig({
  compilation: {
    resolve: {
      alias: {
        "/@": path.join(process.cwd(), "src"),
        stream$: "readable-stream",
      },
    },
  },
});
```

alias is prefix replacement, for the above example `/@/pages` will be replaced by `/root/src/pages`. If you want an exact match, you can add `$`, for example `stream$` will only replace `stream`, but not `stream/xxx`.

#### `resolve. mainFields`

- **default**: `["exports", "browser", "module", "main"]`

When parsing dependencies under node_modules, the fields and order configured in `mainFields` will be parsed from package.json. For `package.json`

```json
{
  "name": "package-a",
  "module": "es/index.js",
  "main": "lib/index.js"
}
```

Will use `es/index.js` first (if the path exists), and will continue to search backwards if it does not exist.

#### `resolve.conditions`

Configuration is not currently supported.

#### `resolve.symlinks`

- **default**: `true`

When parsing a file, whether to track the real directory corresponding to the symlink, and start parsing the next dependency from the real directory. If pnpm is used to manage dependencies, this option must be configured as true.

#### `resolve. strictExports`

- **default**: `false`

Whether to strictly follow the exports defined in `exports` in `package.json`. If set to true, when `exports` is defined in `package.json`, but `exports` does not define the corresponding export, an error will be reported directly. If set to true, it will continue to try other entries according to mainFields.

### define

- **Default value**: `{}`

Global variable injection, the configured variable name and value will be injected into the product at compile time. Farm injects `process.env.NODE_ENV` and some variables used by Farm itself such as `FARM_HMR_PORT` by default

```ts
export default defineConfig({
  compilation: {
    define: {
      MY_VAR: 123,
    },
  },
});
```

:::note

1. In order to enhance performance, define uses the injection form of global variables, which means that variables in the form of objects cannot be injected, for example, variables in the form of `process.env.XXX` cannot be injected, and only variables in the form of `XXX` can be configured .
2. If multiple Farm projects are mounted under the same window, the defines with the same name in multiple projects will overwrite each other.
3. The injection is a string. If it needs to be converted to another type, it needs to be manually converted in the runtime code, such as `Number(MY_VAR)`
   :::

### external

- **default**: `[]`

Configure the imports that are external, and the imports that are external will not appear in the compiled product. However, the corresponding import statement will not be deleted. You need to customize how to deal with external, otherwise an error will be reported at runtime. If targetEnv is an external module under node, it will automatically try to require the module.

It needs to be configured in a regular way, for example:

```ts
export default defineConfig({
  compilation: {
    external: ["^stream$"],
  },
});
```

### mode

- **default**: `development` for start, watch commands, `production` for build commands

Configure the compilation mode. In order to optimize the performance during development, if there is no manual configuration of production optimization related options (minify, tree shake, etc.), the production environment optimization such as compression and tree shake will be disabled by default under `development`. In `production` mode enabled.

### root

- **default**: `process.cwd()`

Configure the root directory for project compilation. This option will affect the search path of the default configuration file, the search of compiled module dependencies, etc.

### runtime

Configure Farm runtime capabilities. The types are as follows:

```ts
interface FarmRuntimeOptions {
  runtime?: {
    path: string;
    plugins?: string[];
    namespace?: string;
  };
}
```

#### `runtime.path`

- **Default value**: The path of Farm's built-in runtime

Customize a Runtime to replace Farm's built-in Runtime.

:::warning
It is not recommended to configure this option under normal circumstances, because once this option is configured, the pointed runtime needs all
:::

#### `runtime.plugins`

- **Default value**: The path of Farm's built-in runtime-plugin-hmr

Configure the Runtime plug-in, through the Runtime plug-in, you can intervene in Runtime behavior, such as module loading, resource loading, etc. For details, please refer to: WIP.

#### `runtime.namespace`

- **default**: name field of project package.json

Configure the namespace of Farm Runtime to ensure that the execution of different products under the same window or global can be isolated from each other. By default, the name field of the project package.json is used as the namespace.

### assets

#### `assets.include`

- **default**: `[]`

Additional file suffixes that are regarded as static resources, such as the following example, `txt` will be regarded as posture resources, and will be treated as static resources when importing txt files:

```ts
export default defineConfig({
  compilation: {
    assets: {
      include: ["txt"],
    },
  },
});
```

### script

#### `script.target`

- **Default value**: `esnext` (dynamically adjusted according to the iteration of Farm)

Configure Farm to parse the AST of `js/jsx/ts/tsx` and support the ES syntax version when generating code. Possible values: `es5`, `es6`, `es2015` - `es2023`, `esnext`

#### `script.parser`

- **default**: same as SWC

Configure the behavior of SWC when parsing AST, configuration item reference: https://swc.rs/docs/configuration/compilation#jscparser

#### `script.plugins`

- **default**: `[]`

Configure the swc plugin array, each item of the array contains three fields:

- **name**: the package name of the swc plugin
- **options**: Configuration items passed to swc plugin
- **filters**: Which modules to execute the plug-in, must be configured, support `resolvedPaths` and `moduleTypes` these two filter items, if both are specified at the same time, take the union.

An example of a configuration that supports JSX for a Vue project is as follows:

```ts
import jsPluginVue from "@farmfe/js-plugin-vue";

/**
 * @type {import('@farmfe/core').UserConfig}
 */
export default {
  compilation: {
    script: {
      plugins: [
        {
          name: "swc-plugin-vue-jsx",
          options: {
            transformOn: true,
            optimize: true,
          },
          filters: {
            // resolvedPaths: [".+"]
            moduleTypes: ["tsx", "jsx"],
          },
        },
      ],
    },
  },
  plugins: [jsPluginVue()],
};
```

#### `script.decorators`

```ts
export interface DecoratorsConfig {
  legacyDecorator: boolean;
  decoratorMetadata: boolean;
  /**
   * The version of the decorator proposal to use. 2021-12 or 2022-03
   * @default 2021-12
   */
  decoratorVersion: "2021-12" | "2022-03" | null;
  /**
   * @default []
   */
  includes: string[];
  /**
   * @default ["node_modules/"]
   */
  excludes: string[];
}
```

It's recommended to use default decorators configuration of Farm, unless you want to improve performance, you can set `includes` and `excludes`.

Options:

- **legacyDecorator**: default to `true`. Using legacy decorator proposal.
- **decoratorMetadata**: default to `false`. You have to set `legacyDecorator` to `false` if you want to set it to true.
- **decoratorVersion**: default to '2021-12', proposal version. The value is 2021-12 or 2022-03.
- **includes**: default to `[]`. If you want to include modules that are excluded, you can set this option. Regex supported.
- **excludes**: default to `['node_modules/']`. Modules under these paths are ignored when transform decorators. Regex supported

### css

#### `css.modules`

Configure Farm CSS Modules.

```ts
interface FarmCssModulesConfig {
  // Configure which paths will be processed as css modules, using regular strings
  // defaults to `.module.css` or `.module.scss` or `.module.less`
  paths?: string[];
  // configure the generated css class name, the default is `[name]-[hash]`
  indentName?: string;
}
```

##### `css.modules.paths`

- **default**: `["\\.module\\.(css|scss|sass|less)"]`

Configure which paths correspond to modules that will be treated as CSS Modules. A regular string needs to be configured. Defaults to files ending in `.module.(css|scss|sass|less)`.

##### `css.modules.identName`

- **default**: `[name]-[hash]`

Configure the generated CSS Modules class name, the default is `[name]-[hash]`, `[name]`, `[hash]` are placeholders (also all currently supported placeholders). `[name]` means the original class name, `[hash]` means the hash of the modified css file id.

#### `css.prefixer`

Configure CSS compatibility prefixes, such as `-webkit-`.

```ts
interface FarmCssPrefixer {
  targets?: string[] | string | BrowserTargetsRecord;
}

type BrowserTargetsRecord = Partial<
  Record<
    | "chrome"
    | "opera"
    | "edge"
    | "firefox"
    | "safari"
    | "ie"
    | "ios"
    | "android"
    | "node"
    | "electron",
    string
  >
> & { [key: string]: string };
```

##### `css.prefixer.targets`

- **Default value**: `undefined`

Configure which target browsers or browser versions to enable, for example:

```ts
import type { UserConfig } from "@farmfe/core";

function defineConfig(config: UserConfig) {
  return config;
}

export default defineConfig({
  compilation: {
    css: {
      prefix: {
        targets: ["last 2 versions", "Firefox ESR", "> 1%", "ie >= 11"],
      },
    },
  },
});
```

### html

#### `html.base`

- **Default value**: `undefined`

All HTML entries will inherit `html.base`, for details, refer to [Guide - HTML](/docs/features/html)

### sourcemap

- **default**: `true`

Configure whether to enable sourcemap, optional configuration items and descriptions are as follows:

- **`true`**: Only generate sourcemap for files not under `node_modules`, and generate a separate sourcemap file
- **`false`**: turn off sourcemap
- **`inline`**: Only generate sourcemap for files not under `node_modules`, and inline sourcemap into the product, do not generate a separate file
- **`all`**: generate sourcemap for all files, and generate a separate sourcemap file
- **`all-inline`**: Generate sourcemaps for all files, and inline sourcemaps into the product, do not generate separate files

### partialBundling

Configure the behavior of Farm's partial bundling. For details, please refer to [Partial Bundling](/docs/features/partial-bundling)

```ts
export interface FarmPartialBundlingConfig {
  targetConcurrentRequests?: number;
  targetMinSize?: number;
  targetMaxSize?: number;
  groups?: {
    name: string;
    test: string[];
    groupType?: "mutable" | "immutable";
    resourceType?: "all" | "initial" | "async";
  }[];
  enforceResources?: {
    name: string;
    test: string[];
  }[];
  enforceTargetConcurrentRequests?: boolean;
  enforceTargetMinSize?: boolean;
  immutableModules?: string[];
}
```

#### `partialBundling.targetConcurrentRequests`

- **default**: `25`

Farm tries to generate resource numbers as closer as possible to this config value for initial resource loading or a dynamic resource loading.

#### `partialBundling.targetMinSize`

- **default**: `20 * 1024` bytes, 20 KB

The minimum size of each generated resources **before minify and gzip**. Note that `targetMinSize` will not be satisfied if `ModuleBucket's size` is less than `targetMinSize`, `ModuleBucket` will be given priority. Config `enforceTargetMinSize` can be used to enforce size.

#### `partialBundling.targetMaxSize`

- **default**: `1500 * 1024` bytes, 1500 KB

The maximum size of generated resources before minify and gzip.

#### `partialBundling.groups`

- **default**: `[]`

A group of modules that should be placed together. Note that this group config is only a hit to the compiler that these modules should be placed together, it may produce multiple resources, if you want to enforce modules in the same resource, you should use `enforceResources`.

Options for each item:

- **name**: Name of this group.
- **test**: Regex array to match the modules which are in this group.
- **groupType**: `mutable` or `immutable`, this group only applies to the specified type of modules.
- **resourceType**: `all`, `initial` or `async`, this group only applies to the specified type of resources.

```ts title="farm.config.ts" {4-9}
export default defineConfig({
  compilation: {
    partialBundling: {
      groups: [
        {
          name: "vendor-react",
          test: ["node_modules/"],
        },
      ],
    },
  },
});
```

#### `partialBundling.enforceResources`

- **default**: `[]`

Array to match the modules that should always be in the same output resource, ignore all other constraints.

Options for each item:

- **name**: Name of this resource.
- **test**: Regex array to match the modules which are in this resource.

```ts title="farm.config.ts" {4-9}
export default defineConfig({
  compilation: {
    partialBundling: {
      enforceResources: [
        {
          name: "index",
          test: [".+"],
        },
      ],
    },
  },
});
```

:::warning
`enforceResources` will ignore all Farm's internal optimization, be careful when you use it.
:::

#### `partialBundling.enforceTargetConcurrentRequests`

- **default**: `false`

Enforce target concurrent requests for every resource loading, when true, smaller resource will be merged into bigger resource to meet the target concurrent requests. this may cause issue for css resource, be careful to use this option.

#### `partialBundling.enforceTargetMinSize`

- **default**: `false`

Enforce target min size for every resource, when tue, smaller resource will be merged into bigger resource to meet the target concurrent requests. this may cause issue for css resource, be careful to use this option

#### `partialBundling.immutableModules`

- **default**: `['node_modules']`

Regex array to match the immutable modules.

```ts title="farm.config.ts"
export default defineConfig({
  compilation: {
    partialBundling: {
      immutableModules: ["node_modules/", "/global-constants"],
    },
  },
});
```

Immutable module can affect bundling and incoming persistent cache, be careful if you want to change it.

#### `partialBundling.immutableModulesWeight`

- **default**: `0.8`

默认为`0.8`，不可变模块将拥有 80%的请求数。 例如，如果`targetConcurrentRequest`为 25，则默认情况下不可变资源将采用`25 * 80% = 20`。 该选项是为了确保可变模块和不可变模块是隔离的，如果更改您的业务代码，node_modules 下的代码不会受到影响。

### lazyCompilation

- **default**: `true` in development mode, `false` in build mode

Whether to enable lazy compilation, configure to false to close. See [lazy compilation](/docs/features/lazy-compilation).

### treeShaking

- **default**: `false` in development mode, `true` in build mode

Whether to enable tree shake, set to false to close. See [Tree Shake](/docs/features/tree-shake).

### minify

- **default**: `false` in development mode, `true` in build mode

Whether to enable compression, the product will be compressed and confused after it is turned on. See [Compression](/docs/features/tree-shake).

### presetEnv

- **default**: `false` in development mode, `true` in build mode

```ts
type FarmPresetEnvConfig =
  | boolean
  | {
      include?: string[];
      exclude?: string[];
      // TODO using swc's config
      options?: any;
      assumptions?: any;
    };
```

By default, polyfills will not be injected into modules under node_modules, if necessary, please use `include` to add polyfills.

#### `presetEnv.include`

- **default**: `[]`

Include additional modules that require polyfill, configure regular strings, for example `include: ['node_modules/(es6-package|my-package)/']`

#### `presetEnv. exclude`

- **default**: `['node_modules/']`

Configure modules that do not require polyfill, and configure regular strings, such as `exclude: ['custom-path/(es5-package|my-package)/']`. By default node_modules is excluded, if you need to include excluded modules, it is recommended to use `include`

#### `presetEnv.options`

- **default**: `downgrade to ES5`

Options passed to swc preset env, see https://swc.rs/docs/configuration/compilation#env.

### persistentCache

- **default**: `true`

Options for [Persistent Cache](/docs/features/persistent-cache). Configuring it `false` to disable cache.

```ts
export type PersistentCache =
  | boolean
  | {
      namespace?: string;
      cacheDir?: string;
      buildDependencies?: string[];
      moduleCacheKeyStrategy?: {
        timestamp?: boolean;
        hash?: boolean;
      };
    };
```

#### `persistentCache.namespace`

- **default**: `farm-cache`

Namespace for the cache, caches under different namespace will be isolated.

#### `persistentCache.cacheDir`

- **default**: `node_modules/.farm/cache`

Cache store directory.

#### `persistentCache.buildDependencies`

- **default**: `farm.config.ts and all its deep dependencies`

File path or package name that may affect the compilation, for example, plugins. By default, `farm.config.ts/js/mjs` and all of its deep dependencies will be treated as build dependencies, if any of these files changed, all cache will be invalidated.

it can be a file path or a package name, for example:

```ts
import { defineConfig } from "@farmfe/core";
import path from "node:path";

export default defineConfig({
  persistentCache: {
    buildDependencies: [
      // a file path
      path.resolve(process.cwd(), "./plugins/my-plugin.js"),
      // a package name, note that this package must expose package.json
      "farm-plugin-custom-xxx",
    ],
  },
});
```

#### `persistentCache.moduleCacheKeyStrategy`

- **default**: `{ timestamp: true, hash: true }`

How to generate cache key when trying to reuse cache. if `timestamp` is true and the timestamp of the module is not changed, then all build stage hooks like `load`, `transform` will be skipped and the cached module will be reused. if `hash` is true and the content of the module is not changed, `load` and `transform` hook will be called to get the transformed content, other hooks will be skipped and the cached module will be reused.

- `timestamp`: whether check timestamp of the module, which has the best performance
- `hash`: whether check content hash after load and transform

#### `persistentCache.envs`

- **default**: [Farm Env](https://farm-fe.github.io/docs/config/farm-config#environment-variable)

Envs used to invalidate cache, if the configured env changed, then all cache will be invalidated.

<!-- #### `presetEnv.assuptions` -->
