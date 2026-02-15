# Midi-Model UIリファレンス（LLM設定値生成用 / 日本語）

このドキュメントは、ChatGPT などに自然言語で作曲要件を渡し、`app.py` の UI に**そのまま入力できる設定値**を生成してもらうための仕様書です。

## 1. 使い方（推奨）

1. このドキュメントを LLM に渡す
2. 「出力は JSON のみ」と指示する
3. `custom prompt` タブ内の `apply JSON` を開く
4. JSON を貼り付けて `Apply JSON` を押す

LLM への最短依頼例:

```text
次のUI仕様に従って、custom prompt用の設定JSONを1つ作ってください。
条件: 落ち着いたピアノ主体、4/4、80 BPM前後、再現しやすい設定。
出力はJSONのみ。
```

## 2. 出力フォーマット（JSON）

### 2.1 custom prompt（主対象）

```json
{
  "tab": "custom_prompt",
  "custom_prompt": {
    "instruments": ["Acoustic Grand", "String Ensemble 1"],
    "drum_kit": "None",
    "bpm": 80,
    "time_signature": "4/4",
    "key_signature": "C",
    "seed": 12345,
    "random_seed": false,
    "generate_max_n_midi_events": 512,
    "temperature": 0.95,
    "top_p": 0.94,
    "top_k": 20,
    "allow_midi_cc_event": true
  }
}
```

### 2.2 midi prompt（補助）

```json
{
  "tab": "midi_prompt",
  "midi_prompt": {
    "use_first_n_midi_events_as_prompt": 128,
    "reduce_control_change_and_set_tempo_events": true,
    "remap_tracks_and_channels": true,
    "add_default_instrument": true,
    "remove_empty_channels": false
  }
}
```

注記: `midi_prompt` は別途 UI の `input midi` に `.mid/.midi` ファイル指定が必要です。

### 2.3 last output prompt（補助）

```json
{
  "tab": "last_output_prompt",
  "last_output_prompt": {
    "select_output_to_continue": "all"
  }
}
```

`select_output_to_continue` は `all` または `output1` `output2` ...（バッチ数分）。

## 3. UI項目の制約一覧（正規仕様）

### 3.1 custom prompt

- `instruments`
  - 型: `string[]`
  - 制約: 0〜15件、値は「楽器128種リスト」から選択
  - UI上の意味: 空なら自動選択
- `drum_kit`
  - 型: `string`
  - 値: `None`, `Standard`, `Room`, `Power`, `Electric`, `TR-808`, `Jazz`, `Blush`, `Orchestra`
  - デフォルト: `None`
- `bpm`
  - 型: `integer`
  - 範囲: 0〜255
  - デフォルト: 0（自動）
- `time_signature`
  - 型: `string`
  - 値: `auto`, `4/4`, `2/4`, `3/4`, `6/4`, `7/4`, `2/2`, `3/2`, `4/2`, `3/8`, `5/8`, `6/8`, `7/8`, `9/8`, `12/8`
  - デフォルト: `auto`
  - 注記: tv2モデルのみ有効
- `key_signature`
  - 型: `string`
  - 値: `auto`, `C♭`, `A♭m`, `G♭`, `E♭m`, `D♭`, `B♭m`, `A♭`, `Fm`, `E♭`, `Cm`, `B♭`, `Gm`, `F`, `Dm`, `C`, `Am`, `G`, `Em`, `D`, `Bm`, `A`, `F♯m`, `E`, `C♯m`, `B`, `G♯m`, `F♯`, `D♯m`, `C♯`, `A♯m`
  - デフォルト: `auto`
  - 注記: tv2モデルのみ有効
- `seed`
  - 型: `integer`
  - 範囲: 0〜2147483647
  - デフォルト: 0
- `random_seed`
  - 型: `boolean`
  - デフォルト: `true`
  - 挙動: `true` の場合、実行時に seed は自動再採番される
- `generate_max_n_midi_events`
  - 型: `integer`
  - 範囲: 1〜4096
  - デフォルト: 512
- `temperature`
  - 型: `number`
  - 範囲: 0.1〜1.2
  - デフォルト: 1.0
- `top_p`
  - 型: `number`
  - 範囲: 0.1〜1.0
  - デフォルト: 0.94
- `top_k`
  - 型: `integer`
  - 範囲: 1〜128
  - デフォルト: 20
- `allow_midi_cc_event`
  - 型: `boolean`
  - デフォルト: `true`

### 3.2 midi prompt

- `use_first_n_midi_events_as_prompt`: `integer`（1〜4097, default 128）
- `reduce_control_change_and_set_tempo_events`: `boolean`（default true）
- `remap_tracks_and_channels`: `boolean`（default true）
- `add_default_instrument`: `boolean`（default true）
- `remove_empty_channels`: `boolean`（default false）

### 3.3 last output prompt

- `select_output_to_continue`
  - 型: `string`
  - 値: `all` または `outputN`

