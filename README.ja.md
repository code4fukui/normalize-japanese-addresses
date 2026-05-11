# @geolonia/normalize-japanese-addresses

[
![build](https://github.com/geolonia/normalize-japanese-addresses/actions/workflows/build.yml/badge.svg)
](https://github.com/geolonia/normalize-japanese-addresses/actions/workflows/build.yml)

経済産業省が提供する[IMIコンポーネントツール](https://info.gbiz.go.jp/tools/imi_tools/)のジオコーディングメカニズムにインスパイアされた、オープンソースの日本の住所正規化ライブラリです。

## デモ

- [CodePen デモ](https://codepen.io/geolonia/pen/oNBrqzL)
- [インタラクティブな例](https://code4fukui.github.io/normalize-japanese-addresses/)

## 特徴

- **標準化**: 住所を一貫した標準的なフォーマットに変換します。
- **漢字の正規化**: 旧字体と新字体の揺れや、その他一般的な文字の揺れ（例: `ヶ` と `ケ`、`釜` と `竈`、`埠頭` と `ふ頭`）を吸収します。
- **数字の変換**: 住所内の数字を統一し、町丁目名に含まれる数字を漢数字に、番地・号をアラビア数字に変換します。
- **文字幅の統一**: 全角の英数字をすべて半角に変換します。
- **住所の補完**: 省略された「郡」の名称を自動的に補完します。
- **京都の住所への対応**: 京都特有の通り名を削除し、住所をシンプルにします。
- **ジオコーディング**: 正規化された住所に対して、データが存在する場合は緯度と経度を返します。

## インストール

お好みのパッケージマネージャーを使用して、npmレジストリからライブラリをインストールします。

```shell
npm install @geolonia/normalize-japanese-addresses
```

## 使い方

### `normalize(address: string, option?: Option)`

指定された日本の住所文字列を正規化します。

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

**ブラウザ / Deno (ES Modules)**

```javascript
import { normalize } from "https://code4fukui.github.io/normalize-japanese-addresses/dist/main-es.js";

const result = await normalize('北海道札幌市西区24-2-2-3-3');
console.log(result);
```

### 正規化レベル

返されるオブジェクトには、住所文字列をどこまで判別できたかを示す `level` プロパティが含まれます。

- `0`: 都道府県も判別できなかった。
- `1`: 都道府県まで判別できた。
- `2`: 市区町村まで判別できた。
- `3`: 町丁目まで判別できた。
- `7`: （高精度モード）住居表示住所の街区までの判別ができた。
- `8`: （高精度モード）住居表示住所の街区符号・住居番号までの判別ができた。

`level` オプションを使用して目的の正規化レベルを指定することで、処理を高速化できます。

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

### 高精度な正規化（住居表示レベル）

住居表示の街区符号や住居番号の解析（レベル7および8）を含むより精度の高い結果を得るには、`geoloniaApiKey` を指定してGeoloniaのバックエンドを利用できます。

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

## 設定

`config` オブジェクトのプロパティを設定することで、ライブラリ全体の動作を変更できます。

```javascript
const { config, normalize } = require('@geolonia/normalize-japanese-addresses');
```

- **`config.townCacheSize: number`**
  ライブラリは市区町村ごとに町丁目のデータを取得し、キャッシュします。このオプションはキャッシュする市区町村の最大数を設定します。デフォルトは `1000` です。

- **`config.japaneseAddressesApi: string`**
  日本の住所データAPIのエンドポイントを指定します。デフォルトは `https://geolonia.github.io/japanese-addresses/api/ja` です。

  `file://` 形式で指定することで、ローカルディレクトリを参照できます。これはオフラインでの利用や、独自の住所データを使用する場合に便利です。

  **例: ローカルの住所データを使用する**

  1.  [geolonia/japanese-addresses](https://github.com/geolonia/japanese-addresses) から住所データをダウンロードします。
      ```shell
      curl -sL https://github.com/geolonia/japanese-addresses/archive/refs/heads/master.tar.gz | tar xvfz -
      ```
  2.  `japaneseAddressesApi` の設定にローカルのパスを指定します。
      ```javascript
      const path = require('path');
      const { config, normalize } = require('@geolonia/normalize-japanese-addresses');

      const localDataPath = path.resolve(__dirname, 'japanese-addresses-master/api/ja');
      config.japaneseAddressesApi = `file://${localDataPath}`;

      normalize('...your address...');
      ```

## 制限事項

- このライブラリは住所のマッチングを目的として設計されているため、京都の通り名など、郵便配達において必須ではない住所の一部を削除する場合があります。
- デフォルトのオープンデータモードでは、町丁目レベルまでの正規化を行います。住居表示の街区・住居番号レベルの正規化には、APIキーを使用した高精度モードが必要です。
- 住居表示未整備の地域では、正規化の精度が低くなる場合があります。

## 開発者向け

ローカルの開発環境をセットアップするには以下の手順を実行します。

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

## コントリビューション

Pull Request や Issue はいつでも歓迎します。

正規化ロジックの詳細については、以下のソースコードを参照してください。
- **文字列の揺れ:** [`src/lib/dict.ts`](https://github.com/geolonia/normalize-japanese-addresses/blob/master/src/lib/dict.ts)
- **変換前後の例:** [`test/main.test.ts`](https://github.com/geolonia/normalize-japanese-addresses/blob/master/test/main.test.ts)

## ライセンス

このプロジェクトは MIT License の下でライセンスされています。

ソーシャルメディアでこのプロジェクトをシェアしていただいたり、[Geolonia](https://geolonia.com/) へのリンクを貼っていただけると、開発者のモチベーション向上に繋がりますので、ぜひよろしくお願いいたします。
