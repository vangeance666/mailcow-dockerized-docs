SOGo wird verwendet, um über einen Webbrowser auf Ihre Mails zuzugreifen und Ihre Kontakte oder Kalender hinzuzufügen und zu teilen. Für eine ausführlichere Dokumentation zu SOGo besuchen Sie bitte die [eigene Dokumentation] (http://wiki.sogo.nu/).

## Benutzerdefiniertes SOGo-Thema (CSS) anwenden
mailcow-Builds nach dem 28. Januar 2021 können das CSS-Thema von SOGo ändern, indem sie `data/conf/sogo/custom-theme.js` bearbeiten.
Bitte schauen Sie sich die AngularJS Material [intro](https://material.angularjs.org/latest/Theming/01_introduction) und [documentation](https://material.angularjs.org/latest/Theming/03_configuring_a_theme) sowie die [material style guideline](https://material.io/archive/guidelines/style/color.html#color-color-palette) an, um zu erfahren, wie das funktioniert. 

Sie können die mitgelieferte `custom-theme.js` als Beispiel verwenden, indem Sie die Kommentare entfernen.
Nachdem Sie `data/conf/sogo/custom-theme.js` modifiziert und Änderungen an Ihrem neuen SOGo-Theme vorgenommen haben, müssen Sie 

1. Bearbeiten Sie `data/conf/sogo/sogo.conf` und fügen Sie `SOGoUIxDebugEnabled = YES;` ein.
2. SOGo und Memcached Container neu starten, indem man `docker compose restart memcached-mailcow sogo-mailcow` ausführt.
3. SOGo im Browser öffnen
4. öffnen Sie die Entwicklerkonsole des Browsers, normalerweise ist die Tastenkombination F12
5. nur wenn Sie Firefox benutzen: schreiben Sie mit der Hand in die Entwicklerkonsole `allow pasting` und drücken Sie Enter
6. fügen Sie den Java-Script-Schnipsel in die Entwicklungskonsole ein:
```
copy([].slice.call(document.styleSheets)
  .map(e => e.ownerNode)
  .filter(e => e.hasAttribute('md-theme-style'))
  .map(e => e.textInhalt)
  .join('\n')
)
```
7. Öffnen Sie den Texteditor und fügen Sie die Daten aus der Zwischenablage ein (Strg+V), Sie sollten ein minimiertes CSS erhalten, speichern Sie es
8. kopieren Sie die CSS-Datei auf den Mailcow-Server `data/conf/sogo/custom-theme.css`
9. editiere `data/conf/sogo/sogo.conf` und setze `SOGoUIxDebugEnabled = NO;`
10. Anhängen/Erstellen von `docker-compose.override.yml` mit:
```
Version: '2.1'

Dienste:
  sogo-mailcow:
    volumes:
      - ./data/conf/sogo/custom-theme.css:/usr/lib/GNUstep/SOGo/WebServerResources/css/theme-default.css:z
```
11. führen Sie `docker compose up -d` aus
12. Ausführen von `docker compose restart memcached-mailcow`

## Zurücksetzen auf das SOGo Standardthema
1. checken Sie `data/conf/sogo/custom-theme.js` aus, indem Sie `git fetch ; git checkout origin/master data/conf/sogo/custom-theme.js data/conf/sogo/custom-theme.js` ausführen
2. Suchen Sie in `data/conf/sogo/custom-theme.js`:
```
// Neue Paletten auf das Standardthema anwenden, einige Farbtöne neu zuordnen
    $mdThemingProvider.theme('default')
      .primaryPalette('green-cow', {
        'default': '400', // Hintergrundfarbe der oberen Symbolleisten
        hue-1': '400',
        'hue-2': '600', // Hintergrundfarbe der Seitenleiste
        'hue-3': 'A700'
      })
      .accentPalette('green', {
        'default': '600', // Hintergrundfarbe der Fab-Schaltflächen und des Anmeldebildschirms
        hue-1': '300', // Hintergrundfarbe der Symbolleiste der mittleren Liste
        hue-2': '300', // Hervorhebungsfarbe für ausgewählte Nachrichten und den aktuellen Tageskalender
        hue-3': 'A700'
      })
      .backgroundPalette('frost-grey');
```
und ersetzen Sie es durch:
```
    $mdThemingProvider.theme('default');
```
3. Entfernen Sie aus `docker-compose.override.yml` Volume Mount in `sogo-mailcow`:
```
- ./data/conf/sogo/custom-theme.css:/usr/lib/GNUstep/SOGo/WebServerResources/css/theme-default.css:z
```
4. führen Sie `docker compose up -d` aus
5. Starten Sie `docker compose restart memcached-mailcow`.

## Favicon ändern
mailcow-Builds nach dem 31. Januar 2021 können SOGo's Favicon ändern, indem sie `data/conf/sogo/custom-favicon.ico` für SOGo und `data/web/favicon.png` für mailcow UI ersetzen.
**Anmerkung**: Sie können `.png` Favicons für SOGo verwenden, indem Sie sie in `custom-favicon.ico` umbenennen.
Für beide, SOGo und mailcow UI Favicons, müssen Sie eine der Standardgrößen verwenden: 16x16, 32x32, 64x64, 128x128 und 256x256.
Nachdem Sie diese Datei ersetzt haben, müssen Sie SOGo und Memcached Container neu starten, indem Sie `docker compose restart memcached-mailcow sogo-mailcow` ausführen.

## Logo ändern
Mailcow-Builds nach dem 21. Dezember 2018 können das SOGo-Logo ändern, indem sie die Datei `data/conf/sogo/sogo-full.svg` ersetzen oder erstellen (falls sie fehlt).
Nachdem Sie diese Datei ersetzt haben, müssen Sie SOGo und Memcached Container neu starten, indem Sie `docker compose restart memcached-mailcow sogo-mailcow` ausführen.

## Domains verbinden (untereinander sichtbar machen)
Domains sind normalerweise voneinander isoliert.

Sie können das ändern, indem Sie `data/conf/sogo/sogo.conf` modifizieren:

Suche...
```
   // SOGoDomainsVisibility = (
    // (domain1.tld, domain5.tld),
    // (domain3.tld, domain2.tld)
    // );
```
...und ersetzen Sie diese durch - zum Beispiel:

```
    SOGoDomainsVisibility = (
      (beispiel.org, beispiel.com, beispiel.net)
    );
```

SOGo neu starten: `docker compose restart sogo-mailcow`

## Deaktivieren Sie die Passwortänderung

Bearbeiten Sie `data/conf/sogo/sogo.conf` und **ändern** Sie `SOGoPasswordChangeEnabled` auf `NO`. Bitte fügen Sie keinen neuen Parameter hinzu.

Führen Sie `docker compose restart memcached-mailcow sogo-mailcow` aus, um die Änderungen zu aktivieren.

## TOTP zurücksetzen / TOTP deaktivieren

Führen Sie `docker compose exec -u sogo sogo-mailcow sogo-tool user-preferences set defaults user@example.com SOGoTOTPEnabled '{"SOGoTOTPEnabled":0}'` aus dem mailcow Verzeichnis aus.

