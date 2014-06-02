# Mroonga for Windows builder

Windows向けのMroonga（MariaDBに同梱）パッケージを作成するスクリプトです。

## Requirements

* Ruby 1.9 or later
* Visual C++ 2010 Professional or 2013 Express
* CMake 2.8.11
* Bison

## Usage

### 1. バージョンの確認

Rakefileの先頭に記述してある以下のバージョンを確認し、必要に応じて修正してください。

  * MariaDB
  * Mroonga
  * Visual C++

### 2. 実行

以下のコマンドを順番に実行してください。

```bat
> gem install bundler   # 初回のみ
> bundle install        # 初回のみ
> rake download         # groonga.orgからソースファイルをダウンロード
> rake build:all        # ビルド実行
> rake enable_mroonga   # Mroongaをデフォルトで有効にする（ZIP用）
> rake rename           # リリース用の名前にリネーム（MSI用）
> set GITHUB_TOKEN=xxx  # 環境変数にGitHubトークンを設定
> rake upload           # GitHubのリリースページにアップロード
```

## TODO

* zipファイルの動作確認

## FAQ

* Archive::Zip.extractでErrno::ENOENTになるんだけど？
  * パス名が長すぎるためです。`C:\work\mrn`等で作業すれば成功すると思います。
* CMakeのバージョンは2.8.12じゃだめ？
  * PDBまわりのバグがあるのでだめです。CMake 3.0で直るらしいです。
    * http://www.cmake.org/Bug/bug_relationship_graph.php?bug_id=14600&graph=dependency&orientation=vertical
