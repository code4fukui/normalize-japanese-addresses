# @geolonia/normalize-japanese-addresses

> 日本語のREADMEはこちらです: [README.ja.md](README.ja.md)

[
![build](https://github.com/geolonia/normalize-japanese-addresses/actions/workflows/build.yml/badge.svg)
](https://github.com/geolonia/normalize-japanese-addresses/actions/workflows/build.yml)

An open-source Japanese address normalization library, inspired by the geocoding mechanism of the [IMI Component Tools](https://info.gbiz.go.jp/tools/imi_tools/) from Japan's Ministry of Economy, Trade, and Industry.

## Demos

- [CodePen Demo](https://codepen.io/geolonia/pen/oNBrqzL)
- [Interactive Example](https://code4fukui.github.io/normalize-japanese-addresses/)

## Features

- **Standardization**: Converts addresses into a consistent, standard format.
- **Kanji Normalization**: Handles variations between old (旧字体) and new (新字体) kanji forms, as well as other common character variations (e.g., `ヶ` vs `ケ`, `釜` vs `竈`, `埠頭` vs `ふ頭`).
- **Number Conversion**: Unifies address numbers, converting numbers in town names (町丁目) to Kanji and block/house numbers (番地号) to Arabic numerals.
- **Character Width**: Converts all full-width (zenkaku) alphanumeric characters to half-width (hankaku).
- **Address Completion**: Automatically complements missing `郡` (county) names.
- **Kyoto Address Support**: Removes non-standard Kyoto street names (通り名) to simplify addresses.
- **Geolocation**: Returns latitude and longitude for normalized addresses where available.

## Installation

Install the library from the npm registry using your package manager of choice.

```shell
npm install @geolonia/normalize-japanese-addresses
```

## Usage

### `normalize(address: string, option?: Option)`

Normalizes a given Japanese address string.

**Node.js (CommonJS)**

```javascript
const { normalize } = require('@geolonia/normalize-japanese-addresses');

normalize('北海道札幌市西区24-2-2-3-3').then(result => {
  console.log(result);
  /*
  {
    pref: '北海道',
    city: '札幌市西区',
    town: '二十四軒二条二丁目',
    addr: '3-3',
    lat: 43.074273,
    lng: 141.315099,
    level: 3
  }
  */
});
```

**Browser / Deno (ES Modules)**

```javascript
import { normalize } from "https://code4fukui.github.io/normalize-japanese-addresses/dist/main-es.js";

const result = await normalize('北海道札幌市西区24-2-2-3-3');
console.log(result);
```

### Normalization Level

The returned object includes a `level` property indicating how much of the address was successfully parsed.

- `0`: Could not identify the prefecture.
- `1`: Identified up to the prefecture.
- `2`: Identified up to the city/ward/county.
- `3`: Identified up to the town/area (町丁目).
- `7`: (High-precision mode) Identified up to the residential block (街区).
- `8`: (High-precision mode) Identified up to the residential block and number (住居番号).

You can set the desired normalization level using the `level` option for faster processing.

```javascript
normalize('北海道札幌市西区24-2-2-3-3', { level: 1 }).then(result => {
  console.log(result);
  /*
  {
    pref: '北海道',
    city: '',
    town: '',
    addr: '札幌市西区二十四軒二条二丁目3-3',
    lat: null,
    lng: null,
    level: 1
  }
  */
});
```

### High-Precision Normalization (Residential Level)

For more accurate results, including residential block and number parsing (levels 7 and 8), you can use the Geolonia backend by providing a `geoloniaApiKey`.

```javascript
normalize('東京都世田谷区北烏山６−２２−２２', {
  geoloniaApiKey: 'YOUR-API-KEY',
}).then(result => {
  console.log(result);
  /*
  {
    pref: '東京都',
    city: '世田谷区',
    town: '北烏山六丁目',
    addr: '',
    lat: 35.68451,
    lng: 139.59951,
    gaiku: '22',
    jyukyo: '22',
    level: 8
  }
  */
});
```

## Configuration

You can modify the library's global behavior by setting properties on the `config` object.

```javascript
const { config, normalize } = require('@geolonia/normalize-japanese-addresses');
```

- **`config.townCacheSize: number`**
  The library fetches and caches town data (町丁目) for each municipality. This option sets the maximum number of municipalities to cache. The default is `1000`.

- **`config.japaneseAddressesApi: string`**
  Specifies the endpoint for the Japanese address data API. The default is `https://geolonia.github.io/japanese-addresses/api/ja`.

  You can point this to a local directory using the `file://` protocol. This is useful for offline use or for using your own address data.

  **Example: Using Local Address Data**

  1.  Download the address data from [geolonia/japanese-addresses](https://github.com/geolonia/japanese-addresses).
      ```shell
      curl -sL https://github.com/geolonia/japanese-addresses/archive/refs/heads/master.tar.gz | tar xvfz -
      ```
  2.  Set the `japaneseAddressesApi` config to the local path.
      ```javascript
      const path = require('path');
      const { config, normalize } = require('@geolonia/normalize-japanese-addresses');

      const localDataPath = path.resolve(__dirname, 'japanese-addresses-master/api/ja');
      config.japaneseAddressesApi = `file://${localDataPath}`;

      normalize('...your address...');
      ```

## Limitations

- This library is designed for address matching and may remove parts of an address, such as Kyoto's traditional street names, that are not required for postal services.
- The default open-data mode normalizes up to the town/area (町丁目) level. For residential block/number level, the high-precision mode with an API key is required.
- Normalization may be less accurate for areas without a standardized residential addressing system (住居表示未整備).

## For Developers

To set up a local development environment:

```shell
# 1. Clone the repository
git clone https://github.com/geolonia/normalize-japanese-addresses.git
cd normalize-japanese-addresses

# 2. Install dependencies
npm install

# 3. Build the project
npm run build

# 4. Run tests
npm test
```

## Contributing

Pull requests and Issues are always welcome.

For details on the normalization logic, please refer to the source code:
- **String variations:** [`src/lib/dict.ts`](https://github.com/geolonia/normalize-japanese-addresses/blob/master/src/lib/dict.ts)
- **Before/after examples:** [`test/main.test.ts`](https://github.com/geolonia/normalize-japanese-addresses/blob/master/test/main.test.ts)

## License

This project is licensed under the MIT License.

We would appreciate it if you could share this project on social media or link to [Geolonia](https://geolonia.com/) to help motivate our developers.