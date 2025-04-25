# XPlanung GML Export

Für den interoperablen Datenaustausch müssen die Bebauungsplaninformationen in XPlan-GML exportiert werden können.
In Django lassen sich hierfür einfache XML-Templates verwenden. Diese werden zur Laufzeit mit den Daten aus der DB gefüllt. 
Das Prinzip ist das gleiche wie bei den HTML-Templates.


## Export XML View

Für den Export brauchen wir einen View. Da immer nur ein einzelner Bebauungsplan exportiert wird, kann man als Grundlage den Standard Detail View nutzen.

xplanung_light/views.py
```python
#...
from django.views.generic import DetailView
#...
class BPlanDetailView(DetailView):
    model = BPlan
#...
``` 

Dieser vererbt seine Struktur an den neuen Export View. Für ein konformes XPlan-GML sind einige Vorarbeiten nötig. Wir brauchen die Geometrien
für den räumlichen Geltungsbereich im EPSG:25832 und im Format GML3. Das kann man relativ einfach mit einer Erweiterung 
des querysets mit einer annotation lösen. Zusätzlich zu den Polygonen brauchen wir noch den Extent der Geometrien. Dieser läßt sich aktuell nicht über eine annotation abfragen, sondern muss zur Laufzeit berechnet werden. Dazu nutzen wir die über Geodjango zur Verfügung stehende GDAL Implementierung. Da wir auch das GML3 noch ändern müssen (Ergänzungen von gml_id Attributen), brauchen wir noch die etree-Bibliothek zum Parsen und Schreiben von XML.

xplanung_light/views.py
```python
#...
from django.contrib.gis.db.models.functions import AsGML, Transform
from django.contrib.gis.gdal import CoordTransform, SpatialReference
from django.contrib.gis.gdal import OGRGeometry
import uuid
import xml.etree.ElementTree as ET
#...
class BPlanDetailXmlRasterView(BPlanDetailView):  

    def get_queryset(self):
        # Erweiterung der auszulesenden Objekte um eine transformierte Geomtrie im Format GML 3
        queryset = super().get_queryset().annotate(geltungsbereich_gml_25832=AsGML(Transform("geltungsbereich", 25832), version=3))
        return queryset

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        # Um einen XPlanung-konformen Auszug zu bekommen, werden gml_id(s) verwendet.
        # Es handelt sich um uuids, die noch das Prefix "GML_" bekommen. Grundsätzlich sollten die 
        # aus den Daten in der DB stammen und dort vergeben werden. 
        # Im ersten Schritt synthetisieren wir sie einfach ;-)
        context['auszug_uuid'] = "GML_" + str(uuid.uuid4())
        context['bplan_uuid'] = "GML_" + str(uuid.uuid4())
        # Irgendwie gibt es keine django model function um direkt den Extent der Geometrie zu erhalten. Daher nutzen wir hier gdal
        # und Transformieren die Daten erneut im RAM
        # Definition der Transformation (Daten sind immer in WGS 84 - 4326)
        ct = CoordTransform(SpatialReference(4326, srs_type='epsg'), SpatialReference(25832, srs_type='epsg'))
        # OGRGeoemtry Objekt erstellen
        ogr_geom = OGRGeometry(str(context['bplan'].geltungsbereich), srs=4326)
        # Transformation nach EPSG:25832
        ogr_geom.transform(ct)
        # Speichern des Extents in den Context
        context['extent'] = ogr_geom.extent
        # Ausgabe der GML Variante zu Testzwecken 
        # print(context['bplan'].geltungsbereich_gml_25832)
        # Da die GML Daten nicht alle Attribute beinhalten, die XPlanung fordert, müssen wir sie anpassen, bzw. umschreiben
        # Hierzu nutzen wir etree
        ET.register_namespace('gml','http://www.opengis.net/gml/3.2')
        root = ET.fromstring("<?xml version='1.0' encoding='UTF-8'?><snippet xmlns:gml='http://www.opengis.net/gml/3.2'>" + context['bplan'].geltungsbereich_gml_25832 + "</snippet>")
        ns = {'gml': 'http://www.opengis.net/gml/3.2',
        }
        # print("<?xml version='1.0' encoding='UTF-8'?><snippet xmlns:gml='http://www.opengis.net/gml/3.2'>" + context['bplan'].geltungsbereich_gml_25832 + "</snippet>")
        # Test ob ein Polygon zurück kommt - damit wäre nur ein einziges Polygon im geometry Field
        polygons = root.findall('gml:Polygon', ns)
        # print(len(polygons))
        if len(polygons) == 0:
            # print("Kein Polygon auf oberer Ebene gefunden - es sind wahrscheinlich mehrere!")
            multi_polygon_element = root.find('gml:MultiSurface', ns)
            uuid_multisurface = uuid.uuid4()
            multi_polygon_element.set("gml:id", "GML_" + str(uuid_multisurface))
            # Füge gml_id Attribute hinzu - besser diese als Hash aus den Geometrien zu rechnen, oder in Zukunft generic_ids der Bereiche zu verwenden 
            polygons = root.findall('gml:MultiSurface/gml:surfaceMember/gml:Polygon', ns)
            for polygon in polygons:
                uuid_polygon = uuid.uuid4()
                polygon.set("gml:id", "GML_" + str(uuid_polygon))
            context['multisurface_geometry_25832'] = ET.tostring(multi_polygon_element, encoding="utf-8", method="xml").decode('utf8')
        else:
            polygon_element = root.find('gml:Polygon', ns)
            polygon_element.set("xmlns:gml", "http://www.opengis.net/gml/3.2")   
            uuid_polygon = uuid.uuid4()
            polygon_element.set("gml:id", "GML_" + str(uuid_polygon))
            # Ausgabe der Geometrie in ein XML-Snippet - erweitert um den MultiSurface/surfaceMember Rahmen
            ET.dump(polygon_element) 
            context['multisurface_geometry_25832'] = '<gml:MultiSurface srsName="EPSG:25832"><gml:surfaceMember>' + ET.tostring(polygon_element, encoding="utf-8", method="xml").decode('utf8') + '</gml:surfaceMember></gml:MultiSurface>'
        return context

    """"
    def get_object(self):
        single_object = super().get_object(self.get_queryset())
        # print(single_object.geltungsbereich)
        # print(single_object.geltungsbereich_gml_25832)
        return single_object
    """

    def dispatch(self, *args, **kwargs):
        response = super().dispatch(*args, **kwargs)
        response['Content-type'] = "application/xml"  # set header
        return response
``` 

