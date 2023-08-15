# KEX-Vorgang-Export-API

> ⚠️ You'll find German domain-specific terms in the documentation, for translations and further explanations please refer to our [glossary](https://docs.api.europace.de/common/glossary/)

## General

An API to read data of KreditSmart-Vorgänge. The URL for this service is:

    https://www.europace2.de/kreditsmart/kex/vorgaenge

All APIs documented here are [GraphQL-APIs](https://docs.api.europace.de/privatkredit/graphql/).

> ⚠️ This API is continuously developed. Therefore we expect
> all users to align with the "[Tolerant Reader Pattern](https://martinfowler.com/bliki/TolerantReader.html)", which requires clients to be
> tolerant towards compatible API changes when reading and processing the data. This means:
>
> 1. unknown properties must not result in errors
>
> 2. Strings with a restricted set of values (Enums) must support new unknown values
>
> 3. sensible usage of HTTP status codes, even if they are not explicitly documented
>

<!-- https://opensource.zalando.com/restful-api-guidelines/#108 -->

### Authentication

These APIs are secured by the OAuth 2.0 client credentials flow using the [Authorization-API](https://docs.api.europace.de/privatkredit/authentifizierung/).
To use these APIs your OAuth2-Client needs the following scopes:

| Scope                      | Label in Partnermanagement | Description                         |
|----------------------------|----------------------------|-------------------------------------|
| privatkredit:vorgang:lesen | KreditSmart-Vorgänge lesen | Scope for reading data of a Vorgang |

### GraphQL-Requests

These APIs accept data with the content-type **application/json** with UTF-8 encoding.
The fields inside a block can be sent in any order.

The APIs support all common GraphQL formats. More information can be found at [https://graphql.org/learn/queries/](https://graphql.org/learn/queries/).

The body of a GraphQL request contains the field `query`, which includes the GraphQL query as a String. Parameters can be set directly in the query or defined as variables. The variables can be sent
in the `variables` field of the body as a key-value map.
All our examples use variables.

    {
      "query": "...",
      "variables": { ... }
    }

### Error Codes

One of the special features in GraphQL is that most errors are not reflected via HTTP error codes.
In many cases you receive a status code 200, even though an error has occurred. These GraphQL errors can be found in the `errors` field of the response body.
More information about error codes can be found [here](https://docs.api.europace.de/privatkredit/graphql/#error-handling).

### HTTP-Status Errors

| Error Code | Message               | Description                     |
|------------|-----------------------|---------------------------------|
| 401        | Unauthorized          | Authentication failed           |
| 403        | Forbidden             | The API client misses a scope   |
| 415        | Unsupported MediaType | The wrong content type was used |

#### GraphQL Errors

| Error Code | Message     | Description                                                                                    |
|------------|-------------|------------------------------------------------------------------------------------------------|
| 400        | Bad Request | Request format is invalid (mandatory fields are missing, wrong parameter names or values, ...) |
| 403        | Forbidden   | The authenticated user does not have sufficient rights to read the Vorgang                     |
| 410        | Gone        | The Vorgang was deleted in the meantime                                                        |

## Read Vorgang

**vorgang** ( vorgangsnummer: String! ) -> [Vorgang](#vorgang)!

> This query is for reading data of a [Vorgang](#vorgang).

### Example

#### POST Request

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

#### POST Response

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

## Read Anträge of a Vorgang

* This query is implemented as a subquery of the Vorgang datatype.
* If you do not specify an antragsnummer all Anträge of a Vorgang will be exported.

**antraege** ( antragsnummer: String ) -> [[Antrag](#antrag)]

> This query is for reading [Antraege](#antrag) of a [Vorgang](#vorgang).

### Example

#### POST Request

    POST https://www.europace2.de/kreditsmart/kex/vorgaenge
    Authorization: Bearer xxxxxxx
    Content-Type: application/json;charset=utf-8

    {
      "query": "query getVorgang($vorgangsnummer: String!, $antragsnummer: String) {
        vorgang(vorgangsnummer: $vorgangsnummer) {
          vorgangsnummer
          antraege(antragsnummer: $antragsnummer) {
            antragsnummer
          }
        }
      }",
      "variables": {
        "vorgangsnummer": "123456",
        "antragsnummer": "123456/1/1"
      }
    }

#### POST Response

    {
      "data": {
        "vorgang": {
          "vorgangsnummer": "123456",
          "antraege": [
            {
              "antragsnummer": "123456/1/1"
            }
          ]
        }
      }
    }
      },
      "errors": []
    }

## Response Types

### Vorgang

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
      "erstelltAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
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

`letzteAenderungAm` is only for the last change of a Vorgang. `letzteAenderungAm` for Anträge will be populated in each Antrag.

#### Partner

    {
      "partnerId": String
    }

The Europace 2 Partner-ID has 5-characters and has the format ABC12.

#### Antragsteller

    {
      "personendaten": Personendaten,
      "wohnsituation": Wohnsituation,
      "beschaeftigung": Beschäftigung
      "herkunft": Herkunft
    }

##### Personendaten

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

##### Wohnsituation

    {
      "anschrift": Wohnanschrift
      "voranschrift": Wohnanschrift
      "gemeinsamerHaushalt": true | false
      "wohnart": "ZUR_MIETE" | "ZUR_UNTERMIETE" | "IM_EIGENEN_HAUS" | "BEI_DEN_ELTERN"
      "anzahlPkw": Integer
      "anzahlPersonenImHaushalt": Integer
    }

The value of `gemeinsamerHaushalt` is only relevant for the second Antragsteller.

###### Wohnanschrift

    {
        "strasse": String,
        "hausnummer": String,
        "plz": String,
        "ort": String,
        "wohnhaftSeit": "YYYY-MM-DD"
    }

##### Beschäftigung

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

The `beschaeftigungsart` determines which data is available. For example the `beschaeftigungsart=ARBEITER` means that data for the field `arbeiter` is available. All other fields will be empty.

###### Arbeiter and Angestellter

    {
      "beschaeftigungsverhaeltnis": {
        "berufsbezeichnung": String,
        "arbeitgeber": Arbeitgeber,
        "befristung": "BEFRISTET" | "UNBEFRISTET",
        "befristetBis": "YYYY-MM-DD",
        "inProbezeit": true | false
        "beschaeftigtSeit": "YYYY-MM-DD"
        "nettoeinkommenMonatlich": BigDecimal
      },
      "vorherigesBeschaeftigungsverhaeltnis": {
        "arbeitgeber": Arbeitgeber,
        "beschaeftigtSeit": "YYYY-MM-DD",
        "beschaeftigtBis": "YYYY-MM-DD"
      }
    }

###### Selbstständiger and Freiberufler

    {
      "berufsbezeichnung": String,
      "firma": Firma,
      "selbststaendigSeit": "YYYY-MM-DD",
      "selbststaendigSeit": "YYYY-MM-DD",
      "nettoeinkommenJaehrlich": BigDecimal,
      "bruttoEinkommenLaufendesJahr": BigDecimal,
      "einkommenssteuerLaufendesJahr": BigDecimal,
      "abschreibungenLaufendesJahr": BigDecimal,
      "bruttoEinkommenLetztesJahr": BigDecimal,
      "einkommenssteuerLetztesJahr": BigDecimal,
      "abschreibungenLetztesJahr": BigDecimal,
      "bruttoEinkommenVor2Jahren": BigDecimal,
      "einkommenssteuerVor2Jahren": BigDecimal,
      "abschreibungenVor2Jahren": BigDecimal,
      "bruttoEinkommenVor3Jahren": BigDecimal,
      "einkommenssteuerVor3Jahren": BigDecimal,
      "abschreibungenVor3Jahren": BigDecimal
    }

###### Beamter

    {
      "beschaeftigungsverhaeltnis": {
        "berufsbezeichnung": String,
        "inProbezeit": true | false,
        "arbeitgeber": Arbeitgeber,
        "verbeamtetSeit": "YYYY-MM-DD"
        "nettoeinkommenMonatlich": BigDecimal
      },
      "vorherigesBeschaeftigungsverhaeltnis": {
        "arbeitgeber": Arbeitgeber,
        "beschaeftigtSeit": "YYYY-MM-DD",
        "beschaeftigtBis": "YYYY-MM-DD"
      }
    }

###### Hausfrau and Arbeitsloser

    {
      "sonstigesEinkommenMonatlich": BigDecimal
    }

###### Rentner

    {
      "rentnerSeit": "YYYY-MM-DD",
      "staatlicheRenteMonatlich": BigDecimal,
      "rentenversicherung": {
        "anschrift": Anschrift,
        "name": String
      }
    }

###### Arbeitgeber and Firma

    {
      "anschrift": Anschrift,
      "brance": Branche,
      "name": String
    }

###### Branche

    "LANDWIRTSCHAFT_FORSTWIRTSCHAFT_FISCHEREI" | "ENERGIE_WASSERVERSORGUNG_BERGBAU" | "VERARBEITENDES_GEWERBE" | "BAUGEWERBE" | "HANDEL" | "VERKEHR_LOGISTIK" | "INFORMATION_KOMMUNIKATION" | "GEMEINNUETZIGE_ORGANISATION" | "KREDITINSTITUTE_VERSICHERUNGEN" | "PRIVATE_HAUSHALTE" | "DIENSTLEISTUNGEN" | "OEFFENTLICHER_DIENST" | "GEBIETSKOERPERSCHAFTEN" | "HOTEL_GASTRONOMIE" | "ERZIEHUNG_UNTERRICHT" | "KULTUR_SPORT_UNTERHALTUNG" | "GESUNDHEIT_SOZIALWESEN"

###### Anschrift

    {
      "strasse": String,
      "hausnummer": String,
      "plz": String,
      "ort": String,
      "land": Country
    }

##### Herkunft

    {
      "staatsangehoerigkeit": Country
      "steuerId": String
      "inDeutschlandSeit": "YYYY-MM-DD"
      "aufenthaltstitel": Aufenthaltstitel
      "aufenthaltBefristetBis": "YYYY-MM-DD"
      "arbeitserlaubnisVorhanden": true | false
      "arbeitserlaubnisBefristetBis": "YYYY-MM-DD"
    }

###### Aufenthaltstitel

    "VISUM" | "AUFENTHALTSERLAUBNIS" | "NIEDERLASSUNGSERLAUBNIS" | "ERLAUBNIS_ZUM_DAUERAUFENTHALT_EU"

#### Haushalt

    {
      "ausgaben": Ausgaben,
      "einnahmen": Einnahmen,
      "immobilien": [ Immobilie ]
      "kinder": [ Kind ],
      "kontoverbindung": {
        "iban": String,
        "bic": String,
        "kreditinstitut": String,
        "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
      },
      "verbindlichkeiten" : Verbindlichkeiten,
      "vermoegen": Vermoegen,
    }

##### Antragstellerzugehoerigkeit

	"ANTRAGSTELLER_1" | "ANTRAGSTELLER_2" | "BEIDE"

##### Kind

    {
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit,
      "kindergeldFuer": "ERSTES_ODER_ZWEITES_KIND" | "DRITTES_KIND" | "AB_VIERTEM_KIND",
      "name": String,
      "unterhaltseinnahmenMonatlich": BigDecimal
    }

##### Verbindlichkeiten

    {
      "ratenkredite" : [RatenkreditVerbindlichkeit]
      "sonstigeVerbindlichkeiten" : [SonstigeVerbindlichkeit]
      "kreditkarten" : [KreditkartenVerbindlichkeit]
      "dispositionskredite" : [DispostionskreditVerbindlichkeit]
      "leasings" : [LeasingVerbindlichkeit]
    }

###### RatenkreditVerbindlichkeit

    {
      "rateMonatlich": BigDecimal
      "schlussrate": BigDecimal
      "datumLetzteRate": "YYYY-MM-DD"
      "restschuld": BigDecimal
      "urspruenglicherKreditbetrag": BigDecimal
      "datumErsteZahlung": "YYYY-MM-DD"
      "abloesen": Boolean
      "iban": String
      "bic": String
      "kreditinstitut": String
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
      "glaeubiger": String
    }

###### SonstigeVerbindlichkeit

    {
      "rateMonatlich": BigDecimal
      "schlussrate": BigDecimal
      "datumLetzteRate": "YYYY-MM-DD"
      "restschuld": BigDecimal
      "urspruenglicherKreditbetrag": BigDecimal
      "datumErsteZahlung": "YYYY-MM-DD"
      "abloesen": Boolean
      "iban": String
      "bic": String
      "kreditinstitut": String
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
      "glaeubiger": String
    }

###### KreditkartenVerbindlichkeit

    {
      "rateMonatlich": BigDecimal
      "zinssatz": BigDecimal
      "beanspruchterBetrag": BigDecimal
      "verfuegungsrahmen": BigDecimal
      "abloesen": Boolean
      "iban": String
      "bic": String
      "kreditinstitut": String
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
      "glaeubiger": String
    }

###### DispostionskreditVerbindlichkeit

    {
      "zinssatz": BigDecimal
      "beanspruchterBetrag": BigDecimal
      "verfuegungsrahmen": BigDecimal
      "abloesen": Boolean
      "iban": String
      "bic": String
      "kreditinstitut": String
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
      "glaeubiger": String
    }

###### LeasingVerbindlichkeit

    {
      "rateMonatlich": BigDecimal,
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit,
      "glaeubiger": String,
      "schlussrate": BigDecimal,
      "datumLetzteRate": "YYYY-MM-DD"
    }

##### Ausgaben

    {
      "mietausgaben": [ Mietausgabe ],
      "privateKrankenversicherungen": [ PrivateKrankenversicherung ],
      "sonstigeAusgaben": [ SonstigeAusgabe ],
      "unterhaltsverpflichtungen": [ Unterhaltsverpflichtung ]
    }

###### Mietausgabe

    {
      "betragMonatlich": BigDecimal,
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
    }   

###### PrivateKrankenversicherung

    {
      "betragMonatlich": BigDecimal,
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
    } 

###### SonstigeAusgabe

    {
      "betragMonatlich": BigDecimal,
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
    } 

###### Unterhaltsverpflichtung

    {
      "betragMonatlich": BigDecimal,
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
    } 

##### Einnahmen

    {
      "ehegattenunterhalt": [ Ehegattenunterhalt ],
      "einkuenfteAusNebentaetigkeit": [ EinkunftAusNebentaetigkeit ],
      "sonstigeEinnahmen": [ SonstigeEinnahme ]
      "unbefristeteZusatzrenten": [ UnbefristeteZusatzrente ],
    }

###### EinkunftAusNebentaetigkeit

    {
      "beginnDerTaetigkeit": "YYYY-MM-DD",
      "betragMonatlich": BigDecimal,
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
    }

###### Ehegattenunterhalt

    {
      "betragMonatlich": BigDecimal,
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
    }

###### UnbefristeteZusatzrente

    {
      "betragMonatlich": BigDecimal,
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
    }

###### SonstigeEinnahme

    {
      "betragMonatlich": BigDecimal,
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit
    }

##### Vermoegen

    {
      "bausparvertraege": [ Bausparvertrag ],
      "lebensversicherungen": [ Lebensversicherung ]
    }

###### Bausparvertrag

    {
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit,
      "sparbeitragMonatlich": BigDecimal
    }

###### Lebensversicherung

    {
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit,
      "praemieMonatlich": BigDecimal
    }

##### Immobilie

    {
      "bezeichnung": String,
      "darlehen": [ Darlehen ],
      "gehoertZuAntragsteller": Antragstellerzugehoerigkeit,
      "immobilienart": "EIGENTUMSWOHNUNG" | "EINFAMILIENHAUS" | "MEHRFAMILIENHAUS" | "BUEROGEBAEUDE",
      "mieteinnahmenKaltMonatlich": BigDecimal,
      "mieteinnahmenWarmMonatlich": BigDecimal,
      "nebenkostenMonatlich": BigDecimal,
      "nutzungsart": "EIGENGENUTZT" | "VERMIETET" | "EIGENGENUTZT_UND_VERMIETET",
      "vermieteteWohnflaeche": Integer,
      "wert": BigDecimal,
      "wohnflaeche": Integer
    }

###### Darlehen

    {
      "rateMonatlich": BigDecimal,
      "restschuld": BigDecimal,
      "zinsbindungBis": "YYYY-MM-DD"
    }

#### Finanzbedarf

    {
      "fahrzeugkauf": Fahrzeugkauf,
      "finanzierungswunsch": Finanzierungswunsch,
      "finanzierungszweck": "UMSCHULDUNG" | "FAHRZEUGKAUF" | "MODERNISIEREN" | "FREIE_VERWENDUNG",
      "ratenschutzAntragsteller1": FinanzbedarfRatenschutz,
      "ratenschutzAntragsteller2": FinanzbedarfRatenschutz
    }

The field `fahrzeugkauf` is only available if the Finanzierungszweck is `FAHRZEUGKAUF`.

##### Fahrzeugkauf

    {
      "anbieter": "HAENDLER" | "PRIVAT",
      "beglicheneKosten": BigDecimal,
      "erstzulassungsdatum": "YYYY-MM-DD",
      "kaufpreis": BigDecimal,
      "kw": Integer, 
      "laufleistung": Integer,
      "marke": String,
      "modell": String,
      "ps": Integer
    }

##### Finanzierungswunsch

    {
      "laufzeitInMonaten": Integer,
      "kreditbetrag": BigDecimal,
      "rateMonatlich": BigDecimal,
      "ratenzahlungstermin": "MONATSMITTE" | "MONATSENDE",
      "provisionswunschInProzent": BigDecimal
    }

##### FinanzbedarfRatenschutz

    {
      arbeitslosigkeitAbsicherung: RatenschutzAbsicherung,
      arbeitsunfaehigkeitAbsicherung: RatenschutzAbsicherung,
      todesfallAbsicherung: RatenschutzAbsicherung
    }

###### RatenschutzAbsicherung

    {
      gewuenscht: Boolean,
      kommentar: String,
      vorhanden: Boolean,
      wichtig: Boolean
    }

#### Country

This Type uses the format [ISO-3166/ALPHA-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements)

In addition there is the value "SONSTIGE" ("other")

    "AD" | "AE" | "AF" | "AG" | "AL" | "AM" | "AO" | "AR" | "AT" | "AU" | "AZ" | "BA" | "BB" | "BD" | "BE" | "BF" | "BG" | "BH" | "BI" | "BJ" | "BN" | "BO" | "BR" | "BS" | "BT" | "BW" | "BY" | "BZ" | "CA" | "CD" | "CF" | "CG" | "CH" | "CI" | "CK" | "CL" | "CM" | "CN" | "CO" | "CR" | "XK" | "CU" | "CV" | "CY" | "CZ" | "DE" | "DJ" | "DK" | "DM" | "DO" | "DZ" | "EC" | "EE" | "EG" | "ER" | "ES" | "ET" | "FI" | "FJ" | "FM" | "FR" | "GA" | "GB" | "GD" | "GE" | "GH" | "GM" | "GN" | "GQ" | "GR" | "GT" | "GW" | "GY" | "HN" | "HR" | "HT" | "HU" | "ID" | "IE" | "IL" | "IN" | "IQ" | "IR" | "IS" | "IT" | "JM" | "JO" | "JP" | "KE" | "KG" | "KH" | "KI" | "KM" | "KN" | "KP" | "KR" | "KW" | "KZ" | "LA" | "LB" | "LC" | "LI" | "LK" | "LR" | "LS" | "LT" | "LU" | "LV" | "LY" | "MA" | "MC" | "MD" | "ME" | "MG" | "MH" | "MK" | "ML" | "MM" | "MN" | "MR" | "MT" | "MU" | "MV" | "MW" | "MX" | "MY" | "MZ" | "NA" | "NE" | "NG" | "NI" | "NL" | "NO" | "NP" | "NR" | "NU" | "NZ" | "OM" | "PA" | "PE" | "PG" | "PH" | "PK" | "PL" | "PS" | "PT" | "PW" | "PY" | "QA" | "RO" | "RS" | "RU" | "RW" | "SA" | "SB" | "SC" | "SD" | "SE" | "SG" | "SI" | "SK" | "SL" | "SM" | "SN" | "SO" | "SR" | "SS" | "ST" | "SV" | "SY" | "SZ" | "TD" | "TG" | "TH" | "TJ" | "TL" | "TM" | "TN" | "TO" | "TR" | "TT" | "TV" | "TZ" | "UA" | "UG" | "US" | "UY" | "UZ" | "VA" | "VC" | "VE" | "VN" | "VU" | "WS" | "YE" | "ZA" | "ZM" | "ZW" | "SONSTIGE"

#### Antrag

    {
      "antragsnummer": String,
      "produktanbieterantragsnummer": String,
      "angenommenAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
      "letzteAenderungAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
      "letztesEreignisAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
      "ausgehaendigtAm": "yyyy-MM-dd'T'HH:mm:ss.SSS",
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
      },
      "produkttyp": String,
      "ratenschutz": Ratenschutz,
      "gesamtkonditionen": Gesamtkonditionen,
      "ratenkredit": Ratenkredit,
      "benoetigteUnterlagen" : [BenoetigteUnterlage],
      "dokumente" : [Dokument],
      "identifikationAntragsteller1" : Identifikation,
      "identifikationAntragsteller2" : Identifikation,
      "machbarkeit": {
        "status": "MACHBAR" | "MACHBAR_UNTER_VORBEHALT" | "NICHT_MACHBAR"
      }
    }

The field `ausgehaendigtAm` shows only the timestamp of the most recent issuing of the Antrag.
The field `produkttyp` can currently be one of the following values: `RATENKREDIT`, `BAUSPARKASSE_MODERNISIERUNGSKREDIT`

##### BenoetigteUnterlage

    {
      "unterlage": String
    }

##### Ratenschutz

    {
      "versicherteRisikenAntragsteller1": [ VersichertesRisiko ]
      "versicherteRisikenAntragsteller2": [ VersichertesRisiko ]
      "praemieMonatlich": BigDecimal
    }

The type `VersichertesRisiko` can currently be one of the following values: `ARBEITSLOSIGKEIT`, `ARBEITSUNFAEHIGKEIT`, `LEBEN`

#### Ratenkredit

    {
      "produktanbieterId": String,
      "produktbezeichnung": String,
      "produktart": String,
      "schlussrate": BigDecimal,
      "vorlaufzinsenProTag": BigDecimal
    }

The field `produktart` can currently be one of the following values: `AUTOKREDIT`, `MODERNISIERUNGSKREDIT`, `RATENKREDIT`, `BUSINESSKREDIT`

#### Gesamtkonditionen

    {
      "auszahlungsbetrag": BigDecimal,
      "effektivzins": BigDecimal,
      "gesamtkreditbetrag": BigDecimal,
      "laufzeitInMonaten": Int,
      "monatlicheRate": BigDecimal,
      "nettokreditbetrag": BigDecimal,
      "sollzins": BigDecimal
    }

The percentage values (`effektivzins`, `sollzins`) are based on 100 (`1.23` instead of `0.0123`).

#### Dokument

    {
      "url": String,
      "name": String
    }

#### Identifikation

    {
      antragstellername: String 
      qesUrl: String
      referenznummer: String
      videolegitimationUrl: String
    }

The field `antragstellername` contains the name in the format "\<first name\> \<last name\>".

## Terms of use

The APIs are made available under the following [Terms of Use](https://docs.api.europace.de/terms/).
