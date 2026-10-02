# Fork de kuackmedia

Fork de [AyogoHealth/cordova-plugin-oauth](https://github.com/AyogoHealth/cordova-plugin-oauth) a partir del tag
`v4.1.0` (Apache 2.0, ver `LICENCE`). Lo usa el login TuID (Antel ID) de las apps Cordova de Kuack (TG-2714): el app
abre la autorización de TuID en `ASWebAuthenticationSession` (iOS) o Custom Tabs (Android) y recibe el retorno
`antelmusic://oauth2redirect` como evento `message` (`oauth::{…}`).

Versión: `4.1.0-kuack.2`. **`main` es lo publicado**: las apps lo instalan desde
`github:kuackmedia-apps/cordova-plugin-oauth` (sin versión, como los demás forks de kuackmedia-apps), declarado en los
`config-app*.xml` de `_common/configTemplates` con `URL_SCHEME` = `shareSchema` y `URL_HOSTNAME` = `oauthCallbackHost`
del Gruntfile de cada app. Un push a `main` llega al próximo `initApp.sh` de **todas** las apps: los cambios se prueban
antes en una branch (en el app, con `config-app-local.xml`, que apunta al checkout local del plugin). El tag
`v4.1.0-kuack.1` marca la primera versión del fork.

## Cambios respecto de upstream

Los puntos 1 a 4 son de Android y el 5 suma una opción en el JS y en iOS. Las líneas cambiadas llevan el comentario
`kuackmedia:`.

1. **`OAuthPlugin.onNewIntent` a prueba de nulos.** `onStart()` le pasa el intent de arranque de la activity en
   cada vuelta a primer plano. Upstream hacía `intent.getAction().equals(…)` y `uri.getHost().equals(…)`: un
   intent sin action, un `VIEW` sin data o una URI sin host tiraba `NullPointerException`.
2. **Se compara también el esquema**, no sólo el host (`oauthscheme` / `oauthhostname`).
3. **No repite el retorno.** Si el app arrancó en frío por el callback, `onStart()` volvía a despachar el mismo
   resultado (con un `code` ya usado) en cada vuelta a primer plano. Se recuerda la última URI despachada
   (`lastDispatchedUri`, estática). No se toca el intent de la activity porque otros plugins
   (`cordova-plugin-customurlscheme`) lo leen.
4. **`androidx.browser` 1.8.0** por default (`ANDROID_SUPPORT_CUSTOM_TABS_VERSION`; upstream 1.3.0).
5. **Sesión privada opcional en iOS** (`4.1.0-kuack.2`): `window.open(url, 'oauth:<nombre>', 'ephemeral=yes')` abre
   `ASWebAuthenticationSession` con `prefersEphemeralWebBrowserSession` (iOS 13+). Sin cookies compartidas con Safari,
   iOS no muestra el aviso "«App» quiere usar «dominio» para iniciar sesión", que no se puede evitar de otra forma
   (no hay whitelist ni entitlement); a cambio, el usuario se identifica en cada login y el "recordar usuario" del
   proveedor no persiste. `www/oauth.js` pasa los `features` de `window.open` como 2º argumento de `startOAuth`
   (`{ ephemeral: bool }`); sin la opción, la sesión es la compartida de upstream. Android lee sólo el 1º argumento
   y la ignora (Custom Tabs no muestra ese aviso).

## Para actualizar desde upstream

`git fetch upstream --tags`, en una branch merge (o rebase) del tag nuevo, revisar que los cinco puntos sigan
aplicando, probarlo en un app con `config-app-local.xml`, llevarlo a `main` y taggear `vX.Y.Z-kuack.N`.
