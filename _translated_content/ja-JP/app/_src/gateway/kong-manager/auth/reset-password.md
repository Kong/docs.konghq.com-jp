---
title: "Kong Managerでパスワードと RBAC トークンをリセットする"
badge: "enterprise"
---
認証に関しては、Kongは管理者の2つの異なる認証情報を使用します。

1. 管理者は **パスワードを使って** Kong Managerにログインします。
2. 管理者は **RBACトークン** を使用してKong Admin APIにリクエストを行います。

[ベーシック認証](/gateway/{{page.release}}/kong-manager/auth/basic/)を使用している場合、管理者は Kong Manager 内からパスワードをリセットできます。 **LDAP** および **OIDC 認証** は、組織が Kong の外部でパスワードを保存および管理することを意味するため、どちらのタイプでもパスワードのリセットはできません。

各RBACトークンはハッシュとしてKongに保存されます。選択した認証オプションに関係なく、管理者はKong Manager内からRBACトークンをリセットできます。機密性を保つため、RBACトークンはハッシュ化されており、作成後は取得できないことに注意してください。ユーザーがトークンを忘れた場合、リセットする以外の手段はありません。

忘れたパスワードをKong Managerでリセット
--------------------------

### 前提条件

* 認証とRBACが[ベーシック認証](/gateway/{{page.release}}/kong-manager/auth/rbac/enable/)で[有効化](/gateway/{{page.release}}/kong-manager/auth/basic/)されること
* [SMTP](/gateway/{{page.release}}/kong-manager/configuring-to-send-email/)は電子メールを送信するように設定されています

### 手順

1. ログインページで、 **パスワードを忘れた場合** をクリックします。
2. アカウントに関連付けられているメールアドレスを入力します。
3. メールに記載されているリンクをクリックします。
4. パスワードをリセットします。リセットが完了したらすぐに再度入力する必要があることに注意してください。
5. 新しいパスワードでログインします。

Kong Managerからパスワードを変更する
------------------------

### 前提条件

* 認証とRBACが[ベーシック認証](/gateway/{{page.release}}/kong-manager/auth/rbac/enable/)で[有効化](/gateway/{{page.release}}/kong-manager/auth/basic/)されること
* [スーパー管理者権限](/gateway/{{page.release}}/kong-manager/auth/super-admin/)があるか、`/admins`と`/rbac`の読み取りと書き込みアクセス権を持つユーザーがいること

### 手順

1. **アカウント名** からドロップダウンメニューを開き、 **プロフィール** を選択します。
2. **パスワードのリセット** セクションでフィールドに入力し、 **パスワードのリセット** ボタンをクリックします。

Kong ManagerでのRBACトークンのリセット
---------------------------

### 前提条件

* 認証とRBACが[有効](/gateway/{{page.release}}/kong-manager/auth/rbac/enable/)である
* [スーパー管理者権限](/gateway/{{page.release}}/kong-manager/auth/super-admin/)があるか、`/admins`と`/rbac`の読み取りと書き込みアクセス権を持つユーザーがいること

### 手順

1. **アカウント名** からドロップダウンメニューを開き、 **プロフィール** を選択します。
2. **RBACトークンリセット** セクションで、 **トークンのリセット** をクリックし、リセットを確認します。
3. **コピー（Copy）** をクリックして、トークンをコピーします。

