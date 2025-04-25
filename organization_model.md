# Organisationsmodell

## Einleitung

Das initiale Modell für verantwortliche Organisationen soll die Gebietskörperschaften des Landes abbilden. Diese sind grundsätzlich hierarchisch geliedert und verfügen über bundesweit einheitlich definierte Schlüssel (AGS). Verantwortlich für die Bereitstellung der Struktur der Gebietskörperschaften ist in Rheinland-Pfalz das Statistische Landesamt. Dieses publiziert das s.g. [**Amtliche Verzeichnis der Gemeinden und Gemeindeteile**](https://www.statistik.rlp.de/fileadmin/statistik.rlp.de/Dokumente_und_Bilder/3_Produkte/3_Verzeichnisse/A1132_202201_ur_G.pdf) in Form einer PDF-Datei. Leider lässt sich das nicht ohne weiteres automatisiert verarbeiten, es gibt jedoch auch ein [**Verzeichnis der Kommunalverwaltungen, Oberbürgermeister, Landräte und Bürgermeister**](https://www.statistik.rlp.de/fileadmin/statistik.rlp.de/Dokumente_und_Bilder/3_Produkte/3_Verzeichnisse/Kommunalverwaltungen_01.01_2025.xlsm). Dieses wird als xslm Mappe publiziert und enthält über verschiedene Tabellen verteilt die für unseren Zweck benötigte information.

Wir benötigen sowohl Kontaktinformationen, als auch Adressen und Zuständigkeitsbereiche. Die Zuständigkeitsbereiche werden vom Landesamt für Vermessung und Geobasisinformation über eine OGC-WFS Schnittstelle publiziert und enthalten auch die jeweiligen einheitlichen AGS. Damit lassen sie sich automatisiert mit den Organisationen verknüpfen.
Um das Modell nicht unnötig kompliziwert zu machen, speichern wir alle Informationen zunächst in einem einfachen flachen Modell.
In Django werden Modelle pro App in einer Datei models.py abgelegt. In unserem Fall also xplanung_light/models.py. Da die Modelle python-Klassen sind, lässt sich vieles über Vererbung umsetzen. Wir definieren dazu einfach eine simple Klasse GenericMetadata, die zunächst einfach eine automatisch generierte UUID beinhaltet, die die davon abgeleiteteten Klassen erben.
Die Integration von django-simple-history ist sehr einfach und erfolgt durch das Attribut **history = HistoricalRecords()**.

## Modelldefinition

xplanung_light/models.py
```python
from django.db import models
from django.contrib.auth.models import User
import uuid
from simple_history.models import HistoricalRecords
from django.contrib.gis.db import models

# generic meta model
class GenericMetadata(models.Model):
    generic_id = models.UUIDField(default = uuid.uuid4)
    #created = models.DateTimeField(null=True)
    #changed = models.DateTimeField(null=True)
    #deleted = models.DateTimeField(null=True)
    #active = models.BooleanField(default=True)
    #owned_by_user = models.ForeignKey(User, on_delete=models.CASCADE, null=True)

    class Meta:
        abstract = True

    """def save(self, *args, **kwargs):
        self.owned_by_user= self.request.user
        super().save(*args, **kwargs)"""


# administrative organizations
class AdministrativeOrganization(GenericMetadata):

    COUNTY = "KR"
    COUNTY_FREE_CITY = "KFS"
    COM_ASS = "VG"
    COM_ASS_FREE_COM = "VFG"
    COM = "GE"
    UNKNOWN = "UK"

    ADMIN_CLASS_CHOICES = [
        (COUNTY,  "Landkreis"),
        (COUNTY_FREE_CITY, "Kreisfreie Stadt"),
        (COM_ASS, "Verbandsgemeinde"),
        (COM_ASS_FREE_COM, "Verbandsfreie Gemeinde"),
        (COM, "Gemeinde/Stadt"),
        (UNKNOWN, "unbekannt"),
    ]

    ls = models.CharField(max_length=2, verbose_name='Landesschlüssel', help_text='Eindeutiger zweistelliger Schlüssel für das Bundesland - RLP: 07', default='07')
    ks = models.CharField(max_length=3, verbose_name='Kreisschlüssel', help_text='Eindeutiger dreistelliger Schlüssel für den Landkreis', default='000')
    vs = models.CharField(max_length=2, verbose_name='Gemeindeverbandsschlüssel', help_text='Eindeutiger zweistelliger Schlüssel für den Gemeindeverband', default='00')
    gs = models.CharField(max_length=3, verbose_name='Gemeindeschlüssel', help_text='Eindeutiger dreistelliger Schlüssel für die Gemeinde', default='000')
    name = models.CharField(max_length=1024, verbose_name='Name der Gebietskörperschaft', help_text='Offizieller Name der Gebietskörperschaft - z.B. Rhein-Lahn-Kreis')
    type = models.CharField(max_length=3, choices=ADMIN_CLASS_CHOICES, default='UK', verbose_name='Typ der Gebietskörperschaft', db_index=True)
    address_street = models.CharField(blank=True, null=True, max_length=1024, verbose_name='Straße mit Hausnummer', help_text='Straße und Hausnummer')
    address_postcode = models.CharField(blank=True, null=True, max_length=5, verbose_name='Postleitzahl', help_text='Postleitzahl')
    
    address_city = models.CharField(max_length=256, blank=True, null=True, verbose_name='Stadt')
    address_phone = models.CharField(max_length=256, blank=True, null=True, verbose_name='Telefon')
    address_facsimile = models.CharField(max_length=256, blank=True, null=True, verbose_name='Fax')
    address_email = models.EmailField(max_length=512, blank=True, null=True, verbose_name='EMail')
    address_homepage = models.URLField(blank=True, null=True, verbose_name='Homepage')
    geometry = models.GeometryField(blank=True, null=True, verbose_name='Gebiet')
    history = HistoricalRecords()

    def __str__(self):
        """Returns a string representation of a administrative unit."""
        return f"'{self.type}' '{self.name}'"
```

Migration
```shell
python3 manage.py makemigrations
python3 manage.py migrate
```

## Datenimport

Da die benötigten Daten für die Organisationen als xslm und in Form von Webservices bereitstehen, benötigen wir zwei weitere python-Bibliotheken.

```shell
python3 -m pip install openpyxl
python3 -m pip install requests
```

Wir laden die xslm Datei zunächst händisch herunter und legen sie in unser Arbeitsverzeichnis.

Die Funktionen für den Import speichern wir in xplanung_light/views.py. Hier ist wichtig, dass die Proxy-Einstellungen in der Datei angepasst werden. Für produktive Zwecke ist es besser, die Proxy-Konfiguration in der zentralen Konfigurationsdatei abzulegen und zu importieren. Hierzu später mehr.

xplanung_light/views.py
```python
#...
from xplanung_light.models import AdministrativeOrganization
from django.contrib.gis.geos import GEOSGeometry
from openpyxl import Workbook, load_workbook
import requests
#...

PROXIES = {
    'http_proxy': 'http://xxx:8080',
    'https_proxy': 'http://xxx:8080',
}

def get_geometry(type, ags):
    if type =='KR' or type =='KFS':
        base_uri = "https://www.geoportal.rlp.de/spatial-objects/314/collections/vermkv:landkreise_rlp"
        param_dict = {'f': 'json', 'kreissch': ags[:3]}
    if type =='VG' or type =='VFG':
        base_uri = "https://www.geoportal.rlp.de/spatial-objects/314/collections/vermkv:verbandsgemeinde_rlp"
        param_dict = {'f': 'json', 'vgnr': ags[:5]}
    if type == 'GE':
        base_uri = "https://www.geoportal.rlp.de/spatial-objects/314/collections/vermkv:gemeinde_rlp"
        param_dict = {'f': 'json', 'ags': "*7" + ags}
    resp = requests.get(url=base_uri, params=param_dict, proxies=PROXIES)
    print(base_uri)
    print(str(param_dict))
    data = resp.json() 
    return str(data['features'][0]['geometry'])

def import_organisations():
    wb = load_workbook('Kommunalverwaltungen_01.01_2025.xlsm')
    # nuts-1 - bundeslandebene
    # nuts-2 - regierungsbezirke
    # nuts-3 - landkreisebene
    # lau-1 - verbandsgemeindeebene
    # lau-2 - gemeideebene

    table_all_admin_units = wb.worksheets[10]
    table_nuts_3_1 = wb.worksheets[5]
    table_nuts_3_2 = wb.worksheets[6]
    table_lau_1_1 = wb.worksheets[8]
    table_lau_1_2 = wb.worksheets[7]
    # table_lau_2 = wb.worksheets[10]

    count_landkreise = 0
    count_kreisfreie_staedte = 0
    count_verbandsgemeinden = 0
    count_verbandsfreie_gemeinden = 0
    count_gemeinden = 0
    # read landkreisebene
    landkreisebene = {}
    i = 0
    for row in table_nuts_3_1.iter_rows(values_only=True):
        i = i + 1
        if i > 2:
            if row[0] != None:
                landkreis = {}
                landkreis['kr'] = row[0]
                landkreis['vg'] = row[2]
                landkreis['ge'] = row[1]
                landkreis['name'] = row[4]
                landkreis['type'] = 'KR'
                landkreis['address'] = {}
                landkreis['address']['street'] = row[9]
                landkreis['address']['postcode'] = row[10]
                landkreis['address']['city'] = row[11]
                landkreis['address']['phone'] = str(row[12]) + '/' + str(row[13])
                landkreis['address']['facsimile'] = str(row[12]) + '/' + str(row[14])
                landkreis['address']['email'] = str(row[15])
                landkreis['address']['homepage'] = "https://" + str(row[16])
                landkreisebene[row[0] + row[2] + row[1]] = landkreis
                count_landkreise = count_landkreise + 1
    i = 0           
    for row in table_nuts_3_2.iter_rows(values_only=True):
        i = i + 1
        if i > 2:
            if row[0] != None:
                kreisfreie_stadt = {}
                kreisfreie_stadt['kr'] = row[0]
                kreisfreie_stadt['vg'] = row[2]
                kreisfreie_stadt['ge'] = row[1]
                kreisfreie_stadt['name'] = row[4]
                kreisfreie_stadt['type'] = 'KFS'
                kreisfreie_stadt['address'] = {}
                kreisfreie_stadt['address']['street'] = row[9]
                kreisfreie_stadt['address']['postcode'] = row[10]
                kreisfreie_stadt['address']['city'] = row[11]
                kreisfreie_stadt['address']['phone'] = str(row[12]) + '/' + str(row[13])
                kreisfreie_stadt['address']['facsimile'] = str(row[12]) + '/' + str(row[14])
                kreisfreie_stadt['address']['email'] = str(row[15])
                kreisfreie_stadt['address']['homepage'] = "https://" + str(row[16])
                landkreisebene[row[0] + row[2] + row[1]] = kreisfreie_stadt
                count_kreisfreie_staedte = count_kreisfreie_staedte + 1
    # read verbandsgemeindeebene
    verbandsgemeindeebene = {}
    i = 0
    for row in table_lau_1_1.iter_rows(values_only=True):
        i = i + 1
        if i > 2:
            if row[0] != None:
                vg = {}
                vg['kr'] = row[0]
                vg['vg'] = row[2]
                vg['ge'] = row[1]
                vg['name'] = row[4]
                vg['type'] = 'VG'
                vg['address'] = {}
                vg['address']['street'] = row[9]
                vg['address']['postcode'] = row[10]
                vg['address']['city'] = row[11]
                vg['address']['phone'] = str(row[12]) + '/' + str(row[13])
                vg['address']['facsimile'] = str(row[12]) + '/' + str(row[14])
                vg['address']['email'] = str(row[15])
                vg['address']['homepage'] = "https://" + str(row[16])
                verbandsgemeindeebene[row[0] + row[2] + row[1]] = vg
                count_verbandsgemeinden = count_verbandsgemeinden + 1
    i = 0
    for row in table_lau_1_2.iter_rows(values_only=True):
        i = i + 1
        if i > 2:
            if row[0] != None:
                vg = {}
                vg['kr'] = row[0]
                vg['vg'] = row[2]
                vg['ge'] = row[1]
                vg['name'] = row[4]
                vg['type'] = 'VFG'
                vg['address'] = {}
                vg['address']['street'] = row[9]
                vg['address']['postcode'] = row[10]
                vg['address']['city'] = row[11]
                vg['address']['phone'] = str(row[12]) + '/' + str(row[13])
                vg['address']['facsimile'] = str(row[12]) + '/' + str(row[14])
                vg['address']['email'] = str(row[15])
                vg['address']['homepage'] = "https://" + str(row[16])
                verbandsgemeindeebene[row[0] + row[2] + row[1]] = vg
                count_verbandsfreie_gemeinden = count_verbandsfreie_gemeinden + 1
    #print(json.dumps(landkreise))
    all_admin_units = {}
    i = 0
    for row in table_all_admin_units.iter_rows(values_only=True):
        i = i + 1
        if i > 2:
            if row[0] != None:
                admin_unit = {}
                admin_unit['kr'] = row[0]
                admin_unit['vg'] = row[1]
                admin_unit['ge'] = row[2]
                admin_unit['name'] = row[3]
                print(admin_unit['name'])
                admin_unit['plz'] = row[4]
                admin_unit['type'] = 'GE'
                if row[1] == '00' and row[2] == '000':
                    admin_unit['type'] = landkreisebene[row[0] + row[1] + row[2]]['type']
                    admin_unit['address'] = landkreisebene[row[0] + row[1] + row[2]]['address']
                if row[1] != '00' and row[2] == '000':
                    admin_unit['type'] = verbandsgemeindeebene[row[0] + row[1] + row[2]]['type']
                    admin_unit['address'] = verbandsgemeindeebene[row[0] + row[1] + row[2]]['address']
                admin_unit['geometry'] = get_geometry(admin_unit['type'], str(row[0]) + str(row[1]) + str(row[2]))
                all_admin_units[str(row[0]) + str(row[1]) +str(row[2])] = admin_unit   
                #save object to database
                obj, created = AdministrativeOrganization.objects.update_or_create(
                    ks=admin_unit['kr'],
                    vs=admin_unit['vg'],
                    gs=admin_unit['ge'],
                    defaults={
                        "ks": admin_unit['kr'],
                        "vs": admin_unit['vg'],
                        "gs": admin_unit['ge'],
                        "name": admin_unit['name'],
                        "type": admin_unit['type'],
                        "geometry": GEOSGeometry(admin_unit['geometry'])
                    },
                )
                """
                administration = AdministrativeOrganization()
                administration.ks = admin_unit['kr']
                administration.vs = admin_unit['vg']
                administration.gs = admin_unit['ge']
                administration.name = admin_unit['name']
                administration.type = admin_unit['type']
                administration.geometry = GEOSGeometry(admin_unit['geometry'])
                administration.save()
                """

    print("Landkreise:" + str(count_landkreise))
    print("Kreisfreie Städte:" + str(count_kreisfreie_staedte))
    print("Verbandsgemeinden:" + str(count_verbandsgemeinden))
    print("Verbandsfreie Gemeinden:" + str(count_verbandsfreie_gemeinden))
    print(i)
```

Der initiale Import wird einfach von der shell gestartet. Wir nutzen hier die Django shell, über die man direkten Zugriff auf die Funktionen im Projekt erhält.

```shell
python3 manage.py shell
```

```python
from xplanung_light.views import import_organisations
import_organisations()
```