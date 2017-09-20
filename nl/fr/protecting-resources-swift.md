---

copyright:
  years: 2017
lastupdated: "2017-05-08"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}


# Protection des ressources Swift
{: #protecting-resources-swift}

Vous pouvez utiliser le SDK serveur {{site.data.keyword.appid_short}} pour protéger les ressources dans votre application Swift.
{:shortdesc}


## Avant de commencer
{: #before-you-begin}

* Familiarisez-vous avec le développement d'applications Swift sur {{site.data.keyword.Bluemix}}.
* Installez les logiciels suivants :
    * Swift 3.0.2
    * Kitura 1.6


## Installation du SDK

1. Ouvre le fichier `Package.swift` situé dans le répertoire de votre application Swift et ajoutez la dépendance `appid-serversdk-swift`. Exemple :

  ```swift
  import PackageDescription

  let package = Package(
      dependencies: [
          .Package(url: "https://github.com/ibm-cloud-security/appid-serversdk-swift.git", majorVersion: 1)
      ]
  )
  ```
  {:pre}

## Protection des ressources dans Swift
{: #protecting}

Le SDK Swift fournit un plug-in de données d'identification Kitura qui permet de
protéger les applications Web. Si vous utilisez ce plug-in, les clients non authentifiés
reçoivent une réponse HTTP 302.
Ils sont redirigés vers la page de connexion hébergée par
{{site.data.keyword.appid_short_notm}} ou vers la page de connexion du fournisseur d'identité, selon votre configuration.



### Protection des applications Web
{: protect-webapps notoc}

Le plug-in WebAppKituraCredentialsPlugin est basé sur le flux d'octroi d'autorisation OAuth2 authorization_code et doit être utilisé pour les applications Web utilisant des navigateurs. Le plug-in fournit des outils pour implémenter les flux d'authentification et d'autorisation. Il fournit également des mécanismes pour détecter des tentatives non authentifiées d'accès à des ressources protégées et redirige alors automatiquement le navigateur de l'utilisateur vers la page d'authentification. Si l'authentification aboutit, l'utilisateur est dirigé vers l'URL de rappel de l'application Web, laquelle utilise le plug-in pour obtenir des jetons d'accès et d'identité auprès d'{{site.data.keyword.appid_short_notm}}. Après avoir obtenu ces jetons, le plug-in les stocke dans une session HTTP sous WebAppKituraCredentialsPlugin.AuthContext.

Le code ci-dessous explique comment utiliser le plug-in
WebAppKituraCredentialsPlugin dans une application Kitura pour protéger le noeud final
`/protected`.


  ```swift
  import Foundation
  import Kitura
  import KituraSession
  import Credentials
  import SwiftyJSON
  import BluemixAppID

  // The following URLs will be used for AppID OAuth flows
  var LOGIN_URL = "/ibm/bluemix/appid/login"
  var CALLBACK_URL = "/ibm/bluemix/appid/callback"
  var LOGOUT_URL = "/ibm/bluemix/appid/logout"
  var LANDING_PAGE_URL = "/index.html"
  // Setup Kitura to use session middleware
  // Must be configured with proper session storage for production
  // environments. See https://github.com/IBM-Swift/Kitura-Session for
  // additional documentation
  let router = Router()
  let session = Session(secret: "Some secret")
  router.all(middleware: session)
  let options = [
  	"clientId": "{client-id}",
  	"secret": "{secret}",
  	"tenantId": "{tenant-id}",
  	"oauthServerUrl": "{oauth-server-url}",
  	"redirectUri": "{app-url}" + CALLBACK_URL
  ]
  let webappKituraCredentialsPlugin = WebAppKituraCredentialsPlugin(options: options)
  let kituraCredentials = Credentials()
  kituraCredentials.register(plugin: webappKituraCredentialsPlugin)

  // Explicit login endpoint
  router.get(LOGIN_URL,
  		   handler: kituraCredentials.authenticate(credentialsType:
webappKituraCredentialsPlugin.name,
  												   successRedirect: LANDING_PAGE_URL,
  												   failureRedirect: LANDING_PAGE_URL
  ))
  // Callback to finish the authorization process. Will retrieve access and identity
tokens from AppID
  router.get(CALLBACK_URL,
  		   handler: kituraCredentials.authenticate(credentialsType:
webappKituraCredentialsPlugin.name,
  												   successRedirect: LANDING_PAGE_URL,
  												   failureRedirect: LANDING_PAGE_URL
  ))
  // Logout endpoint. Clears authentication information from session
  router.get(LOGOUT_URL, handler:  { (request, response, next) in
  	kituraCredentials.logOut(request: request)
  	webappKituraCredentialsPlugin.logout(request: request)
  	_ = try? response.redirect(LANDING_PAGE_URL)
  })

  // Protected area
  router.get("/protected", handler: kituraCredentials.authenticate(credentialsType:
webappKituraCredentialsPlugin.name), { (request, response, next) in
      let appIdAuthContext:JSON? = request.session?[WebAppKituraCredentialsPlugin.AuthContext]
      let identityTokenPayload:JSON? = appIdAuthContext?["identityTokenPayload"]

      guard appIdAuthContext?.dictionary != nil, identityTokenPayload?.dictionary != nil else {
          response.status(.unauthorized)
          return next()
      }

      response.send(json: identityTokenPayload!)
      next()
  })
  var port = 3000
  if let portString = ProcessInfo.processInfo.environment["PORT"]{
      port = Int(portString)!
  }
  print("Starting on \(port)")

  // Add an HTTP server and connect it to the router
  Kitura.addHTTPServer(onPort: port, with: router)
  // Start the Kitura runloop (this call never returns)
  Kitura.run()
  ```
  {:pre}

Pour plus d'informations, voir le
<a href="https://github.com/ibm-cloud-security/appid-serversdk-swift" target="_blank">référentiel
GitHub Swift pour {{site.data.keyword.appid_short_notm}}<img src="../../icons/launch-glyph.svg" alt="Icône de lien externe"></a>.