## URL für Export

Um die Export Funktion nutzen zu können, brauchen wir noch einen neuen Endpunkt.

xplanung_light/urls.py
```python
#...
from xplanung_light.views import BPlanCreateView, BPlanUpdateView, BPlanDeleteView, BPlanListView, BPlanDetailXmlRasterView
#...
 # export xplanung gml
    path("bplan/<int:pk>/xplan/", BPlanDetailXmlRasterView.as_view(template_name="xplanung_light/bplan_template_xplanung_raster_6.xml"), name="bplan-export-xplan-raster-6"),
#...
```

## Link in Table View

Den Link auf den Endpunkt übernehmen wir in die Bebauungsplantabelle

xplanung_light/tables.py
```python
#...
class BPlanTable(tables.Table):
    #download = tables.LinkColumn('gedis-document-pdf', text='Download', args=[A('pk')], \
    #                     orderable=False, empty_values=())
    xplan_gml = tables.LinkColumn('bplan-export-xplan-raster-6', text='Exportieren', args=[A('pk')], \
                         orderable=False, empty_values=())
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
        fields = ("name", "gemeinde", "planart", "xplan_gml", "edit", "delete")
```

## XML Template

Fehlt nur noch das Template ;-)

xplanung_light/bplan_template_xplanung_raster_6.xml
```jinja
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<xplan:XPlanAuszug xmlns:xplan="http://www.xplanung.de/xplangml/6/0" xmlns:gml="http://www.opengis.net/gml/3.2" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:wfs="http://www.opengis.net/wfs" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xsi:schemaLocation="http://www.xplanung.de/xplangml/6/0 http://repository.gdi-de.org/schemas/de.xleitstelle.xplanung/6.0/XPlanung-Operationen.xsd" gml:id="{{ auszug_uuid }}">
  <gml:boundedBy>
    <gml:Envelope srsName="EPSG:25832">
      <gml:lowerCorner>567015.8040 5937951.7580</gml:lowerCorner>
      <gml:upperCorner>567582.8240 5938562.2710</gml:upperCorner>
    </gml:Envelope>
  </gml:boundedBy>
  <gml:featureMember>
    <xplan:BP_Plan gml:id="{{ bplan_uuid }}">
      <gml:boundedBy>
        <gml:Envelope srsName="EPSG:25832">
          <gml:lowerCorner>{{ extent.0 }} {{ extent.1 }}</gml:lowerCorner>
          <gml:upperCorner>{{ extent.2 }} {{ extent.3 }}</gml:upperCorner>
        </gml:Envelope>
      </gml:boundedBy>
      <xplan:name>{{ bplan.name }}</xplan:name>
      <xplan:erstellungsMassstab>1000</xplan:erstellungsMassstab>
      <xplan:raeumlicherGeltungsbereich>
            {% autoescape off %}
            {{ multisurface_geometry_25832 }}
            {% endautoescape %}
      </xplan:raeumlicherGeltungsbereich>
      <xplan:gemeinde>
        <xplan:XP_Gemeinde>
          <xplan:ags>{{ bplan.gemeinde.ls }}{{ bplan.gemeinde.ks }}{{ bplan.gemeinde.vs }}{{ bplan.gemeinde.gs }}</xplan:ags>
          <xplan:gemeindeName>{{ bplan.gemeinde.name }}</xplan:gemeindeName>
        </xplan:XP_Gemeinde>
      </xplan:gemeinde>
      <xplan:planArt>{{ bplan.planart }}</xplan:planArt>
      <xplan:staedtebaulicherVertrag>false</xplan:staedtebaulicherVertrag>
      <xplan:erschliessungsVertrag>false</xplan:erschliessungsVertrag>
      <xplan:durchfuehrungsVertrag>false</xplan:durchfuehrungsVertrag>
      <xplan:gruenordnungsplan>false</xplan:gruenordnungsplan>
    </xplan:BP_Plan>
  </gml:featureMember>
</xplan:XPlanAuszug>
```