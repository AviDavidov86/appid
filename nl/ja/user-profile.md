---

copyright:
  years: 2017
lastupdated: "2017-11-07"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}


# ユーザー・プロファイル
{: #user-profile}

ユーザー・プロファイルは、{{site.data.keyword.appid_full}} によって保管され保守されるエンティティーです。プロファイルには、ユーザーの属性と ID が入ります。プロファイルは、匿名にすることも、ID プロバイダーによって管理されている ID にリンクすることもできます。{:shortdesc}

{{site.data.keyword.appid_short_notm}} には、匿名ログインまたは OpenId Connect (OIDC) [ID プロバイダー](/docs/services/appid/identity-providers.html#setting-up-idp)による認証を使用したログインのための API が用意されています。ユーザー・プロファイル属性 API エンドポイントは、ログインと許可のプロセス中に {{site.data.keyword.appid_short_notm}} が生成するアクセス・トークンによって保護されるリソースです。


## ユーザー属性の保管、読み取り、削除
{: #storing-data}

{{site.data.keyword.appid_short_notm}} には、ユーザーの属性に対する操作を作成、検索、更新、および削除するための <a href="https://appid-profiles.ng.bluemix.net/swagger-ui/index.html#/Attributes" target="_blank">REST API <img src="../../icons/launch-glyph.svg" alt="外部リンク・アイコン"></a> が用意されています。このサービスには、<a href="https://github.com/ibm-cloud-security/appid-clientsdk-android" target="_blank">Android <img src="../../icons/launch-glyph.svg" alt="外部リンク・アイコン"></a> および <a href="https://github.com/ibm-cloud-security/appid-clientsdk-swift" target="_blank">Swift <img src="../../icons/launch-glyph.svg" alt="外部リンク・アイコン"></a> モバイル・クライアント用の SDK も用意されています。


## OAuth ID
{: #oauth}

サービスが OAuth ログイン API を呼び出すと、{{site.data.keyword.appid_short_notm}} は OAuth 2.0 と OIDC プロトコルを使用して、選択された ID プロバイダーで呼び出し元の許可と認証を行います。認証された後、ID は {{site.data.keyword.appid_short_notm}} ユーザー・レコードと関連付けられます。{{site.data.keyword.appid_short_notm}} は、ユーザーの属性にアクセスするために使用できるアクセス・トークンと、ID プロバイダーから提供された ID 情報を保持する識別トークンを返します。同じ ID で認証されたすべてのクライアントから、この同じユーザー・レコードと属性に再びアクセスできます。


## 匿名ユーザー
{: #anonymous}

匿名でログインすると、{{site.data.keyword.appid_short_notm}} は匿名のマークが付いた一時的なユーザー・レコードを作成します。{{site.data.keyword.appid_short_notm}} は、匿名のアクセス・トークンと識別トークンを呼び出し元に返します。この匿名アクセス・トークンを使用して、ユーザー・アプリケーションは、ユーザー・レコードに保管される属性の作成、読み取り、更新、および削除を行うことができます。例えば、開発者は、ユーザーがログインしなくてもすぐにショッピング・カートにアイテムを追加できるアプリケーションを作成できます。


## 識別されたユーザー
{: #identified}

ID プロバイダーによって提供された ID を持っている匿名ユーザーは、識別されたユーザーになることができます。匿名ユーザーから識別済みユーザーになるまでのフローをまとめると、次のようなステップになります。


* 開発者が匿名アクセス・トークンをログイン API に渡します。
* {{site.data.keyword.appid_short_notm}} が、呼び出し元を ID プロバイダーを使用して認証します。
* {{site.data.keyword.appid_short_notm}} が、アクセス・トークンで定義された匿名ユーザー・レコードを見つけ、ID を割り当てます。

    **注**: ID を匿名レコードに割り当てることができるのは、その同じ ID が他のユーザーにまだ割り当てられていない場合のみです。ID が既に別の {{site.data.keyword.appid_short_notm}} ユーザーと関連付けられている場合、アクセス・トークンと識別トークンにはそのユーザーのレコードの情報が含まれ、そのユーザーの属性へのアクセスを提供します。前の匿名ユーザーとその属性に、新しいアクセス・トークンを使用してアクセスすることはできません。トークンの有効期限が切れるまでは、匿名アクセス・トークンにより引き続き情報にアクセスできます。開発者は、匿名ユーザーと既知のユーザーの匿名属性をマージする方法を選択できます。

* {{site.data.keyword.appid_short_notm}} から受け取る新しいアクセス・トークンと識別トークンは既知のユーザーを指し、識別トークンには ID プロバイダーから受け取った公開情報が含まれています。
* 匿名トークンは、そのユーザーでは無効になります。

このユーザーが匿名であったときの属性には、新規アクセス・トークンでアクセスできます。新規トークンは、追加、変更、削除できます。この後、どのクライアントからでも同じ ID でユーザーがログインすると、ユーザーと属性は解決されてアクセス可能になります。


## データ分離と暗号化
{: #data}

{{site.data.keyword.appid_short_notm}} は、ユーザー・プロファイル属性を保管して暗号化します。マルチテナント・サービスでは、すべてのテナントには指定された暗号鍵がありそれぞれのテナントのユーザー・データはそのテナントの鍵だけで暗号化されます。

{{site.data.keyword.appid_short_notm}} は必ず、プライベートな資料を暗号化してから保管します。
