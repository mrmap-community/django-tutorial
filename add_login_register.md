
# Nutzung von Standardfunktionen des auth packages für Anmeldung und Registrierung

## Anpassung der Pfade in xplanung_light/urls.py

```python
from django.contrib.auth import views as auth_views
#...
path("accounts/login/", auth_views.LoginView.as_view(next_page="home"), name="login"),
path("accounts/logout/", auth_views.LogoutView.as_view(next_page="home"), name='logout'),
# https://dev.to/donesrom/how-to-set-up-django-built-in-registration-in-2023-41hg
path("register/", views.register, name = "register"),
#...
```

## HTML-Templates

### Erstellen eines Verzeichnisses für Registrierungstemplates

Erstellung des Verzeichnisses für registration - pwd: komserv2/xplanung_light/templates/
```shell
mkdir xplanung_light/templates/registration
```

### Templates
xplanung_light/templates/registration/login.html
```jinja
{% extends "../xplanung_light/layout.html" %}
{% block content %}
{% if form.errors %}
<p>Your username and password didn't match. Please try again.</p>
{% endif %}
{% if next %}
    {% if user.is_authenticated %}
    <p>Your account doesn't have access to this page. To proceed,
    please login with an account that has access.</p>
    {% else %}
    <p>Please login to see this page.</p>
    {% endif %}
{% endif %}
<form method="post" action="{% url 'login' %}">
{% csrf_token %}
<table>
<tr>
    <td>{{ form.username.label_tag }}</td>
    <td>{{ form.username }}</td>
</tr>
<tr>
    <td>{{ form.password.label_tag }}</td>
    <td>{{ form.password }}</td>
</tr>
</table>
<input type="submit" value="Einloggen">
<input type="hidden" name="next" value="{{ next }}">
</form>
<p>Noch keinen Zugang? <a href="{% url 'register' %}" class="navbar-item">Hier Registrieren</a></p>
{# Assumes you set up the password_reset view in your URLconf #}
{# <p><a href="{% url 'password_reset' %}">Lost password?</a></p> #}
{% endblock %}
```

xplanung_light/templates/registration/register.html
```jinja
{% extends "../xplanung_light/layout.html" %}
{% block content %}
<h2>Registrieren</h2>
    <form method="post">
        {% csrf_token %}
        {{form.as_p}}
        <button type="submit">Registrieren</button>
    </form>
{% endblock %}
```

## Formular für Registrierung

xplanung_light/forms.py
```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class RegistrationForm(UserCreationForm):

    email = forms.EmailField(required=True)

    class Meta:
        model = User
        fields = ['username', 'email', 'password1', 'password2']
```

## Anpassung der views

xplanung_light/views.py
```python
#...
from xplanung_light.forms import RegistrationForm
from django.shortcuts import redirect
from django.contrib.auth import login
#...
# https://dev.to/balt1794/registration-page-using-usercreationform-django-part-1-21j7
def register(request):
    if request.method != 'POST':
        form = RegistrationForm()
    else:
        form = RegistrationForm(request.POST)
        if form.is_valid():
            form.save()
            user = form.save()
            login(request, user)
            return redirect('home')
        else:
            print('form is invalid')
    context = {'form': form}
    return render(request, 'registration/register.html', context)
```

## Stylesheets optimieren

xplanung_light/static/xplanung_light/site.css
```css
/* ... */
.navbar {
    background-color: lightslategray;
    font-size: 1em;
    font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
    color: white;
    padding: 8px 5px 8px 5px;
}

.navbar a {
    text-decoration: none;
    color: inherit;
}

.navbar-brand {
    font-size: 1.2em;
    font-weight: 600;
}

.navbar-item {
    font-variant: small-caps;
    margin-left: 30px;
}

.body-content {
    padding: 5px;
    font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
```

## Was wir bisher haben

1. Nutzung von Templates
2. Einfache function based views
3. Anwendnung von css
4. Formulare
5. Registrierung
6. Authentifizierung

Dokumentation auf xplanung_light/templates/xplanung_light/home.html
```jinja
<!-- ... -->
<h4>Funktionen</h4>
<ul>
    <li>Homepage</li>
    <li>Authentifizierung gegen Datenbank</li>
    <li>Registrierung</li>
    <li>Admin-Backend</li>
    <li>...</li>
</ul>
<!-- ... -->
```

Aussehen nach Anmeldung des admin im Frontend

[http://127.0.0.1:8000/](http://127.0.0.1:8000/)

```{image} img/standardfunktionen_1.png
:alt: Standardfunktionen
:class: bg-primary
:width: 800px
:align: center
```