## 4. LLM出力のバリデーション規則

- 範囲外数値は丸める
  - `bpm`: 0〜255
  - `seed`: 0〜2147483647
  - `generate_max_n_midi_events`: 1〜4096
  - `temperature`: 0.1〜1.2
  - `top_p`: 0.1〜1.0
  - `top_k`: 1〜128
- 整数項目は整数化（`round` ではなく `int` 推奨）
- `instruments` は最大15件に切り詰める
- `instruments` の未知名は除外する
- `drum_kit` は列挙値以外なら `None` にフォールバック
- 再現重視なら `random_seed=false` を使う

`Apply JSON` 実行時も同じ規則で自動補正されます。  
補正が発生した場合は `Apply result` に警告が表示されます。

## 5. 実務向けの設定決定順（LLMに指示しやすい順）

1. まず音楽要件を決める: `instruments`, `drum_kit`, `bpm`, `time_signature`, `key_signature`
2. 次に長さを決める: `generate_max_n_midi_events`
3. 最後に生成の揺らぎを決める: `temperature`, `top_p`, `top_k`, `allow_midi_cc_event`
4. 再現性要件を決める: `random_seed`, `seed`

## 6. LLMへ渡すプロンプトテンプレート

```text
あなたは midi-model の設定値生成アシスタントです。
以下の制約に厳密に従い、JSONのみを返してください。
- instruments は指定リストから最大15件
- drum_kit は列挙値のみ
- bpm: 0..255
- time_signature: 許可値のみ
- key_signature: 許可値のみ
- seed: 0..2147483647
- generate_max_n_midi_events: 1..4096
- temperature: 0.1..1.2
- top_p: 0.1..1.0
- top_k: 1..128
- boolean は true/false

要件:
{ここに自然言語の曲要件を記載}
```

## 7. 楽器128種リスト（`instruments` 用）

```text
Acoustic Grand
Bright Acoustic
Electric Grand
Honky-Tonk
Electric Piano 1
Electric Piano 2
Harpsichord
Clav
Celesta
Glockenspiel
Music Box
Vibraphone
Marimba
Xylophone
Tubular Bells
Dulcimer
Drawbar Organ
Percussive Organ
Rock Organ
Church Organ
Reed Organ
Accordion
Harmonica
Tango Accordion
Acoustic Guitar(nylon)
Acoustic Guitar(steel)
Electric Guitar(jazz)
Electric Guitar(clean)
Electric Guitar(muted)
Overdriven Guitar
Distortion Guitar
Guitar Harmonics
Acoustic Bass
Electric Bass(finger)
Electric Bass(pick)
Fretless Bass
Slap Bass 1
Slap Bass 2
Synth Bass 1
Synth Bass 2
Violin
Viola
Cello
Contrabass
Tremolo Strings
Pizzicato Strings
Orchestral Harp
Timpani
String Ensemble 1
String Ensemble 2
SynthStrings 1
SynthStrings 2
Choir Aahs
Voice Oohs
Synth Voice
Orchestra Hit
Trumpet
Trombone
Tuba
Muted Trumpet
French Horn
Brass Section
SynthBrass 1
SynthBrass 2
Soprano Sax
Alto Sax
Tenor Sax
Baritone Sax
Oboe
English Horn
Bassoon
Clarinet
Piccolo
Flute
Recorder
Pan Flute
Blown Bottle
Skakuhachi
Whistle
Ocarina
Lead 1 (square)
Lead 2 (sawtooth)
Lead 3 (calliope)
Lead 4 (chiff)
Lead 5 (charang)
Lead 6 (voice)
Lead 7 (fifths)
Lead 8 (bass+lead)
Pad 1 (new age)
Pad 2 (warm)
Pad 3 (polysynth)
Pad 4 (choir)
Pad 5 (bowed)
Pad 6 (metallic)
Pad 7 (halo)
Pad 8 (sweep)
FX 1 (rain)
FX 2 (soundtrack)
FX 3 (crystal)
FX 4 (atmosphere)
FX 5 (brightness)
FX 6 (goblins)
FX 7 (echoes)
FX 8 (sci-fi)
Sitar
Banjo
Shamisen
Koto
Kalimba
Bagpipe
Fiddle
Shanai
Tinkle Bell
Agogo
Steel Drums
Woodblock
Taiko Drum
Melodic Tom
Synth Drum
Reverse Cymbal
Guitar Fret Noise
Breath Noise
Seashore
Bird Tweet
Telephone Ring
Helicopter
Applause
Gunshot
```

## 8. 補足（実装依存の挙動）

- `instruments` を手動指定すると、生成中の `patch_change` は抑制されます。
- `time_signature` と `key_signature` は tv2 モデル以外では効果が限定されます。
- このリファレンスは UI 入力制約の仕様書です。作曲のコツは `docs/guide_ja.md` を参照してください。
