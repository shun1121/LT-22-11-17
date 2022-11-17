---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS
css: unocss
---

# お昼休憩スタンプを投稿するSlackアプリを作った話

Fukunishi.S

---

# 背景

<style>
li {
  font-size: 20px;
}
</style>

<div class="flex">
  <p class="w-full text-white pt-10">
    <ul class="">
      <li class="mb-5">#breaktimeチャンネルにスタンプを投稿。</li>
      <li>プロフィールのステータスを変更。</li>
    </ul>
    <p v-click class="pt-15 pl-[70px] text-5">ちょっとめんどくさい...</p>
    <p v-click class="pt-15  text-[24px]">Slashコマンドで1アクションにしよう！</p>
  </p>
  <div class="w-full space-x-6">
    <img src="/post-stamp.png" class="w-92 object-contain mb-5" />
    <img src="/prof-set-stamp.png" class="w-80 object-contain" />
  </div>
</div>

---

# Slashコマンド

<style>
  .slidev-layout h1 + p {
    opacity: initial;
    margin-top: 40px;
  }
</style>

メッセージ入力欄からアプリの機能を呼び出すことができる機能。

メッセージ入力欄に `/` と入力し、コマンドを打つと機能を使うことができる。

<img src="/slash-command.png" class="w-92 object-contain my-8" />

https://slack.com/intl/ja-jp/help/articles/201259356-Slack-%E3%81%AE%E3%82%B9%E3%83%A9%E3%83%83%E3%82%B7%E3%83%A5%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89#h_01EPZ2Z81EJ67RA2BGDKZ9M1AN


---

# 構成

<style>
  ul {
    margin-top: 70px
  }
  li {
    font-size: 20px
  }
</style>

<img src="/structure.png" class="mt-10 object-contain mb-5" />

---

# 使い方

<style>
  p {
    width: 90%;
    word-wrap: break-word;
  }
</style>

<div class="flex w-[400px] mt-10">
  <p class="pt-8">
    https://slack.com/oauth/v2/authorize?client_id=2659222021.3940566559953&scope=commands,im:history,chat:write,chat:write.customize&user_scope=users.profile:write,chat:write <br><br>
    こちらのURLにアクセスして、「許可する」ボタンを押します。<br><br>
    時間を少し空けるとアプリが使えるようになります。
  </p>

  <img src="/request.png" class="ml-20 object-contain w-[700px]" />
</div>

---

# 機能紹介

<style>
  table {
    margin-top: 80px;
  }
</style>

| Slashコマンド | 挙動 |
| --- | --- |
| <kbd class="text-orange">/hiru</kbd>| #breaktimeチャンネルとプロフィールのステータスに<img src="/hiru.png" class="inline ml-1 object-contain w-[30px] h-auto" />を投稿。<br>ステータスのスタンプは60分で自動的に消える。 |
| <kbd class="text-orange">/zenhan</kbd> | #breaktimeチャンネルとプロフィールのステータスに<img src="/zenhan.png" class="inline ml-1 object-contain w-[30px] h-auto" />を投稿。<br>ステータスのスタンプは30分で自動的に消える。 |
| <kbd class="text-orange">/kouhan</kbd> | #breaktimeチャンネルとプロフィールのステータスに<img src="/kouhan.png" class="inline ml-1 object-contain w-[30px] h-auto" />を投稿。<br>ステータスのスタンプは30分で自動的に消える。 |

---

# 詰まった点

<style>
  ul {
    margin-top: 60px;
    margin-left: 140px;
  }
  li {
    font-size: 32px;
    margin: 30px
  }
  ol li {
    font-size: 20px;
    margin: 15px;
  }
  a {
    font-size: 20px;
  }
</style>

- OAuth認証
  https://qiita.com/TakahikoKawasaki/items/e37caf50776e00e733be
  1. 認可コードの取得
  2. アクセストークンの取得
- コールドスタート

---

# AOuth認証
<style>
  span {
    color: #3FA8FF;
  }
</style>

