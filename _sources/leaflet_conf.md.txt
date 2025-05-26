# Zentrale Konfiguration

komserv/settings.py
```python
#...
LEAFLET_CONFIG = {
    # conf here
    'SPATIAL_EXTENT': (6.0, 49.0, 8.5, 52),
    'DEFAULT_CENTER': (7.0, 50.0),
    'DEFAULT_ZOOM': 7,
    'MIN_ZOOM': 2,
    'MAX_ZOOM': 20,
    'DEFAULT_PRECISION': 6,
}
#...
```

# WMS Layer hinzufügen

Muss im im jeweiligen Template erfolgen, da die zentrale Leaflet Konfiguration nur zusätzliche Tiled Layer erlaubt.
Siehe auch [https://stackoverflow.com/questions/66938889/how-to-add-leaflet-extensions-marker-basemap-geocoder-to-django-leaflet](https://stackoverflow.com/questions/66938889/how-to-add-leaflet-extensions-marker-basemap-geocoder-to-django-leaflet)


xplanung_light/templates/xplanung_light/bplan_form.html
```jinja
{% extends "xplanung_light/layout.html" %}
{% block title %}
    Bebauungsplan anlegen
{% endblock %}
{% block content %}
<script type="text/javascript">
    window.addEventListener("map:init", function(e) {
            var detail = e.detail;
            var map = detail.map;
            /* Transparent overlay layers */
            var wmsLayer = L.tileLayer.wms('https://geo5.service24.rlp.de/wms/liegenschaften_rp.fcgi?', {
                layers: 'Flurstueck',
                format: 'image/png',
                transparent: true,
            }).addTo(map);
            // and many more
        }, false
    ); //end of window.addEventListener
</script>
    <form method="post" class="geocoding-form">{% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Speichern">
    </form>
{% endblock %}
```