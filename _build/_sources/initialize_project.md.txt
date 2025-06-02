# Django Projekt initialisieren (Standard-Terminal oder VSCODE-Terminal)

Im Arbeitsverzeichnis komserv2 - venv aktiviert!
Django Boilerplate Code anlegen
```shell
django-admin startproject komserv .
```

Erste App anlegen
```shell
python manage.py startapp xplanung_light
```

Datenbankschema erstellen und automatisch initiale SQLITE DB anlegen
```shell
python manage.py migrate
```

Starten des Django-Servers auf lokalem Port 8000
```shell
python manage.py runserver
```

# Frontend testen

[http://127.0.0.1:8000](http://127.0.0.1:8000)

```{image} img/initial_django_project.png
:alt: Liste der publizierenden Orgas
:class: bg-primary
:width: 800px
:align: center
```