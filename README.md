# WeVocal Plugin SDK
WeVocalシリーズのプラグインを開発するためのSDK。<br />
プラグインの作成、設定の検証、動作テスト、ビルドには付属のスクリプトを使用する。

(正直、Web Audio Modulesも対応しているのでそっちで作る方が汎用性はあるんじゃね・・・？)

## プラグインを作成する
Node.js で以下のコマンドを実行する。

```bash
# my-plugin/ フォルダ に my.gain というIDでプラグインを作成する
node plugin-sdk/scripts/wevocal-plugin.mjs create my-plugin my.gain
node plugin-sdk/scripts/wevocal-plugin.mjs check my-plugin
node plugin-sdk/scripts/wevocal-plugin.mjs test my-plugin
node plugin-sdk/scripts/wevocal-plugin.mjs build my-plugin
```

`my-plugin/` に `my.gain` という ID のプラグインが作成される。

- `check`: 設定を検証する
- `test`: 音声処理の結果を基準データと比較する
- `build`: 配布用ファイルを `dist/` に生成する

## ドキュメント

- [manifest.json](docs/manifest.md) — プラグインの設定項目
- [プラグインの実装](docs/plugins.md) — FX、音源、波形処理、解析
- [独自 UI](docs/custom-ui.md) — HTML による UI の実装と通信方法

## 関連ファイル

- `template/`: プラグインの雛形 (方法1)
- `template-processor/`: プラグインの雛形 (方法2)
- `scripts/wevocal-plugin.mjs`: プラグインの作成・検証・テスト・ビルド
- `types.ts`: TypeScript型定義

TypeScript で標準エフェクトと同様の実装を行う場合は、`wevocal-lib` の `web/src/plugins/` も参考になる。
- https://github.com/PTOM76/wevocal-lib/tree/main/web/src
