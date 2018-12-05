# KEX-Vorgang-Export-API
Die Schnittstelle ermöglicht das automatisierte Auslesen von Vorgängen in KreditSmart.

# Table of Contents

* [Allgemeines](#allgemeines)
* [Authentifizierung](#authentifizierung)
* [TraceId zur Nachverfolgbarkeit von Requests](#traceid-zur-nachverfolgbarkeit-von-requests)
* [Content-Type](#content-type)
* [Beispiel](#beispiel)
   * [POST Request](#post-request)
   * [POST Response](#post-response)
* [Fehlercodes](#fehlercodes)
   * [HTTP-Status Errors](#http-status-errors)
   * [Validation Error](#validation-error)
   * [weitere Fehler](#weitere-fehler)
* [Request Format](#request-format)
* [Vorgang](#vorgang)
   * [Partner](#partner)
   * [Antragsteller](#antragsteller)
      * [Personendaten](#personendaten)
      * [Wohnsituation](#wohnsituation)
      * [Beschäftigung](#beschäftigung)
         * [Arbeiter und Angestellter](#arbeiter-und-angestellter)
         * [Selbstständiger und Freiberufler](#selbstständiger-und-freiberufler)
         * [Beamter](#beamter)
         * [Arbeitgeber](#arbeitgeber)
      * [Haushalt](#haushalt)
         * [Antragstellerzuordnung](#antragstellerzuordnung)
         * [Kind](#kind)
      * [Finanzbedarf](#finanzbedarf)
      * [Antrag](#antrag)
* [Response Format](#response-format)

## Allgemeines

Vorgänge können über unsere GraphQL Schnittstelle via **HTTP POST** ausgelesen werden werden.  
Die URL für das Auslesen von Vorgängen ist:

[//]: # (TODO hier die richtige URL https://www.europace2.de/kreditsmart/kex/vorgang)

    http://kex-vorgang-export.pku.rz-europace.local/vorgaenge
    
Die gewünschten Properties werden als JSON im Body des POST Requests übermittelt.  
Ein erfolgreicher Aufruf resultiert in einer Response mit dem HTTP Statuscode **200 SUCCESS**.  
Die angeforderten Daten werden ebenfalls als JSON übermittelt.


## Authentifizierung

Für jeden Request ist eine Authentifizierung erforderlich. Die Authentifizierung erfolgt über einen HTTP Header.

| Request Header Name | Beschreibung                      |
|---------------------|-----------------------------------|
| X-Authentication    | API JWT der Vertriebsorganisation |


Das API JWT (JSON Web Token) erhalten Sie von Ihrem Ansprechpartner im KreditSmart-Team. Schlägt die Authentifizierung fehl, erhält der Aufrufer eine HTTP Response mit Statuscode **401 UNAUTHORIZED**.

## TraceId zur Nachverfolgbarkeit von Requests

Für jeden Request soll eine eindeutige ID generiert werden, die den Request im EUROPACE 2 System nachverfolgbar macht und so bei etwaigen Problemen oder Fehlern die systemübergreifende Analyse erleichtert.  
Die Übermittlung der X-TraceId erfolgt über einen HTTP-Header. Dieser Header ist optional,
wenn er nicht gesetzt ist, wird eine ID vom System generiert.

| Request Header Name | Beschreibung                    | Beispiel    |
|---------------------|---------------------------------|-------------|
| X-TraceId           | eindeutige Id für jeden Request | sys12345678 |

[//]: # (TODO RM brauchen wir das?) 
## Content-Type

Die Schnittstelle akzeptiert Daten mit Content-Type "**application/json**".  
Entsprechend muss im Request der Content-Type Header gesetzt werden. Zusätzlich das Encoding, wenn es nicht UTF-8 ist.

| Request Header Name |   Header Value   |
|---------------------|------------------|
| Content-Type        | application/json |

## Beispiel 
### POST Request
[//]: # (TODO hier die richtige URL) 

    POST http://kex-vorgang-export.pku.rz-europace.local/vorgaenge
    X-Authentication: xxxxxxx
    Content-Type: application/json;charset=utf-8
    
    {
      "query": "query getVorgang($vorgangsnummer: String!) { vorgang(vorgangsnummer: $vorgangsnummer) { vorgangsnummer bearbeiter{ partnerId } kundenbetreuer{ partnerId } } }",
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
In vielen Fällen bekommt man einen Status 200 zurück, obwohl ein Fehler aufgetreten ist. Dafür gibt es den Parameter `errors` in der Response.

### HTTP-Status Errors

| Fehlercode | Nachricht       | weitere Attribute          | Erklärung                            |
|------------|-----------------|----------------------------|--------------------------------------|
| 401        | Unauthorized    | -                          | Authentifizierung ist fehlgeschlagen |
| 410        | Vorgang deleted | "vorgangsnummer": "123456" | Der Vorgang wurde gelöscht           |

### Validation Error
Wenn die GraphQL-Query nicht verarbeitet werden kann, wird eine Response mit errorType `ValidationError` zurückgegeben.   
Im Beispiel wurde die o.g. query ausgeführt, und das Feld vorgangsnummer falsch geschrieben (vorgangsnummerr).

    {
      "errors": [
        {
          "message": "Validation error of type FieldUndefined: Field 'vorgangsnummerr' in type 'Vorgang' is undefined @ 'vorgang/vorgangsnummerr'",
          "locations": [
            {
              "line": 1,
              "column": 89
            }
          ],
          "description": "Field 'vorgangsnummerr' in type 'Vorgang' is undefined",
          "validationErrorType": "FieldUndefined",
          "queryPath": [
            "vorgang",
            "vorgangsnummerr"
          ],
          "errorType": "ValidationError"
        }
      ]
    }


### weitere Fehler
Wenn der Request nicht erfolgreich verarbeitet werden konnte, liefert die Schnittstelle eine 200, aber in dem response parameter "errors" ist ein Fehler zu finden

    {
      "data": {},
      "errors": [
        {
          "message": MESSAGE,
          "status": STATUS_CODE
        }
      ]
    }

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
      "kundenbetreuer": Partner,
      "bearbeiter": Partner,
      "leadquelle": String,
      "eigeneVorgangsnummer": String,
      "vorgangsnummer": String,
      "antragsteller1": Antragsteller,
      "antragsteller2": Antragsteller,
      "haushalt": Haushalt,
      "finanzbedarf": Finanzbedarf,
      "letzteAenderungAm": "yyyy-MM-dd'T'HH:mm:ss.SSS"
      "antraege": [Antrag]
    }


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
    }


#### Personendaten

    {
      "anrede": "FRAU" | "HERR",
      "email": String
      "familienstand": "LEDIG" | "VERHEIRATET" | "GESCHIEDEN" | "VERWITWET" | "GETRENNT_LEBEND" | "EHEAEHNLICHE_LEBENSGEMEINSCHAFT" | "EINGETRAGENE_LEBENSPARTNERSCHAFT",
      "geburtsdatum": "YYYY-MM-DD",
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
      },
      "gemeinsamerHaushalt": true | false
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
    }

Die befüllten Felder zur Beschäftigung sind abhängig von der Beschäftigungsart.  
__Beispiel:__ *beschaeftigungsart=ARBEITER*, dann wird der Knoten *arbeiter* befüllt

##### Arbeiter und Angestellter

    {
      "beschaeftigungsverhaeltnis": {
        "arbeitgeber": Arbeitgeber,
        "befristung": "BEFRISTET" | "UNBEFRISTET",
        "befristetBis": "YYYY-MM-DD",
        "inProbezeit": true | false
      }
    }

##### Selbstständiger und Freiberufler

    {
      "firma": {
        "name": String
      }
    }

##### Beamter

    {
      "beschaeftigungsverhaeltnis": {
        "inProbezeit": true | false,
        "arbeitgeber": 	Arbeitgeber,
      },
    }

##### Arbeitgeber
  
    {
      "name": String
    }


#### Haushalt

    {
      "kinder": [ kind ],
      "kontoverbindung": {
        "iban": String,
        "bic": String,
        "kreditinstitut": String,
        "gehoertZuAntragsteller": Antragstellerzuordnung
      }
    }

##### Antragstellerzuordnung

	"ANTRAGSTELLER_1" | "ANTRAGSTELLER_2" | "BEIDE"

##### Kind

    {
      "name": String,
      "gehoertZuAntragsteller": Antragstellerzuordnung
    }	
				

#### Finanzbedarf

    {
      "fahrzeugkauf": {
        "kaufpreis": Decimal,
      }
      "finanzierungszweck": "UMSCHULDUNG" | "FAHRZEUGKAUF" | "MODERNISIEREN" | "FREIE_VERWENDUNG",
      "finanzierungswunsch": 			{
        "laufzeitInMonaten": Integer,
        "kreditbetrag": Decimal,
        "rateMonatlich": Decimal
      }
    }

Fahrzeugkauf wird nur befüllt, wenn als Finanzierungszweck "FAHRZEUGKAUF" gesetzt ist.

#### Antrag

    {
      "teilantragsnummer": String
      "letzteAenderungAm": "yyyy-MM-dd'T'HH:mm:ss.SSS"
      "antragstellerstatus": {
        "status": "BEANTRAGT" | "UNTERSCHRIEBEN" | "NICHT_ANGENOMMEN" | "WIDERRUFEN" 
      "produktanbieterstatus": {
        "status": "NICHT_BEARBEITET" | "UNTERSCHRIEBEN" | "ABGELEHNT" | "AUTOMATISCH_ABGELEHNT" | "ZURUECKGESTELLT" 
      }
    }
    

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
