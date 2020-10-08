# GithubActions実践入門

各章の内容は`ch?`ディレクトリの中にまとめることとする。

# 参考

## [Github公式ドキュメント](https://docs.github.com/ja/free-pro-team@latest/actions)
## [GithubActions実践入門](https://www.amazon.co.jp/GitHub-Actions-%E5%AE%9F%E8%B7%B5%E5%85%A5%E9%96%80-%E6%8A%80%E8%A1%93%E3%81%AE%E6%B3%89%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA%EF%BC%88NextPublishing%EF%BC%89-%E5%AE%AE%E7%94%B0/dp/4844378716)
## [GithubActions実践入門リポジトリ](https://github.com/github-actions-up-and-running)


# 目次
- 第1章：Github Actionsの基礎知識
- 第2章：Github Actionsの機能解説
- 第3章：アクション
- 第4章：サンプルレシピ

# 各章メモ


<details>
<summary>第1章：Github Actionsの基礎知識</summary>


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

</details>


<details>
<summary>第2章：Github Actionsの機能解説</summary>


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


#### 秘密情報の取り扱い
- 外部サービスにアクセスするための認証情報などをワークフロー内部で使いたい場合、Settings -> Secretsに秘密情報を登録する。
- 最大100個登録可能で、秘密情報の名前にはアルファベットと_しか使えない。
- ログに表示される場合、自動でマスクされる仕組みにはなっているが極力自前でログに出力されないようコードを書いた方が良い
- JSONのような構造化データを秘密情報に登録した場合、上記のマスク機能が正常に働かない恐れがある
- 秘密情報は最大64KBまでしか保存できない
- ワークフロー内で秘密情報にアクセスする際は`secrets.(env_name)`


`GITHUB_TOKEN`：Github Appのアクセストークンと同様にリポジトリの権限が与えられているので、REST_APIの認証情報として使う。

