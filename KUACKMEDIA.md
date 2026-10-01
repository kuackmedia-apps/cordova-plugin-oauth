# Fork de kuackmedia

Fork de [AyogoHealth/cordova-plugin-oauth](https://github.com/AyogoHealth/cordova-plugin-oauth) a partir del tag
`v4.1.0` (Apache 2.0, ver `LICENCE`). Lo usa el login TuID (Antel ID) de las apps Cordova de Kuack (TG-2714): el app
abre la autorización de TuID en `ASWebAuthenticationSession` (iOS) o Custom Tabs (Android) y recibe el retorno
`antelmusic://oauth2redirect` como evento `message` (`oauth::{…}`).

Versión: `4.1.0-kuack.1`. Se instala desde `github:kuackmedia-apps/cordova-plugin-oauth#v4.1.0-kuack.1` con las
variables `URL_SCHEME` y `URL_HOSTNAME` (en las apps, la clave `oauthPlugin` del Gruntfile genera el bloque del
`config.xml`).

## Cambios respecto de upstream

Todos en Android; iOS queda igual. Las líneas cambiadas llevan el comentario `kuackmedia:`.

1. **`OAuthPlugin.onNewIntent` a prueba de nulos.** `onStart()` le pasa el intent de arranque de la activity en
   cada vuelta a primer plano. Upstream hacía `intent.getAction().equals(…)` y `uri.getHost().equals(…)`: un
   intent sin action, un `VIEW` sin data o una URI sin host tiraba `NullPointerException`.
2. **Se compara también el esquema**, no sólo el host (`oauthscheme` / `oauthhostname`).
3. **No repite el retorno.** Si el app arrancó en frío por el callback, `onStart()` volvía a despachar el mismo
   resultado (con un `code` ya usado) en cada vuelta a primer plano. Se recuerda la última URI despachada
   (`lastDispatchedUri`, estática). No se toca el intent de la activity porque otros plugins
   (`cordova-plugin-customurlscheme`) lo leen.
4. **`androidx.browser` 1.8.0** por default (`ANDROID_SUPPORT_CUSTOM_TABS_VERSION`; upstream 1.3.0).

## Para actualizar desde upstream

`git fetch upstream --tags`, rebase de la branch `kuack` sobre el tag nuevo, revisar que los cuatro puntos sigan
aplicando y taggear `vX.Y.Z-kuack.N`.