<div>
  <ul>
    <li><span>https://slack.com/oauth/authorize</span> にclient_id、scope、user_scopeなどのクエリパラメータを与えてアクセスします。(Settings > Manage Distribution > Sharable URLに記載)</li>
    <li>画面の中で「許可する」ボタンを押下する。</li>
    <li>SlackアプリのRedirect URLsに登録したURLにリクエストが飛ぶ。</li>
    <li>上記で設定のURLにはcodeから始まる<strong class="underline">認可コードが含まれているので取得する</strong>。</li>
    <li>取得した認可コードを使って <span>oauth.v2.access</span> にリクエストし、<strong class="underline">アクセストークンを取得する</strong>。</li>
  </ul>
</div>

```js
app.get("/", async (req, res) => {
  const code = req.query.code;   // req.query.codeで認可コードを取得
              ...
  if (code) {
    const slack = new WebClient();
    const result = await slack.oauth.v2.access({ // 認可コードを使ってアクセストークンを取得
      client_id: clientId,
      client_secret: clientSecret,
      code,
    });
              ...
  }
});
```

---

# コールドスタート

<style>
  .slidev-layout h1 + p {
    color:#FFF;
    opacity: initial;
    margin-top: 50px;
    margin-bottom: 50px;
  }
</style>

<p v-click>
  初めてSlashコマンドを実行したり、時間をあけて実行するとスタンプの投稿に数秒から十数秒時間がかかってしまっていた...。
</p>

<ul v-click>
  <h3 class="mb-3 text-blue">コールドスタート</h3>
  <li>
    Cloud Functionsでは、1つのリクエストに対して、1つのインスタンスが作られ、そこで関数の処理が実行される。
  </li>
  <li>
    インスタンスが作成されると、実行環境の初期化が行われるので、その分の時間がかかる。
  </li>
  <li>
    作成されたインスタンスは一定時間維持され、リクエストが再度あった場合はインスタンスは再利用される。
  </li>
  <li>
    しばらく実行されない場合、そのインスタンスは削除される。
  </li>
  <li>
    インスタンスが削除された後にリクエストが来ると再び１からインスタンスが作られるため時間がかかる。
  </li>
</ul>

---

# コールドスタート問題の解決方法

<style>
  .slidev-layout h1 + ul {
    margin-top: 70px;
    margin-bottom: 20px;
  }
  p {
    margin-top: 50px;
    font-size: 24px;
  }
</style>

- レスポンス時間を短縮するために、米国リージョンで使用していたものを東京リージョンに変更する。
- メモリの割り当てを256MBから512MBに増やす。https://zenn.dev/seisei/articles/7c02fdffa866a0

```js
...
exports.slackApp = functions.region("asia-northeast1") // 東京リージョンを指定
                            .runWith({ memory: '512MB' }) // メモリサイズを512MBに設定
                            .https.onRequest(app);
```

<p v-click class="pt-10">
  この変更で、コールドスタート時でも遅くとも2500ms前後にはレスポンスが返るようになった。
</p>
---

# 終わりに

<style>
  ul {
    margin-top: 80px;
    font-size: 28px;
  }
  li {
    margin-top: 16px;
  }
  a {
    margin-left: 40px;
  }
</style>

- OAuth認証の理解やコードでの記述方法が学べてよかった。
- 自分が作ったアプリを誰かに使ってもらえるのは改めて嬉しいことだと感じた。

https://github.com/shun1121/change-status

---

# 参考
<style>
  .slidev-layout h1 + p {
    margin-top: 30px;
    opacity: initial;
  }
</style>

https://qiita.com/TakahikoKawasaki/items/e37caf50776e00e733be

https://api.slack.com/authentication/oauth-v2#exchanging

https://nado1999.me/articles/slack-oauth-token-20220407

https://qiita.com/subarunari/items/3e4c6060fcefd4c65257

https://qiita.com/dbgso/items/a95a3364d9a8c67f3387

https://qiita.com/takehilo/items/9ab4a25a02b8328a6d5e

https://reffect.co.jp/html/slack#ngrok

https://firebase.google.com/docs/firestore/quickstart?hl=ja

https://www.boel.co.jp/tips/vol124/