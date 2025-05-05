# Planmodell

## Einleitung

Das Modell zur Verwaltung von kommunalen Plänen orientiert sich am deutschen Standard [**XPlanung**](https://xleitstelle.de/xplanung) und am [**Leitfaden für die Bereitstellung kommunaler Pläne und Satzungen im Rahmen der Geodateninfrastruktur Rheinland-Pfalz (GDI-RP)**](https://www.geoportal.rlp.de/metadata/Leitfaden_kommunale_Plaene_GDI_RP.pdf). Dieser Leitfaden wurde ab 2008 auf Basis von XPlanung 2.0 entwickelt. Neben den Vorgaben des Datenaustauschstandards wurden dabei auch Anforderungen aus der kommunalen Praxis übernommen und eine standardisierte Bereitstellung vorgegeben. Der Standard wurde mit den kommunalen Spitzenverbänden des Landes abgestimmt und wird vom Lenkungsausschuss für Geodateninfrastruktur Rheinland-Pfalz herausgegeben.

In einer ersten Version des Modells wird nur eine minimale Zahl von Attributen definiert. Für den proof of concept (POC)  ist das zunächst ausreichend. Das abstrakte Grundmodell heißt XPlan und vererbt seine Attribute auf das konkrete Modell BPlan. Der Geltungsbereich ist als Geometry-Field modelliert und kann damit auch Multipolygone aufnehmen. 

## Modelldefinition

xplanung_light/models.py
```python
"""
https://xleitstelle.de/releases/objektartenkatalog_6_0
"""
class XPlan(models.Model):

    name = models.CharField(null=False, blank=False, max_length=2048, verbose_name='Name des Plans', help_text='Offizieller Name des raumbezogenen Plans')
    #nummer [0..1]
    nummer = models.CharField(max_length=5, verbose_name="Nummer des Plans.")
    #internalId [0..1]
    #beschreibung [0..1]
    #kommentar [0..1]
    #technHerstellDatum [0..1], Date
    #genehmigungsDatum [0..1], Date
    #untergangsDatum [0..1], Date
    #aendertPlan [0..*], XP_VerbundenerPlan
    #wurdeGeaendertVonPlan [0..*], XP_VerbundenerPlan
    #aendertPlanBereich [0..*], Referenz, Testphase
    #wurdeGeaendertVonPlanBereich [0..*], Referenz, Testphase
    #erstellungsMassstab [0..1], Integer
    #bezugshoehe [0..1], Length
    #hoehenbezug [0..1]
    #technischerPlanersteller, [0..1]
    #raeumlicherGeltungsbereich [1], GM_Object
    geltungsbereich = models.GeometryField(null=False, blank=False, verbose_name='Grenze des räumlichen Geltungsbereiches des Plans.')
    #verfahrensMerkmale [0..*], XP_VerfahrensMerkmal
    #hatGenerAttribut [0..*], XP_GenerAttribut
    #externeReferenz, [0..*], XP_SpezExterneReferenz
    #texte [0..*], XP_TextAbschnitt
    #begruendungsTexte [0..*], XP_BegruendungAbschnitt
    
    class Meta:
        abstract = True


class BPlan(XPlan):

    BPLAN = "1000"
    EINFACHERBPLAN = "10000"
    QUALIFIZIERTERBPLAN = "10001"
    BEBAUUNGSPLANZURWOHNRAUMVERSORGUNG = "10002"
    VORHABENBEZOGENERBPLAN = "3000"
    VORHABENUNDERSCHLIESSUNGSPLAN = "3100"
    INNENBEREICHSSATZUNG = "4000"
    KLARSTELLUNGSSATZUNG = "40000"
    ENTWICKLUNGSSATZUNG = "40001"
    ERGAENZUNGSSATZUNG = "40002"
    AUSSENBEREICHSSATZUNG = "5000"
    OERTLICHEBAUVORSCHRIFT = "7000"
    SONSTIGES = "9999"

    BPLAN_TYPE_CHOICES = [
        (BPLAN,  "BPlan"),
        (EINFACHERBPLAN, "EinfacherBPlan"),
        (QUALIFIZIERTERBPLAN, "QualifizierterBPlan"),
        (BEBAUUNGSPLANZURWOHNRAUMVERSORGUNG, "BebauungsplanZurWohnraumversorgung"),
        (VORHABENBEZOGENERBPLAN, "VorhabenbezogenerBPlan"),
        (VORHABENUNDERSCHLIESSUNGSPLAN, "VorhabenUndErschliessungsplan"),
        (INNENBEREICHSSATZUNG, "InnenbereichsSatzung"),
        (KLARSTELLUNGSSATZUNG, "KlarstellungsSatzung"),
        (ENTWICKLUNGSSATZUNG, "EntwicklungsSatzung"),
        (ERGAENZUNGSSATZUNG, "ErgaenzungsSatzung"),
        (AUSSENBEREICHSSATZUNG, "AussenbereichsSatzung"),
        (OERTLICHEBAUVORSCHRIFT, "OertlicheBauvorschrift"),
        (SONSTIGES, "Sonstiges"),
    ]

    #gemeinde [1], XP_Gemeinde
    gemeinde = models.ForeignKey(AdministrativeOrganization, null=True, on_delete=models.SET_NULL)
    #planaufstellendeGemeinde [0..*], XP_Gemeinde
    #plangeber [0..*], XP_Plangeber
    #planArt [1..*], BP_PlanArt
    planart = models.CharField(null=False, blank=False, max_length=5, choices=BPLAN_TYPE_CHOICES, default='1000', verbose_name='Typ des vorliegenden Bebauungsplans.', db_index=True)
	#sonstPlanArt [0..1], BP_SonstPlanArt
    #rechtsstand [0..1], BP_Rechtsstand
    #status [0..1], BP_Status
    #aenderungenBisDatum [0..1], Date
    #aufstellungsbeschlussDatum [0..1], Date
    #veraenderungssperre [0..1], BP_VeraenderungssperreDaten
    #auslegungsStartDatum [0..*], Date
    #auslegungsEndDatum [0..*], Date
    #traegerbeteiligungsStartDatum [0..*], Date
    #traegerbeteiligungsEndDatum [0..*], Date
    #satzungsbeschlussDatum [0..1], Date
    #rechtsverordnungsDatum [0..1], Date
    #inkrafttretensDatum [0..1], Date
    #ausfertigungsDatum [0..1], Date
    #staedtebaulicherVertrag [0..1], Boolean
    #erschliessungsVertrag [0..1], Boolean
    #durchfuehrungsVertrag [0..1], Boolean
    #gruenordnungsplan [0..1], Boolean
    #versionBauNVO [0..1], XP_GesetzlicheGrundlage
    #versionBauGB [0..1], XP_GesetzlicheGrundlage
    #versionSonstRechtsgrundlage [0..*], XP_GesetzlicheGrundlage
    #bereich [0..*], BP_Bereich

    def __str__(self):
        """Returns a string representation of a BPlan."""
        return f"'{self.planart}': '{self.name}'"
```

## Verwaltungsviews

### Vorarbeiten

Um eine komfortable Verwaltung von Geometrien zu ermöglichen, bietet es sich an das Django Leaflet package ([**django-leaflet**](https://django-leaflet.readthedocs.io/en/latest/)) einzusetzen. Standardmäßig ist bei django noch eine älterer OpenLayers-Client (2.13) integriert.

Installation des packages
```shell
python3 -m pip install django-leaflet
```

Aktivieren des packages in komserv/settings.py
```python
#...
    'leaflet',
#...
```

Ersatz des OpenLayer Client im Admin backend  - xplanung_light/admin.py
```python
#...
from leaflet.admin import LeafletGeoAdmin
from xplanung_light.models import BPlan
#...
admin.site.register(BPlan, LeafletGeoAdmin)
#...
```

### Urls

xplanung_light/urls.py
```python
#...
from xplanung_light.views import BPlanCreateView, BPlanUpdateView, BPlanDeleteView, BPlanListView
#...
    # urls for bplan
    path("bplan/", BPlanListView.as_view(), name="bplan-list"),
    path("bplan/create/", BPlanCreateView.as_view(), name="bplan-create"),
    path("bplan/<int:pk>/update/", BPlanUpdateView.as_view(), name="bplan-update"),
    path("bplan/<int:pk>/delete/", BPlanDeleteView.as_view(), name="bplan-delete"),
]
```

### Views

Nutzung von einfachen ClassBasedGenericViews

xplanung_light/views.py
```python
#...
from django.views.generic import (ListView, CreateView, UpdateView, DeleteView)
from xplanung_light.models import AdministrativeOrganization, BPlan
from django.urls import reverse_lazy
#...

class BPlanCreateView(CreateView):
    model = BPlan
    fields = ["name", "nummer", "geltungsbereich", "gemeinde", "planart"]
    success_url = reverse_lazy("bplan-list") 


class BPlanUpdateView(UpdateView):
    model = BPlan
    fields = ["name", "nummer", "geltungsbereich", "gemeinde", "planart"] 
    success_url = reverse_lazy("bplan-list") 


class BPlanDeleteView(DeleteView):
    model = BPlan

    def get_success_url(self):
        return reverse_lazy("bplan-list")


class BPlanListView(ListView):
    model = BPlan
    success_url = reverse_lazy("bplan-list") 
```

### Templates

xplanung_light/templates/xplanung_light/*

#### List

xplanung_light/templates/xplanung_light/bplan_list.html
```jinja
{% extends "xplanung_light/layout.html" %}
{% block title %}
    Liste der Bebauungspläne
{% endblock %}
{% block content %}
<p>Treffer: {{ object_list.count }} Bebauungspläne</p>
{% endblock %}
```

#### Confirm Delete

xplanung_light/templates/xplanung_light/bplan_confirm_delete.html
```jinja
<form method="post">{% csrf_token %}
    <p>Wollen sie das Objekt wirklich löschen? "{{ object }}"?</p>
    {{ form }}
    <input type="submit" value="Bestätigung">
</form>
```

#### Create

xplanung_light/templates/xplanung_light/bplan_form.html
```jinja
{% extends "xplanung_light/layout.html" %}
{% block title %}
    Bebauungsplan anlegen
{% endblock %}
{% block content %}
    <form method="post">{% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Save">
    </form>
{% endblock %}
```

#### Update

xplanung_light/templates/xplanung_light/bplan_form_update.html
```jinja
{% block title %}
    Bebauungsplan editieren
{% endblock %}
{% block content %}
    <form method="post">{% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Aktualisieren">
    </form>
{% endblock %}
```
