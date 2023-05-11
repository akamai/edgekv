# EdgeKV Helper Library

This helper class can be used to simplify the interaction with EdgeKV in EdgeWorker code. It abstracts away the complexity into a few lines of code.

Original code is https://github.com/akamai/edgeworkers-examples/tree/master/edgekv/lib

Your EdgeKV tokens can be passed to the constructor.


## Setup
```sh
npm install edgekv --save
```
Then, put edgekv_tokens.js file along with main.js.
edgekv_tokens.js can be created by Akamai CLI.

## How to use in EdgeWorkers code
JavaScript module bundler would help you to embed this module into main.js.

[rollup.js](https://rollupjs.org/) for example, with following `rollup.config.js` will generate code bundle efficiently. Then create tgz file at `dist` directory.
```js
import resolve from "@rollup/plugin-node-resolve";
import commonjs from "@rollup/plugin-commonjs";

export default {
  input: ["src/main.js", "src/bundle.json", "src/edgekv_tokens.js"],
  external: [
    'create-response',
		'http-request',
		'cookies',
		'text-encode-transform',
		'url-search-params',
		'streams',
		'log',
		'resolvable'
	],
	preserveModules: false,
	output: {
		dir: "dist",
		format: "es"
	},
	plugins: [
		commonjs(),
		resolve()
	]
};
```


## EdgeKV Class
### Constructor
Constructor to allow setting default namespace, group and token.
These defaults can be overriden when making individual GET, PUT, and DELETE operations
* @param {string} [$0.namepsace="default"]
  * the default namespace to use for all GET, PUT, and DELETE operations
  * Namespace must be 32 characters or less, consisting of A-Z a-z 0-9 _ or -
* @param {string} [$0.group="default"]
  * the default group to use for all GET, PUT, and DELETE operations
  * Group must be 128 characters or less, consisting of A-Z a-z 0-9 _ or -
* @param {object} [$0.edgekv_access_tokens={}]
  * the token name and token value for each namespace
  * The structure of this argument assumes the same as the token creation command by Akamai CLI.
* @param {number} [$0.num_retries_on_timeout=0]
  * the number of times to retry a GET requests when the sub request times out
* @param {object} [$0.ew_request=null]
  * passes the request object from the EdgeWorkers event handler to enable access to EdgeKV data in sandbox environments
* @param {boolean} [$0.sandbox_fallback=false]
  * whether to fallback to retrieving staging data if the sandbox data does not exist, instead of returning null or the specified default value

```js
import { edgekv_access_tokens } from './edgekv_tokens.js';

const ekv = new EdgeKV("namespace", "group", edgekv_access_tokens);
```

### getText
async GET text from an item in the EdgeKV.
* @param {string} [$0.namepsace=this.#namespace]
  * specify a namespace other than the default
* @param {string} [$0.group=this.#group]
  * specify a group other than the default
* @param {string} [$0.item]
  * item key to get from the EdgeKV
* @param {string} [$0.default_value=null]
  * the default value to return if a 404 response is returned from EdgeKV
* @param {number} [$0.timeout=null]
  * the maximum time, between 1 and 1000 milliseconds, to wait for the response
* @param {number} [$0.num_retries_on_timeout=null]
  * the number of times to retry a requests when the sub request times out
* @returns {Promise<string>}
  * if the operation was successful, the text response from the EdgeKV or the default_value on 404
* @throws {object}
  * if the operation was not successful, an object describing the non-200 and non-404 response from the EdgeKV: {failed, status, body}

```js
let text = await ekv.getText({item: 'key'});
```

### getJson
async GET json from an item in the EdgeKV.
* @param {string} [$0.namepsace=this.#namespace]
  * specify a namespace other than the default
* @param {string} [$0.group=this.#group]
  * specify a group other than the default
* @param {string} [$0.item]
  * item key to get from the EdgeKV
* @param {object} [$0.default_value=null]
  * the default value to return if a 404 response is returned from EdgeKV
* @param {number} [$0.timeout=null]
  * the maximum time, between 1 and 1000 milliseconds, to wait for the response
* @param {number} [$0.num_retries_on_timeout=null]
  * the number of times to retry a requests when the sub request times out
* @returns {Promise}
  * if the operation was successful, the json response from the EdgeKV or the default_value on 404
* @throws {object}
  * if the operation was not successful, an object describing the non-200 and non-404 response from the EdgeKV: {failed, status, body}

```js
let object = await ekv.getJson({item: 'key'});
```

### putText
async PUT text into an item in the EdgeKV.
* @param {string} [$0.namepsace=this.#namespace]
  * specify a namespace other than the default
* @param {string} [$0.group=this.#group]
  * specify a group other than the default
* @param {string} [$0.item]
  * item key to put into the EdgeKV
* @param {string} [$0.value]
  * text value to put into the EdgeKV
* @param {number} [$0.timeout=null]
  * the maximum time, between 1 and 1000 milliseconds, to wait for the response
* @returns {string}
  * if the operation was successful, the response from the EdgeKV
* @throws {object}
  * if the operation was not successful, an object describing the non-200 response from the EdgeKV: {failed, status, body}

```js
await ekv.putText({item: 'key', value: 'foo'});
```

### putTextNoWait
PUT text into an item in the EdgeKV while only waiting for the request to send and not for the response.
* @param {string} [$0.namepsace=this.#namespace]
  * specify a namespace other than the default
* @param {string} [$0.group=this.#group]
  * specify a group other than the default
* @param {string} [$0.item]
  * item key to put into the EdgeKV
* @param {string} [$0.value]
  * text value to put into the EdgeKV
* @throws {object}
  * if the operation was not successful at sending the request, an object describing the error: {failed, status, body}

```js
await ekv.putTextNoWait({item: 'key', value: 'foo'});
```

### putJson
async PUT json into an item in the EdgeKV.
* @param {string} [$0.namepsace=this.#namespace]
  * specify a namespace other than the default
* @param {string} [$0.group=this.#group]
  * specify a group other than the default
* @param {string} [$0.item]
  * item key to put into the EdgeKV
* @param {object} [$0.value]
  * json value to put into the EdgeKV
* @param {number} [$0.timeout=null]
  * the maximum time, between 1 and 1000 milliseconds, to wait for the response
* @returns {string}
  * if the operation was successful, the response from the EdgeKV
* @throws {object}
  * if the operation was not successful, an object describing the non-200 response from the EdgeKV: {failed, status, body}

```js
await ekv.putJson({item: 'key', value: {name: 'foo', description: 'bar'}});
```

### putJsonNoWait
PUT json into an item in the EdgeKV while only waiting for the request to send and not for the response.
* @param {string} [$0.namepsace=this.#namespace]
  * specify a namespace other than the default
* @param {string} [$0.group=this.#group]
  * specify a group other than the default
* @param {string} [$0.item]
  * item key to put into the EdgeKV
* @param {object} [$0.value]
  * json value to put into the EdgeKV
* @throws {object}
  * if the operation was not successful at sending the request, an object describing the error: {failed, status, body}

```js
await ekv.putJsonNoWait({ namespace = this.#namespace, group = this.#group, item, value } = {})
```

### delete
async DELETE an item in the EdgeKV.
* @param {string} [$0.namepsace=this.#namespace]
  * specify a namespace other than the default
* @param {string} [$0.group=this.#group]
  * specify a group other than the default
* @param {string} [$0.item]
  * item key to delete from the EdgeKV
* @param {number} [$0.timeout=null]
  * the maximum time, between 1 and 1000 milliseconds, to wait for the response
* @returns {string}
  * if the operation was successful, the text response from the EdgeKV
* @throws {object}
  * if the operation was not successful, an object describing the non-200 response from the EdgeKV: {failed, status, body}

```js
await ekv.delete({item: 'key'});
```

### deleteNoWait
DELETE an item in the EdgeKV while only waiting for the request to send and not for the response.
* @param {string} [$0.namepsace=this.#namespace] specify a namespace other than the default
* @param {string} [$0.group=this.#group] specify a group other than the default
* @param {string} $0.item item key to delete from the EdgeKV
* @throws {object} if the operation was not successful at sending the request, an object describing the error: {failed, status, body}

```js
await ekv.deleteNoWait({item: 'key'});
```

### Errors
All errors coming from the use of the EdgeKV class will be in the following format:
```js
{
  "failed": "whatFailed",
  "status": responseStatusCode,
  "body": "descriptionOfFailure"
}
```

## Usage Examples
### Get Product Sale Price
```js
try {
  const edgeKv = new EdgeKV({ group: "ProductSalePrice" }); // the namespace will be "default" if it is not provided
  let salePrice = await edgeKv.getText({ item: productId, default_value: "N/A" });
  // use the salePrice in the page
} catch (error) {
  // do something in case of an error
}
```

### Write the Product Modification Time
```js
try {
  const edgeKv = new EdgeKV({ group: "LastUpdated" });
  let date = new Date().toString();
  await edgeKv.putText({ item: productId, value: date });
  // this information can then be used to see when a product was last updated
} catch (error) {
  // do something in case of an error
}
```

### Write the Product Modification Time Without Wait For Response
```js
try {
  const edgeKv = new EdgeKV({ group: "LastUpdated" });
  let date = new Date().toString();
  edgeKv.putTextNoWait({ item: productId, value: date });
  // this information can then be used to see when a product was last updated
} catch (error) {
  // do something in case of an error
}
```

## Resources
Please see the examples tagged "EKV" [here](https://github.com/akamai/edgeworkers-examples/tree/master/edgecompute/examples) for example usage of this helper library.

For information on using EdgeWorkers and EdgeKV, please review [Akamai's product policy ](https://www.akamai.com/site/en/documents/akamai/2022/edgeworkers-and-edgekv-supplemental-product-policy.pdf)
