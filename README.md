# @geolonia/normalize-japanese-addresses
日本語のREADMEはこちらです: [README.ja.md](README.ja.md)

Open-source address normalization library.

## Demo

- https://codepen.io/geolonia/pen/oNBrqzL
- https://code4fukui.github.io/normalize-japanese-addresses/

## Installation

The library is distributed on the npm registry as `@geolonia/normalize-japanese-addresses`.
Install it using the npm command.

```shell
$ npm install @geolonia/normalize-japanese-addresses -S
```

## Usage

### `normalize(address: string, option: Option)`

Normalizes the address.

```javascript
const { normalize } = require('@geolonia/normalize-japanese-addresses')
normalize('北海道札幌市西区24-2-2-3-3').then(result => {
  console.log(result); // {"pref": "北海道", "city": "札幌市西区", "town": "二十四軒二条二丁目", "addr": "3-3", "lat": 43.074273, "lng": 141.315099, "level": 3}
})
```

The normalized result object includes a `level` property. `level` stores a number indicating how much of the address could be identified:

* `0` - Could not identify the prefecture.
* `1` - Could identify the prefecture.
* `2` - Could identify the city/town.
* `3` - Could identify the neighborhood.

For example, if you only want to normalize the prefecture name, you can specify the `level` option to make the process faster.

```javascript
const { normalize } = require('@geolonia/normalize-japanese-addresses')
normalize('北海道札幌市西区24-2-2-3-3', { level: 1 }).then(result => {
  console.log(result); // {"pref": "北海道", "city": "", "town": "", "addr": "札幌市西区二十四軒二条二丁目3-3", "lat": null, "lng": null, "level": 1}
})
```

Use as an ES module (browser or Deno)
```javascript
import { normalize } from "https://code4fukui.github.io/normalize-japanese-addresses/dist/main-es.js";

const result = await normalize('北海道札幌市西区24-2-2-3-3');
console.log(result);
```

### Global Options

The following parameters can be used to change the overall behavior of the library.

#### `config.townCacheSize: number`

The `@geolonia/normalize-japanese-addresses` library retrieves the latest neighborhood data from a web API and performs address normalization. The `townCacheSize` option changes the number of municipalities cached. The default is 1,000.

#### `config.japaneseAddressesApi: string`

Specify the endpoint of the web API that serves the neighborhood data. The default is `https://geolonia.github.io/japanese-addresses/api/ja`. Refer to the [Geolonia Address Data](https://github.com/geolonia/japanese-addresses/tree/develop/api) for the directory structure of this API.

You can specify a `file://` URL to reference locally saved files.

##### Example Usage

```shell
# Download Geolonia Address Data
$ curl -sL https://github.com/geolonia/japanese-addresses/archive/refs/heads/master.tar.gz | tar xvfz -
```

```javascript
const { config, normalize } = require('@geolonia/normalize-japanese-addresses')
config.japaneseAddressesApi = 'file:///path/to/japanese-addresses-master/api/ja'

(function(){
  for (address of addresses) {
    await normalize(address)
  }
})()
```

## Normalization Details

* Supplements omitted county names (e.g., "Kogun" in addresses).
* Unifies alphabets and numbers to half-width.
* Removes Kyoto street names.
* Absorbs variations between old and new kanji characters to align with the Ministry of Land, Infrastructure, Transport and Tourism's position reference information.
* Absorbs variations in "ヶケが", "ヵカか力", "之ノの", "ッツっつ" to align with the Ministry's position reference information.
* Absorbs variations between "釜" and "竈", "埠頭" and "ふ頭" kanji.
* Converts all numeric characters in neighborhood-level addresses to kanji numerals to align with the Ministry's position reference information.
* Converts numeric characters in address numbers and house numbers to Arabic numerals, and converts strings like "番地" to "-".
* If the address ends with a building name, it is returned as is, but it may be better to separate it beforehand.

References:

* [Please refer to the source code for the strings that are processed for variations.](https://github.com/geolonia/normalize-japanese-addresses/blob/master/src/lib/dict.ts)
* [Please refer to the test code for examples of addresses before and after normalization.](https://github.com/geolonia/normalize-japanese-addresses/blob/master/test/main.test.ts)

## Developer Information

First, set up the environment with the following commands.

```shell
$ git clone git@github.com:geolonia/normalize-japanese-addresses.git
$ cd normalize-japanese-addresses
$ npm install
```

Next, run the following to compile:

```shell
$ npm run build
```

The necessary files will be generated in the dist folder. Then, you can create a sample.js file like this:

```javascript
// sample.js
const { normalize } = require('./dist/main-node.js');
normalize('北海道札幌市西区24-2-2-3-3', { level: 3 }).then(result => {
  console.log(result); // { "pref": "北海道", "city": "", "town": "", "addr": "札幌市西区二十四軒二条二丁目3-3", "level": 1 }
})
```

And run it with:

```shell
$ node sample.js
```

## Notes

* This normalization engine is designed for "address consolidation" and removes Kyoto street names, for example.
  * It should be suitable for use in postal or courier services.
* This normalization engine supports neighborhood and smaller-scale addresses, but does not handle addresses beyond that level.
* It is not good at handling areas where the residential address system is not well-developed.

### How to Contribute

[Pull requests](https://github.com/geolonia/normalize-japanese-addresses/pulls) and [Issues](https://github.com/geolonia/normalize-japanese-addresses/issues) are always welcome.

## License
This project is licensed under the [MIT License](LICENSE).
