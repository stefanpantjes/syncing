# PR-9733 reproduceren

Deze map bevat drie afzonderlijke testschema's. De bestaande mappen `schema/sync` en `prepr/schema` zijn niet gewijzigd.

| Scenario | Map | Inhoud |
| --- | --- | --- |
| Grote validatiefouten | `test-fixtures/pr-9733/validation-errors` | Eén bewust ongeldig model: 32 Asset-velden met elk 8 ongeldige accept-waarden, samen 256. |
| Beginversie | `test-fixtures/pr-9733/preview-v1` | 48 componentdefinities en één model. Content staat component 01 t/m 47 toe. |
| Gewijzigde versie | `test-fixtures/pr-9733/preview-v2` | Dezelfde model- en componentrefs. Content staat nu component 02 t/m 48 toe. |

Selecteer één scenariomap als bron voor de Git-schema-sync; selecteer niet deze overkoepelende map, want de snapshots bevatten dezelfde refs. Als jouw integratie een vaste schemamap gebruikt, kopieer dan uitsluitend de JSON-bestanden uit de gekozen scenariomap naar die map in een aparte testrepository.

Gebruik een lege testomgeving: een volledige sync kan schemaonderdelen verwijderen wanneer ze niet in de geselecteerde Git-map staan. Gebruik de aangepaste frontend met een REST6-backend waarop de `report_uri`-fallback actief is (volgens het ticket: `rest-dev`).

## Groot validatierapport

1. Gebruik `validation-errors` als bron en start Import via de Git-schema-sync.
2. Open DevTools, bekijk de WebSocket/Pusher-events en filter Network op `sync/report`.
3. Verwacht een validatiefout met `report_uri` wanneer het foutenrapport de eventlimiet van 10.240 bytes overschrijdt.
4. De frontend moet het volledige rapport ophalen, alle veldfouten tonen en loading stoppen zonder de import als geslaagd te tonen.

Voor de bestandsimport kun je [pr9733validationerrors.json](validation-errors/pr9733validationerrors.json) downloaden en via Schema → Add model → Or import a model uploaden. Het bestand is bewust ongeldig.

## Grote preview- en importevents

1. Gebruik `preview-v1` als bron en importeer deze beginversie in de lege testomgeving.
2. Gebruik daarna `preview-v2` als bron en start opnieuw Import.
3. Alleen de toegestane componenttypen van Content veranderen: 01 t/m 47 wordt 02 t/m 48. De model- en componentrefs blijven gelijk.
4. Verwacht voor een groot `dry_run.field_updated` event een `report_uri` en daarna een rapportaanvraag. Het opgehaalde rapport bevat `current.types` en `new.types`, elk met 47 typen.
5. Controleer dat de volledige wijziging zichtbaar is. Kies daarna Import schema om ook de Pull/import te controleren. Voltooiing moet wachten op de voorgaande rapportaanvragen.

De minimale berekende voorbeeldpayload bevat circa 12 kB compact JSON. De werkelijke backendpayload bepaalt of de grens wordt overschreden; controleer het werkelijk ontvangen event.

## Mislukte rapportaanvraag

Blokkeer de `sync/report`-aanvraag in DevTools en herhaal een test die een rapport ophaalt. Loading moet stoppen en de bestaande foutweergave moet verschijnen. Een later completion-event mag de operatie niet alsnog succesvol maken.

Voor een echte 404 kun je een bewaarde `report_uri` na de bewaartermijn van één uur opnieuw opvragen, vanuit dezelfde geauthenticeerde omgeving.

## Lokale controle

De JSON-syntaxis, refs en onderlinge verwijzingen zijn lokaal gecontroleerd. Alle bestanden gebruiken `schema_version: 2026-03-05`, zoals de bestaande exports in deze repository. Het validatieschema heeft bewust ongeldige accept-waarden. De bestanden zijn niet live tegen de backend gevalideerd of geïmporteerd.
