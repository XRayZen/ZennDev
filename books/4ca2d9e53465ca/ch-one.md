---
title: "環境構築"
---

# 環境構築

OSを自作するための開発環境を構築します。この章では、必要なツールのインストールと基本的な設定方法について説明します。

## 必要なツール

1. **コンパイラ/アセンブラ**: GCCやNASMなどのツールを使用してソースコードをコンパイルします。
2. **エミュレータ**: QEMUなどの仮想環境でOSを実行・テストします。
3. **デバッガ**: GDBなどを使用してコードのデバッグを行います。
4. **エディタ/IDE**: Visual Studio CodeやVimなどのコードエディタを使用します。

## 開発環境のセットアップ

### Linuxの場合

```bash
# 必要なパッケージのインストール
sudo apt-get update
sudo apt-get install build-essential nasm qemu-system-x86 gdb

# 作業ディレクトリの作成
mkdir -p myos/src
cd myos
```

### macOSの場合

```bash
# Homebrewを使用してパッケージをインストール
brew update
brew install gcc nasm qemu gdb

# 作業ディレクトリの作成
mkdir -p myos/src
cd myos
```

### Windowsの場合

Windows環境では、WSL（Windows Subsystem for Linux）を使用するか、MinGWなどのツールを使用します。

```bash
# WSLのUbuntuで必要なパッケージをインストール
sudo apt-get update
sudo apt-get install build-essential nasm qemu-system-x86 gdb

# 作業ディレクトリの作成
mkdir -p myos/src
cd myos
```

## プロジェクト構成

基本的なプロジェクト構成は以下のようになります：

```
myos/
├── src/           # ソースコード
├── build/         # ビルド成果物
├── tools/         # ビルドスクリプトなど
└── Makefile       # ビルド設定
```

次の章では、この環境を使って実際に文字を画面に描画する方法について学びます。
