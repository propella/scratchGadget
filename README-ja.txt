-*- outline -*-

* Scratch を Google Gadget にする方法。

<script src="http://www.gmodules.com/ig/ifr?url=http://metatoys.org/scratchGadget/scratch.xml&amp;up_url=http%3A%2F%2Fscratch.mit.edu%2Fprojects%2Fabee%2F1843337&amp;synd=open&amp;w=482&amp;h=410&amp;title=How+to+play+Scratch+(WIP)&amp;lang=ja&amp;country=ALL&amp;border=%2523ffffff%7C3px%252C1px+solid+%2523999999&amp;output=js"></script>

阿部さんに教えてもらった
http://scratch.mit.edu/forums/viewtopic.php?pid=921594#p921594 の質問に触発されて、Scratch 用の Google Gadget を作ったメモ

** Scratch Gadget の使い方

最初に出来たガジェットの使い方を書きます。まず当初の目的だったはてなにスクラッチ作品を貼るには、

http://www.gmodules.com/ig/creator?url=http://metatoys.org/scratchGadget/scratch.xml&up_url=&synd=open&w=482&h=410&title=%E3%82%B9%E3%82%AF%E3%83%A9%E3%83%83%E3%83%81+%E4%BD%9C%E5%93%81&lang=ja&country=ALL&border=%23ffffff|3px%2C1px+solid+%23999999

にアクセスして、タイトルと、なぜか ??? になっている所に作品の URL を入れて「コードを取得」を押して下さい。残念ながらプレビューは効かないです。

google のホームページに追加するには、+ Google と書いてあるボタンを押します。Google に行くとガジェットがあるので、ガジェットの右上の歯車アイコンから設定を変数を押し、URL を入力します。以下メモ。

** 単純に貼り付ける方法。

Scratch にはすでにアプレットをホームページに貼りつけるためのソースが用意されてるので、Google Gadget 用の xml に貼るだけでガジェットは完成します。例えば http://scratch.mit.edu/projects/Suzu/1993994 のガジェットを作るには、ページに右側にある Embed リンクから As an applet: の中にある HTML コードを以下のように xml の Content 要素内に書くだけです。

<?xml version="1.0" encoding="UTF-8" ?> 
<Module>
  <ModulePrefs title="block buster" />
  <Content type="html">
     <![CDATA[
<applet id='ProjectApplet' style='display:block' code='ScratchApplet' codebase='http://scratch.mit.edu/static/misc' archive='ScratchApplet.jar' height='387' width='482'><param name='project' value='../../static/projects/Suzu/1993994.sb'></applet> <a href='http://scratch.mit.edu/projects/Suzu/1993994'>Learn more about this project</a>
     ]]>
  </Content> 
</Module>

ただ、この方法だとわざわざプロジェクトを作るたびに xml をアップロードしないと行けないので、汎用で使えるガジェットを考えます。ガジェットには利用者が編集画面でパラメーターを入力する機能があるので、作品の URL を入れるとその作品用のガジェットが出来上がるようにしましょう。

** パラメーターの設定

パラメーターを設定するには UserPref 要素を使います。例えば、

>|xml|
  <UserPref name="url" display_name="Project URL"/>
||<

の要素を Module に含めると、url という名前のパラメーターになります。スクリプトでパラメーターを取得するには Gadget API の gadget.Profs オブジェクトを使います。

>|javascript|
  var prefs = new gadgets.Prefs();
  var url = prefs.getString("url");
||<

** リモートコンテンツ

もう一つ面白い機能がリモートコンテンツです。他のホームページにアクセスして内容を取ってくる事が出来ます。この Scratch Gadget では、作品のページからタイトルを取ってくる事にします。Gadget API では gadget.io.makeRequest(url, response, params) を使います。url がアクセス先、response がテータ取得後のコールバック、params がデータの種類など様々なオプションです。今回は普通にテキストとして取得して正規表現でタイトルを抜き出しました。

>|javascript|
  function setTitle(url) {
    var params = {};  
    params[gadgets.io.RequestParameters.CONTENT_TYPE] = gadgets.io.ContentType.TEXT;
    gadgets.io.makeRequest(url, response, params);
  }

  function response(obj) {
    var doc = obj.text;
    var match = /project_title">([^<]*)</.exec(doc);
    if (!match) return;
    var title = match[1];
    gadgets.window.setTitle(title);
    document.getElementById("projectLink").innerText = title;
  }
||<

** 国際化

次に、国際化について考えます。この機能を使うとガジェットの中の単語を色んな国の言葉で表示する事が出来ます。このガジェットにはほとんど言葉がありませんが、折角なのでやってみます。

まず、デフォルトで使われるメッセージファイルを ALL_ALL.xml という名前で作ります。name 属性がプログラムが読む識別子で、中にテキスト要素として人が読む言葉を書きます。

>|xml|
<messagebundle>
  <msg name="Project_URL">Project URL</msg> 
</messagebundle> 
||<

次に翻訳後の内容を、日本語なら ja_ALL.xml という名前で作ります。

>|xml|
<messagebundle>
  <msg name="Project_URL">プロジェクトの URL</msg> 
</messagebundle> 
||<

次にこのファイルをアップして、ガジェットファイルにファイルの場所を書きます。Lang="ja" のように言語を指定します。言語がない場合は、この場合デフォルトとして ALL_ALL.xml が使われます。

>|xml|
    <Locale messages="http://metatoys.org/scratchGadget/ALL_ALL.xml"/>
    <Locale lang="ja" messages="http://metatoys.org/scratchGadget/ja_ALL.xml"/> 
||<

ガジェットファイル内で設定した単語にアクセスするには、__MSG_識別子__ という形式を使います。例えば上のパラメーター設定時の表示内容を以下のように書き換えます。Javascript 内で使う場合は gadget.Prefs にある getMsg() を使うらしいです。

>|xml|
  <UserPref name="url" display_name="__MSG_Project_URL__"/>
||<

** 投稿

ガジェットが出来たら折角なので投稿してみます。投稿は http://www.google.com/ig/submit にガジェット URL を書きます。公式マニュアルには <ModulePrefs> の必須の属性について書いてありますが、割りと適当でいいみたいです。ここで公開すると以下のように見えます。

http://www.google.com/ig/directory?url=metatoys.org/scratchGadget/scratch.xml

** 参考

- ユーザー設定の定義: http://code.google.com/intl/ja/apis/gadgets/docs/basic.html#Userprefs
- iGoogle コンテンツ ディレクトリへの公開:　http://code.google.com/intl/ja/apis/gadgets/docs/publish.html#Submitting
- リモート コンテンツの処理: http://code.google.com/intl/ja/apis/gadgets/docs/remote-content.html
- ガジェットと国際化 (i18n): http://code.google.com/intl/ja/apis/gadgets/docs/i18n.html
- ガジェットの公開: http://code.google.com/intl/ja/apis/gadgets/docs/publish.html

http://localhost/src/scratchGadget/
