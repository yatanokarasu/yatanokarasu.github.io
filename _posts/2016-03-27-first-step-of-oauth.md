---
layout:         post
title:          "The first step of OAuth"
categories:     [Authorization]
tags:           [OAuth]
---

OAuthにより、Github等のソーシャルサービスが提供する機能を、その他のサービスが利用することができます。
まずは基礎の理解をしていきたいと思います。

<!-- more -->

OAuthは認可(Authorization)を行うための標準プロトコルです。IETFで[RFC6749](http://tools.ietf.org/html/rfc6749)で規定されています。
認可とは「そのサービスを利用するための権限を与えること」です。認証(Authentication)とは異なります、認証は「その人を本人であることを証明すること」です。
ですので、OAuthを行えばなりすましなどの対策が出来るといえば違います。
その話はまた今度(気になる場合は、詳細は[コチラ](http://www.thread-safe.com/2012/01/problem-with-oauth-for-authentication.html))。


# Overall sequence

OAuthで覚えておきたい用語は下表の通りです。

| Terms                 | Means                                                                       |
|:----------------------|:----------------------------------------------------------------------------|
| Service Provider      | OAuthに対応したAPIを提供するWebサービス                                             |
| Consumer              | サービスプロバイダのAPIへのアクセス認可を受け、APIにアクセスするアプリケーション                   |
| User (Resource Owner) | サービスプロバイダのWebサービスに自分のアカウントを持っており、コンシューマのアプリケーションを利用するユーザ |
| Consumer Key (Client ID) | サービスプロバイダがコンシューマを一意に識別するためのID |
| Request Token         | コンシューマが、これと引き換えにサービスプロバイダからアクセストークンを取得します |
| Access Token          | コンシューマがAPIへのアクセス認可を受け、サービスプロバイダから取得する文字列。このトークンを使用して、APIにアクセスすることができる |
{: rules="rows"}


では、OAuthのフローは以下の通りです。ここでは例として、
__ユーザはQiita(コンシューマ)のログインにGithub(サービスプロバイダ)を使用する__ ことをイメージしてみます。



0.  これはユーザが気にするフローではないですが、コンシューマは事前にサービスプロバイダからOAuth利用認可を得ておく。
    ここでコンシューマは、`Consumer Key (Consumer ID)`と`Consumer Secret`を取得しておきます。
    すなわち、Qiitaはサービス提供にあたり、事前にGithubからOAuth利用認可をもらっておきます。

1.  ユーザ(リソースオーナ)は、コンシューマにサービスプロバイダと連携機能の使用を開始する。
    Qiitaのログインページで __Githubでログイン__ を選択するイメージです。

2.  コンシューマは、バックグラウンドでサービスプロバイダにアクセスし、 __未認可のリクエストトークン__ を取得します。
    Qiitaは、バックグラウンドでGithubにアクセスして、未認可のリクエストトークンをもらってきます。
    この未認可のリクエストトークンでは、アクセストークンと交換してもらえません。

3.  ユーザはサービスプロバイダにリダイレクトさせます。この際、コンシューマはリダイレクトURLのURLストリングに未認可のリクエストトークンを付加します。
    QiitaがユーザをGithubの認証ページにリダイレクトし、その際未認可のリクエストトークンを付加することになります。

4.  ユーザはサービスプロバイダで認証処理をして、コンシューマへのアクセス権委譲を許可します。
    Githubのログイン処理をするイメージです。
    この際、受け取ったリクエストトークンを認可済トークンとして扱います。

5.  サービスプロバイダは、ユーザをコンシューマのコールバックURLにリダイレクトさせます。Githubで認証後、QiitaのコールバックURLにリダイレクトするイメージです。
    この際、URLストリングに認可済リクエストトークンを付加します。

6.  コンシューマは、バックグラウンドでサービスプロバイダに認可済リクエストトークンを送り、アクセス権を行使できるアクセストークンと交換します。
    QiitaがGithubからアクセス権を得るイメージです。

7.  コンシューマｈは、アクセストークンを利用して、ユーザが求める情報にアクセスします。


これをシーケンス図は以下の通りです。

{% include gravizo
  src='
@startuml;
hide footbox;

participant "User\n(Resource Owner)" as user;
participant "Consumer\n(ex. Qiita)" as consumer;
participant "Service Provider\n(ex. Github)" as provider;

consumer ->  provider : 0. receive authorization and get Consumer Key & Secret;
consumer <-- provider : Consumer Key & Secret;

|||;

user     ->  consumer : 1. use service;
consumer ->  provider : 2. get unauthorized token with Consumer Key & Secret;
consumer <-- provider : unauthorized token;
user     <-- consumer : <<302 Found>> with unauthorized token;
user     ->  provider : 3. redirect to Service Provider with unauthorized token;
user     <-- provider : show authentication page to authorize consumer;

|||;

user     ->  provider : 4. authenticate (login);
user     <-- provider : <<302 Found>> with <b>authorized token</b>;
user     ->  consumer : 5. redirect to Consumer callback URL with <b>authorized token</b>;

|||;

consumer ->  provider : 6. exchange authorized token for <b>access token</b>;
consumer <-- provider : <b>access token</b>;

|||;

consumer ->  provider : 7. access to API with <b>access token</b>;
consumer <-- provider : result;
user     <-- consumer : result;

@enduml;'
%}
