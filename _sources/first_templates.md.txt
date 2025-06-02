
# Erstellung erster Versionen der Templates

## Basis Template xplanung_light/templates/xplanung_light/layout.html

```jinja
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <meta name="description" content="Author: Armin Retterath, XPlanung, Django, Formular, Easy, kostenfrei, Open Source"/>
    <title>{% block title %}{% endblock %}</title>
    {% load static %}
    <link rel="stylesheet" type="text/css" href="{% static 'xplanung_light/site.css' %}"/>
    
</head>
<body>
    <div class="navbar">
        <a href="{% url 'home' %}" class="navbar-brand">XPlanung light</a>
        <a href="{% url 'about' %}" class="navbar-item">Über</a>
        {% if user.is_authenticated %}
            <p>
                Angemeldeter Benutzer: {{ user.username }} <br>
                <a href="{% url 'logout' %}" class="navbar-item">Abmelden</a>
            </p>
            <p><a href="{% url 'admin:index' %}" >Admin Backend</a></p>
        {% else %}
            <a href="{% url 'login' %}" class="navbar-item">Anmelden</a>
        {% endif %}
    </div>
    <div class="body-content">
        {% block content %}
        {% endblock %}
        <hr/>
        <footer>
            <p>&copy; 2025</p>
            <p>Letzte Änderung: 2025-04-04 11:40 Initiales Anlegen</p>
        </footer>
    </div>
</body>
</html>
```

## Anpassung der views

Function based views für home und about Seiten - xplanung_light/views.py
```python
#...
from django.shortcuts import render

def home(request):
    return render(request, "xplanung_light/home.html")
    
def about(request):
    return render(request, "xplanung_light/about.html")
#...
```

url für about zu xplanung_light/urls.py hinzufügen
```python
#...
path("about/", views.about, name="about"),
#...
```

## Template für home - xplanung_light/templates/xplanung_light/home.html

```jinja
{% extends "xplanung_light/layout.html" %}
{% block title %}
Home
{% endblock %}
{% block content %}
<p>Einfache Webanwendung zur Verwaltung kommunaler Pläne gemäß dem aktuellen Standard <a href="https://xleitstelle.de/xplanung" target="_blank">XPlanung</a></p>
<p>Der Betreiber übernimmt keinerlei Verantwortung für die Funktionsfähigkeit und Zuverlässigkeit der Anwendung. Es handelt sich nur um ein <b>Proof of Concept</b>. Solange die Anwendung online ist, kann und darf sie von jedermann kostenfrei verwendet werden. Für die Sicherheit der Daten wird ebenfalls keine Verantwortung übernommen. Nutzer sollten nur ihre eigenen Daten sehen und editieren können. Das Projekt dient als Test zur Prüfung von Funktionen des zugrundeliegenden Webframeworks <a href="https://www.djangoproject.com/" target="_blank">Django</a> und wurde innerhalb weniger Tage unter Nutzung von <a href="https://docs.djangoproject.com/en/5.0/topics/class-based-views/generic-display/" target="_blank">class-based generic views</a> umgesetzt ;-) .</p>
<h4>Funktionen</h4>
<ul>
    <li>...</li>
    <li>...</li>
</ul>
<h4>Technische Informationen</h4>
<ul>
    <li><a href="#" target="_blank">Projekt auf GitHub</a></li>
    <li><a href="#" target="_blank">Github Repo der ...</a></li>
    <li><a href="#" target="_blank">Standard</a></li>
</ul>
<h4>Validatoren</h4>
<ul>
    <li><a href="#" target="_blank">...</a></li>
</ul>
{% endblock %}
```

## Template für about - xplanung_light/templates/xplanung_light/about.html

```jinja
{% extends "xplanung_light/layout.html" %}
{% block title %}
Über
{% endblock %}
{% block content %}
<p>Anwendung zur einfachen Verwaltung von kommunalen Plänen, konform zum deutschen Standard <b>XPlanung</b>.</p>
{% endblock %}
```
