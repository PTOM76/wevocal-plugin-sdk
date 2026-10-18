# manifest.json

`manifest.json` は、プラグインの識別情報、パラメータ、使用するファイルなどを定義するファイル。

## 基本項目

| 項目 | 説明 |
| --- | --- |
| `id` | プラグイン ID。小文字の英数字と `.`, `_`, `-` を使用できる。`wevocal.` で始まる ID は標準プラグイン用のため使用できない |
| `api` | API バージョン。現在は `1` |
| `kind` | プラグインの種類。FX は `fx` |
| `format` | プラグイン形式。現在は `wevocal` |
| `version` | プラグインのバージョン |
| `author` | 作者 |
| `name` | `lang/` に定義した名前の翻訳キー |
| `params` | パラメータの定義 |
| `processor` | processor のファイル |
| `wasm` | 任意。WASM ファイルを指定する |
| `permissions` | 要求する権限。導入時に利用者へ表示される。現在は宣言のみ |
| `tail` | 余韻の長さ（秒）。曲の終了位置の計算に使う |
| `latency` | 処理の遅延（サンプル数） |
| `realtime` | `false` の場合は書き出し専用。通常の再生では素通しになる |
| `anticipative` | `false` の場合は先行計算の対象にしない。既定では対象になる |
| `sidechain` | `true` の場合はサイドチェーン入力を追加する |
| `ui` | 独自 UI の HTML ファイル。省略時は標準 UI を生成する |
| `presets` | 同梱するプリセット |
| `migrate` | 古いパラメータ ID から新しい ID への変換設定 |

`wasm` に指定したファイルは、processor の `options.processorOptions.wasm` に `ArrayBuffer` として渡される。

`latency` が異なるトラックは、ホスト側で再生位置を調整してそろえる。

`anticipative` を `false` にするのは、乱数や時刻などによって処理結果が変わる場合など。

## パラメータ

`params` には、プラグインのつまみなどを定義する。

| 項目 | 説明 |
| --- | --- |
| `id` | パラメータ ID |
| `name` | `lang/` に定義した名前の翻訳キー |
| `min` | 最小値 |
| `max` | 最大値 |
| `default` | 初期値 |
| `step` | 値の変更単位 |
| `unit` | 単位。`%`、`dB`、`Hz`、`ms`、`s` など |
| `log` | 対数スケールで扱う場合に指定する |
| `choices` | 選択肢の翻訳キー。値には選択肢のインデックスを使う |

FX の `AudioParam` は、ここで定義した `id` と同じ名前にする。

## 翻訳

`lang/` に言語ごとの JSON ファイルを配置する。

例:

```text
lang/
  ja_jp.json
  en_us.json
```

- `name`: プラグイン一覧に表示する名前
- `description`: 一覧で名前の下に表示する説明と、ツールチップの説明
- `keywords`: 一覧で検索するときに使う単語。別言語での名前や言い換えなどを指定できる

`description` と `keywords` は省略できる。

`manifest.json` の `name` やパラメータの `name` には、翻訳に使用するキーを指定する。

## プリセットと移行設定

`presets` には同梱するプリセットを定義する。各プリセットは `name` と `values` を持つ。

`migrate` は、古いパラメータ ID を新しい ID に読み替えるために使用する。既存のプロジェクトで使われているパラメータ ID を変更する場合に利用する。