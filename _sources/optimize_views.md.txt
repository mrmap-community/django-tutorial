# Geometrie Editor

## Aktivieren des Leaflet-Clients in den Views

Leaflet Integration im Basis-Template xplanung_light/templates/xplanung_light/layout.html
```jinja
{# ... #}
{# load leaflet specific parts #}
{% load leaflet_tags %}
{% leaflet_css plugins="ALL" %}
{% leaflet_js plugins="ALL" %}
{# ... #}
```

Anpassung der BPlanCreateView und BPlanUpdateView in xplanung_light/views.py
```python 
#...
from leaflet.forms.widgets import LeafletWidget
#...
class BPlanCreateView(CreateView):
    model = BPlan
    fields = ["name", "nummer", "geltungsbereich", "gemeinde", "planart"]
    success_url = reverse_lazy("bplan-list") 

    def get_form(self, form_class=None):
        form = super().get_form(form_class)
        form.fields['gemeinde'].queryset = form.fields['gemeinde'].queryset.only("pk", "name", "type")
        form.fields['geltungsbereich'].widget = LeafletWidget(attrs={'geom_type': 'MultiPolygon', 'map_height': '500px', 'map_width': '50%','MINIMAP': True})
	return form

class BPlanUpdateView(UpdateView):
    model = BPlan
    fields = ["name", "nummer", "geltungsbereich", "gemeinde", "planart"] 
    success_url = reverse_lazy("bplan-list") 

    def get_form(self, form_class=None):
        form = super().get_form(form_class)
        form.fields['gemeinde'].queryset = form.fields['gemeinde'].queryset.only("pk", "name", "type")
        form.fields['geltungsbereich'].widget = LeafletWidget(attrs={'geom_type': 'MultiPolygon', 'map_height': '500px', 'map_width': '50%','MINIMAP': True})
        return form
```

# Tabellenanzeige

Um mit geringem Aufwand eine einfach zu pflegende Tabellenanzeige zu erhalten, bietet sich das package [**django-tables2**](https://django-tables2.readthedocs.io/en/latest/) an.

```shell
python3 -m pip install django-tables2
```

Package zu komserv/settings.py hinzufügen
```python
#...
    'django_tables2',
#...
```

Erstellen einer Python-Datei für das Management von Tabellen

xplanung_light/tables.py
```python
import django_tables2 as tables
from .models import BPlan
from django_tables2 import Column
from django_tables2.utils import A

class BPlanTable(tables.Table):
    #download = tables.LinkColumn('gedis-document-pdf', text='Download', args=[A('pk')], \
    #                     orderable=False, empty_values=())
    edit = tables.LinkColumn('bplan-update', text='Bearbeiten', args=[A('pk')], \
                         orderable=False, empty_values=())
    delete = tables.LinkColumn('bplan-delete', text='Löschen', args=[A('pk')], \
                         orderable=False, empty_values=())
    """
    geojson = Column(
        accessor=A('geojson'),
        orderable=False,
        # ...
    )
    """

    class Meta:
        model = BPlan
        template_name = "django_tables2/bootstrap5.html"
        fields = ("name", "gemeinde", "edit", "delete")
```

Anpassung der Klasse BPlanListView in xplanung_light/views.py - Integration des Tabellenmoduls
```python
#...
from django_tables2 import SingleTableView
from xplanung_light.tables import BPlanTable
#...
class BPlanListView(SingleTableView):
    model = BPlan
    table_class = BPlanTable
    success_url = reverse_lazy("bplan-list") 
```

# Anpassung Templates

Anpassung der Liste - Hinzufügen eines Create Buttons - xplanung_light/templates/xplanung_light/bplan_list.html
```jinja
{# .... #}
{% block content %}
<p><a href="{% url 'bplan-create' %}">BPlan anlegen</a></p>
{# .... #}
```

Menüeintrag für Bebauungspläne hinzufügen - xplanung_light/templates/xplanung_light/layout.html
```jinja
{# .... #}
<ul class="navbar-nav me-auto mb-2 mb-lg-0">
              {% if user.is_authenticated %}
              <li class="nav-item">
                <a class="nav-link" aria-current="page" href="{% url 'bplan-list' %}">Bebauungspläne</a>
              </li>
              {% endif %}
              <li class="nav-item">
{# .... #}
```

Datenmodelle migrieren
```shell
python3 manage.py makemigrations
python3 manage.py migrate
```
