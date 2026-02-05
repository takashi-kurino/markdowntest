# 認証付きTodoApp

ステートフル認証付きtodoアプリ
    [サイト](http://apple.com)
    
## 1. 技術スタック　選定理由
- Frontend:React/Next.js/Tailwind<br>
    React：コンポーネント分割と状態管理により、TodoのCRUDやモーダル管理を整理しやすいため。<br>
    Next：ルーティング・環境変数・ビルドを含めた実運用前提の構成を学ぶため。今回はApp Routerを利用<br>
    Tailwind:UI設計よりもロジックと認証実装に集中するため。

- Backed:Django <br>
    認証・管理画面やORMが揃っており、webアプリの基礎構造を学ぶのに適している。<br>
    将来的な拡張（非同期通信、API分離）も想定。

- Auth: dj_rest_auth + JWT + HttpOnly Cookie<br>
    JWTを含めた認証の仕組みを歴史的背景（セッション / トークン）から理解したかった<br>
    フロントにトークンを持たせない設計を採用し、セキュリティ面も考慮

- Infra : Docker / Nginx<br>
    環境差異をなくし、ローカル〜本番で同一構成を保つため<br>
    リバースプロキシの役割を理解する目的も含む

- deploy : さくらVPS<br>
    マネージドサービスではなく自分で構成・運用する経験を重視<br>
    認証メール（SMTP）や費用面を考慮し日本のVPSを選択

## 2. 機能一覧　

- ユーザー認証<br>
    登録/ログイン/ログアウト・アカウント削除・パスワード再設定/更新・メール認証
- Todo管理<br>
    作成/更新/削除/完了切り替え


## 3. 設計・構成の考え方

- フロント/バックを分けた理由<br>
    webだけでなく、ネイティブへ対応を考慮<br>
    責務の明確化。フロントは表示のみ、バックでコアプログラムの管理。<br>
    他言語やフレームワークへの移行の簡略化。

- 認証の管理<br>
    dj_rest_authを利用。

- url設計<br>
    [スプレットシートへ](https://docs.google.com/spreadsheets/d/1EidbhsEDfPob-SpcRWWKjWHrd_fIpGc0WlMhUA_Qgu4/edit?usp=sharing)

## 4. 苦労した点
### dj_rest_authのパスワードリセットメールの解決。

- 以下の問題が上がっていた。<br>
    [stack overflow](https://stackoverflow.com/questions/77077297/dj-rest-auth-password-reset-serializer-is-not-working)
    
    フロントのURLへ飛ばしたかったため、カスタムserializerを作成したが、uidが文字化けを起こす。<br>

    理想url：http://localhost/password-reset/confirm?uid=1&token={...}<br>
    結果url：http://localhost/password-reset/confirm?uid=A&token={...}<br>

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
    uidを作成している生のコードをインポート
    ``` 
    from dj_rest_auth.forms import user_pk_to_url_str
    ```

    カスタムのurlに反映
    ``` 
    def custom_url_generator(request, user, temp_key):
        id = user_pk_to_url_str(user)
        return f'{NEXT_PUBLIC_URL}/password-reset/confirm?uid={id}&token={temp_key}'
    ```

    メールオプションに反映
    ``` 
    class CustomPasswordResetSerializer(PasswordResetSerializer):

        def get_email_options(self, **kwargs):
            return {
                'url_generator': custom_url_generator,
            }
    ```

    結果url：http://localhost/password-reset/confirm?uid=1&token={...}<br>
    AIに聞いても解決できなかったことが、生のコードを見ることで解決した。<br>
    かなり苦労したが、自力で解決できたことが嬉しかった。

### リフレッシュトークンの扱い。無限ループ


## 次回
Nextのfetchを中心としたauth0の利用。
BFFの採用。
家計簿アプリの作成。