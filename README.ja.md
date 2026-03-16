# @geolonia/normalize-japanese-addresses
日本語のREADMEはこちらです: [README.ja.md](README.ja.md)

オープンソースのアドレス正規化ライブラリ。

## デモ

- https://codepen.io/geolonia/pen/oNBrqzL
- https://code4fukui.github.io/normalize-japanese-addresses/

## インストール

このライブラリは npmレジストリに `@geolonia/normalize-japanese-addresses` として配布されています。
npm コマンドを使ってインストールできます。

```shell
$ npm install @geolonia/normalize-japanese-addresses -S
```

## 使用方法

### `normalize(address: string, option: Option)`

住所を正規化する。

```javascript
const { normalize } = require('@geolonia/normalize-japanese-addresses')
normalize('北海道札幌市西区24-2-2-3-3').then(result => {
  console.log(result); // {"pref": "北海道", "city": "札幌市西区", "town": "二十四軒二条二丁目", "addr": "3-3", "lat": 43.074273, "lng": 141.315099, "level": 3}
})
```

正規化された結果オブジェクトには `level` プロパティが含まれています。 `level` は、住所の識別できた部分を示す数値を格納しています:

* `0` - 都道府県を識別できなかった。
* `1` - 都道府県を識別できた。
* `2` - 市区町村を識別できた。
* `3` - 町名まで識別できた。

例えば、都道府県名のみを正規化したい場合は、 `level` オプションを指定することで処理が高速化されます。

```javascript
const { normalize } = require('@geolonia/normalize-japanese-addresses')
normalize('北海道札幌市西区24-2-2-3-3', { level: 1 }).then(result => {
  console.log(result); // {"pref": "北海道", "city": "", "town": "", "addr": "札幌市西区二十四軒二条二丁目3-3", "lat": null, "lng": null, "level": 1}
})
```

ES モジュールとして使用する (ブラウザや Deno)
```javascript
import { normalize } from "https://code4fukui.github.io/normalize-japanese-addresses/dist/main-es.js";

const result = await normalize('北海道札幌市西区24-2-2-3-3');
console.log(result);
```

### グローバルオプション

ライブラリの全体的な動作を変更するには、以下のパラメーターを使用できます。

#### `config.townCacheSize: number`

`@geolonia/normalize-japanese-addresses` ライブラリは、最新の町名データをWeb APIから取得して住所の正規化を行います。 `townCacheSize` オプションは、キャッシュする市町村数を変更します。デフォルトは1,000です。

#### `config.japaneseAddressesApi: string`

町名データを提供するWeb APIのエンドポイントを指定します。デフォルトは `https://geolonia.github.io/japanese-addresses/api/ja` です。このAPIのディレクトリ構造については、[Geolonia Address Data](https://github.com/geolonia/japanese-addresses/tree/develop/api)を参照してください。

`file://` URLを指定して、ローカルに保存したファイルを参照することもできます。

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

* 住所の中に県名が省略されている場合(例: "虎ノ門"など)、県名を補完します。
* アルファベットと数字を全角から半角に統一します。
* 京都の町名を削除します。
* 国土交通省の位置参照情報に合わせて、旧字体と新字体の差異を吸収します。
* 国土交通省の位置参照情報に合わせて、"ヶケが"、"ヵカか力"、"之ノの"、"ッツっつ"の表記の差異を吸収します。
* 国土交通省の位置参照情報に合わせて、"釜"と"竈"、"埠頭"と"ふ頭"の漢字の差異を吸収します。
* 町丁目レベルの住所の数字を全て漢数字に統一し、国土交通省の位置参照情報に合わせます。
* 号番地の数字を全て半角アラビア数字に変換し、"番地"の文字列は"-"に変換します。
* 住所の末尾に建物名がある場合、そのままの形で出力しますが、事前に分離するのがよい場合があります。

参考:

* [処理対象となる表記の差異については、ソースコードの dict.ts をご参照ください。](https://github.com/geolonia/normalize-japanese-addresses/blob/master/src/lib/dict.ts)
* [正規化前後の住所例については、テストコードの main.test.ts をご参照ください。](https://github.com/geolonia/normalize-japanese-addresses/blob/master/test/main.test.ts)

## 開発者情報

まず、以下のコマンドで環境を設定してください。

```shell
$ git clone git@github.com:geolonia/normalize-japanese-addresses.git
$ cd normalize-japanese-addresses
$ npm install
```

次に、以下を実行してコンパイルしてください:

```shell
$ npm run build
```

必要なファイルがdistフォルダに生成されます。それから、次のようにsample.jsファイルを作成できます:

```javascript
// sample.js
const { normalize } = require('./dist/main-node.js');
normalize('北海道札幌市西区24-2-2-3-3', { level: 3 }).then(result => {
  console.log(result); // { "pref": "北海道", "city": "", "town": "", "addr": "札幌市西区二十四軒二条二丁目3-3", "level": 1 }
})
```

そして、以下のように実行してください:

```shell
$ node sample.js
```

## メモ

* このノーマライゼーションエンジンは「住所のコンソリデーション」を目的に設計されており、京都の街路名などを削除します。
  * 郵便や宅配サービスで使用するのに適しています。
* このノーマライゼーションエンジンは近隣および小規模な住所をサポートしますが、それ以上の住所レベルは処理できません。
* 住所システムが十分に発達していない地域での対応は得意ではありません。

### 貢献方法

[プルリクエスト](https://github.com/geolonia/normalize-japanese-addresses/pulls)と[イシュー](https://github.com/geolonia/normalize-japanese-addresses/issues)は常に歓迎されます。

## ライセンス
このプロジェクトは [MIT License](LICENSE) のもとで公開されています。
