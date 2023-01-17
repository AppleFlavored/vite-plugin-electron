<p align="center">
  <img width="170" src="https://github.com/electron-vite/vite-plugin-electron/blob/main/logo.svg?raw=true">
</p>
<div align="center">
  <h1>vite-plugin-electron</h1>
</div>
<p align="center">Electron 🔗 Vite</p>
<p align="center">
  <a href="https://npmjs.com/package/vite-plugin-electron">
    <img src="https://img.shields.io/npm/v/vite-plugin-electron.svg">
  </a>
  <a href="https://npmjs.com/package/vite-plugin-electron">
    <img src="https://img.shields.io/npm/dm/vite-plugin-electron.svg">
  </a>
  <a href="https://discord.gg/YfjFuEgVUR">
    <img src="https://img.shields.io/badge/chat-discord-blue?logo=discord">
  </a>
</p>
<p align="center">
  <strong>
    <span>English</span>
    |
    <a href="https://github.com/electron-vite/vite-plugin-electron/blob/main/README.zh-CN.md">简体中文</a>
  </strong>
</p>

<br/>

## Features

- 🚀 Fully compatible with Vite and Vite's ecosystem <sub><sup>(Based on Vite)</sup></sub>
- 🎭 Flexible configuration
- 🐣 Few APIs, easy to use
- 🔥 Hot restart

![vite-plugin-electron.gif](https://github.com/caoxiemeihao/blog/blob/main/vite/vite-plugin-electron.gif?raw=true)

## Install

```sh
npm i vite-plugin-electron -D
```

## Examples

- [quick-start](https://github.com/electron-vite/vite-plugin-electron/tree/main/examples/quick-start)
- [multiple-windows](https://github.com/electron-vite/vite-plugin-electron/tree/main/examples/multiple-windows)
- [custom-start-electron-app](https://github.com/electron-vite/vite-plugin-electron/tree/main/examples/custom-start-electron-app)
- [bytecode](https://github.com/electron-vite/vite-plugin-electron/tree/main/examples/bytecode)

## Usage

vite.config.ts

```js
import electron from 'vite-plugin-electron'

export default {
  plugins: [
    electron({
      entry: 'electron/main.ts',
    }),
  ],
}
```

You can use `process.env.VITE_DEV_SERVER_URL` when the vite command is called 'serve'

```js
// electron main.js
const win = new BrowserWindow({
  title: 'Main window',
})

if (process.env.VITE_DEV_SERVER_URL) {
  win.loadURL(process.env.VITE_DEV_SERVER_URL)
} else {
  // load your file
  win.loadFile('yourOutputFile.html');
}
```

## API <sub><sup>(Define)</sup></sub>

`electron(config: Configuration | Configuration[])`

```ts
export interface Configuration {
  /**
   * Shortcut of `build.lib.entry`
   */
  entry?: import('vite').LibraryOptions['entry']
  vite?: import('vite').InlineConfig
  /**
   * Triggered when Vite is built every time -- `vite serve` command only.
   * 
   * If this `onstart` is passed, Electron App will not start automatically.  
   * However, you can start Electroo App via `startup` function.  
   */
  onstart?: (this: import('rollup').PluginContext, options: {
    /**
     * Electron App startup function.  
     * It will mount the Electron App child-process to `process.electronApp`.  
     * @param argv default value `['.', '--no-sandbox']`
     */
    startup: (argv?: string[]) => Promise<void>
    /** Reload Electron-Renderer */
    reload: () => void
  }) => void | Promise<void>
}
```

## JavaScript API

`vite-plugin-electron`'s JavaScript APIs are fully typed, and it's recommended to use TypeScript or enable JS type checking in VS Code to leverage the intellisense and validation.

- `Configuration` - type
- `defineConfig` - function
- `resolveViteConfig` - function, Resolve the default Vite's `InlineConfig` for build Electron-Main
- `withExternalBuiltins` - function
- `build` - function
- `startup` - function

**Example**

```js
build(
  withExternalBuiltins( // external Node.js builtin modules
    resolveViteConfig( // with default config
      {
        entry: 'foo.ts',
        vite: {
          mode: 'foo-mode', // for .env file
          plugins: [{
            name: 'plugin-build-done',
            closeBundle() {
              // Startup Electron App
              startup()
            },
          }],
        },
      }
    )
  )
)
```

## How to work

It just executes the `electron .` command in the Vite build completion hook and then starts or restarts the Electron App.

## Recommend structure

Let's use the official [template-vanilla-ts](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-vanilla-ts) created based on `create vite` as an example

```diff
+ ├─┬ electron
+ │ └── main.ts
  ├─┬ src
  │ ├── main.ts
  │ ├── style.css
  │ └── vite-env.d.ts
  ├── .gitignore
  ├── favicon.svg
  ├── index.html
  ├── package.json
  ├── tsconfig.json
+ └── vite.config.ts
```

## Be aware

- 🚨 By default, the files in `electron` folder will be built into the `dist-electron`
- 🚨 Currently, `"type": "module"` is not supported in Electron
- 🚨 In general, Vite may not correctly build Node.js packages, especially C/C++ native modules, but Vite can load them as external packages. So, put your Node.js package in `dependencies`. Unless you know how to properly build them with Vite.
  ```js
  electron({
    entry: 'electron/main.ts',
    vite: {
      build: {
        rollupOptions: {
          // Here are some C/C++ plugins that can't be built properly.
          external: [
            'serialport',
            'sqlite3',
          ],
        },
      },
    },
  }),
  ```

<!-- You can see 👉 [dependencies vs devDependencies](https://github.com/electron-vite/vite-plugin-electron-renderer#dependencies-vs-devdependencies) -->
