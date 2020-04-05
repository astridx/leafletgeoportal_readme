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

```
	"Sidebar": {
		"show": true
	},
```


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

```
	"Settings": {
		"show": true,
		"opacityMin": 0,
		"opacityStep": 0.1,
		"opacityMax": 1,
		"popupMinZoom": 9,
		"popupMaxZoom": 18
	},
```


##### Popupinfo
##### Deckkraftschieber


#### Sprachdateien
### Popupinfo
### Deckkraftschieber





### Adresssuche


#### Besonderheiten
#### Konfiguration

```
	"Adresssuche": {
		"show": true,
		"queryMinLength": 1,
		"suggestMinLength": 3,
		"suggestTimeout": 250,
		"defaultMarkGeocode": true,
		"showUniqueResult": true,
		"showResultIcons": false,
		"collapsed": true,
		"expand": "click"
	},

```
#### Sprachdateien

### Infosuche
Bei Cancel muss danach Gemarkung neu gesetzt werden.
Ich habe absichtlich nicht das Popup geöffnet.
#### Besonderheiten
#### Konfiguration

```
	"Infosuche": {
		"show": true
	},

```

#### Sprachdateien
Default für Flur (placeholder_fln) muss eine Zahl sein!


### Hilfe
#### Besonderheiten

Die Hilfe wird in der Seitenleiste angezeigt.
Sie wird nur angezeigt wenn auch die Seitenleiste in der Konfiguration 
auf `anzeigen` gesetzt ist.

#### Konfiguration

```
	"Hilfe": {
		"show": true
	}
```

#### Sprachdateien




### TreeLayers

