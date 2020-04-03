# leafletgeoportal

## Allgemeine Anmerkungen

### Mobil

Die mobile Umsetzung ist nicht notwendig.

### Grenzen

Die Grenzen können in der Konfiguration eingestellt werden. Diese sind die 
maximalen Grenzen und auf diese wird zu beginnt gezoomt.

```
map.fitBounds(window.config.bounds);
map.setMaxBounds(window.config.bounds);
```

### Besonderheiten

### Warnungen und Hinweise - Alerts

Siehe [Gestaltungselemente](https://github.com/astridx/leafletgeoportal#gestaltungselemente)


### Map

#### Zoom mit Doppelklick

Üblicherweise kann eine digitale Landkarte durch Doppelklicken vergrößert und 
durch Doppelklicken bei gedrückter 
Umschalttaste verkleinert werden kann. Ich habe diese 
[Funktion](https://leafletjs.com/reference-1.6.0.html#map-doubleclickzoom) deaktiviert, weil 
dies zu Missverständnissen führen kann. Zum Beispiel beim Zeichnen oder Messen 
kann ein schnelles Klicken zu einem nicht gewollten Zoomen führen.

```
map.doubleClickZoom.disable(); 
```

#### CSS

Die Breite und Höhe der Karte wird per CSS an den Viewport ausgerichtet. 
Als Viewport wird der Bereich des Anzeigefensters bezeichnet, 
der für die Darstellung der Inhalte zur Verfügung steht. 

```
html,
body
{
	height: 100vh;
	width: 100vw;
}
```

## Geoserver

### Konfiguration

#### CORS

Um die Informationen zum WMS-Layer in einem Pop up anzeigen zu können, muss ich CORS aktivieren:

Risiko: Clickjacking. Clickjacking ist eine Technik, bei der ein Computerhacker die Darstellung einer Internetseite überlagert und dann deren Nutzer dazu veranlasst, scheinbar harmlose Mausklicks und/oder Tastatureingaben durchzuführen. (https://de.wikipedia.org/wiki/Clickjacking). Mit anderen Worten: Ein Hacker könnte den Geoserver auf seiner Website als IFrame so dominant einbinden, dass der Website Besucher glaubt, er sehe Inhalte des Ihres Webservers. Wenn er etwas anklickt, klickt er aber auf Buttons der eigentlichen bösartigen Seite und führt ein Schadprogramm aus.

Um CORS zu aktivieren habe ich in der Datei `web.xml` im Verzeichnis `webapps/Geoserver/WEB-INF` folgendes ausgekommentierte [aktiviert](https://docs.geoserver.org/latest/en/user/production/container.html).

```
   <!-- Uncomment following filter to enable CORS -->
   <filter>
        <filter-name>cross-origin</filter-name>
        <filter-class>org.eclipse.jetty.servlets.CrossOriginFilter</filter-class>
       <init-param>
           <param-name>chainPreflight</param-name>
           <param-value>false</param-value>
       </init-param>
       <init-param>
           <param-name>allowedOrigins</param-name>
           <param-value>*</param-value>
       </init-param>
       <init-param>
           <param-name>allowedMethods</param-name>
           <param-value>GET,POST,PUT,DELETE,HEAD,OPTIONS</param-value>
       </init-param>
       <init-param>
           <param-name>allowedHeaders</param-name>
           <param-value>*</param-value>
       </init-param>
    </filter>

...

   <!-- Uncomment following filter to enable CORS -->
    <filter-mapping>
        <filter-name>cross-origin</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

```

#### SQL View

##### Gemarkungsname

```
SELECT distinct gemarkungsname 
FROM sg_flurstueck_f
ORDER BY gemarkungsname ASC
```

##### Flur

```
SELECT distinct fln::int
FROM sg_flurstueck_f
WHERE gemarkungsname like '%gemarkungsname%'
ORDER BY fln::int ASC
```

##### Zähler

```
SELECT distinct fsn_zae::int
FROM sg_flurstueck_f
WHERE fsn_zae like '%fsn_zae%'
and fln = %fln% and
gemarkungsname like '%gemarkungsname%'
ORDER BY fsn_zae::int ASC
```

##### Nenner

```
SELECT distinct fsn_nen::int
FROM sg_flurstueck_f 
WHERE fsn_nen like '%fsn_nen%' and
fln = %fln% and
 fsn_zae like '%fsn_zae%' and
 gemarkungsname like '%gemarkungsname%'
ORDER BY fsn_nen::int ASC
```


#### Gespeicherte Abfrage

#### Schriftart

Für die Anzeige eines Layers muss die die Schriftart intalliert werden. 
Dies fürht zu Problemen bei der Abzeige der Legende.

```
20 Dez 11:23:44 WARN [geotools.styling] - can't parse ttf://Dingbats as a java resource present in the classpath
20 Dez 11:23:46 ERROR [geoserver.ows] -
java.lang.IllegalArgumentException: Unknown font Dingbats
        at org.geotools.renderer.style.TTFMarkFactory.getShape(TTFMarkFactory.java:98)
        at org.geotools.renderer.style.TTFMarkFactory.getShape(TTFMarkFactory.java:153)
        at org.geotools.renderer.style.SLDStyleFactory.getShape(SLDStyleFactory.java:1364)
        at org.geotools.renderer.style.SLDStyleFactory.createMarkStyle(SLDStyleFactory.java:643)
        at org.geotools.renderer.style.SLDStyleFactory.createPointStyle(SLDStyleFactory.java:608)
        at org.geotools.renderer.style.SLDStyleFactory.createPointStyle(SLDStyleFactory.java:483)
        at org.geotools.renderer.style.SLDStyleFactory.createStyleInternal(SLDStyleFactory.java:332)
```

Als Notbehelf habe im Verzeichnis 

`C:\Program Files (x86)\GeoServer 2.15.1\data_dir\workspaces\KRE_KANAL\styles\manhole.sld`

die Zeile 

```
<se:OnlineResource xlink:href="ttf://Dingbats" xlink:type="simple"/>
```

gelöscht.

### Druck-Modul

Das Druckmodul für GeoServer ermöglicht das einfache Hosten des 
Mapfish-Druckdienstes innerhalb einer GeoServer-Instanz. 
Das Mapfish-Druckmodul bietet eine HTTP-API zum Drucken, 
die in JavaScript-Anwendungen wie Leaflet hilfreich ist.

#### Installation

Ich habe das Druck Modul wie folgt installiert:

1. Ich habe die Erweiterung geoserver-2.15.1-printing-plugin.zip von der 
[Download-Seite](http://geoserver.org/release/2.15.1/) heruntergelden.
2. Danach habe ich den Inhalt des ZIP-Archivs in das Verzeichnis `/WEB-INF/lib/` 
der GeoServer-Web-App kopiert. 
3. Dann habe ich den Host `ows.mundialis.de` in der Datei 
`C:\Program Files (x86)\GeoServer 2.15.1\data_dir\printing\config.yaml` in der Liste 
der erlaubten Hosts eingetragen.
```
#===========================================================================
# allowed DPIs
#===========================================================================
dpis: [75, 150, 300]

#===========================================================================
# the allowed scales
#===========================================================================
scales:
  - 25000
  - 50000
  - 100000
  - 200000
  - 500000
  - 1000000
  - 2000000
  - 4000000

#===========================================================================
# the list of allowed hosts
#===========================================================================
hosts:
  - !localMatch
    dummy: true
  - !ipMatch
    ip: www.camptocamp.org
  - !dnsMatch
    host: demo.opengeo.org
    port: 80
  - !dnsMatch
    host: terraservice.net
    port: 80
  - !dnsMatch
    host: ows.mundialis.de 
  - !dnsMatch
    host: sigma.openplans.org
  - !dnsMatch
    host: demo.mapfish.org

layouts:
  Legal:
    mainPage:
      pageSize: LEGAL
      items:
        - !map
          spacingAfter: 30
          width: 440
          height: 483
  #===========================================================================
  A4 portrait:
  #===========================================================================
    metaData:
#      title: '${mapTitle}'
      author: 'MapFish print module'
      subject: 'Simple layout'
      keywords: 'map,print'
      creator: 'MapFish'

    titlePage:
      pageSize: A4
      items:
        - !text
          spacingAfter: 150
        - !text
          font: Helvetica
          fontSize: 40
          spacingAfter: 100
          align: center
          text: '${layout}'
        - !image
          maxWidth: 160
          maxHeight: 160
          spacingAfter: 100
          align: center
          url: http://trac.mapfish.org/trac/mapfish/attachment/ticket/3/logo_v8_sphere.svg?format=raw

      footer: &commonFooter
        height: 30
        items:
          - !columns
            config:
              cells:
                - paddingBottom: 5   
            items:
              - !text
                backgroundColor: #FF0000
                align: left
                text: a red box
              - !text
                align: right
                text: 'Page ${pageNum}'
              - !image
                align: center
                maxWidth: 100
                maxHeight: 30
                spacingAfter: 200
                url: 'http://geoserver.org/img/geoserver-logo.png'

    #-------------------------------------------------------------------------
    mainPage:
      pageSize: A4
      rotation: true
#      header:
#        height: 50
#        items:
#          - !text
#            font: Helvetica
#            fontSize: 30
#            align: right
#            text: '${layout}'
      items:
        - !text
          text: '${mapTitle}'
          fontSize: 30
          spacingAfter: 30
        - !map
          spacingAfter: 30
          width: 440
          height: 483
        - !columns
          # columns can have an absolute position. In that case, they need the 3 following fields:
          absoluteX: 410
          absoluteY: 310
          width: 100
          items:
            - !scalebar
              type: bar
              maxSize: 100
              barBgColor: white
              fontSize: 8
              align: right
        - !text
          text: '${comment}'
          spacingAfter: 30
        - !attributes
          source: data
          spacingAfter: 30
          columnDefs:
            id:
              columnWeight: 2
              header: !text
                text: ID
                backgroundColor: #A0A0A0
              cell: !text
                text: '${id}'
            name:
              columnWeight: 5
              header: !text
                text: Name
                backgroundColor: #A0A0A0
              cell: !columns
                config:
                  cells:
                    - backgroundColor: '${nameBackgroundColor}'
                      borderWidth: 1
                      borderColor: '${nameBorderColor}'
                items:
                  - !text
                    text: '${name}'
            icon:
              columnWeight: 2
              header: !text
                text: Symbol
                backgroundColor: #A0A0A0
              cell: !image
                align: center
                maxWidth: 15
                maxHeight: 15
                url: 'http://www.mapfish.org/svn/mapfish/trunk/MapFish/client/mfbase/mapfish/img/${icon}.png'
        - !text
          font: Helvetica
          fontSize: 9
          align: right
          text: '1:${scale} ${now MM.dd.yyyy}'
      footer: *commonFooter
```
4. Zuletzt habe ich den GeoServer neu gestartet, damit die Änderungen wirksam 
werden.  

Beim ersten Start nach der Installation sollte der GeoServer eine 
Druckmodul-Konfigurationsdatei im Verzeichnis 
`GEOSERVER_DATA_DIR/printing/config.yaml` erstellen.  

Die Einstellungen des Druckmoduls können nur durch Öffnen dieser 
Konfigurationsdatei in einem Texteditor geändert werden.

Eine Liste der konfigurierten Druckparameter habe ich über 
http://localhost:8080/geoserver/pdf/info.json abgerufen. Ich habe die Standards 
belassen.


> Die von mir verwendeten Installationsdateien habe ich im Unterverzeichnis 
`sik` abgelegt.

















## Elemente

### Seitenleiste

In der Seiteleiste sind die Steuerelemente/Controls 
- Einstellungen
- Adresssuche
- Flursuche
- Legende
- Zeichnen
- Hilfe
untergebracht.

Das Steuerelement in der Seitenleiste öffnen Sie, indem Sie es mit der Maus anklicken. 

Während der Arbeit mit einem Control ist es möglich, ein anderes zu aktivieren. 
Dazu reicht ein Klick auf den Namen in der Seitenleiste. 
Daraufhin wird das Aktuelle ausgeblendet und das Gewünschte eingeblendet. 
So wechseln Sie beliebig oft hin und her.  

Sie schließen die Seitenleiste, indem Sie auf den Namen des geöffneten 
Steuerelements in der Leiste klicken.

#### Besonderheiten
#### Konfiguration
#### Sprachdateien



### Einstellungen

Einstellungen, die für den Benutzer änderbar sind, sind in einem Tabulator 
in der Seitenleiste zusammengefasst. 

#### Besonderheiten
##### Popupinfo
Weil ein Mausklick für mehrere Funktionen genutzt wird, legt der Benutzer 
in den Einstellungen fest, ob Popups mit Informationen zu geografischen Lage 
per Klick angezeigt werden. Zum Zeichnen oder Messen deaktiviert er die Infos 
per Mausklick.  
Zusätzlich wird hier festgelegt, bei welchen Zoomstufen das Popup aktiviert ist.
##### Deckkraftschieber
Die Grundebene ist standardmäßig mit einer Deckkraft von 1 (100%) versehen. 
Mit dem Deckkraftschieber ist diese veränderbar, wenn Overlays betrachtet werden.

#### Konfiguration
##### Popupinfo
##### Deckkraftschieber


#### Sprachdateien
### Popupinfo
### Deckkraftschieber





### Adresssuche


#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### Infosuche
Bei Cancel muss danach Gemarkung neu gesetzt werden.
Ich habe absichtlich nicht das Popup geöffnet.
#### Besonderheiten
#### Konfiguration
#### Sprachdateien
Default für Flur (placeholder_fln) muss eine Zahl sein!


### Hilfe
#### Besonderheiten

Die Hilfe wird in der Seitenleiste angezeigt.
Sie wird nur angezeigt wenn auch die Seitenleiste in der Konfiguration 
auf `anzeigen` gesetzt ist.

#### Konfiguration


#### Sprachdateien




### TreeLayers

#### Besonderheiten
#### Konfiguration
#### Sprachdateien


### Measure

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### Draw

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### Legende

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### Print

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### ZoomDisplay

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### Scale

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### Scaledisplay

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### Overview

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### Mausposition

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

## Gestaltungselemente

### Alert

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### Toolbar

#### Besonderheiten
#### Konfiguration
#### Sprachdateien

### Sidebar

#### Besonderheiten
#### Konfiguration
#### Sprachdateien
