# midi-model (macOS fork)

## 概要

このリポジトリは [SkyTNT/midi-model](https://github.com/SkyTNT/midi-model) をフォークし、macOS で動かしやすいように調整したものです。  
本 README は、macOS でのセットアップと起動を最短で再現できる手順に絞って説明します。

![](./banner.png)

## 対応環境

- macOS (Apple Silicon 推奨)
- Python `3.11.14`（`asdf` 利用例）
- Homebrew

## ドキュメント

- 初心者向け作曲ガイド: `docs/guide_ja.md`
- LLM設定値生成仕様: `docs/ui_reference_for_llm_ja.md`
- custom prompt JSONサンプル: `docs/custom_prompt_example_ja.json`

## Quickstart (macOS)

以下をそのまま実行してください。

```bash
asdf install python 3.11.14
asdf set python 3.11.14

brew install fluid-synth

python -m venv venv
source venv/bin/activate

pip install torch
pip install -r requirements.txt
pip install requests
```

補足:
- `torch` は環境ごとの差分が出やすいため、先に単独インストールしています。
- `pyfluidsynth` は `requirements.txt` に含まれますが、`fluid-synth` 本体は Homebrew で別途必要です。

## モデルの準備

`models/midi-model-tv2o-medium` にモデルをダウンロードします。

```bash
source venv/bin/activate
python - <<'PY'
from huggingface_hub import snapshot_download
snapshot_download(
    repo_id="skytnt/midi-model-tv2o-medium",
    local_dir="models/midi-model-tv2o-medium",
    local_dir_use_symlinks=False
)
PY
```

## 起動方法

```bash
source venv/bin/activate
export HOMEBREW_PREFIX="$(brew --prefix)"
python -c "import fluidsynth; print('fluidsynth import ok')"

export NO_PROXY="127.0.0.1,localhost"
export no_proxy="$NO_PROXY"
unset GRADIO_SHARE

python app.py --port 7862 --device mps
```

起動後、ブラウザで UI が開きます。  
`Model option` で `models/midi-model-tv2o-medium` 配下のモデルを選択し、`Load` を押してください。

## CLI オプション

`app.py` は以下のオプションを受け付けます。

- `--port` (default: `7860`)
- `--device` (default: `auto`, values: `auto/cpu/cuda/mps`)
- `--batch` (default: `4`)
- `--share` (default: `False`)

macOS では `--device mps` を推奨します。  
MPS が使えない環境では `--device auto` または `--device cpu` を使ってください。

## apply JSON で一括設定

`custom prompt` タブの `apply JSON` を使うと、複数パラメータをまとめて反映できます。

1. `custom prompt` タブを開く
2. `apply JSON` を開く
3. JSON を貼り付けて `Apply JSON` を押す
4. 必要なら `Apply result` の警告を確認する

`Apply JSON` は `docs/ui_reference_for_llm_ja.md` の制約に沿って値を補正します。

## LLMにJSONを作らせる方法

ChatGPT などの LLM に自然言語要件を渡して、`custom prompt` 用 JSON を生成できます。  
その際は `docs/ui_reference_for_llm_ja.md` を一緒に渡し、必ず「JSONのみ」で返すよう指示してください。

LLMへの依頼例:

```text
あなたは midi-model の設定値生成アシスタントです。
docs/ui_reference_for_llm_ja.md の制約に厳密準拠し、
custom_prompt 形式のJSONを1つ作成してください。
要件:
- 落ち着いたピアノ主体
- 4/4
- 78〜84 BPM
- 明るすぎず切なめ
- 再現重視（random_seed=false）
出力はJSONのみ（説明文・コードフェンス禁止）。
```

## よくあるハマりどころ

- `import fluidsynth` で失敗する:
  - `brew install fluid-synth` が完了しているか確認してください。
  - 必要に応じて `export HOMEBREW_PREFIX="$(brew --prefix)"` を設定してください。
- 起動はするがアクセスが不安定:
  - `NO_PROXY/no_proxy` を `127.0.0.1,localhost` に設定してください。
- モデルが見つからない:
  - `models/midi-model-tv2o-medium` にダウンロード済みか確認してください。

## Demo / 参考リンク

- [online: huggingface](https://huggingface.co/spaces/skytnt/midi-composer)
- [online: colab](https://colab.research.google.com/github/SkyTNT/midi-model/blob/main/demo.ipynb)
- [download windows app](https://github.com/SkyTNT/midi-model/releases)

## Pretrained model

- [huggingface](https://huggingface.co/skytnt/midi-model-tv2o-medium)

## Dataset

- [projectlosangeles/Los-Angeles-MIDI-Dataset](https://huggingface.co/datasets/projectlosangeles/Los-Angeles-MIDI-Dataset)

## Train

```bash
python train.py
```

## Citation

```bibtex
@misc{skytnt2024midimodel,
  author = {SkyTNT},
  title = {Midi Model: Midi event transformer for symbolic music generation},
  year = {2024},
  howpublished = {\url{https://github.com/SkyTNT/midi-model}},
}
```