#### Besonderheiten
#### Konfiguration
```
	"TreeLayers": {
		"position": "bottomright",
		"collapsed": true,
		"namedToggle": true,
		"baseTree": [{
				"label": "Kremme Alkis",
				"noShow": true,
				"children": [{
						"label": "KRE_ALKIS",
						"layer": {
							"url": "/KRE_ALKIS/wms",
							"titel": "KRE_ALKIS",
							"properties": {
								"id": "DEBBAL650003XlMY",
								"id_hash": "aec26c728b24ecdfa578e858a83b3abe",
								"area": 2635.44726799473,
								"kennung": 11001,
								"convnr": 7,
								"convnr2": 7,
								"beginnt": "2015-10-14T04:40:32Z",
								"endet": null,
								"std_modell": "DLKM",
								"son_modell": null,
								"gmk_lan": "12",
								"gmk_gmn": "3649",
								"fsn_zae": "114",
								"fsn_nen": null,
								"fsk": "12364900500114______",
								"flstalb": "1236490050011400000",
								"afl": 2730,
								"fln": 5,
								"fsf": null,
								"arz": false,
								"rbv": null,
								"zfm": null,
								"obkx": 366393.168,
								"obky": 5847596.529,
								"zde": "1000-01-01",
								"gkz_lan": "12",
								"gkz_rbz": "0",
								"gkz_krs": "65",
								"gkz_gem": "165",
								"gkz_gmt": null,
								"istgebucht": "DEBBAL010008Xtdq",
								"istgebucht_hash": "513d808326c441f93329ef415a741734",
								"sk": "2028",
								"dprio": 700,
								"overlay": "AX_Flurstueck",
								"geotyp": 3,
								"gemeindename": "Kremmen",
								"gemarkungsname": "Kremmen",
								"marker": null
							},
							"layers": "KRE_ALKIS",
							"format": "image/png",
							"maxZoom": 20,
							"minZoom": 0,
							"transparent": true,
							"opacity": 1,
							"attribution": "NTI CWSM GmbH"
						},
						"name": "KRE_ALKIS"
					}]
			}],
		"overlaysTree": [{
				"label": "Kremmen",
				"selectAllCheckbox": true,
				"children": [{
						"label": "Kremmen Kanal",
						"selectAllCheckbox": true,
						"children": [{
								"label": "aw_anschldruckl",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"titel": "aw_anschldruckl",
									"properties": {
										"material": "Polyethylen",
										"id_utilisation": 14,
										"utilisation": "Druckabfluss, Schmutzwassersystem",
										"disposition_state": "vorhanden (in Betrieb)",
										"name_number": "ADL Spargelhof",
										"name_number_1": null,
										"diameter_inside": null,
										"diameter_outside": null,
										"length": 1045.79532637,
										"input_z": null,
										"output_z": null,
										"pipe_length": null,
										"pipe_slope": null,
										"aw_baujahr": 2016,
										"dimension_1": "100.00000000",
										"function": "Haltung ,Transportkanal"
									},
									"layers": "aw_anschldruckl",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "aw_anschldruckl"
							}, {
								"label": "aw_anschlussleitung",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"titel": "aw_anschlussleitung",
									"properties": {
										"material": "Steinzeug",
										"id_utilisation": 10,
										"utilisation": "Freispiegelabfluss im geschlossenen Profil, Schmutzwassersystem",
										"name_number": "KR_00001269",
										"name_number_1": null,
										"diameter_inside": null,
										"diameter_outside": null,
										"id_function": 1,
										"input_z": 36.68,
										"output_z": 36.29,
										"pipe_length": null,
										"pipe_slope": null,
										"length": 5.88110542,
										"aw_laenge": null,
										"aw_baujahr": 1995,
										"dimension_1": "150.00000000",
										"disposition_state": "vorhanden (in Betrieb)",
										"function": "Haltung ,Transportkanal"
									},
									"layers": "aw_anschlussleitung",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "aw_anschlussleitung"
							}, {
								"label": "aw_anschlusspunkt",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"titel": "aw_anschlusspunkt",
									"properties": {
										"ownership": null,
										"utilisation": "Freispiegelabfluss im geschlossenen Profil, Schmutzwassersystem",
										"id_utilisation": 10,
										"name_number": "KR_00001367",
										"name_number_1": null,
										"disposition_state": "vorhanden (in Betrieb)",
										"aw_baujahr": null,
										"id_aw_punktkennung": 1,
										"aw_punktkennung": "Anschlusspunkt allgemein"
									},
									"layers": "aw_anschlusspunkt",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "aw_anschlusspunkt"
							}, {
								"label": "aw_druckleitung",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"titel": "aw_druckleitung",
									"layers": "aw_druckleitung",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "aw_druckleitung"
							}, {
								"label": "aw_pumpwerk",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"titel": "aw_druckleitung",
									"properties": {
										"id_disposition_state": 6,
										"material": "Polyethylen",
										"id_utilisation": 14,
										"utilisation": "Druckabfluss, Schmutzwassersystem",
										"name_number": "PW_Flatow 01",
										"name_number_1": null,
										"disposition_state": "vorhanden (in Betrieb)",
										"diameter_inside": null,
										"diameter_outside": null,
										"input_z": 35.5,
										"output_z": 41.49,
										"pipe_length": null,
										"pipe_slope": null,
										"length": 592.17881149,
										"aw_laenge": null,
										"aw_baujahr": 2011,
										"dimension_1": "90.00000000",
										"function": "Haltung ,Transportkanal",
										"total_length": 592.17881149
									},
									"layers": "aw_pumpwerk",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "aw_pumpwerk"
							}, {
								"label": "ww_manhole",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"titel": "ww_manhole",
									"properties": {
										"orientation": 0,
										"aw_gelaendehoehe": null,
										"bottom_elevation": 37.06,
										"ownership": null,
										"material": null,
										"utilisation": "Freispiegelabfluss im geschlossenen Profil, Schmutzwassersystem",
										"aw_baujahr": 1995,
										"function": null,
										"name_number": "KR_00001271",
										"name_number_1": null,
										"id_function": 16,
										"id_disposition_state": 6,
										"disposition_state": "vorhanden (in Betrieb)",
										"id_mhole_lower_part_shape": null,
										"id_utilisation": 10
									},
									"layers": "ww_manhole",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.5,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "ww_manhole"
							}, {
								"label": "ww_section",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"titel": "ww_section",
									"properties": {
										"material": "Steinzeug",
										"id_utilisation": 10,
										"id_lining": null,
										"input_z": 37.16,
										"output_z": 36.57,
										"pipe_length": 43.50995014,
										"pipe_slope": 0.01356136,
										"utilisation": "Freispiegelabfluss im geschlossenen Profil, Schmutzwassersystem",
										"name_number": "KR_K2.7",
										"name_number_1": null,
										"disposition_state": "vorhanden (in Betrieb)",
										"diameter_inside": null,
										"diameter_outside": null,
										"length": 43.50594972,
										"aw_laenge": 43.50995014,
										"aw_baujahr": 1995,
										"dimension_1": "200.00000000",
										"function": "Haltung ,Transportkanal",
										"total_length": 43.50594972
									},
									"layers": "ww_section",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "ww_section"
							}, {
								"label": "ww_valve",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"titel": "ww_valve",
									"properties": {
										"test": "test"
									},
									"layers": "ww_valve",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "ww_valve"
							}]
					}, {
						"label": "Kremmen Kanal Beschriftung",
						"selectAllCheckbox": true,
						"children": [{
								"label": "aw_anschldruckl_tbl",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"layers": "aw_anschldruckl_tbl",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "aw_anschldruckl_tbl"
							}, {
								"label": "aw_anschlussleitung_tbl",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"layers": "aw_anschlussleitung_tbl",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "aw_anschlussleitung_tbl"
							}, {
								"label": "aw_anschlusspunkt_tbl",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"layers": "aw_anschlusspunkt_tbl",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "aw_anschlusspunkt_tbl"
							}, {
								"label": "aw_druckleitung_tbl",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"layers": "aw_druckleitung_tbl",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "aw_druckleitung_tbl"
							}, {
								"label": "aw_pumpwerk_tbl",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"layers": "aw_pumpwerk_tbl",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "aw_pumpwerk_tbl"
							}, {
								"label": "ww_manhole_tbl",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"layers": "ww_manhole_tbl",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.5,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "ww_manhole_tbl"
							}, {
								"label": "ww_section_tbl_bs",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"layers": "ww_section_tbl_bs",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "ww_section_tbl_bs"
							}, {
								"label": "ww_section_tbl_fs",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"layers": "ww_section_tbl_fs",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "ww_section_tbl_fs"
							}, {
								"label": "ww_valve_tbl",
								"layer": {
									"url": "/KRE_KANAL/wms",
									"layers": "ww_valve_tbl",
									"format": "image/png",
									"maxZoom": 20,
									"minZoom": 0,
									"transparent": true,
									"opacity": 0.9,
									"attribution": "NTI CWSM GmbH"
								},
								"name": "ww_valve_tbl"
							}]
					}]
			}]
	},

```
#### Sprachdateien


