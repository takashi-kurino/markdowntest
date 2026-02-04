# 認証付きTodoApp

## なんのアプリか
- ステートフル認証付きtodoアプリ
    - [サイト](http://apple.com)
    
1. 技術スタック　選定理由
- Frontend:React/Next.js/Tailwind
    - React：コンポーネント分割と状態管理により、TodoのCRUDやモーダル管理を整理しやすいため。
    - Next：ルーティング・環境変数・ビルドを含めた実運用前提の構成を学ぶため。
    今回はApp Routerを利用
    - Tailwind
    UI設計よりもロジックと認証実装に集中するため。

- Backed:Django 
    - 認証・管理画面やORMが揃っており、webアプリの基礎構造を学ぶのに適している。
    将来的な拡張（非同期通信、API分離）も想定。
- Auth: dj_rest_auth + JWT + HttpOnly Cookie
    - JWTを含めた認証の仕組みを歴史的背景（セッション / トークン）から理解したかった
    - フロントにトークンを持たせない設計を採用し、セキュリティ面も考慮
- Infra : Docker / Nginx
    - 環境差異をなくし、ローカル〜本番で同一構成を保つため
    - リバースプロキシの役割を理解する目的も含む

- deploy : さくらVPS
    - マネージドサービスではなく自分で構成・運用する経験を重視
    - 認証メール（SMTP）や費用面を考慮し日本のVPSを選択

2. 機能一覧　

- ユーザー認証
    - 登録/ログイン/ログアウト・アカウント削除・パスワード再設定/更新・メール認証
- Todo管理
    - 作成/更新/削除/完了切り替え


3. 設計・構成の考え方

- フロント/バックを分けた理由
    - webだけでなく、ネイティブへ対応を考慮
    - 責務の明確化。フロントは表示のみ、バックでコアプログラムの管理。
    - 他言語やフレームワークへの移行の簡略化。

- 認証の管理
    - dj_rest_authを利用。

- url設計
    - [スプレットシートへ](https://docs.google.com/spreadsheets/d/1EidbhsEDfPob-SpcRWWKjWHrd_fIpGc0WlMhUA_Qgu4/edit?usp=sharing)

- 

4. 苦労した点
- リフレッシュトークンの扱い。無限ループ

## 次回
Nextのfetchを中心としたauth0の利用。
BFFの採用。
家計簿アプリの作成。