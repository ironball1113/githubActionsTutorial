# GithubActions実践入門

各章の内容は`ch?`ディレクトリの中にまとめることとする。

公式ドキュメント https://docs.github.com/ja/free-pro-team@latest/actions

GithubActions実践入門 https://www.amazon.co.jp/GitHub-Actions-%E5%AE%9F%E8%B7%B5%E5%85%A5%E9%96%80-%E6%8A%80%E8%A1%93%E3%81%AE%E6%B3%89%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA%EF%BC%88NextPublishing%EF%BC%89-%E5%AE%AE%E7%94%B0/dp/4844378716

GithubActions実践入門リポジトリ https://github.com/github-actions-up-and-running


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

### ワークフロー設定ファイルの構文は？
大雑把に整理すると、以下のように書く。

```
name: ワークフロー名
on: トリガーイベント

jobs:
    ジョブ名(パイプケース):
        name: ジョブ名(先頭大文字 単語間空白)
        runs-on: VM環境
        steps:
            - name: ステップ名
              uses: アクション名
              .
              .
              .
              .
```

メモ
- `.github/workflows/???.yml`の変更プルリクエストを作成すると自動判定してくれる

- on
    - トリガーイベントは複数指定可能
- env
    - 環境変数の設定はワークフローのトップレベルで行える
    - 環境変数名をキー、バリューが値 ex) DEBUG: true
    - jobs直下で定義することでそのjobs配下でのみ使用可能な環境変数を設定可能
    - stepsについても同様
    - 非推奨：`GITHUB_`で始まる環境変数名の使用（予約語の可能性が高い）
    - 推奨： ディレクトリなどのパスを環境変数にする場合、末尾に`_PATH`をつける
- jobs
    - job_id
        - jobs直下のジョブ名はjob_idであり、重複不可。先頭に使えるのはアルファベットor_であり、使える文字列はアルファベット・数字・-・_の四種類
        - job_id以下で定義されるstepsが実際に実行される。stepsは異なるプロセスで実行されるため、変更が後のstepsには反映されない
    - steps
        - uses
            - 予め用意されているアクションの実行を定義する
            - アクションの定義ではパブリックリポジトリを指定する（コミット、タグ、ブランチで指定可能）
            - おすすめはコミットSHAでの指定。後方互換性を壊さずに済むが、可読性の面で補強が必要（コメントするとか）
        - with
            - アクションのパラメータを定義する
        - entrypoint
            - Dockerコンテナアクションを実行できる
        - args
            - Dockerに渡す引数を定義する
        - run
            - コマンドラインの実行を定義する
            - パイプ|をつけることで複数行定義可能
    - shell
        - shellを指定可能
        - bash、pwsh、pythonは全てのOSで実行可能
        - shはLinux・macOSで、cmd・powershellはwindows環境で使用可能
    - timeout-minutes
        - job_id直下またはsteps直下で定義可能
        - タイムアウトのデフォルトは360minutes
    - counter-on-error
        - そのジョブ、ステップが失敗してもワークフローを継続するかを定める
        - job_id直下またはsteps直下で定義可能
        - デフォルトでfalse（trueにすると失敗してもワークフロー継続）


### 秘密情報の取り扱い
- 外部サービスにアクセスするための認証情報などをワークフロー内部で使いたい場合、Settings -> Secretsに秘密情報を登録する。
- 最大100個登録可能で、秘密情報の名前にはアルファベットと_しか使えない。
- ログに表示される場合、自動でマスクされる仕組みにはなっているが極力自前でログに出力されないようコードを書いた方が良い
- JSONのような構造化データを秘密情報に登録した場合、上記のマスク機能が正常に働かない恐れがある
- 秘密情報は最大64KBまでしか保存できない
- ワークフロー内で秘密情報にアクセスする際は`secrets.(env_name)`


`GITHUB_TOKEN`：Github Appのアクセストークンと同様にリポジトリの権限が与えられているので、REST_APIの認証情報として使う。

権限の詳細についてはこちら(https://docs.github.com/en/free-pro-team@latest/rest/reference/permissions-required-for-github-apps)

### イベントフィルタ

`.github/workflows/???.yml`の冒頭にある`on`にはイベントトリガーを指定するが、これに条件付けすることでフィルタリングが可能である。
条件づけの方法はいくつかある。

- ブランチ・タグ
    - push・pull_requestイベントでは特定のブランチ・タグで条件づけ可能
    - ブランチ・タグのどちらかしか指定しない場合、指定されていないイベントは全て除外される
    - 正規表現でブランチ・タグの指定ができる(例：release/で始まるブランチを指定)

例：マスターにpushされた時のみWF実行
```
on:
  push:
    branches:
      - master
```

- ファイルパス
    - パスが指定されている場合、ファイルに変更がない場合にはワークフローは実行されない


例：任意のJSファイルが変更された時のみ実行する
```
on: 
  push:
    paths:
      - '**.js'
```