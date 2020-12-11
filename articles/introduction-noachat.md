---
title: "Neovimでchatがしたい"
emoji: "♨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Neovim"]
published: true
---
# 始めに
こんにちは。higashiです。

この記事は[Vim Advent Calendar 2020](https://qiita.com/advent-calendar/2020/vim) 10日目の記事となっています。

今回はNeovimでchatがしたくなったのでchatをします。

# 使うもの
- Neovim v0.5.0
- Python

# 作ったもの
作りました。
こちらです！
Vim対応もしたかったのですが現在Neovimのみの対応となっています。
[noachat.nvim](https://github.com/higashi000/noachat.nvim)
[![Image from Gyazo](https://i.gyazo.com/de5b59d3ca11f7555c350497be9f22c4.gif)](https://gyazo.com/de5b59d3ca11f7555c350497be9f22c4)

# 解説
リアルタイムチャットをできるようにしてみました。

## リアルタイムチャットの部分
WebSocketを使用しています。
当初は「VimScriptでWebSocketできないかな〜」と思っていたのですが、私の現状の技術力では無理という判断をし、Neovimのremote pluginとPythonを使うことにしました。

## 投稿の部分
チャットの受信に使うrecvをwhileで回す都合上、受信中はsendを使っての送信ができません。
じゃあどうやってメッセージ送信するんだよって話です。
悩みながらechoコマンドで遊んでいたところ、「echoが使えるならsystemも使えるのでは？」と思い、
急遽サーバ側にpostでメッセージを送るとつながっているclientすべてにそのメッセージを送信する機能を実装しました。
プラグイン側にもcurlでメッセージを送る関数を実装し試したところこちらでうまく行くとわかったのでこのまま行くことにしました。
発想の勝利です。（私はremote pluginにまったく詳しくないのでうまい実装方法があったのかもしれませんが）

## サーバ
Goで実装しました。
リポジトリはこちらです。
[noachat](https://github.com/higashi000/noachat)
DBを使ったメッセージの保存はしてません。
リロードすると過去ログは読めなくなります。

# インストール
じゃあ実際に使ってみましょう。
サーバはGCPを使ってホスト済みです。

まずは必要なPythonのライブラリをインストールしてきます。
```
pip install --user pynvim websocket-client
```

次にプラグインのインストールです。

dein.vim
```
call dein#add('higashi000/noachat.nvim')
```

vim-plug
```
Plug 'higashi000/noachat.nvim'
```

インストールが終わったら`:UpdateRemotePlugin`の実行をお願いします。

上記すべてが終了したらinit.vimに以下の設定を書き込んでください。
設定部分はShougoさんの[denite.nvim](https://github.com/Shougo/denite.nvim)を参考にさせていただきました。
```vim
autocmd FileType noachat call s:noachat_settings()
function! s:noachat_settings() abort
    map <silent> nl <Plug>(noachat_leave)
endfunction

let g:noachat#ServerURL = 'noa.higashi.dev'
let g:noachat#https = v:true

let g:noachat#UserName = 'your user name'
```

`<Plug>(noachat_leave)` ではチャットの終了キーバインドを設定します。
ちなみにNeovimを`:q!`で終了しても勝手に接続は切られます。

`g:noachat#ServerURL`にはサーバのドメインを書いてあげます。
サーバがhttpsに対応している場合は`g:noachat#https`をtrueにしてあげます。
`g:noachat#UserName`には自分の名前を書いてください。
チャットのユーザー名になります。（未記入だとnonameになります）

メッセージの送信は`:NoaChatPostMsg`で行えます。

これで準備は終わりです！
init.vimを読み込み直し、`:StartNoachat`でチャットを始めましょう！

# Web版について
NeovimだけではなくWeb版も実装してみました。
[こちら](https://noa.higashi.dev)
ここと自分のNeovimでメッセージを送りあうと一人でリアルタイムチャット気分が味わえます。

# 課題
メッセージ受信中はかなり不自由になってしまいます。（他のremote pluginが使えなくなってしまう等）
不自由になってしまう部分はひとつずつ解消していきたいと思います。

# 最後に
このサーバはおそらく来年の1月ころまで動いていると思います。（動いてない報告は[僕の Twitter](https://twitter.com/higashi136_2)までおねがいします）
攻撃はやめてください。課金で私の財布が死ぬと思います。（攻撃できそうなところがあったら教えてくださると助かります）
思い立ったときに誰かを誘ってチャットを楽しんでください。
