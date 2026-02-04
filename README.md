# 認証付きTodoApp

## なんのアプリか
- ステートフル認証付きtodoアプリ
    -[サイト](http://apple.com)
    
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
    - JWTを含めた認証の仕組みを歴史的背景（セッション / トークン）から理解したかった
    - フロントにトークンを持たせない設計を採用し、セキュリティ面も考慮
- Infra : Docker / Nginx
    - 環境差異をなくし、ローカル〜本番で同一構成を保つため
    - リバースプロキシの役割を理解する目的も含む

- deploy : さくらVPS
    - マネージドサービスではなく自分で構成・運用する経験を重視
    - 認証メール（SMTP）や費用面を考慮し日本のVPSを選択

## 機能一覧　

機能全般；
- ユーザー認証
    - 登録/ログイン/ログアウト,アカウント削除,パスワード再設定/更新,メール認証
- Todo管理
    - 作成・更新・削除・完了切り替え
- [url設計](https://docs.google.com/spreadsheets/d/1EidbhsEDfPob-SpcRWWKjWHrd_fIpGc0WlMhUA_Qgu4/edit?usp=sharing)
