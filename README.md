# なんのアプリか
- 認証付きtodoアプリ
    現在の技術は難解な部分やが多く、アプリを作ってもサイトに公開できない等あり、基礎中の基礎を歴史を踏まえて設計・製作・デプロイする必要があったため。
    
# 技術スタック　選定理由
- Frontend:React/Next.js/Tailwind
    Next 人気なフレームワーク React学習中だったため今回はルートディレクトリを使いたかった。
    React 
- Backed:Django 
    管理画面やパッケージが豊富で使いやすいため。将来的な拡張も視野に入れて（非同期通信・
- Auth: dj_rest_auth + HttpOnly Cookie
    reactを利用しての認証が必要だったため、dj_rest_authを利用。

- Infra : Docker / Nginx
    Dcokerは環境が異なってもすぐに実行できることと、

- deploy : さくらVPS
    awsやGCP,azuruも考えたが、まずは歴史をおさえて、自分の思想での設計を重視したかったため。nextはvercelを,djangoはrenderを使えば楽だが、認証につき、無料枠でsmtpメール設定ができなかったため、総合で安く済む日本サーバーでの運用を考えた。
# 機能一覧

[機能全般](https://docs.google.com/spreadsheets/d/1EidbhsEDfPob-SpcRWWKjWHrd_fIpGc0WlMhUA_Qgu4/edit?usp=sharing)
- ユーザー認証(ログイン/ログアウト,アカウント作成,アカウント削除,pass再設定/更新,メール認証/)
