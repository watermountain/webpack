This example demonstrates the AggressiveSplittingPlugin for splitting the bundle into multiple smaller chunks to improve caching. This works best with a HTTP2 web server, otherwise there is an overhead for the increased number of requests.

AggressiveSplittingPlugin splits every chunk until it reaches the specified `maxSize`. In this example it tries to create chunks with <50kB raw code, which typically minimizes to ~10kB. It groups modules together by folder structure, because modules in the same folder are likely to have similar repetitive text, making them gzip efficiently together. They are also likely to change together.

AggressiveSplittingPlugin records its splitting in the webpack records. When it is next run, it tries to use the last recorded splitting. Since changes to application code between one run and the next are usually in only a few modules (or just one), re-using the old splittings (and chunks, which are probably still in the client's cache), is highly advantageous.

Only chunks which are bigger than the specified `minSize` are stored into the records. This ensures that these chunks fill up as your application grows, instead of creating many records of small chunks for every change.

If a module changes, its chunks are declared to be invalid, and are put back into the module pool. New chunks are created from all modules in the pool.

There is a tradeoff here:

The caching improves with smaller `maxSize`, as chunks change less often and can be reused more often after an update.

The compression improves with bigger `maxSize`, as gzip works better for bigger files. It's more likely to find duplicate strings, etc.

The backward compatibility (non HTTP2 client) improves with bigger `maxSize`, as the number of requests decreases.

```js
var path = require("path");
var webpack = require("../../");
module.exports = {
	// mode: "development || "production",
	cache: true, // better performance for the AggressiveSplittingPlugin
	entry: "./example",
	output: {
		path: path.join(__dirname, "dist"),
		filename: "[chunkhash].js",
		chunkFilename: "[chunkhash].js"
	},
	plugins: [
		new webpack.optimize.AggressiveSplittingPlugin({
			minSize: 30000,
			maxSize: 50000
		}),
		new webpack.DefinePlugin({
			"process.env.NODE_ENV": JSON.stringify("production")
		})
	],
	recordsOutputPath: path.join(__dirname, "dist", "records.json")
};
```

# Info

## Unoptimized

```
Hash: 0a1b2c3d4e5f6a7b8c9d
Version: webpack 5.0.0-beta.6
                  Asset      Size
7598e98a1006fa74c6e5.js  7.15 KiB  [emitted] [immutable]
bd7487f4ebb1f501a0fa.js  17.7 KiB  [emitted] [immutable]  [name: main]
d90620a8d5b803dfc7e9.js   117 KiB  [emitted] [immutable]  [id hint: vendors]
Entrypoint main = bd7487f4ebb1f501a0fa.js
chunk bd7487f4ebb1f501a0fa.js (main) 8.77 KiB (javascript) 4.28 KiB (runtime) [entry] [rendered]
    > ./example main
 (webpack)/node_modules/object-assign/index.js 2.06 KiB [built]
 (webpack)/node_modules/react/cjs/react.production.min.js 6.49 KiB [built]
 (webpack)/node_modules/react/index.js 190 bytes [built]
 ./example.js 42 bytes [built]
     + 4 hidden chunk modules
chunk 7598e98a1006fa74c6e5.js 6.52 KiB [rendered]
    > react-dom ./example.js 2:0-22
 (webpack)/node_modules/react-dom/index.js 1.33 KiB [built]
 (webpack)/node_modules/scheduler/cjs/scheduler.production.min.js 4.99 KiB [built]
 (webpack)/node_modules/scheduler/index.js 198 bytes [built]
chunk d90620a8d5b803dfc7e9.js (id hint: vendors) 116 KiB [rendered] split chunk (cache group: defaultVendors)
    > react-dom ./example.js 2:0-22
 (webpack)/node_modules/react-dom/cjs/react-dom.production.min.js 116 KiB [built]
```

## Production mode

```
Hash: 0a1b2c3d4e5f6a7b8c9d
Version: webpack 5.0.0-beta.6
                          Asset       Size
        9d197b298deac2e3d4d5.js    114 KiB  [emitted] [immutable]  [id hint: vendors]
9d197b298deac2e3d4d5.js.LICENSE  247 bytes  [emitted]
        d060f216c0ec95bc7558.js   8.71 KiB  [emitted] [immutable]  [name: main]
d060f216c0ec95bc7558.js.LICENSE  295 bytes  [emitted]
        db2eac6661b345a54896.js   4.98 KiB  [emitted] [immutable]
db2eac6661b345a54896.js.LICENSE  246 bytes  [emitted]
Entrypoint main = d060f216c0ec95bc7558.js
chunk d060f216c0ec95bc7558.js (main) 8.77 KiB (javascript) 4.28 KiB (runtime) [entry] [rendered]
    > ./example main
 (webpack)/node_modules/object-assign/index.js 2.06 KiB [built]
 (webpack)/node_modules/react/cjs/react.production.min.js 6.49 KiB [built]
 (webpack)/node_modules/react/index.js 190 bytes [built]
 ./example.js 42 bytes [built]
     + 4 hidden chunk modules
chunk db2eac6661b345a54896.js 6.52 KiB [rendered]
    > react-dom ./example.js 2:0-22
 (webpack)/node_modules/react-dom/index.js 1.33 KiB [built]
 (webpack)/node_modules/scheduler/cjs/scheduler.production.min.js 4.99 KiB [built]
 (webpack)/node_modules/scheduler/index.js 198 bytes [built]
chunk 9d197b298deac2e3d4d5.js (id hint: vendors) 116 KiB [rendered] split chunk (cache group: defaultVendors)
    > react-dom ./example.js 2:0-22
 (webpack)/node_modules/react-dom/cjs/react-dom.production.min.js 116 KiB [built]
```

## Records

```
{
  "aggressiveSplits": [],
  "chunks": {
    "byName": {
      "main": 0
    },
    "bySource": {
      "0 ./example.js react-dom": 2,
      "1 ./example.js react-dom": 1
    },
    "usedIds": [
      0,
      1,
      2
    ]
  },
  "modules": {
    "byIdentifier": {
      "../../node_modules/object-assign/index.js": 3,
      "../../node_modules/react-dom/cjs/react-dom.production.min.js": 5,
      "../../node_modules/react-dom/index.js": 4,
      "../../node_modules/react/cjs/react.production.min.js": 2,
      "../../node_modules/react/index.js": 1,
      "../../node_modules/scheduler/cjs/scheduler.production.min.js": 7,
      "../../node_modules/scheduler/index.js": 6,
      "./example.js": 0
    },
    "usedIds": [
      0,
      1,
      2,
      3,
      4,
      5,
      6,
      7
    ]
  }
}
```
