# Einführung

Das Tutorial soll an einem konkreten Beispiel zeigen, wie man mit Django in relativ kurzer Zeit eine einfache Anwendung zur Verwaltung von Geodaten erstellen kann. Als Beispiel werden hier kommunale Bauleitpläne genommen.

Die Anwendung verfügt über folgende Funktionen/Eigenschaften:

* Nutzung der Django-Standardbenutzerverwaltung für Registrierung und Authentifizierung
* Import der rheinland-pfälzischen Gebietskörperschaften mit deren geometrischen Abgrenzungen
* Historisierung der Gebietskörperschaften
* Erstellen, Editieren, Löschen von Bebauungsplänen mit Referenzen auf die Gebietskörperschaften
* Editieren multipolygonaler Abgrenzungen von Geltungsbereichen der Bebauungspläne
* Layout mit bootstrap5 optimiert (responsive)
* Export von konformen XPlan-GML Dateien 

Verwendete Django packages:

* django-bootstrap5
* django-simple-history
* django-leaflet
* django-tables2
* django-debug-toolbar
* django-filter
* django-crispy-forms
* crispy-bootstrap5

Weitere python libs:

* requests
* openpyxl
* mappyfile
* mapscript==7.6.0

# Weitere Informationen

* [django](https://www.djangoproject.com/)
* [xplanung](https://xleitstelle.de/releases/objektartenkatalog_6_0)
* [Gebietskörperschaften RLP](https://www.statistik.rlp.de/publikationen/verzeichnisse-und-adressarien)
* [Leitfaden kommunale Pläne GDI-RP](https://www.geoportal.rlp.de/mediawiki/images/c/c7/Leitfaden_Bereitstellung_komm_Plaene.pdf)
* [Testdaten xleitstelle](https://gitlab.opencode.de/xleitstelle/xplanung/testdaten)
* [Beispiel Dateien XPlanung 6.0](https://gitlab.opencode.de/diplanung/ozgxplanung/-/tree/xplanbox-7.1.3/xplan-tests/xplan-tests-resources/src/main/resources/de/latlon/xplan/xplan60?ref_type=tags)
* [QGIS XPlan-Reader](https://github.com/kreis-viersen/xplan-reader/)
* [QGIS XPlan-Umring](https://github.com/kreis-viersen/xplan-umring)
* [Validator - alt](https://www.xplanungsplattform.de/xplan-validator/)
* [Validator - neu](https://xleitstelle.de/validator)

# Beispielimplementierungen

## Beispiel Hosting BW [Komm.ONE](https://www.komm.one/wer-wir-sind) - WFS Abgabe BPlan in Rasterform

| Link | Operation | Erläuterung |
|----------|----------|----------|
| [Eigenschaftsabfrage](https://geodienste.komm.one/ows/services/org.641.1b95f33f-e4f3-4898-bcba-8319409933fc_wfs/org.641.db827e93-33f7-4b4e-97ce-9772690d4b6b?SERVICE=WFS&Request=GetCapabilities)    | GetCapabilities   | keine   |
| [BP_Plan](https://geodienste.komm.one/ows/services/org.641.1b95f33f-e4f3-4898-bcba-8319409933fc_wfs/org.641.db827e93-33f7-4b4e-97ce-9772690d4b6b?VERSION=2.0.0&SERVICE=WFS&typename=BP_Plan&Request=GetFeature)    | GetFeature   | keine   |
| [XP_Rasterdarstellung](https://geodienste.komm.one/ows/services/org.641.1b95f33f-e4f3-4898-bcba-8319409933fc_wfs/org.641.db827e93-33f7-4b4e-97ce-9772690d4b6b?VERSION=2.0.0&SERVICE=WFS&typename=XP_Rasterdarstellung&Request=GetFeature)    | GetFeature   | keine   |
| [BP_Bereich](https://geodienste.komm.one/ows/services/org.641.1b95f33f-e4f3-4898-bcba-8319409933fc_wfs/org.641.db827e93-33f7-4b4e-97ce-9772690d4b6b?SERVICE=WFS&VERSION=2.0.0&typename=BP_Bereich&Request=GetFeature)    | GetFeature   | keine   |

## Netgis Mannheim

* [Übersicht](https://service.gis-mannheim.de/mannheim/mod_plan_bw/)
* [Einzelner Plan](https://service.gis-mannheim.de/mannheim/mod_plan_bw/get_xplan_gml.php?gmlid=GML_ef9a23ca-9613-d9b0-ac3e-2ff516bee276)

# Leitfäden

* [Bayern 10/2024](https://www.stmb.bayern.de/assets/stmi/miniwebs/digitale_planung/xplanung_by_leitfaden_a4_pre.pdf)
* [Baden-Württemberg 12/2022](https://www.geoportal-bw.de/documents/d/guest/leitfaden_bauleitplan_v3-1)
    * [Anlage](https://www.geoportal-bw.de/documents/d/guest/anlage1_v3_1_221215) 
* [Brandenburg 05/2022](https://lbv.brandenburg.de/download/Raumbeobachtung/Pflichtenheft_2022.pdf)
* [Rheinland-Pfalz 10/2017](https://www.geoportal.rlp.de/mediawiki/images/c/c7/Leitfaden_Bereitstellung_komm_Plaene.pdf)
* [Sachsen-Anhalt 03/2024](https://mid.sachsen-anhalt.de/fileadmin/Bibliothek/Politik_und_Verwaltung/MLV/MID/Ministerium/Publikationen/XPlanung-Leitfaden-Sachsen-Anhalt.pdf)