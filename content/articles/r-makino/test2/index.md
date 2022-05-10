---
title: "[第１回]`TypeScript` x `Node.js` x `Nodegui(Qt)` でマルチプラットフォームGUIアプリケーションを作ろう"
slug: "test2"
date: 2022-05-09T16:42:44+09:00
authors: ["b-bar"]
tags: ["aws", "s3"]
draft: false
resources:
  -
    name: "featuredImage"
    src: "images/featuredImage.png"
    params:
      alt: "featured image"
---

## はじめに
突然ですが、スタンドアロンなGUIアプリケーションを作ったことはありますか？

筆者は、Webアプリを普段作成していることもあって、普段の業務で作成する機会はあまりありません。GUIといえば、HTML+CSS+JSで構成される、いわゆるWebページの作成がメインです。

なので、豊富な知見に基づく様なお話は難しいのですが、今回社内向けのマルチプラットフォームツールにGUIを実装する事があったので、どうせならその過程を記事にしてしまおうということで、本連載を書くことにしました。

一応、GUIツールキット・フレームワークとしては、下記の様なものに触れたことがあります。

- Windows向け
  - Form with Visual Basic
  - MFC with Visual C++
  - PyGTK with python3
- Linux向け
  - GTK with C++
- Mac向け
  - GLFW with C++
  - Electron with JavaScript

本連載で作成するツール自体は非常に単純なものではありますが、単純だからこそGUIアプリケーションを作りたいと考えている方の一助になれば幸いと考えています。

また、今回は環境構築がメインとなりますので、具体的なアプリケーションについては次回以降に詳しく解説を行っていきます。


## 今回の目標
- 言語・GUIツールキットの選定理由について
- NodeGui公式のサンプルアプリケーションが動作する環境の構築
- NodeGui公式のサンプルアプリケーションによる動作確認


## 本連載での使用言語・GUIツールキットの選定理由について
### 使用言語について
言語に関しては、特別のこだわり~~というか理由~~は無いのですが、最近JavaScriptのコードをTypeScriptで書き直しする機会があったので、TypeScriptの練度向上も兼ねてTypeScriptを選択しました。

以前なら、C++やC#やObjective-C等のC系の言語で作成するのが普通(何を持って？)だったかと思いますが、2022年現在においては、大抵の主要な(よく使われるというニュアンス)言語において、主だったGUIツールキットのラッパーが作成されています。

したがって、ツールキットをこだわることは有っても、言語をこだわることはそれほど無い気がします。基本的にはご自身の使い慣れた言語で作れば良いと思います。

もちろん、実行速度やメンテナンス性という観点ではキチンと考える必要はあるかと思いますが。


### GUIツールキットについて
#### GUIツールキットの大まかな分類

|種類|具体例|マルチプラットフォーム|ソース記述量|
|-|-|-|-|
|ローレベルAPI|GDI/Carbon/GLFW|🔺|❌|
|OS提供のフレームワーク|MFC/WPF/Cocoa/SwiftUI|❌|⭕️|
|ローレベルAPIを利用したフレームワーク|GTK/Qt|⭕️|⭕️|
|Webブラウザ/Web Viewを利用したフレームワーク|Electron/Wails|⭕️|⭕️|

#### 使用GUIツールキットについて
マルチプラットフォーム対応のため、基本的にはWeb系ないしはローレベルのAPIのラッパー系フレームワーク以外は候補外となります。

GLFWを利用してもマルチプラットフォームアプリケーションは作成できますが、ソースコードの記述量が増えるため微妙です。

TS/JSでは、Electronがフレームワークとしては有名ですが、メモリ消費量が多いことと、QtのJSラッパーとして動作するNodeGuiに興味を惹かれたので、今回はNodeGuiを使用することとしました。


## 環境構築

今回は、MacOS上での開発を前提としますが、必要なパッケージがインストール可能であれば、WindowsやLinux上でも問題無く開発は可能だと思います。

参考までに、MacOS(Homebrew有)・Linux(Redhat系・Debian系)におけるCMake・Make・npm・yarnの導入までは記載しますが、ディストリビューション・バージョン差異やソースコードから導入をしたいといった際には、各プロジェクトの公式サイトをご参照ください。

### CMake・C++コンパイルツールのインストール

#### MacOS(Homebrew有)
```sh
# CMake・C++コンパイルツールのインストール
brew install cmake make
```

#### Linux(Debian系)
```sh
# CMake・C++のコンパイルツールのインストール
sudo apt-get install pkg-config build-essential
sudo apt-get install cmake make
sudo apt-get install mesa-common-dev libglu1-mesa-dev
```

#### Linux(Redhat系)
```sh
# CMake・C++のコンパイルツールのインストール
sudo dnf groupinstall "Development Tools" "Development Libraries"
sudo dnf groupinstall "C Development Tools and Libraries"
sudo dnf install cmake
sudo dnf install mesa-libGL mesa-libGL-devel
```

### Node.js・npmのインストール

#### MacOS(Homebrew有)
```sh
# Node.js・npmのインストール
brew install node
# Node.jsのバージョン確認
node -v
# npmのバージョン確認
npm -v
```

#### Linux(Debian系)
```sh
# Node.js・npmのインストール
sudo apt-get install nodejs npm
# Node.jsのバージョン確認
node -v
# npmのバージョン確認
npm -v
```

#### Linux(Redhat系)
```sh
# Node.js・npmのインストール
sudo dnf install nodejs npm
# Node.jsのバージョン確認
node -v
# npmのバージョン確認
npm -v
```

### yarnのインストール

#### 全OS共通
```sh
# yarnのインストール
npm i -g yarn
# yarnのバージョン確認
yarn -v
```


## Sampleプロジェクトを動かしてみる
ほとんど公式のドキュメントのままですが、手早く動作確認が取れますので、ここまでのインストールパッケージに不足が無いかの確認ができます。

```sh
# sampleプロジェクトのリポジトリクローン
git clone https://github.com/nodegui/nodegui-starter
# sampleプロジェクトルートへ移動
cd nodegui-starter
# sampleプロジェクトの依存パッケージのインストール
yarn install
# sampleプロジェクトの開始
yarn start
```


## 最後に
今回は、環境構築だけでしたが、次回からは実際のアプリケーションの作成に移ります。
