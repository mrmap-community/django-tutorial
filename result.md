# Zusammenfassung

Wir haben innerhalb kürzester Zeit eine sehr einfache Verwaltungssoftware für Bebauungspläne erstellt. 
Die exportierbaren GML-Dateien lassen sich mit Hilfe des Validators prüfen und sind valide.
Einen praktischen Nutzwert hat die Software in diesem Stadium aber noch nicht.

Um die Software zur Produktionsreife zu bringen, müssen noch ein paar Dinge entwickelt werden. 

# TODOs

* Ersetzen des Felds **geltungsbereich** durch eine m2m Relation zu einem neuen Modell **bereich**
* Aktivieren der Pflichtfelder entprechend der Vorgaben in RLP
* Entwickeln notwendiger Validierungsfunktionen
* Erstellung eines Mapfile-Generators zur Publikation von WMS- und WFS-Interfaces
* Schaffung der Ablagemöglichkeit für Dokumente
* Importmöglichkeit für BPlan-GML Dokumente
* ...

```{note}
This text is **standard** _Markdown_
```