### Measure
	"Measure": {
		"show": true
	},

#### Besonderheiten
#### Konfiguration
```
```
#### Sprachdateien

### Draw

#### Besonderheiten
#### Konfiguration
```
	"Draw": {
		"show": true
	},
```
#### Sprachdateien

### Legende

#### Besonderheiten
#### Konfiguration
```
	"Legende": {
		"show": true,
		"maxheight": 100,
		"uri": "http://92.79.80.53/geoserver/KRE_KANAL/ows?service=WMS&request=GetLegendGraphic&format=image/png&width=20&height=20&layer=KRE_KANAL:KRE_KANAL"
	},
```
#### Sprachdateien

### Print

#### Besonderheiten
#### Konfiguration
```
	"Print": {
		"show": true,
		"position": "bottomright",
		"tileWait": "500",
		"showbordertime": "12000"
	},
```
#### Sprachdateien

### ZoomDisplay

#### Besonderheiten
#### Konfiguration
```
	"ZoomDisplay": {
		"show": true,
		"position": "topleft"
	},
```
#### Sprachdateien

### Scale

#### Besonderheiten
#### Konfiguration
```
	"Scale": {
		"show": true,
		"position": "bottomleft"
	},
```
#### Sprachdateien

### Scaledisplay

#### Besonderheiten
#### Konfiguration
```
	"ScaleDisplay": {
		"show": true,
		"position": "bottomleft",
		"updateWhenIdle": true,
		"formatieren": true,
		"ThousandsSeparators": "."
	},
```
#### Sprachdateien

### Overview

#### Besonderheiten
#### Konfiguration
```
	"Overview": {
		"show": true,
		"position": "bottomright",
		"WMSLayer": {
			"url": "http://ows.mundialis.de/services/service?",
			"layers": "OSM-WMS",
			"format": "image/png",
			"maxZoom": 20,
			"minZoom": 0,
			"transparent": true,
			"opacity": 0,
			"attribution": "NTI CWSM GmbH"
		},
		"zoomDifference": 1,
		"minZoom": 8,
		"maxZoom": 18
	},
```
#### Sprachdateien

### Mausposition

#### Besonderheiten
#### Konfiguration
-
#### Sprachdateien

## Gestaltungselemente

### Alert

#### Besonderheiten
#### Konfiguration
```
```
#### Sprachdateien

### Toolbar

#### Besonderheiten
#### Konfiguration
```
	"Toolbar": {
		"show": true,
		"text": "Geoportal des Zweckverband Kremmen"
	},
```
#### Sprachdateien

### Sidebar

#### Besonderheiten
#### Konfiguration
```
	"Sidebar": {
		"show": true
	},
```
#### Sprachdateien
