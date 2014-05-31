# Mroonga for Windows builder

Windows向けのMroonga（MariaDBに同梱）パッケージを作成するスクリプトです。

## Requirements

* Ruby 1.9 or later
* Visual C++ 2010 Professional or 2013 Express
* CMake 2.8.11
* Bison

## Usage

以下のコマンドを順番に実行してください。

```bash
> bundle install
> rake download   # groonga.orgからソースファイルをダウンロード
> rake build:all  # ビルド実行
> rake rename     # リリース用の名前にリネーム
>  GITHUB_TOKEN=xxx rake upload  # GitHubのリリースページにアップロード
```

## TODO

* **Windowsで動作確認する**
* Mroongaをデフォルトで有効にする仕組みを入れる
  * zipを展開
  * 中のmysqldを起動
  * mysqlで中にあるshare/mroonga/install.sqlを実行
  * mysqldを終了
  * zipで圧縮
    * （欲を言うと、ここで、zipのファイル名と展開したときにできるディレクトリー名を同じにしたい）

## FAQ

* Archive::Zip.extractでErrno::ENOENTになるんだけど？
  * パス名が長すぎるためです。`C:\work\mrn`等で作業すれば成功すると思います。
* CMakeのバージョンは2.8.12じゃだめ？
  * PDBまわりのバグがあるのでだめです。CMake 3.0で直るらしいです。
    * http://www.cmake.org/Bug/bug_relationship_graph.php?bug_id=14600&graph=dependency&orientation=vertical
