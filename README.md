
# KEX-Vorgang-Export-API

Die Schnittstelle ermöglicht das automatisierte Auslesen von Vorgängen in KreditSmart.  
Alle hier dokumentierten Schnittstellen sind [GraphQL-Schnittstellen](https://docs.api.europace.de/privatkredit/graphql/).

> :warning: Diese Schnittstelle wird kontinuierlich weiterentwickelt. Daher erwarten wir
> von allen Nutzern dieser Schnittstelle, dass sie das "[Tolerant Reader Pattern](https://martinfowler.com/bliki/TolerantReader.html)" nutzen, d.h.
> tolerant gegenüber kompatiblen API-Änderungen beim Lesen und Prozessieren der Daten sind:
>
> 1. unbekannte Felder dürfen keine Fehler verursachen
>
> 2. Strings mit eingeschränktem Wertebereich (Enums) müssen mit neuen, unbekannten Werten umgehen können
>
> 3. sinnvoller Umgang mit HTTP-Statuscodes, die nicht explizit dokumentiert sind  
>

<!-- https://opensource.zalando.com/restful-api-guidelines/#108 -->

## Allgemeines

Vorgänge können über unsere [GraphQL Schnittstelle](https://docs.api.europace.de/privatkredit/graphql/#allgemeines) via **HTTP POST** ausgelesen werden.  
Die URL für das Auslesen von Vorgängen ist:

    https://www.europace2.de/kreditsmart/kex/vorgaenge

Die gewünschten Properties werden als JSON im Body des POST Requests übermittelt.  
Ein erfolgreicher Aufruf resultiert in einer Response mit dem HTTP Statuscode **200 SUCCESS**.  
Die angeforderten Daten werden ebenfalls als JSON übermittelt.


## Authentifizierung

Für jeden Request ist eine Authentifizierung erforderlich. Die Authentifizierung erfolgt über den OAuth 2.0 Client-Credentials Flow.

| Request Header Name | Beschreibung           |
|---------------------|------------------------|
| Authorization       | OAuth 2.0 Bearer Token |


Das Bearer Token kann über die [Authorization-API](https://docs.api.europace.de/privatkredit/authentifizierung/) angefordert werden.
Dazu wird ein Client benötigt der vorher von einer berechtigten Person über das Partnermanagement angelegt wurde,
eine Anleitung dafür befindet sich im [Help Center](https://europace2.zendesk.com/hc/de/articles/360012514780).

Damit der Client für diese API genutzt werden kann, muss im Partnermanagement die Berechtigung **KreditSmart-Vorgänge lesen	** (Scope `privatkredit:vorgang:lesen`) aktiviert sein.  

Schlägt die Authentifizierung fehl, erhält der Aufrufer eine HTTP Response mit Statuscode **401 UNAUTHORIZED**.

Hat der Client nicht die benötigte Berechtigung um die Resource abzurufen, erhält der Aufrufer eine HTTP Response mit Statuscode **403 FORBIDDEN**.


## TraceId zur Nachverfolgbarkeit von Requests

Für jeden Request soll eine eindeutige ID generiert werden, die den Request im EUROPACE 2 System nachverfolgbar macht und so bei etwaigen Problemen oder Fehlern die systemübergreifende Analyse erleichtert.  
Die Übermittlung der X-TraceId erfolgt über einen HTTP-Header. Dieser Header ist optional,
wenn er nicht gesetzt ist, wird eine ID vom System generiert.

| Request Header Name | Beschreibung                    | Beispiel    |
|---------------------|---------------------------------|-------------|
| X-TraceId           | eindeutige Id für jeden Request | sys12345678 |

## Content-Type

Die Schnittstelle akzeptiert Daten mit Content-Type "**application/json**".  
Entsprechend muss im Request der Content-Type Header gesetzt werden. Zusätzlich das Encoding, wenn es nicht UTF-8 ist.

| Request Header Name |   Header Value   |
|---------------------|------------------|
| Content-Type        | application/json |

## Beispiel
### POST Request

    POST https://www.europace2.de/kreditsmart/kex/vorgaenge
    Authorization: Bearer xxxxxxx
    Content-Type: application/json;charset=utf-8

    {
      "query": "query getVorgang($vorgangsnummer: String!) {
        vorgang(vorgangsnummer: $vorgangsnummer) {
          vorgangsnummer
          bearbeiter {
            partnerId
          }
          kundenbetreuer {
            partnerId
          }
        }
      }",
      "variables": {
        "vorgangsnummer": "123456"
      }
    }

### POST Response

    {
      "data": {
        "vorgang": {
          "vorgangsnummer": "123456",
          "bearbeiter": {
            "partnerId": "11111"
          },
          "kundenbetreuer": {
            "partnerId": "11111"
          }
        }
      },
      "errors": []
    }


## Fehlercodes

Die Besonderheit in GraphQL ist u.a., dass die meisten Fehler nicht über HTTP-Fehlercodes wiedergegeben werden.
In vielen Fällen bekommt man einen Status 200 zurück, obwohl ein Fehler aufgetreten ist. Dafür gibt es das Attribut `errors` in der Response.
Weitere Infos gibt es [hier](https://docs.api.europace.de/privatkredit/graphql/)

### HTTP-Status Errors

| Fehlercode | Nachricht       | weitere Attribute          | Erklärung                            |
|------------|-----------------|----------------------------|--------------------------------------|
| 401        | Unauthorized    | -                          | Authentifizierung ist fehlgeschlagen |
| 410        | Vorgang deleted | "vorgangsnummer": "123456" | Der Vorgang wurde gelöscht           |

### GraphQL Fehler

| Fehlercode | Nachricht                  | Erklärung                                                                                      |
|------------|----------------------------|------------------------------------------------------------------------------------------------|
| 403        | Insufficient access rights | Es wird versucht auf einen Vorgang zuzugreifen, den die Vertriebsorganisation nicht lesen darf |


## Request Format

Die Angaben werden als JSON im Body des Requests gesendet. Für eine bessere Lesbarkeit wird das Gesamtformat in *Typen* aufgebrochen, die an anderer Stelle definiert sind, aber an verwendeter Stelle eingesetzt werden müssen.
Die Attribute innerhalb eines Blocks können in beliebiger Reihenfolge angegeben werden.  
Für einen erfolgreichen Request muss die Query in folgendem Format vorhanden sein (siehe auch den [Beispiel Request](#post-request)):

    vorgang(vorgangsnummer: <vorgangsnummer>) {
        <gewünschte Felder>
    }


## Vorgang

    {
      "vorgangsnummer": String,
      "datenkontext": "ECHTGESCHAEFT" | "TESTUMGEBUNG",
      "kundenbetreuer": Partner,
      "bearbeiter": Partner,
      "tippgeber": Partner,      
      "leadquelle": String,
      "eigeneVorgangsnummer": String,      
      "antragsteller1": Antragsteller,
      "antragsteller2": Antragsteller,
      "haushalt": Haushalt,
      "finanzbedarf": Finanzbedarf,
      "aufbewahrung": {
        "grund": "LEAD", "BERATERHAFTUNG", "GESCHAEFTSNACHWEIS",
  	    "endetAm": "yyyy-MM-dd"
      },
      "letzteAenderungAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
      "letztesEreignisAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
      "prioritaet": "HOCH", "NEUTRAL", "NIEDRIG",
      "vorgangsstatus": {
        "status": "AKTIV" | "ARCHIVIERT"
      },
      "wiedervorlage": {
        "kommentar": String,
	    "wiedervorlageAm": "yyyy-MM-dd"
      }
      "antraege": [Antrag]
    }

:heavy_exclamation_mark: "letzteAenderungAm" zeigt NUR die letzte Änderung der Vorgangs-Daten an. Für Änderungen an den Anträgen wird das Feld "letzteAenderungAm" in jedem Antrag befüllt.


### Partner

    {
      "partnerId": String
    }

Die Europace 2 PartnerID ist 5-stellig und hat das Format ABC12.

### Antragsteller

    {
      "personendaten": Personendaten,
      "wohnsituation": Wohnsituation,
      "beschaeftigung": Beschäftigung
      "herkunft": Herkunft
    }

#### Personendaten

    {
      "anrede": "FRAU" | "HERR",
      "email": String
      "familienstand": "LEDIG" | "VERHEIRATET" | "GESCHIEDEN" | "VERWITWET" | "GETRENNT_LEBEND" | "EHEAEHNLICHE_LEBENSGEMEINSCHAFT" | "EINGETRAGENE_LEBENSPARTNERSCHAFT",
      "geburtsdatum": "YYYY-MM-DD",
      "geburtsland": Country
      "geburtsort": String
      "geburtsname": String
      "nachname": String,
      "telefonGeschaeftlich": String,
      "telefonPrivat": String,
      "titel": [ "DOKTOR" | "PROFESSOR" ]
      "vorname": String,
    }

#### Wohnsituation

    {
      "anschrift": {
        "strasse": String,
        "hausnummer": String,
        "plz": String,
        "ort": String,
        "wohnhaftSeit": "YYYY-MM-DD"
      },
      "gemeinsamerHaushalt": true | false
      "wohnart": "ZUR_MIETE" | "ZUR_UNTERMIETE" | "IM_EIGENEN_HAUS" | "BEI_DEN_ELTERN"
      "anzahlPkw": Integer
      "anzahlPersonenImHaushalt": Integer
    }

Die Angabe *gemeinsamerHaushalt* ist nur beim zweiten Antragsteller ausgefüllt.

#### Beschäftigung

  	{
      "beschaeftigungsart": "ANGESTELLTER" | "ARBEITER" | "ARBEITSLOSER" | "BEAMTER" | "FREIBERUFLER" | "HAUSFRAU" | "RENTNER" | "SELBSTSTAENDIGER",
      "angestellter": Angestellter,				
      "arbeiter": Arbeiter,
      "beamter": Beamter,
      "freiberufler": Freiberufler,
      "selbststaendiger": Selbstständiger
      "rentner": Rentner
      "hausfrau": Hausfrau
      "arbeitsloser" : Arbeitsloser
    }

:heavy_exclamation_mark: Die befüllten Felder zur Beschäftigung sind abhängig von der Beschäftigungsart.  
__Beispiel:__ *beschaeftigungsart=ARBEITER*, dann wird der Knoten *arbeiter* befüllt

##### Arbeiter und Angestellter

    {
      "beschaeftigungsverhaeltnis": {
        "arbeitgeber": Arbeitgeber,
        "befristung": "BEFRISTET" | "UNBEFRISTET",
        "befristetBis": "YYYY-MM-DD",
        "inProbezeit": true | false
        "beschaeftigtSeit": "YYYY-MM-DD"
        "nettoeinkommenMonatlich": BigDecimal
      }
    }

##### Selbstständiger und Freiberufler

    {
      "firma": {
        "name": String
      }
      "selbststaendigSeit": "YYYY-MM-DD"
      "bruttoEinkommenLaufendesJahr": BigDecimal
    }

##### Beamter

    {
      "beschaeftigungsverhaeltnis": {
        "inProbezeit": true | false,
        "arbeitgeber": Arbeitgeber,
        "verbeamtetSeit": "YYYY-MM-DD"
        "nettoeinkommenMonatlich": BigDecimal
      },
    }

##### Hausfrau und Arbeitsloser

    {
      "sonstigesEinkommenMonatlich": BigDecimal
    }

##### Rentner

    {
      "rentnerSeit": "YYYY-MM-DD"
      "staatlicheRenteMonatlich": BigDecimal
    }

##### Arbeitgeber

    {
      "name": String
    }

#### Herkunft

    {
      "staatsangehoerigkeit": Country
      "steuerId": String
      "inDeutschlandSeit": "YYYY-MM-DD"
      "aufenthaltstitel": Aufenthaltstitel
      "aufenthaltBefristetBis": "YYYY-MM-DD"
      "arbeitserlaubnisVorhanden": true | false
    }

##### Aufenthaltstitel

    "VISUM" | "AUFENTHALTSERLAUBNIS" | "NIEDERLASSUNGSERLAUBNIS" | "ERLAUBNIS_ZUM_DAUERAUFENTHALT_EU"


### Haushalt

    {
      "kinder": [ Kind ],
      "kontoverbindung": {
        "iban": String,
        "bic": String,
        "kreditinstitut": String,
        "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
      }
      verbindlichkeiten : Verbindlichkeiten
    }

#### Antragstellerzugehoerigkeit

	"ANTRAGSTELLER_1" | "ANTRAGSTELLER_2" | "BEIDE"

#### Kind

    {
      "name": String,
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
    }

#### Verbindlichkeiten
    {
      "ratenkredite" : [RatenkreditVerbindlichkeit]
    }

##### RatenkreditVerbindlichkeit

    {
      "rateMonatlich": BigDecimal
      "schlussrate": BigDecimal
      "datumLetzteRate": "YYYY-MM-DD"
      "restschuld": BigDecimal
      "urspruenglicherKreditbetrag": BigDecimal
      "datumErsteZahlung": "YYYY-MM-DD"
      "abloesen": Boolean
      "iban": String
      "kreditinstitut": String
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
      "glaeubiger": String
    }

### Finanzbedarf

    {
      "fahrzeugkauf": {
        "kaufpreis": BigDecimal,
      }
      "finanzierungszweck": "UMSCHULDUNG" | "FAHRZEUGKAUF" | "MODERNISIEREN" | "FREIE_VERWENDUNG",
      "finanzierungswunsch": {
        "laufzeitInMonaten": Integer,
        "kreditbetrag": BigDecimal,
        "rateMonatlich": BigDecimal
      }
    }

Fahrzeugkauf wird nur befüllt, wenn als Finanzierungszweck "FAHRZEUGKAUF" gesetzt ist.

### Country

    Die Übermittlung erfolgt im Format [ISO-3166/ALPHA-2](https://de.wikipedia.org/wiki/ISO-3166-1-Kodierliste)

    Zusätzlich gibt es den Wert "SONSTIGE"

### VersichertesRisiko

Das versicherte Risiko kann aktuell die folgenden Werte annhemen: `ARBEITSLOSIGKEIT`, `ARBEITSUNFAEHIGKEIT`, `LEBEN`

### Antrag

    {
      "antragsnummer": String,
      "produktanbieterantragsnummer": String,
      "angenommenAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
      "letzteAenderungAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
      "letztesEreignisAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
      "antragstellerstatus": {
        "status": "BEANTRAGT" | "UNTERSCHRIEBEN" | "NICHT_ANGENOMMEN" | "WIDERRUFEN",
        "letzteAenderungAm": "yyyy-MM-dd'T'HH:mm:ss.SSS"
      },
      "produktanbieterstatus": {
        "status": "NICHT_BEARBEITET" | "UNTERSCHRIEBEN" | "ABGELEHNT" | "AUTOMATISCH_ABGELEHNT" | "ZURUECKGESTELLT",
        "letzteAenderungAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
        "kommentar": String
      },
      "provisionsforderungsstatus": {
        "status": "VOLLSTAENDIG_AUSGEZAHLT",
        "letzteAenderungAm": "yyyy-MM-dd'T'HH:mm:ss.SSS"
      }
      "produkttyp": String,
      "ratenschutz": Ratenschutz,
      "gesamtkonditionen": Gesamtkonditionen,
      "ratenkredit": Ratenkredit
      "benoetigteUnterlagen" : [BenoetigteUnterlage]
    }

#### BenoetigteUnterlage

    {
      "unterlage": String
    }

#### Ratenschutz

    {
      "versicherteRisikenAntragsteller1": [ VersichertesRisiko ]
      "versicherteRisikenAntragsteller2": [ VersichertesRisiko ]
      "praemieMonatlich": BigDecimal
    }

Der Produkttyp kann aktuell die folgenden Werte annehmen: `RATENKREDIT`, `BAUSPARKASSE_MODERNISIERUNGSKREDIT`

### Ratenkredit

    {
      "produktanbieterId": String,
      "produktbezeichnung": String,
      "produktart": String,
      "schlussrate": BigDecimal,
      "vorlaufzinsenProTag": BigDecimal
    }

Die Produktart kann aktuell die folgenden Werte annhemen: `AUTOKREDIT`, `MODERNISIERUNGSKREDIT`, `RATENKREDIT`, `BUSINESSKREDIT`

### Gesamtkonditionen

    {
      "auszahlungsbetrag": BigDecimal,
      "effektivzins": BigDecimal,
      "gesamtkreditbetrag": BigDecimal,
      "laufzeitInMonaten": Int,
      "monatlicheRate": BigDecimal,
      "nettokreditbetrag": BigDecimal,
      "sollzins": BigDecimal
    }

Prozentwerte wie der Sollzins sind 100-basiert.

## Response Format

Die erfragten Felder werden - sofern vorhanden- als JSON im Body der Response gesendet. Nicht befüllte Felder werden nicht zurückgegeben.

    {
      "data": {
        "vorgang": {
          << ANGEFRAGTE FELDER >>
        }
      },
      "errors": [
        << EVENTUELL AUFGETRETENE FEHLER >>
      ]
    }

## Tools

Es gibt verschiedene [Tools](https://docs.api.europace.de/privatkredit/graphql/) für GraphQL.
Für [Postman](https://www.getpostman.com/) stellen wir im [Schnellstarter-Projekt](https://github.com/europace/api-schnellstart/)
auch eine Collection mit einem Beispiel für die "KreditSmart Vorgänge API" zur Verfügung, wenn man keinen GraphQL-Client verwenden möchte.

## Nutzungsbedingungen
Die APIs werden unter folgenden [Nutzungsbedingungen](https://docs.api.europace.de/nutzungsbedingungen/) zur Verfügung gestellt
