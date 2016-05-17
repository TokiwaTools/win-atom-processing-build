
# WindowsでAtomのパッケージ｢processing｣が動かない問題を解決する

超参考: [Atom + Processingの快適な環境をWindows 10でゲットするまで](http://caughtacold.hatenablog.com/entry/2016/03/23/173433)

---

## 経緯
+ まさしく上記のページ通りのバグに遭遇した
+ Sublime TextからAtomに移行してProcessingをエディタ上でビルドしたい
+ 基本的には参考サイト通りに進めていけば解決するがここでは補足をするので、
参考サイトを見ながら参考にしてください

## Processing側の設定
+ ここでは｢processing-java｣というProcessingをコマンドライン上でビルドするためのツールがある場所に｢パスを通す｣という作業をする
	+ 実はこの後に導入するprocessingというパッケージの設定上でパスを通せば動くのだが、今回は参考サイト通りの手順を踏む
+ 図2番目、PATHの値をいじるところでは単に一番最後に｢;｣を追加してからProcessingのあるフォルダのパスを挿入すればよい

## Atom側の設定、楽勝かと思いきや……
+ ここで[Atom](https://atom.io/)を導入する
+ Atomにはパッケージという拡張機能的なものがある

### パッケージ導入方法
+ Atomを開いて Ctrl+,(カンマ) で設定画面を開く
+ 左カラムの Install から図3番目通り以下のパッケージを検索してインストール
 (似た名前のパッケージが多いので注意) 
 
	+ **processing**
		+ ビルドする 今回バグを直すパッケージ
	+ **processing-autocomplete**
		+ 補完機能
	+ **processing-language**
		+ 文法ハイライト (メソッド名などに色が付く)

+ 下2つのパッケージに関しては今回説明は省略する
Atomでprocessingを使うための3点セットと思ってもらってよいと思う
+ この状態で適当なスケッチを開いて Ctrl+Alt+B で実行しようとすると**最初からエラーがでるかもう一度実行するとエラーがでる**と思う
+ エラーメッセージと共に消しても消しても右からコンソールがでてくるが、実は実行自体は成功している (実際にウィンドウが出る)

## トラブルの張本人、ps-treeをいじってみる。
+ ここから実際にエラーを解決するのだが、この章の最後、｢実はwmicというコマンドが何故か通らず…｣の部分が問題で私の環境ではこれが原因で動かなかった
+ ここの問題解決を具体的に補足する

1. Ctrl+,で設定画面を開く
2. 左カラムの Open Config Folder
3. 左のツリーから 以下のファイルを開く
	
		.atom\packages\processing\node_modules\ps-tree\index.js

4. 書き換える

		es.connect(
			spawn('ps', ['-A', '-o', 'ppid,pid,stat,comm']).stdout,	//ここを
			es.split(),
			es.map(function (line, cb) { //this could parse alot of unix command output
			var columns = line.trim().split(/\s+/);
			if (!headers) {
			headers = columns;
				return cb();
			}

  ↑から↓に書き換え

		es.connect(
			spawn('C:/Windows/System32/wbem/WMIC.exe', ['PROCESS GET Name','ProcessId,ParentProcessId']).stdout,	//こう書き換える
			es.split(),
			es.map(function (line, cb) { //this could parse alot of unix command output
			var columns = line.trim().split(/\s+/);
			if (!headers) {
			headers = columns;
				return cb();
			}

お疲れ様でした
