# @geolonia/normalize-japanese-addresses

English README is here: [README.md](README.md)

日本語のREADMEはこちらです: [README.ja.md](README.ja.md)

オープンソースのアドレス正規化ライブラリ。

## デモ

- https://codepen.io/geolonia/pen/oNBrqzL
- https://code4fukui.github.io/normalize-japanese-addresses/

## インストール

このライブラリは `@geolonia/normalize-japanese-addresses` としてnpmレジストリで配布されています。
npmコマンドを使ってインストールします。

```shell
$ npm install @geolonia/normalize-japanese-addresses -S
```

## 使用法

### `normalize(address: string, option: Option)`

住所を正規化します。

```javascript
const { normalize } = require('@geolonia/normalize-japanese-addresses')
normalize('北海道札幌市西区24-2-2-3-3').then(result => {
  console.log(result); // {"pref": "北海道", "city": "札幌市西区", "town": "二十四軒二条二丁目", "addr": "3-3", "lat": 43.074273, "lng": 141.315099, "level": 3}
})
```

正規化された結果オブジェクトには `level` プロパティが含まれます。 `level` は、住所の識別度合いを示す数値を格納しています:

* `0` - 都道府県を識別できませんでした。
* `1` - 都道府県を識別できました。
* `2` - 市区町村を識別できました。
* `3` - 町丁目を識別できました。

例えば、都道府県名のみを正規化したい場合は、 `level` オプションを指定することで処理を高速化できます。

```javascript
const { normalize } = require('@geolonia/normalize-japanese-addresses')
normalize('北海道札幌市西区24-2-2-3-3', { level: 1 }).then(result => {
  console.log(result); // {"pref": "北海道", "city": "", "town": "", "addr": "札幌市西区二十四軒二条二丁目3-3", "lat": null, "lng": null, "level": 1}
})
```

ES モジュールとしての使用 (ブラウザやDeno)
```javascript
import { normalize } from "https://code4fukui.github.io/normalize-japanese-addresses/dist/main-es.js";

const result = await normalize('北海道札幌市西区24-2-2-3-3');
console.log(result);
```

### グローバルオプション

ライブラリの全体的な動作を変更するには、以下のパラメータを使用できます。

#### `config.townCacheSize: number`

`@geolonia/normalize-japanese-addresses` ライブラリは、最新の町丁目データをWebAPIから取得し、住所の正規化を行います。 `townCacheSize` オプションは、キャッシュする市区町村の数を変更します。デフォルトは1,000です。

#### `config.japaneseAddressesApi: string`

町丁目データを提供するWebAPIのエンドポイントを指定します。デフォルトは `https://geolonia.github.io/japanese-addresses/api/ja` です。このAPIのディレクトリ構造については、[Geolonia Address Data](https://github.com/geolonia/japanese-addresses/tree/develop/api) を参照してください。

`file://` URLを指定して、ローカルで保存したファイルを参照することもできます。

##### 使用例

```shell
# Geolonia Address Dataをダウンロード
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

## 正規化の詳細

* 住所から県名が欠落している場合（例："Kogun"）を補完します。
* アルファベットと数字を全角から半角に統一します。
* 京都の町名を削除します。
* 旧字体と新字体の差異を国土交通省の位置参照情報に合わせて吸収します。
* "ヶケが", "ヵカか力", "之ノの", "ッツっつ"の表記の違いを国交省の位置参照情報に合わせて吸収します。
* "釜" と "竈", "埠頭" と "ふ頭"の漢字の違いを吸収します。
* 町丁目レベルの住所の数字を全て漢数字に変換し、国交省の位置参照情報に合わせます。
* 番地と建物番号の数字をアラビア数字に変換し、"番地"などの文字列は "-" に変換します。
* 住所が建物名で終わる場合はそのまま返しますが、事前に分離することをお勧めします。

参考:

* [変換対象の文字列については、ソースコードを参照してください。](https://github.com/geolonia/normalize-japanese-addresses/blob/master/src/lib/dict.ts)
* [正規化前後の住所例については、テストコードを参照してください。](https://github.com/geolonia/normalize-japanese-addresses/blob/master/test/main.test.ts)

## 開発者情報

まずは、以下のコマンドでenvironments をセットアップしてください。

```shell
$ git clone git@github.com:geolonia/normalize-japanese-addresses.git
$ cd normalize-japanese-addresses
$ npm install
```

次に、以下のコマンドでコンパイルしてください。

```shell
$ npm run build
```

必要なファイルがdistフォルダに生成されます。そして、次のように sample.js ファイルを作成できます。

```javascript
// sample.js
const { normalize } = require('./dist/main-node.js');
normalize('北海道札幌市西区24-2-2-3-3', { level: 3 }).then(result => {
  console.log(result); // { "pref": "北海道", "city": "", "town": "", "addr": "札幌市西区二十四軒二条二丁目3-3", "level": 1 }
})
```

そして、以下のコマンドで実行できます。

```shell
$ node sample.js
```

## 注意事項

* この正規化エンジンは「住所統合」用に設計されており、例えば京都の通り名を削除します。
  * 郵便や宅配便サービスでの使用に適しています。
* このエンジンは地区や小規模な住所に対応していますが、それ以上のレベルの住所は扱えません。
* 住宅地の住所システムが十分に発達していない地域での対応は得意ではありません。

### 貢献方法

[プルリクエスト](https://github.com/geolonia/normalize-japanese-addresses/pulls)と[課題](https://github.com/geolonia/normalize-japanese-addresses/issues)は常に歓迎されています。

## ライセンス
このプロジェクトは [MIT License](LICENSE) のもとで公開されています。
