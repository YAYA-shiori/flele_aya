# GHOST.md

「FLELE」に固有の情報です。AI エージェントは、作業の前に読みます。
作者が自由に書き換えるファイルです。

## このゴーストについて

- 伺か（ukagaka）のゴースト「FLELE」。トークはせず、音楽ファイルを演奏する**サウンドプレイヤー**として動く。SHIORI は YAYA（`yaya.dll`）。
- 原作は FLELE（元の配布サイトは終了）。配布物の再配布・改変配布が許されていたので、YAYA-shiori project が引き取って YAYA に移したもの。
- 配布元: https://github.com/YAYA-shiori/flele_aya （nar は Releases から）
- SSP の「現在のフルセット」で動かす前提。演奏は SSP 本体の `\![sound,...]` タグと `currentghost.sound.*` プロパティで行い、曲のタグは SAORI over DirectSSTP の `internal_file` / `MediaInfo` で読む。

## ライセンス・著作

- 辞書など原作の部分は、原作の再配布・改変配布許可に従う（`README.md`、`thanks.txt` 参照）。
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
| `lang.dic` | 表示言語の切り替え（`OnLanguageChange`）と、文言表の引き方（`文言` / `文言n`） |
| `lang_ja.dic` / `lang_en.dic` | 言語ごとの文言表（`□文言表_ja` / `□文言表_en`）。利用者に見える文字列はすべてここに置く |

## 書き方の決まり

- 関数は `名前` の次の行に `{` を書く。`if` などの `{` は同じ行に置く。中身は 1 行にまとめず、タブでインデントする。
- 名前から働きが分からない関数には、直前にコメントを書く（`//---` で囲む）。
- 外部プログラムや SAORI の戻り値を使わないときは `void SAORI(...)` とする（戻り値がスクリプトに混ざらない）。
- 変数は、`○` で始まるものが保存される大域変数。実行中だけ使うものは `□終了前` で `ERASEVAR` する。1つの関数の中だけで使うものは `_` で始まる局所変数にする。

## 表示言語（日本語・英語）

- SSPが通知する言語（`OnLanguageChange`。起動直後と、言語リソースを切り替えたとき）に合わせて、応える言語を切り替える。言語IDで判定し、内蔵の日本語は言語IDが 0 で届くので、そのときだけ言語名（Japanese）で判定する。日本語以外で、文言表の無い言語は英語にする。選んだ言語は `○言語`（`ja` / `en`）に保存する。
- 利用者に見える文字列（メニュー・トーク・ツールチップ・ボタンの表示・ダイアログのタイトル）は、辞書に直接書かず、文言表に置いて `文言('キー')` で引く。個数で言い方が変わるもの（1曲 / 2曲）は `文言n('キー', 個数, ...)`。
  - 文言の中の `{0}` `{1}` は引数、`{fcl}` `{fcc}` はシェルに合わせた強調色。差し込む値（ファイル名など）は置き換えるだけで解釈しないので、`%` や `{` を含んでいてもよい。
  - `文言n` は、個数が 1 のとき「キー.1」があればそちらを使う。表に無いキーは、現在の言語 → `en` → `ja` の順に探す。
  - 文言表は ○文言（ハッシュ）に読み込み、ゴーストの起動のたびに作り直して、終了時に消す。
  - 文言表の中の文字列は `"..."` で書く（`%(` を書かない。展開されるので、差し込む値は `{0}` で受ける）。
- 選択肢のID・イベント名・関数名は言語によらず日本語のまま（`\q[表示,ID]` の表示だけを訳す）。Weirds の曲ごとの項目は、表示名（`ASC(4)`）とイベント名の末尾（`ASC(5)`）を別に埋める。
- 言語を足すには、`lang_en.dic` を写して `lang_言語コード.dic`（`□文言表_言語コード`）を作って訳す。SSPの言語IDとの対応が `□言語IDの言語コード` に無い言語は、そこに足す。`lint.ps1` は `□文言表_*` を「未使用」と報告するが、`EVAL` で呼ぶので問題ない。
- 日本語と英語で同じキーが揃っているかは、`文言表` の `_t['…']` のキーを両方から拾って突き合わせる。

## SAORI

| ファイル | 用途 |
|---|---|
| `saori/Weirds.dll` | 曲リストメニュー（右クリックで出るメニュー） |
| `saori/bln.dll` | 曲名ウィンドウ |
| `saori/hotkey.dll` | ゴーストが非アクティブなときのホットキー |

## 配布（GitHub Actions）

- ネットワーク更新: `https://yaya-shiori.github.io/flele_aya/`（GitHub Pages）。`On_homeurl` は `ghost/master/dic/normal/core.dic` に書く。
- 配布物は `.github/workflows/auto_deploy.yml` が作る。辞書チェック（`auto check`）が通った `main` / `master` の push ごとに、`.github/scripts/build-site.ps1` が nar（`.narignore` で除外）と更新ファイル一式（`.updateignore` で除外）を作って Pages に出し、nar を Releases の `latest` に置き直す。**push するとリリースが公開される**ので、`main` への push は作者の確認を取ってから行う。
- 手元でも `pwsh ./.github/scripts/build-site.ps1 -OutDir site`（`-ListOnly` で一覧だけ）で同じものを作れる。配布物は `git ls-files --recurse-submodules` で数える。`dic/system` は submodule だが、nar にも更新にも普通のファイルとして入る。リポジトリの Settings > Pages > Source は「GitHub Actions」にしておく。
- `updates2.dau` と `updates.txt` は作らず、コミットもしない（`.gitignore` で除外してある）。`delete.txt` は手で書く。配布しているファイル（フォルダを含む。例: `ghost/master/dic/system`）は書かない。
- nar の固定の URL は `https://github.com/YAYA-shiori/flele_aya/releases/latest/download/flele_aya.nar` と `.../flele_balloon2.nar`。
- `flele_balloon/` は同梱バルーン（`install.txt` の `balloon.directory`）。`flele_balloon2/` は単独で配るバルーンで、独自の `install.txt` を持つ。どちらもゴーストの nar と更新には入れず（`.narignore`、`.updateignore`）、`descript.txt` の `homeurl`（`https://yaya-shiori.github.io/flele_aya/flele_balloon/`、`.../flele_balloon2/`）から別に更新される。更新ファイルは `build-site.ps1` が作る。`flele_balloon2` は `build-site.ps1` の `-StandaloneBalloons` に書いてあり、独自の nar（`flele_balloon2.nar`）も作る。
- `cmd_flele.nar` `cmd_to_flele.nar` は caramelldansen 用のおまけ（リポジトリの raw から配る。nar と更新には入れない）。
- 旧 URL（`https://raw.githubusercontent.com/YAYA-shiori/flele_aya/main/`）から更新する利用者を新しい URL へ移すため、ルートと `flele_balloon/`、`flele_balloon2/` に固定の `updates2.dau` をコミットしてある（`.gitignore` の対象なので `git add -f` で追加した）。
    - 載せているのは、新しい homeurl を運ぶファイル（ルートは `core.dic`、バルーンは `descript.txt`）と、ルートの `delete.txt` だけ。
    - これらは作り直さない。新しい URL からは `build-site.ps1` が作る更新ファイルが配られる。
