# 認証付きTodoApp

CookieベースJWT認証付きTodoApp
    [サイト](http://apple.com)
    
## 1. 技術スタック　選定理由
- Frontend : React / Next.js / Tailwind 
    - React：コンポーネント分割と状態管理により、TodoのCRUDやモーダル管理を整理しやすいため。 
    - Next：ルーティング・環境変数・ビルドを含めた実運用前提の構成を学ぶため。今回はApp Routerを利用 
    - Tailwind:UI設計よりもロジックと認証実装に集中するため。

- Backed : Django  
    - 認証・管理画面やORMが揃っており、webアプリの基礎構造を学ぶのに適している。 
    - 将来的な拡張（非同期通信、API分離）も想定。

- Auth : dj_rest_auth + JWT + HttpOnly Cookie
    - JWTを含めた認証の仕組みを歴史的背景（セッション / トークン）から理解したかった
    - フロントにトークンを持たせない設計を採用し、セキュリティ面も考慮

- Infra : Docker / Nginx
    - 環境差異をなくし、ローカル〜本番で同一構成を保つため
    - リバースプロキシの役割を理解する目的も含む

- deploy : さくらVPS
    - マネージドサービスではなく自分で構成・運用する経験を重視
    - 認証メール（SMTP）や費用面を考慮し日本のVPSを選択

## 2. 機能一覧

- ユーザー認証
    - Cookieベースの認証を利用したログイン状態管理
    - ユーザー登録 / ログイン / ログアウト
    - アカウント削除
    - パスワード再設定 / 更新
    - メール認証
- Todo管理
    - 認証ユーザーごとのTodo管理
    - Todo作成 / 更新 / 削除
    - 完了・未完了の切り替え

## 3. 設計・構成の考え方

- フロント/バックを分けた理由
    - Webだけでなく、将来的なモバイル・ネイティブクライアントからの利用も想定し、
      APIを中心とした構成とした
    - 責務を明確に分離し、フロントエンドは表示・入力・API呼び出しに専念し、
      ビジネスロジックやデータ管理はバックエンドで一元管理するため
    - フレームワークや言語を変更する場合でも、
      影響範囲を限定できる構成にするため

- 認証方式
    - JWTをHttpOnly Cookieに保存する方式を採用
    - フロントエンドではトークンを保持せず、認証状態は常にバックエンドに問い合わせる設計
    - アクセストークン失効時はリフレッシュトークンによって再発行されるため、フロント側での状態管理は「ログインしている前提」ではなく「都度確認」とした

- url設計
    - 認証・Todo操作をREST APIとして整理し、フロントエンドや将来的な他クライアントからも利用しやすい構成とした
    - 実装と仕様のズレを防ぐため、エンドポイント一覧をスプレッドシートで管理
    - [スプレットシートへ](https://docs.google.com/spreadsheets/d/1EidbhsEDfPob-SpcRWWKjWHrd_fIpGc0WlMhUA_Qgu4/edit?usp=sharing)

## 4. 苦労した点

### 設計を途中で変更したことによる手戻り

- React（CSR）前提で、コンポーネントや hooks を先に作成して開発を進めた

    - 開発途中で Next.js における SSR の考え方を理解し、「SSR 前提の設計」に寄せようとした
    - その時点で UI やデータ取得処理がほぼ完成しており、サーバーコンポーネント／クライアントコンポーネントの分離やデータ取得方法を大きく見直す必要が生じた
    - 作り直しの工数が大きいと判断し、今回は CSR を前提とした構成に戻り、フロントエンドからバックエンドへは axios を用いて連携する形を採用した

- 学び

    - Next.js を利用する場合、SSR / CSR の責務を最初に整理し、それを前提に設計を固めることが重要だと実感した
    - 途中で設計を変更すると、実装コストだけでなく思考の再整理も必要になり、開発効率やモチベーションに影響することを学んだ

### SSRとCSRの扱いの違い

- Next.jsではSSRを前提とした設計が可能だが、
  Client Componentとの責務分担やデータ取得方法の切り分けに難しさを感じた
- これまでReactのCSRでAPI通信を実装していたため、
  今回は既存の実装方針（axios + Client Component）をそのまま採用した
- サーバーでレンダリングする部分と、クライアントで状態を持つ部分の
  組み合わせを途中で変更すると設計が崩れると感じたため、
  当初決めた構成を優先し、まずは完成させることを重視した
- 次回はNext.jsが推奨するfetchを用いたデータ取得や、
  SSRを前提とした認証・通信設計に取り組む予定

### dj_rest_authのパスワードリセットメールの解決。

- 以下の問題が上がっていた。

    [stack overflow](https://stackoverflow.com/questions/77077297/dj-rest-auth-password-reset-serializer-is-not-working)
    
    フロントのURLへ飛ばしたかったため、カスタムserializerを作成したが、uidが文字化けを起こす。 

    理想url：http://localhost/password-reset/confirm?uid=1&token={...} 
    ```
    uid = 1
    ```
    結果url：http://localhost/password-reset/confirm?uid=A&token={...} 
    ```
    uid = A
    ```
    
    chatgptへ聞いたが解決せず、デフォルトコードを自分で見てみる

    ```
    class AllAuthPasswordResetForm(DefaultPasswordResetForm):
        def clean_email(self):
            """
            Invalid email should not raise error, as this would leak users
            for unit test: test_password_reset_with_invalid_email
            """
            email = self.cleaned_data["email"]
            email = get_adapter().clean_email(email)
            self.users = filter_users_by_email(email, is_active=True)
            return self.cleaned_data["email"]

        def save(self, request, **kwargs):
            current_site = get_current_site(request)
            email = self.cleaned_data['email']
            token_generator = kwargs.get('token_generator', default_token_generator)

            for user in self.users:

                temp_key = token_generator.make_token(user)

                # save it to the password reset model
                # password_reset = PasswordReset(user=user, temp_key=temp_key)
                # password_reset.save()

                # send the password reset email
                url_generator = kwargs.get('url_generator', default_url_generator)
                url = url_generator(request, user, temp_key)
                uid = user_pk_to_url_str(user)

                context = {
                    'current_site': current_site,
                    'user': user,
                    'password_reset_url': url,
                    'request': request,
                    'token': temp_key,
                    'uid': uid,
                }
                if (
                    allauth_account_settings.AUTHENTICATION_METHOD != allauth_account_settings.AuthenticationMethod.EMAIL
                ):
                    context['username'] = user_username(user)
                get_adapter(request).send_mail(
                    'account/email/password_reset_key', email, context
                )
            return self.cleaned_data['email']
    ```

    uidを制御しているコードを発見。受け取ったユーザーをuser_pk_to_url_strで変換していることがわかった。
    ```
    uid = user_pk_to_url_str(user)
    ```
    これをカスタムシリアライザーに反映する。

- 解決策コード
    ``` 
    import os
    from dj_rest_auth.serializers import PasswordResetSerializer
    from dj_rest_auth.forms import user_pk_to_url_str

    NEXT_PUBLIC_URL = os.getenv("NEXT_PUBLIC_URL")

    def custom_url_generator(request, user, temp_key):
        id = user_pk_to_url_str(user)
        return f'{NEXT_PUBLIC_URL}/password-reset/confirm?uid={id}&token={temp_key}'

    class CustomPasswordResetSerializer(PasswordResetSerializer):

        def get_email_options(self, **kwargs):
            return {
                'url_generator': custom_url_generator,
            }
    ```

- 解説
    - uidを作成している生のコードをインポート
    ``` 
    from dj_rest_auth.forms import user_pk_to_url_str
    ```

    - カスタムのurlに反映
    ``` 
    def custom_url_generator(request, user, temp_key):
        id = user_pk_to_url_str(user)
        return f'{NEXT_PUBLIC_URL}/password-reset/confirm?uid={id}&token={temp_key}'
    ```

    - メールオプションに反映
    ``` 
    class CustomPasswordResetSerializer(PasswordResetSerializer):

        def get_email_options(self, **kwargs):
            return {
                'url_generator': custom_url_generator,
            }
    ```

    - 結果url：http://localhost/password-reset/confirm?uid=1&token={...} 
    - 公式実装を確認し、uid生成ロジックを特定したことで解決した。 

### リフレッシュトークンの扱い。

- リフレッシュ処理時、リフレッシュトークンがない場合にホーム画面とログイン画面で無限ループに陥った。（ログイン状態がヘッダーにあるため）

    コード全体。今回の通信はaxiosを利用したためインターセプターで処理を分岐。
    ``` axiosapi.ts

    import axios from "axios";
    import endpoints from "./apiEndpoints";

    const getBaseURL = () => {
    if (typeof window !== "undefined") {
        return `${window.location.protocol}//${window.location.host}/api/`;
    }
    return process.env.NEXT_PUBLIC_API_URL || "http://localhost/api/";
    };

    // Axiosインスタンス作成
    const api = axios.create({
        baseURL: getBaseURL(),
        withCredentials: true,
    });

    export const refreshApi = axios.create({
        baseURL: getBaseURL(),
        withCredentials: true,
    });

    api.interceptors.response.use(
    res => res,
    async error => {

        // ログインページ,homeではリフレッシュ処理を行わない
        if (window.location.pathname === "/login" || window.location.pathname === "/") {
            return Promise.reject(error);
        }

        const original = error.config;

        // refresh API 自身は無視
        if (original.url?.includes(endpoints.auth.refresh())) {
            return Promise.reject(error);
        }

        if (
            error.response?.status === 401 &&
            !original._retry
        ) {
            original._retry = true;
        

            try {
                await refreshApi.post(endpoints.auth.refresh());
                return api(original);
            } catch {
                // refresh token 無い or 期限切れ
                window.location.href = "/login";
                return Promise.reject(error);
            }
        }

        return Promise.reject(error);
    }
    );
    export default api;
    ```
    下記コードを追加して無限ループを止めるようにした。

    ```
    // ログインページ,homeではリフレッシュ処理を行わない
    if (window.location.pathname === "/login" || window.location.pathname === "/") {
        return Promise.reject(error);
    }
    ```
## 次回（学習・発展予定）

- Next.js の fetch を中心とした認証フローの実装
    - サーバーコンポーネントと認証の関係を理解するため
- Auth0 を利用した外部認証の検証
    - 自前実装との違いや責務分離を比較する目的
- BFF（Backend For Frontend）構成の検討
    - フロント専用APIを設けた場合の設計と実装を試す
- 家計簿アプリの作成
    - 認証・CRUD・集計処理を含む別ドメインでの再実装
