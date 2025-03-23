# セットアップ(開発者用は下にあり)


## 環境構築



# 開発者用 環境構築
使用ツール
- pythonバージョン,ライブラリ管理：uv
- タスクランナー：Task

1. uvのセットアップ
2. taskのセットアップ
3. pre-commitの登録

## uvを用いて仮想環境を作成
pythonのバージョンとライブラリの依存関係を管理できるツール
今までpip install で行っていたことを uv add で行う
pipと違って、uvだけでpythonのバージョンも固定できるのと、uv.lockファイルで依存関係までのバージョンを厳密に管理できる

### セットアップ
**インストール**
```
# 参照：https://docs.astral.sh/uv/getting-started/installation/ 

# macOS/Linuxなら以下を実行 (brewを使ってもいい)
curl -LsSf https://astral.sh/uv/install.sh | sh

# 存在確認
uv --version
> uv 0.6.9 (3d9460278 2025-03-20)
```
**環境構築**
```
# uv.lockがあるディレクトリで以下を実行
uv sync
> (champion-crm) tomoyamiyake@tomoyamiyakenoMacBook-Air-2 champion-crm % こんな感じのchampion-crm環境が立ち上がる

# もしdev環境がいらないときは
uv sync --no-dev
```

### 開発時使用方法
**パッケージの追加、削除**
```
# numpyを追加したい時
# バージョンまで指定したいときは numpy==1.22.3 など
uv add numpy

# 開発環境にのみ追加したい時は --devオプションをつけるとそっちだけ追加
uv add pytest --dev

# パッケージの削除
uv remove numpy
```

**uv環境を使ったコードの実行**
```
# uv runの後にいつものコマンド
uv run ruff .
uv run python main.py
```


## go-Task
上のコマンドを打たずとも簡単に一連の流れをまとめて定義できるタスクランナー

### セットアップ
**インストール**
```
# 参照：https://taskfile.dev/installation/

# macOS/Linuxなら以下を実行
brew install go-task

# 存在確認
task --version
> Task version: 3.42.1 (Homebrew)
```

### 開発時使用方法
**タスク定義**
Taskfile.ymlのtasks:の下に記述
```
tasks:
  format:
    desc: コードをフォーマット
    cmds:
      - uv run ruff format src tests

  # 以下このように追加していく
  task2:
    desc: ..
    cmds: ..
```

**task実行**
```
# formatterを実行
task format

# linterを実行
task lint
```

## コードディング規約 :star pushする前にこれを整えてpush
以下を使用
- ruff : linter, formatter
- mypy : 型チェック

### 手動でチェックを行う方法
```
task format   # formatを修正
task lint-fix # lintを行ってその後自動修正
# もしチェックだけしたいなら task lint-check
```

### commitした時に自動でチェックを行う方法
**セットアップ**
これを実行すれば今後commit時に自動で linter formatterが走ってチェック
```
# uv syncでuv環境が立ち上がっていることが前提
pre-commit install
```

### vscodeで規約を反映する
以下のVSCode拡張機能をダウンロード
- Ruff
- Mypy Type Checker

これで、上書き保存した時に自動でフォーマットをかけるようになる
嫌なら .vscode/settings.json を編集
mypyとlintはできていないとエラー表示が起きるようになる

**使い方**
```
git add hoge.py
git commit -m "commit1"

# ルールに沿っていなければここでエラーが生じてcommitできない
# 修正できるものに関してはこの実行で勝手に修正してくれる (その変更をaddする必要あり)
# 自動修正できないものは自力で治す

# もしエラーがあって修正した場合再度 add commit
git add hoge.py
git commit -m "commit1"
> これでcommitが通る

# 色々な都合でとにかくcommitしたい時 --no-verifyオプションで無効化
# ⭐️actionsで結局エラーが起きるのでマージする際にはチェック必須
git commit -m "無理やりcommit" --no-verify
```

## CI
pushした時にactionsを使ってチェックを行う
actionsの挙動は.github/workflow/~.ymlに定義

実行条件
- push時
- pull request作成時

行うこと
1. lintの確認(ruff)
2. 型チェック(mypy)
3. test実行(pytest)

注意点
- actionsの実行環境はubuntuなので普段の開発環境とそこだけ違うことに注意

# API document
document/openapi.yamlを変更した場合、document/api-document.htmlを最新版にしてpushする。
```
# redoc-cliをインストール, 初めだけ
npm install -g redoc-cli

# htmlファイルを作成 -> その後最新版をpush
npx @redocly/cli build-docs document/openapi.yaml -o document/api-document.html
```

## ドキュメント参照の仕方
htmlを開いて見る(api-document.htmlがあることが前提)
```
open api-document.html
```
