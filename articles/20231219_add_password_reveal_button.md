---
title: "「パスワードを表示する」ボタンを実装する"
emoji: "👁️"
type: "tech"
topics: ["HTML", "JavaScript", "ShadowDOM"]
published: true
published_at: 2023-12-19 07:00
---

## TL;DL

こちらはQiitaアドカレで書いた記事の焼き直しです。
最近流行りの「パスワードを表示する」ボタンをShadowDOMを利用して実装してみます

## 「パスワードを表示する」ボタンって？

こういうやつ

![sample_image.png](/images/20231219_add_password_reveal_button/sample_image.png)

これを作るメリットとしては、ログイン試行回数が設定してある場合にCapsLockなどが原因でアカウントロックされてしまった！系の問い合わせを減らすとかがあると思います。
逆に、デメリットとしては覗き見されてしまう可能性が上がること、悪意あるブラウザプラグインなどから読み取られる可能性が増えるといったことが挙げられます。

ざっとググってみたところ、どうにも`inputのtypeを単純にtext<->passwordでtoggleする`やつがいっぱい出てきて、セキュリティ的にビミョーな感じがしたのでここに書きます。
杞憂かつ意味ないかもしれませんがちょっとだけしっかり隠すことを意識して書いてみます。

## ShadowDOMって？

さまざまな方が記事にしており、mozillaにも[記事](https://developer.mozilla.org/ja/docs/Web/API/Web_components/Using_shadow_DOM)があるのでここでは割愛します。
簡単に言うと、「CSSの適用範囲を限定させたコンポーネントをjsで動的に作成し、外側から中身が見えないように**も**できるやつ」って認識でいいかもしれないです。
要はカプセル化を実現できるやつ。DOMにpublicとかprivateとかつけるようなもの。

こいつを使えば、デメリットの1つであった他ブラウザプラグインや他スクリプトから内容を保護することができるため、マスクされていないinputにパスワードが入力されても安心という感じです。現在（2023年末）ではほぼ全てのブラウザで対応しています。

## 作ってみる

### マークアップ,スタイル

テキトーに書きます。本当にテキトーなので許してください。
idどこに書いてあるかなくらいの確認でOKです。

```html
<div id="password-container">
  <div class="password-block">
    <span>PASS:</span>
    <span id="password"><input type="password" /></span>
  </div>
  
  <div class="password-block">
    <span>REVEALED_PASS:</span>
    <span id="password-revealed"></span>  
  </div>
  
  <div class="password-block">
    <a id="toggle_button">[hide]</a>
  </div>
</div>


<style>
  .password-block {
    margin: 1rem 0;
    padding: 0 0.5rem;
  }
</style>
```

### スクリプト

基本はテキトーのままですがこんな感じ。
あと諸事情によりjQueryを使っています。

```javascript
// 各要素をjQueryオブジェクトで取得
var $pass_area = $('#password');
var $pass_revealed_area = $('#password-revealed');

// shadow化
var $pass_revealed_area_shadow = $($pass_revealed_area.get(0).attachShadow({ mode: "closed" }));

// input要素をshadow化したものの中に入れる
var $input = $('<input type="text">');
$pass_revealed_area_shadow.append($input);

// 以降は確認用の実装
$pass_area.find('input').on('keyup', function(e) { 
    // 見せ方によってはchangeイベントでもいいかも
	$input.val(e.target.value);
});
$('#toggle_button').on('click', function(e){
	switch (e.target.innerText) {
  	case '[hide]':
	    $pass_revealed_area.toggle();
    	e.target.innerText = '[show]';
    	break;
  	case '[show]':
	    $pass_revealed_area.toggle();
    	e.target.innerText = '[hide]';
    	break;
  }
});
```
### 解説

上記を入力するとこんな感じで出力されます。
passwordを入力すると下のテキストフォームにも隠されずに表示されます。

![sample_code_html.png](/images/20231219_add_password_reveal_button/sample_code_html.png)

上のコードの

```javascript
// shadow化
var $pass_revealed_area_shadow = $($pass_revealed_area.get(0).attachShadow({ mode: "closed" }));
```

で今回の肝であるshadowDOMを作成しています。
`$pass_revealed_area.get(0)`は単純にDOMを取得するために書いてあるだけで、`<Dom Element>.attachShadow()`するだけでshadow化された要素が返ってきます。

オプションの`{ mode: "closed" }`ですが、これは後ほどshadow化した要素を`<Dom Element>.shadowRoot`で再取得できるかどうかの動きを指定しています。
今回は再取得禁止のため`closed`にしてます。有効にするには`open`と記載します。

また、shadow化できる要素は決まっており、div,spanなどコンテンツをまとめるようなタグにしか使用できません。詳しくは[mozzilaの記事](https://developer.mozilla.org/en-US/docs/Web/API/Element/attachShadow)を参照。

shadow化したあとは普通に操作できなくなるので、`attachShadow()`で返ってきたDOMに対して色々appendしていっているだけです。
こうすると`console.log($('body').html())`とかをコンソールで叩いて書き出しても、今回の例でappendしたinput要素は参照できなくなっています。

こうすることで、単純に`<input type=text>`に変換するのに比べて若干安全になります。
あとは見た目とか動きとか実装し、コードを綺麗にすればそのまま使えると思います。
加えてrevealしておく上限時間など実装しておくといいかもしれませんね。
