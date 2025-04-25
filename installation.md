# Installation der Python Umgebung

## Benötigte Software

1. VSCODE
2. Virtuelle python Umgebung
3. Internetzugang

## Los geht's am Terminal

Verzeichnis anlegen
```shell
mkdir komserv2
```
Ins Verzeichnis wechseln
```shell
cd komserv2
```
Virtuelle python-Umgebung anlegen
```shell
python3 -m venv .venv
```
Virtualle Umgebung aktivieren
```shell
source .venv/bin/activate
```
Paketmanager pip aktualisieren
```shell
python3 -m pip install --upgrade pip
```
Django installieren
```shell
python3 -m pip install django
```
VSCODE im Verzeichnis starten
```shell
code .
```
## VSCODE einrichten über Oberfläche

1. Python Umgebung wählen
2. Debugger konfigurieren - launch.json Datei in .vscode Ordner
```json
{
    "version": "0.2.0",
    "configurations": [
        
        {
            "name": "Python Debugger: Django",
            "type": "debugpy",
            "request": "launch",
            "args": [
                "runserver"
            ],
            "django": true,
            "autoStartBrowser": false,
            "program": "${workspaceFolder}/manage.py"
        }
    ]
}
```
3. Restart VSCODE - manchmal notwendig, um launch.json einlesen zu lassen

## Weitere Infos 
https://code.visualstudio.com/docs/python/tutorial-django