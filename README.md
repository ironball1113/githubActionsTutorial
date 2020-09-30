# GithubActions入門

## 目次
- 第1章：Github Actionsの基礎知識
- 第2章：Github Actionsの機能解説
- 第3章：アクション
- 第4章：サンプルレシピ


## 第1章：Github Actionsの基礎知識

- Git:ソースコードのバージョンを管理するツール
- Github: Gitを利用した、開発者を支援するWebサービス
- Github Actions: Github組み込みのCI/CDツール

Githubにコードをpushすると、その変更に応じてビルド、テスト、デプロイと言った一連の流れを自動実行してくれる。
しかし、基本は様々なイベントにフックして、処理の自動実行をしてくれるという物なのでCI/CDに止まらない。


支払いや制限周りは公式ドキュメント参照　https://docs.github.com/ja/free-pro-team@latest/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-actions


#### Hello world!
.github/workflows/hello.yml

- pushされるたびにhello worldが実行される
- commit名がワークフローの名前としてActionsページに表示される


## 第2章：Github Actionsの機能解説

### ワークフローとは？
ワークフローとは、ソフトウェア開発における何かしらの処理を自動実行する物。
- 構成要素:
    - ジョブ（タスク）：ある程度まとまった処理のこと（前処理、学習、テストなど）
    - ステップ：ジョブの中の処理のこと(API呼び出し、コマンド実行など)
    - アクション：公開・共有されている処理のこと
- ワークフロー例:
    - pushをトリガーにビルド、テスト（テスト結果をslack通知）、デプロイまで行う
    - ストレージに新たなデータの格納を検知、前処理、学習、学習結果の保存まで行う
    - 毎朝6時にあるブログサイトの新着でキーワードに引っかかったものをslack通知する
- 用語メモ
    - ビルド：実行可能ファイルや配布パッケージを作成する処理や操作のこと[[e-words](http://e-words.jp/w/%E3%83%93%E3%83%AB%E3%83%89.html#:~:text=%E3%83%93%E3%83%AB%E3%83%89%E3%81%A8%E3%81%AF%E3%80%81%E7%B5%84%E3%81%BF%E7%AB%8B%E3%81%A6%E3%82%8B%E3%80%81%E5%BB%BA%E3%81%A6%E3%82%8B,%E3%82%92%E6%8C%87%E3%81%99%E5%A0%B4%E5%90%88%E3%82%82%E3%81%82%E3%82%8B%E3%80%82)]
    - デプロイ：実行可能ファイルなどを運用環境に配置・展開して実用に供すること[[e-words](http://e-words.jp/w/%E3%83%87%E3%83%97%E3%83%AD%E3%82%A4.html)]

### ワークフローの保存場所は？
- リポジトリのルートにある`.github/workflows`直下にワークフローは保存される。
- ファイル形式は`yml`か`yaml`
- 名前は自由だが、役割の分かるように命名した方が良い。（workflows以下にディレクトリ分割が可能かは不明）

### ワークフローが実行される仮想環境は？

使用できる仮想環境は以下の二つ
- Github提供のVM(仮想マシン)環境
- ユーザー定義の環境

Githubが提供するVm環境はジョブの実行ごとに毎回用意される。そのため、ジョブ内であれば情報のやりとりは容易だが、ジョブ間で情報のやりとりを行う場合は後述するアーティファクトを使う必要がある。

以下に提供されるOSについて簡単に記載。
- Linux, Windows: Azure Standard_DS2_v2相当
    - Windows Server 2019
    - Ubuntu 20.04、Ubuntu 18.04、Ubuntu 16.04
- macOS: MacStudium(https://www.macstadium.com/)
    - macOS Catalina 10.15
管理者権限を必要とする操作も基本的に実行可能。詳細は公式doc(https://docs.github.com/en/free-pro-team@latest/actions/reference/specifications-for-github-hosted-runners)


また、Ubuntu 18.04で[既にインストールされているソフトウェア](https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu1804-README.md)を見てみる。各項目一部抜粋である。

- 言語
    - Python: 
    - Node 12.18.4
    - Python 2.7.17
    - Python3 3.6.9
    - Ruby 2.5.1p57
    - Swift 5.3
    - Julia 1.5.1

- パッケージマネージャー
    - Homebrew 2.5.1
    - Gem 3.1.4
    - Miniconda 4.8.3
    - Npm 6.14.8
    - Yarn
    - Pip 9.0.1
    - Pip3 9.0.1

....と続く。