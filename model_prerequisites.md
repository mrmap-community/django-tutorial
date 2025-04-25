# Vorbereitung

Neben der Verwaltung räumlicher Daten, wie z.B. den Zuständigkeitsbereichen von Verwaltungen 
und Geltungsbereichen von Plänen beinhaltet das Modell organisatorische Zuständigkeiten, die 
einem zeitlichen Wandel unterliegen. Organisationsstrukturen könnne sich im Lauf von mehreren Jahren 
ändern und daher ist es sinnvoll diese zu historisieren. 
Man kann sich ein eigenes Historienkonzept überlegen, oder man nutzt ein vorhandenes Package. 
In diesem Fall wird [**django-simple-history**](https://django-simple-history.readthedocs.io/en/latest/) genutzt.

## Historie

Installation des packages
```shell
python3 -m pip install django-simple-history
```

Aktivierung in komserv/settings.py
```python
#...
INSTALLED_APPS = [
    # ...
    'simple_history',
]
#...
MIDDLEWARE = [
    # ...
    'simple_history.middleware.HistoryRequestMiddleware',
]
#...
```

Migration des Datenmodells
```shell
python3 manage.py makemigrations
python3 manage.py migrate
```

## Spatial Data

Da GeoDjango verwendet werden soll, müssen wir zunächst die Datenbank von SQLITE auf SPATIALITE ändern.
Hierzu reicht eine Anpassung in der globalen Konfigurationsdatei.

Aktivierung in komserv/settings.py
```python
DATABASES = {
    'default': {
        #'ENGINE': 'django.db.backends.sqlite3',
        'ENGINE': 'django.contrib.gis.db.backends.spatialite',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

Damit sind die Vorbreitungen auch schon abgeschlossen. Jetzt folgt die Definition der benötigten Datenmodelle.
