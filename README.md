# h5wasm-plugins

A collection of pre-compiled compression plugins to be used with [h5wasm](https://github.com/usnistgov/h5wasm)

_(h5wasm is a javascript/webassembly library for reading and writing HDF5 files from the browser or node.js or deno or...)_

## Installation
`npm install h5wasm-plugins`

(this will also install h5wasm >= 0.6.7)

## Included plugins
 - blosc
 - bz2
 - lz4
 - lzf
 - szf
 - zfp
 - zstd

(Note that `gzip` and `szip` filters are built-in to h5wasm and don't require a plugin)

## Usage: browser
The default export from h5wasm (which has a `Module` property defined, for accessing the C-module directly) should be passed to the functions defined here.

```js
import h5wasm from "h5wasm";
import { plugin_names, install_plugins } from "h5wasm-plugins";

await h5wasm.ready;
// installs libH5Zzfp.so to default folder, /usr/local/hdf5/lib/plugin
install_plugins(h5wasm, ["zfp"]);

// Or, to install all plugins to new folder /tmp/h5plugins
// install_plugins(h5wasm, plugin_names, "/tmp/h5plugins");

// open existing file - see h5wasm docs for writing files to
// virtual Emscripten filesystem
const f = new h5wasm.File("my_zfp.h5", "r");
const data = f.get("zfp_data").value;
```

## Usage: server side (e.g. nodejs)
```js
const h5wasm = await import("h5wasm");
const h5wasm_plugins = await import("h5wasm-plugins");

await h5wasm.ready;
// this inserts the node_modules/h5wasm-plugins/plugins path
// at the start of the HDF5 plugin search paths
h5wasm_plugins.install_local_plugins(h5wasm);

const f = new h5wasm.File('./test_zfp.h5', 'r');
// File {
//   path: '/',
//   file_id: 72057594037927936n,
//   type: 'Group',
//   filename: './test_zfp.h5',
//   mode: 'r'
// }

f.get('data').filters
// [
//   {
//     id: 32013,
//     name: 'H5Z-ZFP-1.1.0 (ZFP-0.5.5)',
//     cd_values: [ 89149712, 91252346, 146, 4293918720, 3767009280, 493487 ]
//   }
// ]

f.get('data').value 
// Float32Array(10) [
//   0, 1, 2, 3, 4,
//   5, 6, 7, 8, 9
// ]

h5wasm.Module.get_plugin_search_paths()
// [
//   '/home/bbm/dev/test_plugins/node_modules/h5wasm-plugins/plugins',
//   '/usr/local/hdf5/lib/plugin'
// ]
```
