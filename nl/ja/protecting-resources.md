---

copyright:
  years: 2017
lastupdated: "2017-11-02"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}

# バックエンド・リソースの保護
{: #protecting-resources}

Client SDK を使用して保護リソースにアクセスできます。
{:shortdesc}

必要な場合は、保護リソースの呼び出しによってログイン・ウィジェットが開始します。有効なトークンが既に取得されている場合、ログイン・ウィジェットは開始されず、リソースに直接アクセスします。


## iOS Swift SDK の使用
{: #requesting-swift}

1. BMSCore をインポートします。

  ```swift
  import BMSCore
  ```
  {: codeblock}

2. 保護リソース要求を呼び出します。

  ```swift
  BMSClient.sharedInstance.initialize(bluemixRegion: AppID.<region>)
  BMSClient.sharedInstance.authorizationManager = AppIDAuthorizationManager(appid:AppID.sharedInstance)
  var request:Request =  Request(url: "<your protected resource url>")
  request.send(completionHandler: {(response:Response?, error:Error?) in
      //応答処理コードをここに記述
  })
  ```
  {: codeblock}


## Android SDK の使用
{: #requesting-android}

1. 保護リソース要求を呼び出します。

  ```java
  BMSClient bmsClient = BMSClient.getInstance();
  bmsClient.initialize(getApplicationContext(), AppID.REGION_UK);

  AppIDAuthorizationManager appIdAuthMgr = new AppIDAuthorizationManager(AppID.getInstance())
  bmsClient.setAuthorizationManager(appIdAuthMgr);

  Request request = new Request("<your protected resource url>", Request.GET);
  request.send(this, new ResponseListener() {
  @Override
		public void onSuccess (Response response) {
			//応答処理コードをここに記述
  }
  @Override
		public void onFailure (Response response, Throwable t, JSONObject extendedInfo) {
			//エラー処理コードをここに記述
  });
  ```
  {: codeblock}
