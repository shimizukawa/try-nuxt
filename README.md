# try-nuxt2

https://nuxtjs.org/ja/ を読んで何か色々やる練習用リポジトリです。

- source-map を使ってブラウザでminify前の元コードを把握しながらデバッグしたい
- production向けのsource-mapをSentry向けに用意する
- uglify-jsでsource-map出す

## 前提

- node 14
- yarn
- nuxt 2

## 用語

- minify: 複数のJSファイルを1つにまとめたり、変数名を1文字にしたり、改行なしにして、小さくしている状態
- source-map: minify前のソースコードが分かるように、ファイル名や変数名をマッピングしてくれる

## 起動手順

```
$ cd frontend
$ yarn install
```

### devサーバー起動

- `yarn dev`
    - `nuxt` のalias, packages.jsonで定義
- http://127.0.0.1:3000/ にアクセス
- ブラウザの開発者コンソール(devtool)でJSを見る
    - http://127.0.0.1:3000/_nuxt/app.js minifyされたJS
    - `webpack:///layouts/default.vue` 生vueファイル（breakpoint設定できる）

### 本番用ビルド

`nuxt.config.js` が `target: "server"` の場合
```
$ yarn build  # nuxt build
$ yarn start  # nuxt start
```

`nuxt.config.js` が `target: "static"` の場合
```
$ yarn generate  # nuxt generate
$ yarn start  # nuxt start
```

- `frontend/dist/_nuxt/7bccea6.js` 等が生成されている
    - minifyされており改行もないので読めない
    - ファイル名が元のコードからかけ離れている
- http://127.0.0.1:3000/ にアクセス
- ブラウザのdevtoolでJSを見る
    - http://127.0.0.1:3000/_nuxt/7bccea6.js
    - devtoolの機能で読みやすく改行されている
    - 変数名などは短くなっていて元コードは分からない
    - ファイル名が元のコードからかけ離れている

### 本番用ビルド + source-map

`nuxt.config.js` に以下を追加して、 `source-map` を生成させる
```js
  build: {
    extend (config) {
      config.devtool = 'source-map';
    }
  }
```

- `yarn generate` を再実行
- `frontend/dist/_nuxt/7bccea6.js.map` が追加で生成される
    - このファイルがあれば元ソースのようにブラウザ上で読める
    - ファイル内を見ると、 `webpack:///layouts/default.vue` のような文字列が見える
- `yarn start` してブラウザでアクセスし、devtoolで `webpack://` 以下をみると元ソースのように読める
    - デバッガでbreakpointも設定できる


`config.devtool` に設定できる値は https://webpack.js.org/configuration/devtool/ にある。

- `source-map`: 遅いが本番用途に使える。本番向けにデプロイまでしなくても、ブラウザでソースマップを指定したりSentryにアップロードしたりできる
    - webpack-internal:///.nuxt/App.js 
- `eval-source-map`: 開発サーバーなどに置く用？リビルドが速いらしい


## Sentry向けsource-mapの扱い

とりあえずメモ
https://zenn.dev/kazuma1989/scraps/a67a7d2d17b131
