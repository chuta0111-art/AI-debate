# Qwen3-32B 導入ノート

## 前提環境

- GPU: VRAM 16GB
- 用途: LLMチャット + 動画解析（併用したい）
- ツール: Ollama

---

## Qwen3-32B の基本情報

- パラメータ数: 32B
- 量子化 Q4_K_M 時のサイズ: 約18-20GB
- VRAM 16GBには全部載らない → RAM にはみ出す

---

## VRAM/RAM の使われ方

Ollama はVRAMを先に埋めてから、残りをRAMに逃がす。

| 項目 | 使用量 |
|------|--------|
| VRAM | 16GB（ほぼ満杯） |
| RAM  | 2-4GB（はみ出し分） |

- VRAMは余らない（デフォルト設定の場合）
- 手動でVRAM制限も可能:

```bash
OLLAMA_MAX_VRAM=8g ollama serve
# → VRAM 8GB + RAM 10-12GB（ただし遅くなる）
```

---

## 動画解析との併用について

### VRAM要件の目安

| 動画解析の種類 | VRAM目安 | Qwen3-32Bと併用 |
|---|---|---|
| FFmpeg（CPU処理） | 0GB | 問題なし |
| OpenCV（CPU） | 0GB | 問題なし |
| YOLO物体検出 | 2-4GB | 厳しい |
| Whisper音声認識 | 1-4GB | 厳しい |
| 動画生成AI | 8-16GB+ | 無理 |

### 結論: 16GB VRAMでは同時実行は基本的に厳しい

- CPU処理（FFmpeg, OpenCV）→ 併用可
- GPU必要な処理（YOLO, Whisper等）→ 切り替え運用が現実的

---

## 切り替え運用

### 基本操作

```bash
# LLM停止 → VRAM即解放
ollama stop

# 動画解析が終わったら再起動
ollama run qwen3:32b
```

### 切り替え時の注意

| 項目 | 内容 |
|------|------|
| モデルロード | 毎回20-60秒かかる |
| 会話コンテキスト | ollama stop で消える |

### おすすめ: 時間帯で分ける

- 日中: 動画解析
- 夜: LLMチャット

---

## 会話履歴の保持方法

### 方法1: Ollama の /save /load

```bash
# 会話中に保存
>>> /save my-session

# 再起動後に復元
>>> /load my-session
```

- テキスト履歴は復元されるが、KVキャッシュ（内部状態）は消える
- 履歴が長いほど復元に時間がかかる

### 方法2: Open WebUI（おすすめ）

```bash
docker run -d -p 3000:8080 ghcr.io/open-webui/open-webui:main
# ブラウザで http://localhost:3000
```

- 会話履歴がDBに自動保存（Ollamaと独立）
- Ollama停止/再開しても履歴が残る
- 複数の会話を管理できる
- ChatGPTのようなUIで使いやすい
- **切り替え運用との相性が良い**

### 方法3: 手動で要約コピペ

- 原始的だが、要約して貼れば軽い

---

## TODO（次のセッションで進める）

- [ ] Ollama のインストール
- [ ] Qwen3-32B のダウンロード (`ollama pull qwen3:32b`)
- [ ] 動作確認・速度検証
- [ ] Open WebUI の導入検討
- [ ] 動画解析ツールの選定と切り替えスクリプト作成
- [ ] AI Debate アプリとの連携方法を検討
