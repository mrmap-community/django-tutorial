# Erste Schritte

## Startseite anpassen (einfache FunctionBasedView)

xplanung_light/views.py
```python
from django.http import HttpResponse

def home(request):
    return HttpResponse("Hello, XPlanung!")
```

urls.py erstellen: xplanung_light/urls.py
```python
from django.urls import path
from xplanung_light import views

urlpatterns = [
    path("", views.home, name="home"),
]
```

Anpassung der Projekt urls.py: komserv/urls.py
```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("", include("xplanung_light.urls")),
    path('admin/', admin.site.urls)
]
```

App xplanung_light zur Konfiguration in komserv/settings.py hinzufügen
```python
#...
# Application definition
INSTALLED_APPS = [
  #...
    'xplanung_light',
  #...
]
#...
```

Erstellung der Verzeichnisse für staticfiles und templates - pwd: komserv2/xplanung_light/
```shell
mkdir templates
cd templates
mkdir xplanung_light
cd ..
mkdir static
cd static
mkdir xplanung_light
cd ..
```

Erstellung eines minimalen stylesheets: xplanung_light/static/xplanung_light/site.css
```css
.message {
    font-weight: 600;
    color: blue;
}
```
Anlegen der Konfiguration für die Ablage der static files in komserv/settings.py
```python
#...
STATIC_ROOT = BASE_DIR / 'static_collected'
#...
```

Kopieren der static files in die vorgesehenen Ordner
```shell
python3 manage.py collectstatic
```
Ausgabe
```shell
126 static files copied to 'XXX/komserv2/static_collected'.
```

Superuser anlegen
```shell
python3 manage.py createsuperuser --username=admin --email=admin@example.com
```

## Ausprobieren der Admin Oberfläche und der Startseite

dev-Server beenden (je nach Umgebung) und neu starten - jetzt am besten in VSCODE über F5 - Run->Start Debugging

http://127.0.0.1:8000/

http://127.0.0.1:8000/admin/

## Aktivieren der Debugtoolbar

[django-debug-toolbar](https://django-debug-toolbar.readthedocs.io)

Installation des Pakets

```shell
python3 -m pip install django-debug-toolbar
```

Anpassen der Konfiguration

komserv/settings.py
```python
#...
# Application definition
INSTALLED_APPS = [
  #...
    'debug_toolbar',
  #...
]
#...
MIDDLEWARE = [
    #...
    'debug_toolbar.middleware.DebugToolbarMiddleware',
]
#...
INTERNAL_IPS = [
    # ...
    "127.0.0.1",
    # ...
]
```

Erweiterung der URLs

komserv/urls.py
```python
#...
from debug_toolbar.toolbar import debug_toolbar_urls
#...
urlpatterns += staticfiles_urlpatterns() + debug_toolbar_urls()
```
