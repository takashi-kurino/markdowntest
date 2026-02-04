# 認証付きTodoApp

## なんのアプリか
- ステートフル認証付きtodoアプリ
    
## 技術スタック　選定理由
- Frontend:React/Next.js/Tailwind
    - React：コンポーネント分割と状態管理により、TodoのCRUDやモーダル管理を整理しやすいため。
    - Next：ルーティング・環境変数・ビルドを含めた実運用前提の構成を学ぶため。
    今回はApp Routerを利用
    - Tailwind
    UI設計よりもロジックと認証実装に集中するため。

- Backed:Django 
    - 認証・管理画面やORMが揃っており、webアプリの基礎構造を学ぶのに適している。
    将来的な拡張（非同期通信、API分離）も想定。
- Auth: dj_rest_auth + HttpOnly Cookie
    - 認証に関しての歴史と実装を兼ねた

- Infra : Docker / Nginx
    - Dcokerは環境が異なってもすぐに実行できることと、

- deploy : さくらVPS
    - awsやGCP,azuruも考えたが、まずは歴史をおさえて、自分の思想での設計を重視したかったため。nextはvercelを,djangoはrenderを使えば楽だが、認証の際に無料枠でsmtpメール設定ができなかったため、総合で安く済む日本サーバーでの運用を考えた。
## 機能一覧　[スプレットシート](https://docs.google.com/spreadsheets/d/1EidbhsEDfPob-SpcRWWKjWHrd_fIpGc0WlMhUA_Qgu4/edit?usp=sharing)

機能全般；
- ユーザー認証(ログイン/ログアウト,アカウント作成,アカウント削除,パスワード再設定/更新,メール認証/)
