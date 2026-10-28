# GHOST.md

「FLELE」に固有の情報です。AI エージェントは、作業の前に読みます。
作者が自由に書き換えるファイルです。

## このゴーストについて

- 伺か（ukagaka）のゴースト「FLELE」。トークはせず、音楽ファイルを演奏する**サウンドプレイヤー**として動く。SHIORI は YAYA（`yaya.dll`）。
- 原作は FLELE（元の配布サイトは終了）。配布物の再配布・改変配布が許されていたので、YAYA-shiori project が引き取って YAYA に移したもの。
- 配布元: https://github.com/YAYA-shiori/flele_aya （nar は Releases から）
- SSP の「現在のフルセット」で動かす前提。演奏は SSP 本体の `\![sound,...]` タグと `currentghost.sound.*` プロパティで行い、曲のタグは SAORI over DirectSSTP の `internal_file` / `MediaInfo` で読む。

## ライセンス・著作

- 辞書など原作の部分は、原作の再配布・改変配布許可に従う（`readme.txt`、`readme2.txt`、`thanks.txt` 参照）。
- `shell/` の追加シェルは、それぞれの作者の作品。シェルごとの `readme.txt` / `install.txt` に従い、画像は編集しない。
- `ghost/master/dic/system/` は yaya-dic（サブモジュール）。

## 辞書の構成

- `ghost/master/yaya.txt` が `dic/system`（システム辞書）と `dic/normal`（このゴーストの辞書）を読む。`dic/normal/` に新しい `.dic` を置くと自動で読み込まれる。
- `ghost/master/dic/system/` は yaya-dic の submodule。**編集しない**（変更が必要なら上流に提案する）。更新は `git submodule update --remote` か、スキルの `update-shiori`。
- 関数名の `□` で始まるものは、このゴースト内部の補助関数や、画面（メニュー）を作る関数。`On_` で始まるものは SSP の `\![raise]` や選択肢の ID から呼ばれる入口。選択肢（`\q[表示,ID]`）の ID は関数名なので、関数名を変えるときは `\q[...]` 側も直す。
- `reference0` ではなく `reference[0]`、`res_reference0` ではなく `res_reference[0]` と書く（`reference0` への代入は `reference[0]` に届かない）。

| ファイル（`ghost/master/dic/normal/`） | 主な中身 |
|---|---|
| `core.dic` | 選択肢・アンカーの選択（`OnChoiceSelect`）、ウィンドウ情報、ゴースト間通信（`OnCommunicate`）、SSP が問い合わせるリソース（ボタンの文字、リンク、`On_homeurl`） |
| `boot_end.dic` | 初回起動、起動時の変数初期化（`□起動`）、終了時の後始末（`□終了前`） |
| `shell_balloon.dic` | シェル・バルーン・着せ替え・最小化・サーフェスの変化、`OnTranslate`、タスクトレイのアイコン、シェル別の文字色・メニュー背景（`fcl` `fcc` `mffc` `mfbc` `menu_bg` など）。アイコンの `\_b[img\icon_名前.png,inline]` と、文字色を戻す `\f[color,default]` は関数にせず直接書く |
| `input.dic` | クリック・ダブルクリックでのボタン操作、なでの反応、The Hand、キー入力とホットキー（`hotkey.dll`） |
| `keyconfig.dic` | ショートカットキーを変えるメニュー |
| `menu.dic` | ヘルプメニューのトップ画面、演奏中の曲情報、設定画面（動作設定・再生方法） |
| `song_add.dic` | 曲の手動追加（ダイアログ・パス入力）、`OnUserInput`、ファイル・フォルダ・m3u のドロップ、URL からの曲のダウンロード |
| `weirds.dic` | 曲リストメニュー（`Weirds.dll`）、曲名ウィンドウ（`bln.dll`） |
| `player.dic` | 演奏（再生・一時停止・停止・次の曲の選び方・再生エラー） |
| `songlist.dic` | 曲リストの管理（曲数、絶対パス、タグ、クリア、ダウンロード曲の削除待ち）、曲リストの m3u 出力 |
| `network_event.dic` | 時刻合わせ・メール・ヘッドライン・RSS・インストール・ネットワーク更新のイベント |

## 書き方の決まり

- 関数は `名前` の次の行に `{` を書く。`if` などの `{` は同じ行に置く。中身は 1 行にまとめず、タブでインデントする。
- 名前から働きが分からない関数には、直前にコメントを書く（`//---` で囲む）。
- 外部プログラムや SAORI の戻り値を使わないときは `void SAORI(...)` とする（戻り値がスクリプトに混ざらない）。
- 変数は、`○` で始まるものが保存される大域変数。実行中だけ使うものは `□終了前` で `ERASEVAR` する。1つの関数の中だけで使うものは `_` で始まる局所変数にする。

## SAORI

| ファイル | 用途 |
|---|---|
| `saori/Weirds.dll` | 曲リストメニュー（右クリックで出るメニュー） |
| `saori/bln.dll` | 曲名ウィンドウ |
| `saori/hotkey.dll` | ゴーストが非アクティブなときのホットキー |

## 配布（GitHub Actions）

- `.github/workflows/release.yml` が `main` への push のたびに `updates2.dau` / `delete.txt` を作り直し、変更があれば nar を Releases に出す。**push するとリリースが公開される**ので、`main` への push は作者の確認を取ってから行う。
- `updates2.dau` と `delete.txt` は CI が作るので、手で編集しない。
- `release_ghost.py` は `git ls-files --recurse-submodules` で配布物を数える。`dic/system` は submodule だが nar には普通のファイルとして入る。
- 配布しないファイルは `.narignore`、ネットワーク更新に含めないファイルは `.updateignore`。
- `flele_balloon/` `flele_balloon2/` は同梱バルーン。`cmd_flele.nar` `cmd_to_flele.nar` は caramelldansen 用のおまけ。
