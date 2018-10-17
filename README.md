# Vorarbeiten

Im AD werden die Testbenutzer mit Email angelegt.

# privacyIDEA installieren

privacyIDEA ist eine Python Web-Applikation und kann auf viele verschiedene Weisen 
installiert werden.
(Siehe  https://privacyidea.readthedocs.io/en/latest/installation/index.html)

Im Workshop wählen wir die Installation auf einem Ubuntu 16.04LTS:

~~~~bash
add-apt-repository ppa:privacyidea/privacyidea
apt update
apt install privacyidea-apache2
~~~~

## Administratoren

Nach der Installation existiert kein Standardadmin!
Der erste *interne* Administrator wird über das Tool ``pi-manage`` angelegt.

Mehr Infos zu Admins: https://privacyidea.readthedocs.io/en/latest/faq/admins.html

## Benutzer

Benutzer liest privacyIDEA aus existierenden Benutzerquellen wie:

* Flatfile
* SQL
* LDAP

Benutzerquellen werden über "UserResolver" angesprochen. 
Mehrere UserResolver können zu einem Realm zusammengefasst werden.

## Aufgaben

* Installieren Sie privacyIDEA.
* Legen Sie Administratoren an.
* Nutzen Sie den Administrator, um sich am UI anzumelden.
* Wo wird der Administrator gespeichert?
* Legen Sie einen LDAP Resolver an und einen Realm, um Benutzer in privacyIDEA zu sehen.

# Token

privacyIDEA unterstützt verschiedene Tokentypen, verschiedene Algorithmen, Hardware- und Software-Token.

Manche Token können selber initialisiert werden wie (Yubikey, Smartphone-App, Feitian C200).
Andere Token kommen vom Hersteller mit einem Tokenseed.

Bei der Authentifizierung wird PIN und OTP-Wert hintereinander eingegeben.

Bei Challenge-Response-Token wird erst nur die PIN eingegeben und dann bspw. eine SMS oder Email versendet.

Siehe Tokentypen: https://privacyidea.readthedocs.io/en/latest/configuration/token_config.html

## Aufgaben

* Tokendatei importieren.
* Token einem Benutzer zuweisen.
* Token in den Tokendetails testen.
* Authentifizierung im Audit-Log betrachten.
* Google Authenticator oder privacyIDEA Authenticator ausrollen.

# Authentifizierung via RADIUS

Die Authentifizierung gegen privacyIDEA erfolgt über eine REST-API ``/validate/check``.
Siehe https://privacyidea.readthedocs.io/en/latest/modules/api/validate.html

Die Authentifizierung kann bspw. mit dem Kommandozeilen-Tool ``httpie`` gut getestet werden.

~~~~bash
http --verify no POST https://localhost/validate/check user=hasn pass=secret123456
~~~~

Es gibt Plugins für Web-Applikationen, SSH, Windows Credential Provider, SAML... und FreeRADIUS.
Unter Ubuntu kann das Paket ``privacyidea-radius`` installiert werden und ein Test wie folgt erfolgen:

~~~~bash
echo "User-Name=hans, User-Password=secret123456" | radclient -x -s localhost auth testing123
~~~~

Alle Firewalls und VPNs lassen sich i.d.R. über RADIUS anbinden.

## Aufgaben

* Testen Sie die Authentifizierung mit der REST API. 
  Wie verhält sich der Response, wenn man falsche PINs oder OTP-Werte eingibt?
* Testen Sie einen RADIUS-Request gegen privacyIDEA.

# Policies / Richtlinien

Richtlinien bestimmen, wie sich privacyIDEA in gewissen Fällen verhalten soll.
So kann bspw. definiert werden, ob der Benutzer eine PIN eingeben soll oder ein LDAP-Passwort.
Die Antwort von privacyIDEA kann modifziert werden.
Die Rechte von Administratoren und Benutzern können beschnitten werden.

Siehe https://privacyidea.readthedocs.io/en/latest/policies/index.html

## Aufgaben

* Legen Sie eine Richtlinie an, die als PIN das LDAP-Passwort des Benutzers erfordert.
* Legen Sie eine Richtlinie an, die den Benutzer zwingt, einen privacyIDEA Authenticator
  im 2step-enrollment auszurollen. Damit wird der Token an genau das eine Smartphone
  gebunden.

## Migration und Passthru

Die Passthru-Richtlinie im Scope *authentication* ermöglicht eine leichte Migration, wenn schon vorher ein RADIUS Server existiert.

Die Authentifizierungsanfrage wird, wenn der Benutzer in privacyIDEA keinen Token hat, an den angegebenen RADIUS-Server weitergeleitet.
Dieser beantwortet dann die an privacyIDEA gesendete Anfrage. Sobald dem Benutzer in privacyIDEA ein Token zugewiesen wurde, wird die Anfrage
nicht mehr weitergeleitet. Wenn allen Benutzern in privacyIDEA ein Token zugewiesen wurde, werden keine Anfragen an den alten RADIUS-Server
weitergeleitet und das alte System kann abgeschaltet werden.

Siehe https://privacyidea.readthedocs.io/en/latest/policies/authentication.html#passthru

### Aufgaben

* Definieren Sie einen externen RADIUS-Server.
* Definieren Sie eine Passthru-Policy auf diesen RADIUS-Server.
* Testen Sie die Weiterleitung der Anfrage.

## Helpdesk Benutzer

Im Echtbetrieb möchte man nicht nur mit internen Administratoren arbeiten!

### Aufgaben

* Im AD gibt es eine Gruppe "IT". Machen Sie diese Gruppe zu Administratoren in einem Realm "helpdesk".
* Verbieten Sie diesen Helpdesk-Benutzern, die Konfiguration des Systems zu ändern.

# Event Handler

Event Handler können zusätzliche Aktionen auslösen, wenn ein vorher definiertes Ereignis eintritt.
Das Ereignis kann mit verschiedenen Bedingungen verknüpft werden. Die Ereignisse entsprechen dabei 
den verschiedenen REST API endpoints.

Die ausgelösten Aktionen definieren sich aus Event Handler Modulen.
Es gibt zur Zeit die folgenden Module:

* UserNotification
* Token
* Federation
* Script
* Counter

Siehe https://privacyidea.readthedocs.io/en/latest/eventhandler/index.html

## Aufgaben

* Legen Sie einen Event Handler an, der den Benutzer mit einer Email benachrichtigt, wenn 
  der Administrator für ihn einen neuen Token ausrollt.
* Legen Sie einen Event Handler an, der den Token deaktiviert, wenn der Administrator
  einen TOTP-Token ausrollt.
* Legen Sie einen Event Handler an, der dem Benutzer einen Email-Token ausrollt, wenn der
  Benutzer sich authentisiert, aber keinen Token hat! Diesen Token kann der Benutzer sofort
  für die Authentifizierung verwenen.
* Legen Sie einen Event Handler an, der die fehlgeschlagenen Authentifizierungsanfragen zählt.

# Periodic Tasks

*Periodic Tasks* sind regelmäßig ausgeführte Aufgaben. Diese werden über einen cronjob gesteuert.

Die *Periodic Tasks* nutzen Module, die definieren, was regelmäßig wiederholt werden soll.
Derzeit existieren die Module

* EventCounter
* SimpleStats

Mit dem Modul *EventCounter* können aus den "gezählten" Ereignissen (aus dem Event Handler) Zeitreihen erzeugt
werden.

SimpleStats schreibt in regelmäßigen Abständen "Messwerte" wie die Tokenanzahl in eine Zeitreihe.

.. note:: Prinzipiell sind verschiedene PeriodicTaskModule denkbar. So könnte es auch
   Module geben, die bspw. die Tokendatenbank aufräumen oder das Auditlog rotieren. 

Siehe https://privacyidea.readthedocs.io/en/latest/periodictask/index.html

## Aufgaben

* Erstellen Sie eine Aufgabe, die die fehlgeschlagenen Authentifizierungen in eine Zeitreihe schreibt.
* Erstellen Sie eine Aufgabe, die Anzahl der Token in eine Zeitreihe schreibt.

## Statistiken und Grafana

In der Datenbanktabelle ``monitoringstats`` werden die Zeitreihen gespeichert.
Die Spalte "stats_key" bezeichnet dabei jeweils einen Messwert.
Diese Werte können mit beliebigen Tools dargestellt werden.

Bspw. bringt Grafana bereits ein Plugin, um Statistik-Daten aus einer MySQL-Datenbank auszulesen.

### Aufgaben

* Installieren Sie die Pakete ``grafana`` und ``grafana-data``.
* Erstellen Sie in Grafana ein View, der die fehlgeschlagenen Authentifizierungen über die Zeit anzeigt.
