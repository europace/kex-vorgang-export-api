# KEX-Vorgang-Export-API
Die Schnittstelle ermöglicht das automatisierte auslesen von Vorgängen in Kreditsmart.


## Auslesen eines Vorgangs

Vorgänge können über unsere GraphQL Schnittstelle via **HTTP POST** ausgelesen werden werden.

Die URL für das Auslesen von Vorgängen ist:

[//]: # (TODO hier die richtige URL https://www.europace2.de/kreditsmart/kex/vorgang)
    http://kex-vorgang-export.pku.rz-europace.local/vorgaenge
    
Die gewünschten Properties werden als JSON im Body des POST Requests übermittelt.

Ein erfolgreicher Aufruf resultiert in einer Response mit dem HTTP Statuscode **200 SUCCESS**.

Die angeforderten Daten werden ebenfalls als JSON übermittelt.

## Authentifizierung

Für jeden Request ist eine Authentifizierung erforderlich.

Die Authentifizierung erfolgt über einen HTTP Header.

<table>
<tr>
<th>
Request Header Name</th><th>	Beschreibung</th>
</tr>
<tr>
<td>X-Authentication</td><td>	API JWT der Vertriebsorganisation</td>
</tr>
</table>


Das API JWT (JSON Web Token) erhalten Sie von Ihrem Ansprechpartner im **Kredit**Smart-Team. 

Schlägt die Authentifizierung fehl, erhält der Aufrufer eine HTTP Response mit Statuscode **401 UNAUTHORIZED**.

## TraceId zur Nachverfolgbarkeit von Requests

Für jeden Request soll eine eindeutige ID generiert werden, die den Request im EUROPACE 2 System nachverfolgbar macht und so bei etwaigen Problemen oder Fehlern die systemübergreifende Analyse erleichtert.

Die Übermittlung der X-TraceId erfolgt über einen HTTP Header. Dieser Header ist optional,
wenn er nicht gesetzt ist, wir eine ID vom System generiert.

<table>
<tr>
<th>Request Header Name</th><th>Beschreibung</th><th>Beispiel</th>
<tr>
<td>X-TraceId</td>
<td>eindeutige Id für jeden Request</td>
<td>sys12345678</td>
</tr>
</table>


[//]: # (TODO RM brauchen wir das?) 
## Content-Type

Die Schnittstelle akzeptiert Daten mit Content-Type "**application/json**".

Entsprechend muss im Request der Content-Type Header gesetzt werden. Zusätzlich das Encoding, wenn es nicht UTF-8 ist.

<table>
<tr>
<th>
Request Header Name</th><th>	Header Value</th>
</tr>
<tr>
<td>Content-Type</td><td>	application/json</td>
</tr>
</table>

### POST Request Beispiel:
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
    
### POST Response Beispiel:

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

Die Besonderheit in GraphQL ist u.a., dass nicht alle Fehler direkt als Fehlercodes wiedergegeben werden.
In vielen Fällen bekommt man einen Status 200 zurück, obwohl ein Fehler passiert ist. Dafür gibt es den Parameter `errors` in der response.

### HTTP-Status Errors
 
<table>
<tr><th>Fehlercode</th><th>Nachricht</th><th>	Erklärung</th></tr>
<tr><td>401</td><td>Unauthorized</td><td>Authentifizierung ist fehlgeschlagen</td></tr>
</table>

### Validation Error
Wenn die GraphQL Query nicht verarbeitet werden kann, werden sie eine response mit dem errorType ~ValidationError` erhalten.  

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
Achtung: In diesem Fall wird kein Vorgang in **Kredit**Smart importiert und angelegt.

<table>
<tr><th>Fehlercode</th><th>Nachricht</th><th>	Erklärung</th></tr>
<tr><td>403</td><td>Insufficient access rights</td><td>Es wird versucht auf einen Vorgang zuzugreifen, den die Vertriebsorganisation nicht lesen darf</td></tr>
</table>


## Request Format

Die Angaben werden als JSON im Body des Requests gesendet.

Für eine bessere Lesbarkeit wird das Gesamtformat in *Typen* aufgebrochen, die an anderer Stelle definiert sind, aber an verwendeter Stelle eingesetzt werden müssen.
Die Attribute innerhalb eines Blocks können in beliebiger Reihenfolge angegeben werden.

Für einen erfolgreichen Request gibt es derzeit nur ein definiertes Pflichtfeld (siehe „Vorgang“).

Alle übermittelten Daten werden in **Kredit**Smart übernommen, mit Ausnahme von:

* Angaben, die aufgrund eines abweichenden Formats nicht verstanden werden (z. B. "1" statt "true", "01.01.2016" statt "2016-01-01"), und 
* Angaben, die aufgrund der Datenkonstellationen überflüssig bzw. unstimmig sind (z. B. Angabe beim 1. Antragsteller zu gemeinsamerHaushalt).


An verschiedenen Stellen im Request ist die Angabe eines Landes oder der Staatsangehörigkeit notwendig:
Die Übermittlung erfolgt im Format [ISO-3166/ALPHA-2](https://de.wikipedia.org/wiki/ISO-3166-1-Kodierliste)

### Vorgang

    {
      "kundenbetreuer": Partner,
      "bearbeiter": Partner,
      "leadquelle": String,
      "eigeneVorgangsnummer": String,
      "antragsteller1": Antragsteller,
      "antragsteller2": Antragsteller,
      "haushalt": Haushalt,
      "finanzbedarf": Finanzbedarf,
      "letzteAenderungAm": "YYYY-MM-DD hh-mm-ss.sss"
    }
[//]: # (TODO RM Datumsformat?)


### Partner

	{
		"partnerId": String
	}

Die Europace 2 PartnerID ist 5-stellig und hat das Format ABC12. 

#### Antragsteller

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
Beispiel: *beschaeftigungsart=ARBEITER*, dann wird der Knoten *arbeiter* befüllt

#### Arbeiter und Angestellter

	{
		"beschaeftigungsverhaeltnis": {
			"arbeitgeber": Arbeitgeber,
			"befristung": "BEFRISTET" | "UNBEFRISTET",
			"befristetBis": "YYYY-MM-DD",
			"inProbezeit": true | false
		}
	}

#### Selbstständiger und Freiberufler

	{
		"firma": {
			"name": String
		}
	}

#### Beamter

	{
		"beschaeftigungsverhaeltnis": {
			"inProbezeit": true | false,
			"arbeitgeber": 	Arbeitgeber,
		},
	}

#### Arbeitgeber

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

#### Antragstellerzuordnung

	"ANTRAGSTELLER_1" | "ANTRAGSTELLER_2" | "BEIDE"

#### Kind

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

## Response Format

Die erfragten Felder werden - sofern vorhanden- als JSON im Body der Response gesendet.  
Nicht befüllte felder werden nicht zurückgegeben

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
