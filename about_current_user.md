# Sorceryのメソッドたちを頑張って細かくみてみた。
Railsで認証機能を作るときに便利なsorceryというgem

ヘルパーメソッドがいくつかあり、それについてGithubでは

「名前のままの機能です」的なことが英語で書かれてます

[GitHub \- Sorcery/sorcery: Magical Authentication](https://github.com/Sorcery/sorcery#api-summary)

 
でもこのメソッドたちが実際に中で何を行っているかはわからなかったので、中身を見ていこうと思いました。

途中で雑な理解になっておりますがご了承ください。


今回は「current_user」を見ていきます。

## current_userについて
[rubydoc](https://rubydoc.info/gems/sorcery/Sorcery%2FController%2FInstanceMethods:current_user)を見てみるとこう書いてあります。

「 attempts to auto-login from the sources defined (session, basic_auth, cookie, etc.) returns the logged in user if found, nil if not 」

「 定義されたソース（session、basic_auth、cookieなど）から自動ログインを試みる。見つかった場合はログインしたユーザーを、見つからない場合はnilを返す。」


そしてコードはこちら
```
def current_user
    unless defined?(@current_user)
        @current_user = login_from_session || login_from_other_sources || nil
    end
    @current_user
end
```
とりあえず最後にcurrent_userを返しているのはわかる（そりゃそうだ(^^;）

とりあえず説明文を二つに分けて考えてみます
1. 定義されたソース（session、basic_auth、cookieなど）から自動ログインを試みる
2. 見つかった場合はログインしたユーザーを、見つからない場合はnilを返す

## 1. 定義されたソース（session、basic_auth、cookieなど）から自動ログインを試みる
定義されたソースとは何か？

たぶんですが、session、basic_auth、cookieなどのことですかね？書いてあるし。

ここでCookieについて少し調べます
### Cookieについて
Webにおいてユーザーデータの管理を実現させる方法の一つがCookieというもの。

例えばcookieについては[Web技術の基本](https://www.amazon.co.jp/%E3%82%A4%E3%83%A9%E3%82%B9%E3%83%88%E5%9B%B3%E8%A7%A3%E5%BC%8F-%E3%81%93%E3%81%AE%E4%B8%80%E5%86%8A%E3%81%A7%E5%85%A8%E9%83%A8%E3%82%8F%E3%81%8B%E3%82%8BWeb%E6%8A%80%E8%A1%93%E3%81%AE%E5%9F%BA%E6%9C%AC-%E5%B0%8F%E6%9E%97-%E6%81%AD%E5%B9%B3-ebook/dp/B06XNMMC9S/ref=sr_1_1?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=3IEP55SMC7TBE&dchild=1&keywords=web%E6%8A%80%E8%A1%93%E3%81%AE%E5%9F%BA%E6%9C%AC&qid=1630415768&sprefix=web%E6%8A%80%E8%A1%93%E3%81%AE%2Caps%2C316&sr=8-1)にはこう書いてあります

> 「HTTPはステートレスなプロトコルであるため、webブラウザとwebサーバーの一連のやり取りに置いて、状態を保持し管理する仕組みがありません。そのため、ショッピングサイトなどで状態を保持し管理する必要がある場合にはCookieと呼ばれるデータが用いられます」

また、MDNも見てみます

[HTTP Cookie の使用 \- HTTP \| MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/Cookies)

> Cookie は、ステートレスな HTTP プロトコルのためにステートフルな情報を記憶します。

自分が通っているプログラミングスクールRUNTEQの講師だいそんさんの記事も見つけました！[RailsでCookieとセッションを理解してログインが何かを解説 \| RUNTEQ \- 公式ブログ｜RUNTEQ](https://runteq.jp/blog/programming-school/knowledge/3111/)

HTTPは一連のやり取りの状態を保てない。だから状態を保たせる仕組みが必要。

そしてRailsはCookieによってセッションの仕組みの一部が実現されている。
「 現場で使えるRuby on Rails 」p149

まとめてみると、
- 定義されたソースっていうのはCookieなどのこと
- sorceryはcookieにアクセスして、自動ログインを試みている？

## 自動ログインを試みるって？
一旦current_userのソースコードをみてみます。
```
    def current_user
        unless defined?(@current_user)
            @current_user = login_from_session || login_from_other_sources || nil
        end
        @current_user
    end
```
`login_from_session`と`login_from_other_sources`を詳しくみてみるとこんな感じ

```
    def login_from_other_sources
        result = nil
        Config.login_sources.find do |source|
            result = send(source)
        end
        result || false
    end

    def login_from_session
        @current_user = if session[:user_id]
                            user_class.sorcery_adapter.find_by_id(session[:user_id])
                        end
    end
```
途中で出てくる`user_class`はこんな感じ
```
    def user_class
        @user_class ||= Config.user_class.to_s.constantize
    rescue NameError
        raise ArgumentError, 'You have incorrectly defined user_class or have forgotten to define it in intitializer file (config.user_class = \'User\').'
    end
```
もうわからん(^^;

ここで自分なりに憶測を立ててみます
1. `user_class`っていうのは自分で定義するUserモデルのこと
2. `login_from_session`は、もしセッションの中にuser_idがあるなら、同じuser_idのユーザーをUserモデルから探して、@current_userに格納
3. `login_from_other_sources`は定義されたソースがcookieじゃない場合に、その別のソースを引っ張ってきて返している。

一応こんな感じだと思うのですが…違うかも(^^;

## 2. 見つかった場合はログインしたユーザーを、見つからない場合はnilを返す
これはもうそのまんまですね。先ほどの説明と重複しますが、

`login_from_session`でユーザーが見つかったら@current_userへ格納

もし見つからず、`login_from_other_sources`でユーザー（？）が見つかったら@current_userへ格納

見つからない場合は＠current_userにnillを格納。

そして@current_userを返す。


以上です。

## コメント

最後は雑な理解になってしまいました(^^;

というか、current_userに関しては自分で定義した方がわかりやすいのではないか？
と思い始めています。

```
    current_user
        User.find_by(id: session[:user_id])
    end
```
めちゃくちゃシンプルですw

ご覧いただきありがとうございました！

余裕があればsorceryの他のメソッドも見ていきたいです!