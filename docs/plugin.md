# プラグインの実装
WeVocal では、FX、音源、波形処理、解析の各プラグインを実装できる。それぞれ `manifest.json` の `kind` で種類を指定する。

## 開発コマンド

```bash
node plugin-sdk/scripts/wevocal-plugin.mjs create my-plugin my.gain
node plugin-sdk/scripts/wevocal-plugin.mjs check my-plugin
node plugin-sdk/scripts/wevocal-plugin.mjs test my-plugin
node plugin-sdk/scripts/wevocal-plugin.mjs build my-plugin
```

`create` は `template-processor/` の雛形からプラグインを作成する。

`check` は設定を検証する。`test` は Node.js 上で processor を動かし、正弦波・雑音・インパルスを入力した結果を `test/baseline.json` と比較する。処理の遅れ、余韻、処理時間も確認できる。

意図した変更によってテスト結果が変わった場合は、`--update` で基準データを更新する。

`build` は `dist/` に配布用の `manifest.json`、`processor.js`、WASM ファイルなどを出力する。配布するのは `dist/` の中身。

## FX プラグイン

FX は AudioWorklet で音声を加工するプラグイン。`manifest.json` の `kind` を `fx` にする。

### processor の実装

processor は `src/processor.js` に実装する。

- `import` は使用できない。必要なコードはバンドラーなどでまとめ、1 ファイルにする
- `registerProcessor()` に登録する名前は `wevocal-fx-` にプラグイン ID を続けたもの。ID の英数字以外は `-` に置き換える
- 例: `my.gain` の場合は `wevocal-fx-my-gain`
- `AudioParam` の名前は `params` の `id` と一致させ、`parameterDescriptors` にも同じ ID で定義する
- パラメータの値は基本的に 128 サンプル単位で渡される。配列の長さが 1 の場合は、そのブロック内で一定の値になる
- `process()` 内でメモリ確保などの負荷が大きい処理を避け、必要な状態は processor のインスタンス内に保持する
- 重い処理を WASM に移す場合は、`process()` から 128 サンプルごとに 1 回呼び出す構成にする

### サイドチェーン

`manifest.json` の `sidechain` を `true` にすると、2 つ目の入力を使用できる。

processor は `inputs[1]` から音声を取得する。何も接続されていなければ空になる。`numberOfInputs` はホスト側で `2` に設定する。

### 状態の保存

パラメータ以外の状態を保存する場合は、`MessagePort` でホストに通知する。

```js
this.port.postMessage({
  __wevocalState: '文字列'
});
```

ホストは受け取った状態をプロジェクトに保存し、processor の生成時に `options.processorOptions.state` で渡す。

ほかの場所で状態が変更された場合は、ホストから次の形式で通知される。

```js
{ __wevocalSetState: '文字列' }
```

状態は文字列で渡す。必要に応じて Base64 などでシリアライズする。

## 音源プラグイン

`kind` を `instrument` にすると、MIDI の音符を鳴らす音源を作成できる。作成した音源は、トラックの「音源…」から選択する。

音声入力は持たず、`numberOfInputs` は `0` にする。音声の出力とパラメータの扱いは FX と同様。

音符などのイベントは `port` 経由で届く。

| メッセージ | 内容 |
| --- | --- |
| `{ __wevocalNotes: [{ time, dur, pitch, vel }] }` | 指定時刻に発音し、`dur` 秒後に離す音符。イベントは事前に届くため、`currentTime` と `sampleRate` から発音するサンプル位置を計算する |
| `{ __wevocalNoteOn: { pitch, vel } }` | 指定した音程・ベロシティですぐに発音する。キーボード入力などに使う |
| `{ __wevocalNoteOff: { pitch } }` | 指定した音程の音を止める |
| `{ __wevocalClear: true }` | 予約済みの音符を取り消し、発音中の音も止める |
| `{ __wevocalSample: { channels, sampleRate } }` | サンプラーで使用する元の音声データ |

`__wevocalSample` は、マニフェストの `params` に `sample` がある音源で使用する。

書き出し時には、すべての音符が `processorOptions.events` から渡される。

標準のシンセとサンプラーの実装は `web/src/plugins/instruments.ts` を参照。

## 波形処理・解析プラグイン

`kind` を `tool` または `analyzer` にすると、選択した波形ブロックに対して処理や解析を実行できる。

processor では classic script として `registerTool(fn)` を呼び出す。

`fn(input, values)` には音声データとパラメータの値が渡される。

```js
input = {
  channels: Float32Array[],
  sampleRate
};
```

`values` は各パラメータの値を持つオブジェクト。関数は同期処理でも `Promise` を返す非同期処理でもよい。

### tool

加工した音声を返す。

```js
{ channels: Float32Array[] }
```

元の音声と長さが異なっていてもよい。

### analyzer

解析結果を返す。

```js
{
  rows: [
    { key, value, unit? }
  ]
}
```

`key` には `lang/` に定義した翻訳キーを指定する。解析結果は画面に一覧表示される。

processor は Worker 内で動作するため、アプリ内部のオブジェクトにはアクセスできない。音声データとパラメータの値を受け取って処理する。

標準の処理には、ノーマライズ、逆再生、直流成分の除去、モノラル化などがある。解析ではラウドネス、ピーク、スペクトルの特徴などを取得できる。

実装は `web/src/plugins/toolsStandard.ts` を参照。

## TypeScript で実装する

`plugin-sdk/types.ts` に型定義がある。また、`FxBase` は Web Audio のノードを構成するプラグイン向けの基底クラス。

標準エフェクトも TypeScript で実装されているため、`wevocal-lib` の `web/src/plugins/` が参考になる。

## 安全性と処理負荷

FX プラグインはアプリと同じ AudioContext・スレッド上で動作し、プラグインごとに処理が隔離されているわけではない。処理に時間がかかりすぎると、ほかのトラックを含めて音が途切れる可能性がある。

導入時にはこの点を利用者に伝え、確認してもらう。FX のダイアログには音声処理の負荷を表示する。

## プラグインの導入とプロジェクト

設定画面の「プラグイン」で `manifest.json` の URL を指定すると、プラグインを追加できる。

追加したプラグインは端末に保存され、オフラインでも利用できる。

プロジェクトにはプラグイン ID とパラメータの値が保存される。プラグインがインストールされていない端末で開くと、そのエフェクトは素通しになるが、パラメータの値は残る。