権限の詳細についてはこちら(https://docs.github.com/en/free-pro-team@latest/rest/reference/permissions-required-for-github-apps)

#### イベントフィルタ

`.github/workflows/???.yml`の冒頭にある`on`にはイベントトリガーを指定するが、これに条件付けすることでフィルタリングが可能である。
条件づけの方法はいくつかある。

- ブランチ・タグ
    - push・pull_requestイベントでは特定のブランチ・タグで条件づけ可能
    - ブランチ・タグのどちらかしか指定しない場合、指定されていないイベントは全て除外される
    - 正規表現でブランチ・タグの指定ができる(例：release/で始まるブランチを指定)


- ファイルパス
    - パスが指定されている場合、ファイルに変更がない場合にはワークフローは実行されない


- 特定のアクティビティ（issue_comment -> created, edited, deleted）
    - 上記三種類の内一つを指定するなどができる

#### ジョブ間の依存関係
通常、jobsで定義したジョブは並列実行される。しかし、ジョブ1とあとにジョブ2を実行したいという場合がある。この場合、job_id下で`needs`を指定すれば、依存元を定義できる.

#### ジョブ間のアウトプットの受け渡し
`needs`でジョブ間の依存関係を定義すると、アウトプットの受け渡しもしたくなる。
この場合はjob_id下で`outputs`を定義する。`outputs`はsteps終了時に評価される。


#### マトリクスビルド
ワークフローの実行を特定のパラメータの組み合わせで繰り返し実行したい場合がある。例えば、OSとプログラミング言語のバージョンの組み合わせとか。その場合、jobs下で`strategy`を定義する。

例:osとnodeバージョンの組み合わせ
```
unit-test:
    name: Unit Test
    strategy:
        matrix:
          os: [ubuntu-latest, windows-latest, macos-latest]
          node: [10,12]
    runs-on: ${{ matrix.os }}
```

また、`strategy.matrix.include`を用いると、特定の組み合わせの時のみ新しい変数の追加・存在しない新しい組み合わせの追加が可能。
逆に`strategy.matrix.exclude`は特定の組み合わせを実行しないようにできる。

#### スケジュール実行

依存ライブラリの脆弱性チェックなどを定期的に行いたい場合、スケジュール実行が可能。
しかし、スケジュール実行はデフォルトブランチの最新コミットでのみ設定されることに注意。
スケジュール設定はcronで行う。

#### コンテキスト
ワークフローに与えられる実行時の情報をコンテキストという。jobuやstepsなどをさす。
詳しくは[こちら](https://docs.github.com/ja/free-pro-team@latest/actions/reference/context-and-expression-syntax-for-github-actions)

#### ジョブのコンテナ内実行
ワークフロー内のジョブはVM環境上で実行してきたが、ジョブをコンテナ内で実行することが可能。

詳しくは[こちら](https://docs.github.com/ja/free-pro-team@latest/actions/creating-actions/creating-a-docker-container-action)
```
jobs:
    unit0test:
        runs-on: ubuntu-latest
        container:
            image: node:12.14.1-streatch
            env:
                NODE_ENV: development
            ports:
                - 8080
            options: --cpus 1
        steps:
            .....
```


例えば、ワークフローの実行中に何かしらのサービスを立ち上げたい場合に使用する。例としてDBを立ち上げておいて、それを参照するなどである。コレをサービスコンテナと良いjobs直下で`services`を定義する。


### キャッシュ
CI/CDではリポジトリの依存するパッケージのダウンロードが理由でビルド時間が長くなることがある。コレは実行ごとにクリーンな環境を構築するため、ダウンロードファイルが持ち越されないことが理由である。

そこでCI/CDが用意するキャッシュ機能を用いて、ダウンロードパッケージの使い回すことでクリーンな環境を用意しつつダウンロードの重複を回避することが一般的である。コレはGithubActionsでも使用可能である。

### アーティファクト
CI/CDではビルド中に生成したファイルをビルド後に利用したい場合がある。例えば、アーカイブなどの成果物やログやテスト結果などである。アーティファクトという機能はコレら成果物を保存しておくための機能である。

アーティファクトを保存する際には`actions/upload-artifact`を使う。これは公式で提供されているアクションである。

```
- name: Upload test coverage
  uses: actions/upload-artifact@v2
  with:
    name: test-coverage-${{ matrix.os }}-${{ matrix.node }}
    path: coverage
```

`download-artifact`もある。一つ前のjobでuploadされたアーティファクトを以降のjobでdownloadすることがこれで可能になる。しかし、ワークフローをまたいでは実行できない。
また、保存期間は90日である。

### ログによるコマンド実行
ワークフロー実行中echoなどでログに特定のフォーマットの文字列を出力することで以下のことが可能。

- 環境変数の設定
- アウトプットの設定
- PATHの追加
- debug, warning, errorメッセージの追加
- ログのマスク
- コマンド実行の停止・再開

コマンド実行の雛形
```
$ echo "::<COMMAND> <PARAM1>=<VALUE1>, <PARAM2>=<VALUE2>::<COMMAND VALUE>"
```

例えば、環境変数の設定の場合
```
$ echo "::set-env name=TZ::Asia/Tokyo"
# echo "::<set-env> <name>=<TZ>::<Asia/Tokyo>"
```
これによりTZというパラメータにAsia/Tokyoが設定される。

以下に実行できるコマンドについて大雑把に整理する。

- 環境変数の設定:set-env
    - 設定したstepの次のstepからアクセス可能
- アウトプットの設定：set-output
- PATHの追加：add-path
    - 設定したstepの次のstepからアクセス可能
- デバッグメッセージの出力
    - debug  `$ echo "::debug::Debug Message"`
    - warning  `$ echo "::warning::Warning Message"`
    - error  `$ echo "::error::Error Message"`
    - 静的解析ツールでファイルに対しての警告メッセージをpull requestに仕込める
- ログのマスク：add-mask
    - `$echo "::add-mask::password"`
    - passwordで設定した値がログ上でマスクされる
- コマンド実行の停止・再開:stop-commands
    - `$echo "::stop-commands::stop"`
    - これまで紹介したコマンドを文字列として出力する
    - このコマンド実行後のログからコマンド処理がされなくなる

### ワークフローのデバック
秘密情報でACTIONS_STEP_DEBUG=trueを設定するとデバッグ情報が出力される 


### 権限
read権限があれば以下のことが可能
- Actionsタブの閲覧
- forkしてプルリクエストイベントでワークフローの実行

以下のことはできない
- 秘密情報のアクセス、閲覧

### 通知
settings-> Nortificationsから設定可能。


### ランナーのセルフホスティング

通常のワークフローではGithubの提供するVM環境で実行される。しかし、セルフホストランナーを使うことで、カスタムしたVM環境でワークフローを実行できる。
つまりは次のようなケースで使うことがある。

- CPUやメモリを激しく消費するジョブを実行したい
- プライベートネットワーク内でジョブを実行したい
- Githubが提供していないOS上でジョブを実行したい

詳しくは[こちら](https://docs.github.com/ja/free-pro-team@latest/actions/hosting-your-own-runners/about-self-hosted-runners)

Githubの提供する環境とセルフホストランナーの違いはいくつかある

- Github提供
    - 環境の自動更新
    - 運用コストがかからない
    - ジョブ実行ごとにクリーンになる
    - プライベートリポジトリだと料金がかかる
- セルフホストランナー
    - 既存のマシンやクラウドサービスのリソースを使える
    - 環境をカスタマイズ（ハードウェア、OS、セキュリティ、ソフトウェア）
    - ジョブ実行間で環境持ち越し
    - プライベートリポジトリでも無料
    - 運用コストがかかる


### REST API
REST APIを使うと以下のことが可能。

- アーティファクトの取得、削除
- 秘密情報の設定、削除
- セルフホストランナーのステータス取得
- etc

詳しくは[こちら](https://docs.github.com/en/free-pro-team@latest/rest/reference/actions)


</details>


<details>
<summary>第3章：アクション</summary>

### アクションとは？
GithubActionsの用語で、アクションという単位で実行可能なタスクを作成できる。
アクションには現状二種類存在する。

- javascript：Linux, Windows, macOSで利用可能
- Docker: Linuxで利用可能

公開するアクションの管理には専用のリポジトリを作成することが推奨されている。
非公開のアクションの管理にはそのアクションを利用するリポジトリの`.github/actions`に保存することが推奨されている。

またアクションの制御として使用できるアクションの制限が行える。利用するアクションをorganization内に限定する場合、公式のcheckout等も使えなくなるため、その場合にはforkしておく必要がある。

### README.mdについて
公開するアクションを保存するリポジトリのREADME.mdには次の情報を含めることが推奨されている。

- アクションが何をするかの詳細な説明
- inputsとoutputsについての説明
- アクションが使用する秘密情報・環境変数の説明
- アクションのワークフロー内での使用例

### 公式テンプレート

Githubが提供するアクション作成のテンプレートがある。

- actions/javascriptaction https://github.com/actions/javascript-action
- actions/typescript-action https://github.com/actions/typescript-action
- actions/container-action https://github.com/actions/container-action

</details>

<details>
<summary>第4章：サンプルレシピ</summary>
 
 サンプルレシピの実装例については[こちら](https://github.com/github-actions-up-and-running)
 
- ジョブ失敗時にSlack通知
- ワークフロー実行環境にSSHしてデバッグ
- DockerイメージをGithubパッケージで公開
- reviewdogでLintの結果をプルリクエストに表示
- Terraform Github ActionsでAWSにデプロイ

</details>
