# Midi-Model 作曲ガイド（初心者向け）

このガイドは、`custom prompt` タブを使って「イメージに近い曲」を作るための実践マニュアルです。  
自然言語で指示するタイプではないため、**設定をレシピ化して少しずつ調整する**運用を前提にしています。

## 1. 5分クイックスタート

1. `Model option` でモデルを選び、`Load` を押して `success` を確認します。
2. `custom prompt` タブで、下の「8曲風レシピ」から1つをそのまま入力します（または `apply JSON` にLLM出力JSONを貼り付けて `Apply JSON`）。
3. `generate` を押します。
4. 良い結果が出たら `last output prompt` タブで続きを生成します。

失敗しやすい点:
- `Load` 前に `generate` を押すと生成できません。
- `random seed` が ON だと毎回結果が変わります。
- 最初は `generate max n midi events=512` で試すと比較しやすいです。

## 2. パラメータ早見表（custom prompt）

- `instruments (auto if empty)`
  - 曲で使う楽器を指定。空だと自動選択。
- `drum kit`
  - ドラムの質感。`None` でドラムなし寄り。
- `BPM`
  - テンポ。`0` は自動。
- `time signature`（tv2モデルのみ）
  - 拍子。例: `4/4`, `3/4`, `6/8`。
- `key signature`（tv2モデルのみ）
  - 調性。例: `C`, `Am`, `G`。
- `seed`
  - 乱数の種。同じ設定+同じseedなら再現しやすい。
- `random seed`
  - ON: 毎回seedを自動変更。OFF: `seed` を固定して再現。
- `generate max n midi events`
  - 曲の長さ・展開量。
- `temperature`
  - 高いほど冒険、低いほど安定。
- `top p`
  - 候補の広さ（核サンプリング）。
- `top k`
  - 上位何候補から選ぶか。
- `allow midi cc event`
  - 表情変化（CC）を許可するか。

## 3. 実用レンジ（まずはここから）

- `temperature`: `0.85`〜`1.05`（標準 `1.00`）
- `top p`: `0.90`〜`0.98`（標準 `0.94`）
- `top k`: `12`〜`40`（標準 `20`）
- `generate max n midi events`: `256`（短い）/`512`（標準）/`1024`（長い）

目安:
- 破綻が多い: `temperature` を下げる、`top k` を下げる
- 無難すぎる: `temperature` を上げる、`top p` を上げる

## 4. 8曲風レシピ（そのまま試せる設定）

以下は「出発点」です。気に入らなければ1〜2項目ずつ変更して比較してください。

| 曲風 | instruments | drum kit | BPM | time signature | key signature | random seed | seed | events | temperature | top p | top k | allow cc |
|---|---|---|---:|---|---|---|---:|---:|---:|---:|---:|---|
| Lo-fi Hip Hop | `Electric Piano 1`, `Acoustic Bass`, `Vibraphone` | `TR-808` | 78 | 4/4 | Am | OFF | 1201 | 512 | 0.92 | 0.94 | 16 | ON |
| Piano Ballad | `Acoustic Grand`, `String Ensemble 1`, `Cello` | `None` | 72 | 4/4 | C | OFF | 2302 | 512 | 0.88 | 0.92 | 14 | ON |
| EDM / Dance | `Lead 2 (sawtooth)`, `Synth Bass 1`, `Pad 1 (new age)` | `Electric` | 128 | 4/4 | Em | OFF | 3403 | 768 | 1.02 | 0.97 | 28 | ON |
| Orchestral Cinematic | `String Ensemble 1`, `French Horn`, `Oboe`, `Timpani` | `Orchestra` | 96 | 4/4 | Dm | OFF | 4504 | 1024 | 0.90 | 0.93 | 18 | ON |
| Jazz Trio | `Acoustic Grand`, `Acoustic Bass`, `Ride Cymbal 1` | `Jazz` | 132 | 4/4 | F | OFF | 5605 | 640 | 0.96 | 0.95 | 20 | ON |
| Rock Band | `Overdriven Guitar`, `Distortion Guitar`, `Electric Bass(finger)` | `Power` | 140 | 4/4 | E | OFF | 6706 | 768 | 0.98 | 0.95 | 24 | ON |
| Ambient / New Age | `Pad 2 (warm)`, `Choir Aahs`, `Flute` | `None` | 60 | 4/4 | C | OFF | 7807 | 768 | 0.86 | 0.92 | 14 | ON |
| 8-bit / Retro Game | `Lead 1 (square)`, `Lead 2 (sawtooth)`, `Synth Bass 1` | `Standard` | 150 | 4/4 | Gm | OFF | 8908 | 512 | 1.00 | 0.96 | 22 | OFF |

レシピ共通の調整順:
1. まず `seed` 固定で `temperature` だけ上下させる
2. 次に `top p` を `0.02` ずつ動かす
3. 最後に `top k` を `4` ずつ動かす

## 5. 目的別の微調整

- もっと安定させたい
  - `temperature` を `-0.05`
  - `top k` を `-4`
- もっと意外性を出したい
  - `temperature` を `+0.05`
  - `top p` を `+0.02`
- 同じ曲風でバリエーションを増やしたい
  - `random seed=ON` にするか、`seed` だけ変更
- 長く展開させたい
  - `events` を `768` or `1024` に上げる

## 6. 他タブの最低限ガイド

- `midi prompt`
  - 手元MIDIの冒頭イベントを「お手本」にして続きを生成するタブ。
  - 既存曲の雰囲気を引き継ぎたい時に使います。
- `last output prompt`
  - 直前出力の続きを作るタブ。
  - 良い出力を伸ばす反復に向いています。

## 7. よくある質問（FAQ）

- Q. `Generate` を押してもエラーになる
  - A. モデルが未ロードの可能性があります。`Load` 後に `success` を確認してください。

- Q. 毎回まったく違う曲になる
  - A. `random seed` を OFF にして `seed` を固定してください。

- Q. 音が多すぎてまとまらない
  - A. `temperature` を下げ、`top k` を下げ、`events` を 512 以下にしてください。

- Q. 単調で盛り上がらない
  - A. `events` を増やし、`temperature` を少し上げ、`allow midi cc event` を ON にしてください。

## 8. まず試す推奨手順（初心者向け）

1. `Piano Ballad` レシピをそのまま実行
2. 同じ `seed` のまま `temperature` を `0.88 -> 0.95` に変更
3. 良い方を `last output prompt` で継続生成
4. 最後に `events` を増やして尺を延ばす

この順番だと、設定変更の効果を理解しやすく、狙った曲風に寄せやすくなります。
