コンテキストモードの代わりに自動セッション管理を使用する例題です。

__はじめに__

__自動セッション管理__を使用すれば，従来のコンテキストモードと同じように，Webアプリケーションの__コンテキスト__と4Dの__プロセス__』を関連付けることができます。ただし，プログラミングのスタイルや，使用するコマンドなどは違います。コンテキストモードから自動セッション管理に移行するためには，自動セッション管理に関する基本的な理解をしっかりと習得することが必要です。

__概要__

自動セッション管理は，__v13以降__，使用することができます。以前のバージョンでも，__HTTPクッキー__を自分で管理し，Webコンテキストを再現するために必要な__プロセス変数__や__セレクション__をレコードなどに保存すれば，Webセッションを管理することができました。自動セッション管理は，その名称が示すように，__HTTPクッキーの送受信__・Webコンテキストと4Dプロセスの関連付けなどを4Dが自動的に管理するというものです。さらに，パフォーマンスを向上させるために待機させておいた__プロセスを再利用__したり，そのような__プロセスが消滅する前__にコンテキストをデータベースに保存するイベントなども提供されています。自動セッション管理を活用すれば，とても簡単にWebセッションを管理することができます。

__セットアップ__

自動セッション管理は[データベース設定](http://doc.4d.com/4Dv13/4D/13.5/Web-Sessions-Management.300-1457382.ja.html)または[WEB SET OPTION](http://doc.4d.com/4Dv13/4D/13.5/WEB-SET-OPTION.301-1457388.ja.html)で有効にすることができます。

```
WEB SET OPTION(Web keep session;1)
```

__スコープ__

デフォルトの設定では，ランゲージで処理されるURLはすべてWebセッション管理が有効にされています。ランゲージで処理されるURLとは，つまりスタティックWebサーバー（Webフォルダー内に存在するファイルを自動的に配信すること）以外のURLであり，具体的には，[On Web Connection](http://doc.4d.com/4Dv13/4D/13.5/On-Web-Connection-Database-Method.300-1457408.ja.html)イベントが発生するようなURLのことです。

__注記__：2003以前は，そのようなURLには``4DCGI``という文字列を付けることが要求されていました。2004以降，``4DCGI``は省略できるようになりました。Webフォルダー内にファイルが存在しなければ，必然的に``On Web Connection``イベントが発生します。なお，ドキュメントでは，便宜上，Webフォルダー内に対応するファイルが存在しないことを『無効なリクエスト』と呼んでいます。

__セッションID__

自動セッション管理では，ブラウザから送信されるHTTPクッキーとIPアドレスの組み合わせでセッションを識別します。クッキーの名称はデフォルトで``4DSID``ですが，[WEB SET OPTION](http://doc.4d.com/4Dv13/4D/13.5/WEB-SET-OPTION.301-1457388.ja.html)で変更することもできます。すでに開かれたセッションであるとWebサーバーが判断すれば，停止中のプロセスが再開され，前回の処置で作成したセレクションやプロセス変数をそのまま使い続けることができます。新しいセッションであるとWebサーバーが判断すれば，新規プロセスが作成されます。

デベロッパーは，``On Web Connection``，あるいは``4DACTION``メソッドコールなど，``On Web Connection``を実行しないようなアクセスであれば，そのコールされたメソッドの冒頭で，[WEB Get Current Session ID](http://doc.4d.com/4Dv13/4D/13.5/WEB-Get-Current-Session-ID.301-1457386.ja.html)をコールし，__セッションIDをプロセス変数に代入__します。以降，メソッド開始時にプロセス変数を参照し，すでに値が代入されていれば，それは継続中のコンテキストであると判断することができます。

```
If (Length(Web_currentSessionId)=0)
  Web_currentSessionId:=WEB Get Current Session ID
End if 
```

__Web変数__

v12以前には，プロセス変数を``Compiler_Web``でプロセス変数を宣言しておけば，[HTTP POSTまたはGETで送信されたWebフォーム変数を4Dのプロセス変数に自動代入する](http://doc.4d.com/4Dv13/4D/13.5/Binding-4D-objects-with-HTML-objects.300-1457417.ja.html)，というメカニズムが存在しました。しかし，Webフォーム変数は[WEB GET VARIABLES](http://doc.4d.com/4Dv13/4D/13.5/WEB-GET-VARIABLES.301-1457406.ja.html)，マルチパートでポストされたファイルデータなどは[WEB GET BODY PART](http://doc.4d.com/4Dv13/4D/13.5/WEB-GET-BODY-PART.301-1457383.ja.html)でより効率的に受け取ることができ，そうすれば，Web変数名と同名のプロセス変数を用意する必要もないので，[『Webから送信された変数をプロセス配列に自動代入する』は，v13で廃止されました](http://doc.4d.com/4Dv13/4D/13.4/Compatibility-page.300-1226529.ja.html)。

``WEB GET VARIABLES``は，Webフォーム変数を名前と値の配列ペアで返すコマンドです。目的の変数は，``Find in array``でチェックした後，任意のローカル変数に代入することができます。Web変数と同名のプロセス変数である必要はありません。v14であれば，オブジェクト変数にまとめて管理することもできます。

```
C_OBJECT($variables)
ARRAY TEXT($names;0)
ARRAY TEXT($values;0)
WEB GET VARIABLES($names;$values)
For ($i;1;Size of array($names))
  OB SET($variables;$names{$i};$values{$i})
End for 
```

自動セッション管理が有効にされていれば，Webプロセスのコンテキストにおけるプロセス変数の値は保持されています。ただし，そのような変数は，``Compiler_WEB``で宣言されていなければなりません。たとえば，HTMLに下記のようなコードが記述されているとします。


```html
<form method="post" action="/4daction/Web_得意先履歴一覧/">
<input type="text" name="v得意先名検索" value="<!--#4dtext v得意先名検索-->" />
<input type="submit" value="検索" />
</form>
```

はじめてこのページからフォームを送信したときには，``Compiler_WEB``が実行され，``v得意先名検索``のようなプロセス変数を4D側で宣言する機会が与えられます。前述したように，__Web変数の自動代入は実行されない__ので，``4DACTION``でコールされたメソッドの中で``WEB GET VARIABLES``を使用し，受け取った値を明示的に代入しなければなりません。しかし，そうしておけば，次回のアクセスからは，[Compiler_WEBが実行されません](http://doc.4d.com/4Dv13/4D/13.5/Web-Sessions-Management.300-1457382.ja.html)し，プロセス変数には，前回のアクセスで代入された値が残っていることになります。

__動的なHTMLを出力__

HTMLはテキストなので，テキスト変数の連結でデータベースの値を挿入することもできるかもしれませんが，それではコードが複雑になってしまい，管理が非常に面倒です。動的なHTMLを出力するのであれば，是非，[4D TAGS](http://doc.4d.com/4Dv13/4D/13.5/4D-HTML-Tags.300-1457419.ja.html)を使用してください。

たとえば，セレクションの一部をHTMLページに出力する場合，メソッド内でループを実行し，``NEXT RECORD``などを使用する代わりに，``SELECTION RANGE TO ARRAY``で特定の範囲を配列に取り出すことができます。

```
$start:=1+((Web_currentPage-1)*$lines)
$end:=$start+$lines
$end:=Choose($end<Web_recordsInSelection;$end;Web_recordsInSelection)

SELECTION RANGE TO ARRAY($start;$end;\
 [売上累積]得意先CODE;Web_得意先CODE;\
 [売上累積]日付;Web_日付;\
 [売上累積]納品書NO;Web_納品書NO;\
 [売上累積]品名;Web_品名;\
 [売上累積]数量;Web_数量;\
 [売上累積]単位;Web_単位;\
 [売上累積]単価;Web_単価;\
 [売上累積]金額;Web_金額;\
 [売上累積]入金金額;Web_入金金額;\
 [売上累積]摘要;Web_摘要)
```

4DLOOPタグを活用したHTMLテンプレートを用意しておけば，[WEB SEND FILE](http://doc.4d.com/4Dv13/4D/13.5/WEB-SEND-FILE.301-1457395.ja.html)または[SEND HTML BLOB](http://doc.4d.com/4Dv12/4D/12.4/SEND-HTML-BLOB.301-977184.ja.html)で自動的に配列の値がHTMLに挿入されます。

__注記__: ファイルを使用するのでれば，拡張子は``html, htm, shtml, shtm``のいずれかでなければなりません。BLOBを使用するのであれば，MIME指定が``text/html``でなければなりません。

```html
        <div>
            <table class="t1">
                <tr class="t2">
                    <th width="50">日付</th>
                    <th width="50">納品書NO</th>
                    <th width="170">品名</th>
                    <th width="30">数量</th>
                    <th width="30">単位</th>
                    <th width="40">単価</th>
                    <th width="55">売上金額</th>
                    <th width="55">入金金額</th>
                    <th width="60">摘要</th>
                </tr>
                
                <!--#4dloop Web_得意先CODE-->                
                <!--#4dif (Web_得意先CODE%2=0)-->
                <tr class="t3">
                <!--#4delse-->
                <tr class="t4">
                <!--#4dendif-->                
                    <th width="50"><!--4dtext String(Web_日付{Web_得意先CODE};4)--></th>
                    <th width="50"><!--4dtext Web_納品書NO{Web_得意先CODE}--></th>
                    <th class="t5" width="170"><!--4dtext Web_品名{Web_得意先CODE}--></th>
                    <th class="t6" width="30"><!--4dtext String(Web_数量{Web_得意先CODE};"###,###,##0")--></th>
                    <th width="30"><!--4dtext Web_単位{Web_得意先CODE}--></th>
                    <th class="t6" width="40"><!--4dtext String(Web_単価{Web_得意先CODE};"###,###,##0")--></th>
                    <th class="t6" width="55"><!--4dtext String(Web_金額{Web_得意先CODE};"###,###,##0")--></th>
                    <th class="t6" width="55">&yen;<!--4dtext String(Web_入金金額{Web_得意先CODE};"###,###,##0")--></th>
                    <th width="60"><!--4dtext Web_摘要{Web_得意先CODE}--></th>
                </tr>                
                <!--#4dendloop-->
            </table>
        </div>
```
__注記__: HTMLファイルに通貨記号を直接，記述した場合，Macのブラウザではバックスラッシュが表示されるかもしれません。上記のように，エンティティ参照``&yen;``を使用することにより，そうした問題を避けることができます。

__まとめ__

Webコンテキストと4Dプロセスを関連付ける方法は，v13以降，おおきく変更されました。

___廃止されたもの___

* コンテキストモード全般
* ``SEND HTML BLOB``の第3引数に``false``を渡すこと
* ``SEND HTML TEXT``の第3引数に``false``を渡すこと
* ``Web Context``
* ``/4DMETHOD/``から始まるURL
* Web変数の自動代入

___代わりに使用されるもの___

* 自動セッション管理
* ``WEB GET VARIABLES``
* ``/4DACTION/``から始まるURL
* ``On Web Connection``
