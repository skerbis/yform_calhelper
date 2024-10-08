# YFormCalendar

![Screenshot](https://github.com/FriendsOfRedaxo/yform_calendar/blob/assets/screenshot.png?raw=true)

YFormCalendar ist ein umfassendes Paket für REDAXO, das erweiterte Funktionen zur Verwaltung, zum Export und zur Anzeige von Kalenderereignissen bietet. Es liefert eine ModelClass um mit Kalenderdaten umzugehen. Die Daten werden Ical-konform gespeichert so dass ein späterer Export vereinfacht wird. Mit einem RRULE-Widget können Wiederholungen definiert werden. 

## YFormCalendar Feature-Liste

### CalRender-Klasse
- Abrufen von Kalenderereignissen basierend auf Datum, Zeit und anderen Parametern
- Unterstützung für wiederkehrende Ereignisse mit RRULE
- Sortierung von Ereignissen nach Start- oder Enddatum
- Benutzerdefinierte Abfragen mit YForm-Query-Unterstützung
- Generierung der nächsten X Ereignisse ab einem bestimmten Datum

### ICalExporter-Klasse
- Generierung von iCal-Dateien aus Kalenderereignissen
- Unterstützung für den Download von iCal-Dateien
- Erstellung von iCal-Strings für die direkte Ausgabe

### CalendarJsonExporter-Klasse
- Export von Kalenderereignissen im JSON-Format für FullCalendar
- Benutzerdefinierte Link-Generierung für Ereignisse
- Sortieroptionen für Ereignisse
- Unterstützung für Datumsbereiche beim Export

### RRULE-Widget
- Benutzeroberfläche zur Erstellung und Bearbeitung von Wiederholungsregeln
- Unterstützung für verschiedene Wiederholungsfrequenzen (täglich, wöchentlich, monatlich, jährlich)
- Einstellung von Intervallen, Wochentagen, Monatstagen
- Definition von Wiederholungsanzahl oder Enddatum

### Allgemeine Features
- Integration mit YForm und YForm Manager
- Unterstützung für ganztägige Ereignisse
- Handhabung von Ausnahmedaten (EXDATE) für wiederkehrende Ereignisse
- Kompatibilität mit FullCalendar für Frontend-Darstellung
- Flexibles Datenmodell mit Unterstützung für benutzerdefinierte Felder

## Installation

AddOn über den Installer installieren. 

Dieses Paket muss in einem REDAXO-Projekt-AddOn oder einem eigenen AddOn als abhängiges AddOn verwendet werden. Es ist sicherzustellen, dass die YForm und der YForm Manager installiert und aktiviert sind.

### Demo-Tableset für den Start verwenden

#### 1. install.php im Projekt-AddOn anlegen. 

```php 
<?php
if (rex_addon::get('yform') && rex_addon::get('yform')->isAvailable()) {
    rex_yform_manager_table_api::importTablesets(rex_file::get(rex_path::addon('yform_calendar', 'tableset/tableset.json')));
}
```
**Projekt-AddOn reinstallieren**. Danach sollte eine Tabelle erscheinen: YFormCalender
Diese kann nach Belieben erweitert werden. 

#### 2. boot.php des Projekt-AddOn erweitern

```php 
// Am Anfang einsetzen 
use FriendsOfRedaxo\YFormCalendar\CalRender;
// Einsetzen wo es Sinn ergibt

rex_yform_manager_dataset::setModelClass(
            'rex_yformcalendar',CalRender::class
);
```

Um weitere Tabellen zu verwenden sollten abgeleitete Classes der CalRender erstellt werden. Meist eine leere extended Class um weitere Tabellen anzumelden. 
Im Ordner /tablesets des AddOns befindet sich ein fertiges Tableset. Dieses kann als Ausgangspunkt für eigene Tabellen verwendet werden. 
Alternativ müssen folgende [Tabellenfelder](#erforderliche-tabellenfelder) angelegt sein. 



## Verwendung

```php
<?php
// Im Template oder Modul

// Alle Ereignisse im Juni 2024
$events = MeineCal::getEventsByDate('2024-06-01', '2024-06-30');

// Die nächsten 5 Ereignisse ab jetzt
$nextEvents = MeineCal::getNextEvents(1, 5, date('Y-m-d H:i:s'));

foreach ($events as $event) {
    // $event ist nun eine Instanz von MeineCal
    echo $event->getStartDate();
    // Verwenden Sie hier Ihre spezifischen Methoden
}
```

## CalRender-Klasse

Die `CalRender`-Klasse ist das Herzstück des AddOns. Sie ermöglicht das Abrufen, Filtern, Sortieren und Bearbeiten von Ereignissen.

### Methoden

#### `getCalendarEvents`

```php
public static function getCalendarEvents(array $params = [], rex_yform_manager_query $customQuery = null): Generator
```

Parameter:
- `$params` (optional): Ein Array mit folgenden möglichen Schlüsseln:
  - `startDate`: (string) Start-Datum/Zeit im Format 'Y-m-d' oder 'Y-m-d H:i:s'
  - `endDate`: (string) End-Datum/Zeit im Format 'Y-m-d' oder 'Y-m-d H:i:s'
  - `sortByStart`: (string) Sortierrichtung für Startdatum ('ASC' oder 'DESC')
  - `sortByEnd`: (string) Sortierrichtung für Enddatum ('ASC' oder 'DESC')
  - `whereRaw`: (string) Zusätzliche WHERE-Bedingung für die Abfrage
  - `limit`: (int) Maximale Anzahl der zurückzugebenden Ereignisse
- `$customQuery` (optional): Eine benutzerdefinierte YForm-Query

Rückgabewert: Ein Generator, der Objekte vom Typ `rex_yform_manager_dataset` liefert.

#### `getEventsByDate`

```php
public static function getEventsByDate(string $startDate, ?string $endDate = null, int $limit = PHP_INT_MAX): array
```

Parameter:
- `$startDate`: (string) Start-Datum im Format 'Y-m-d' oder 'Y-m-d H:i:s'
- `$endDate`: (string, optional) End-Datum im Format 'Y-m-d' oder 'Y-m-d H:i:s'
- `$limit`: (int, optional) Maximale Anzahl der zurückzugebenden Ereignisse

Rückgabewert: Ein Array von Objekten vom Typ `rex_yform_manager_dataset`.

#### `getNextEvents`

```php
public static function getNextEvents(int $eventId, int $limit, ?string $startDateTime = null): array
```

Parameter:
- `$eventId`: (int) Die ID des Referenzereignisses
- `$limit`: (int) Maximale Anzahl der zurückzugebenden Ereignisse
- `$startDateTime`: (string, optional) Start-Datum/Zeit im Format 'Y-m-d H:i:s'

Rückgabewert: Ein Array von Objekten vom Typ `rex_yform_manager_dataset`.

### Beispiele

```php
use FriendsOfRedaxo\YFormCalendar\CalRender;

// Beispiel 1: Alle Ereignisse im Juni 2024
$events = CalRender::getEventsByDate('2024-06-01', '2024-06-30');

// Beispiel 2: Die nächsten 5 Ereignisse ab jetzt
$nextEvents = CalRender::getNextEvents(1, 5, date('Y-m-d H:i:s'));

// Beispiel 3: Benutzerdefinierte Abfrage
$customQuery = rex_yform_manager_table::get('rex_calendar_events')->query()
    ->where('status', 'CONFIRMED');

$params = [
    'startDate' => '2024-01-01',
    'endDate' => '2024-12-31',
    'limit' => 50
];

$events = CalRender::getCalendarEvents($params, $customQuery);
foreach ($events as $event) {
    // $event ist ein rex_yform_manager_dataset Objekt
    echo $event->getValue('summary');
}
```

## Edit-Methoden 

### Zusätzliche Methoden für die CalRender-Klasse

#### `createEvent`

```php
public static function createEvent(array $data): ?rex_yform_manager_dataset
```

Diese Methode erstellt ein neues Ereignis in der Datenbank.

Parameter:
- `$data`: (array) Ein assoziatives Array mit Feldnamen und ihren Werten für das neue Ereignis.

Rückgabewert: Das neu erstellte Ereignis als `rex_yform_manager_dataset`-Objekt oder `null`, wenn die Erstellung fehlgeschlagen ist.

Beispiel für die Verwendung:
```php
$neuesEreignisDaten = [
    'summary' => 'Neues Ereignis',
    'description' => 'Dies ist ein neues Ereignis',
    'dtstart' => '2024-01-01 10:00:00',
    'dtend' => '2024-01-01 12:00:00',
    // ... weitere Felder nach Bedarf
];
$neuesEreignis = CalRender::createEvent($neuesEreignisDaten);
if ($neuesEreignis) {
    echo "Ereignis erfolgreich erstellt mit ID: " . $neuesEreignis->getId();
} else {
    echo "Ereignis konnte nicht erstellt werden";
}
```

#### `updateEventById`

```php
public static function updateEventById(int $eventId, array $data): bool
```

Diese Methode aktualisiert ein bestehendes Ereignis in der Datenbank anhand seiner ID.

Parameter:
- `$eventId`: (int) Die ID des zu aktualisierenden Ereignisses.
- `$data`: (array) Ein assoziatives Array mit Feldnamen und ihren neuen Werten.

Rückgabewert: `true`, wenn die Aktualisierung erfolgreich war, ansonsten `false`.

Beispiel für die Verwendung:
```php
$ereignisId = 123; // Die ID des zu aktualisierenden Ereignisses
$aktualisierungsDaten = [
    'summary' => 'Aktualisierter Ereignistitel',
    'description' => 'Dieses Ereignis wurde aktualisiert',
    // ... weitere zu aktualisierende Felder
];
$erfolg = CalRender::updateEventById($ereignisId, $aktualisierungsDaten);
if ($erfolg) {
    echo "Ereignis erfolgreich aktualisiert";
} else {
    echo "Aktualisierung des Ereignisses fehlgeschlagen";
}
```

#### `deleteEventById`

```php
public static function deleteEventById(int $eventId): bool
```

Diese Methode löscht ein Ereignis aus der Datenbank anhand seiner ID.

Parameter:
- `$eventId`: (int) Die ID des zu löschenden Ereignisses.

Rückgabewert: `true`, wenn das Löschen erfolgreich war, ansonsten `false`.

Beispiel für die Verwendung:
```php
$ereignisId = 123; // Die ID des zu löschenden Ereignisses
$erfolg = CalRender::deleteEventById($ereignisId);
if ($erfolg) {
    echo "Ereignis erfolgreich gelöscht";
} else {
    echo "Löschen des Ereignisses fehlgeschlagen";
}
```


## ICalExporter-Klasse

Die `ICalExporter`-Klasse ermöglicht den Export von Kalenderereignissen im iCal-Format.

### Methoden

#### `generateICalFile`

```php
public static function generateICalFile(string $filename, array $events): void
```

Parameter:
- `$filename`: (string) Der Name der zu generierenden Datei (ohne .ics Erweiterung)
- `$events`: (array) Ein Array von `rex_yform_manager_dataset` Objekten

Rückgabewert: Void. Diese Methode generiert eine Datei zum Download.

#### `generateICal`

```php
public static function generateICal(array $events): string
```

Parameter:
- `$events`: (array) Ein Array von `rex_yform_manager_dataset` Objekten

Rückgabewert: Ein String im iCal-Format.

### Beispiel

```php
use FriendsOfRedaxo\YFormCalendar\CalRender;
use FriendsOfRedaxo\YFormCalendar\ICalExporter;

$events = CalRender::getEventsByDate('2024-01-01', '2024-12-31');
$icalString = ICalExporter::generateICal($events);
echo $icalString; // Gibt den iCal-String aus

// Oder zum Herunterladen einer Datei:
ICalExporter::generateICalFile('kalender_2024', $events);
```

## CalendarJsonExporter-Klasse

Die `CalendarJsonExporter`-Klasse dient zum Exportieren von Kalenderereignissen im JSON-Format für FullCalendar.

### Konstruktor

```php
public function __construct(callable $linkCallback, string $modelClass)
```

Parameter:
- `$linkCallback`: (callable) Eine Funktion, die einen Link für jedes Ereignis generiert
- `$modelClass`: (string) Der Name der Modellklasse für die Ereignisse

### Methode

#### `generateJson`

```php
public function generateJson(?string $startDate = null, ?string $endDate = null, string $sortByStart = 'ASC', string $sortByEnd = 'ASC'): string
```

Parameter:
- `$startDate`: (string, optional) Start-Datum im Format 'Y-m-d' oder 'Y-m-d H:i:s'
- `$endDate`: (string, optional) End-Datum im Format 'Y-m-d' oder 'Y-m-d H:i:s'
- `$sortByStart`: (string, optional) Sortierrichtung für Startdatum ('ASC' oder 'DESC')
- `$sortByEnd`: (string, optional) Sortierrichtung für Enddatum ('ASC' oder 'DESC')

Rückgabewert: Ein JSON-String mit den Ereignisdaten.

### Beispiel

```php
<?php
use FriendsOfRedaxo\YFormCalendar\CalendarJsonExporter;
use FriendsOfRedaxo\YFormCalendar\CalRender; //ggf die eigene Modelclass angeben

// Callback-Funktion zur Linkgenerierung
$linkCallback = function($id) {
    return rex_getUrl('', '', ['cal' => $id]);
};

// Erstellen Sie die CalendarJsonExporter-Instanz, CalRender ggf. durch eigene Modelclass ersetzen
$calendarEventJson = new CalendarJsonExporter($linkCallback, CalRender::class);

// Generieren Sie das JSON für FullCalendar
$startDate = (new DateTime('today'))->format('Y-m-d');
$endDate = (new DateTime('+48 months'))->format('Y-m-d');
$eventsJson = $calendarEventJson->generateJson($startDate, $endDate, 'ASC', 'DESC');

// Wenn Sie das JSON überprüfen möchten, können Sie diesen Code verwenden:
// echo '<pre>' . json_encode(json_decode($eventsJson), JSON_PRETTY_PRINT) . '</pre>';
?>

<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <title>FullCalendar Beispiel</title>
    <!-- FullCalendar CSS -->
    <link href='https://cdn.jsdelivr.net/npm/fullcalendar@5.11.0/main.min.css' rel='stylesheet' />
    <!-- FullCalendar JavaScript -->
    <script src='https://cdn.jsdelivr.net/npm/fullcalendar@5.11.0/main.min.js'></script>
    <script src='https://cdn.jsdelivr.net/npm/fullcalendar@5.11.0/locales-all.min.js'></script>
    <!-- Tippy.js CSS -->
    <link href="https://unpkg.com/tippy.js@6/dist/tippy.css" rel="stylesheet">
    <!-- Tippy.js JavaScript -->
    <script src="https://unpkg.com/@popperjs/core@2"></script>
    <script src="https://unpkg.com/tippy.js@6"></script>
</head>
<body>
    <div id='calendar'></div>
    <script>
        document.addEventListener('DOMContentLoaded', function() {
            var calendarEl = document.getElementById('calendar');
            var calendar = new FullCalendar.Calendar(calendarEl, {
                initialView: 'dayGridMonth',
                locale: 'de',
                views: {
                    listMonth: { buttonText: 'Liste' },
                    timeGridWeek: { buttonText: 'Woche' }
                },
                headerToolbar: {
                    left: 'prev,next today',
                    center: 'title',
                    right: 'timeGridWeek,dayGridMonth,listMonth'
                },
                events: <?php echo $eventsJson; ?>,
                eventDidMount: function(info) {
                    tippy(info.el, {
                        content: info.event.extendedProps.description,
                        placement: 'top',
                        trigger: 'mouseenter',
                        theme: 'light',
                    });
                }
            });
            calendar.render();
        });
    </script>
</body>
</html>
```

## RRULE-Widget

Das RRULE-Widget ist eine Benutzeroberfläche zur Erstellung und Bearbeitung von Wiederholungsregeln für Ereignisse.

### RRULE-Wert Erklärung

Der RRULE-Wert ist ein String, der die Wiederholungsregel für ein Ereignis definiert. Komponenten:

- `FREQ`: Häufigkeit (DAILY, WEEKLY, MONTHLY, YEARLY)
- `INTERVAL`: Intervall zwischen Wiederholungen
- `BYDAY`: Wochentage für wöchentliche/monatliche Wiederholungen
- `BYMONTHDAY`: Tag des Monats für monatliche Wiederholungen
- `COUNT`: Anzahl der Wiederholungen
- `UNTIL`: Enddatum für Wiederholungen

Beispiel:
```
FREQ=WEEKLY;INTERVAL=2;BYDAY=MO,WE,FR;UNTIL=20240630T235959Z
```

### Verwendung des RRULE-Widgets

Das RRULE-Widget wird automatisch in YForm-Formularen für Felder vom Typ `rrule` angezeigt. Es generiert einen RRULE-String, der in der Datenbank gespeichert wird.

## Erforderliche Tabellenfelder

Für die korrekte Funktion des YFormCalendar-Pakets sind folgende Felder erforderlich:

1. **summary**: Titel des Ereignisses (Text)
2. **description**: Beschreibung des Ereignisses (Text)
3. **location**: Ort des Ereignisses (Text)
4. **dtstart**: Startdatum/-zeit (DateTime, Format: YYYY-MM-DD HH:MM:SS)
5. **dtend**: Enddatum/-zeit (DateTime, Format: YYYY-MM-DD HH:MM:SS)
6. **all_day**: Ganztägiges Ereignis (Boolean, 0 oder 1)
7. **rrule**: Wiederholungsregel (Text, RRULE-Format)
8. **exdate**: Ausnahmedaten (Text, Format: YYYY-MM-DD oder YYYY-MM-DD/YYYY-MM-DD, kommagetrennt)

## Weitere Beispiele

### Modul mit Performance-Test

Gibt eine Liste aller Termine für den angegebenen Zeitraum aus und die nächsten Termine einer ausgewählten ID. 


```php
<?php
use FriendsOfRedaxo\YFormCalendar\CalRender;

$startDate = date('Y-m-d');
$endDate = date('Y-m-d', strtotime('+10000 days'));
$eventId = 1; // Ersetzen Sie dies durch eine tatsächliche Event-ID aus Ihrer Datenbank
$limit = 10;
?>
<div class="calrender-test">
    <h2>CalRender Test Ausgabe</h2>
    <h3>1. Alle Events im Zeitraum (<?= $startDate ?> bis <?= $endDate ?>)</h3>
    <ul>
    <?php
    $events = CalRender::getEventsByDate($startDate, $endDate);
    foreach ($events as $event): ?>
        <li>
            <?= $event->getValue('summary') ?> - 
            Start: <?= rex_formatter::intlDateTime(strtotime($event->getValue('dtstart')), [IntlDateFormatter::MEDIUM, IntlDateFormatter::SHORT]) ?>, 
            Ende: <?= rex_formatter::intlDateTime(strtotime($event->getValue('dtend')), [IntlDateFormatter::MEDIUM, IntlDateFormatter::SHORT]) ?>
        </li>
    <?php endforeach; ?>
    </ul>
    <h3>2. Nächste maximal <?= $limit ?> Events für Event ID <?= $eventId ?></h3>
    <ul>
    <?php
    $nextEvents = CalRender::getNextEvents($eventId, $limit);
    foreach ($nextEvents as $event): ?>
        <li>
            <?= $event->getValue('summary') ?> - 
            Start: <?= rex_formatter::intlDateTime(strtotime($event->getValue('dtstart')), [IntlDateFormatter::MEDIUM, IntlDateFormatter::SHORT]) ?>, 
            Ende: <?= rex_formatter::intlDateTime(strtotime($event->getValue('dtend')), [IntlDateFormatter::MEDIUM, IntlDateFormatter::SHORT]) ?>
        </li>
    <?php endforeach; ?>
    </ul>
    <h3>3. Speicherverbrauch Test</h3>
    <?php
    $memoryBefore = memory_get_usage();
    $largeNumberOfEvents = iterator_to_array(CalRender::getCalendarEvents([
        'startDate' => $startDate,
        'endDate' => date('Y-m-d', strtotime('+10 year')),
        'limit' => 1000
    ]));
    $memoryAfter = memory_get_usage();
    $memoryUsed = $memoryAfter - $memoryBefore;
    ?>
    <p>Speicherverbrauch für 1000 Events: <?= number_format($memoryUsed / 1024 / 1024, 2) ?> MB</p>
    <h3>4. Leistungstest</h3>
    <?php
    $startTime = microtime(true);
    $events = iterator_to_array(CalRender::getCalendarEvents([
        'startDate' => $startDate,
        'endDate' => date('Y-m-d', strtotime('+10 year')),
        'limit' => 1000
    ]));
    $endTime = microtime(true);
    $executionTime = $endTime - $startTime;
    ?>
    <p>Zeit zum Abrufen von 1000 Events: <?= number_format($executionTime, 4) ?> Sekunden</p>
</div>
```

## Autor

**Friends Of REDAXO**

* http://www.redaxo.org
* https://github.com/FriendsOfREDAXO

**Projektleitung**

[Thomas Skerbis](https://github.com/skerbis)
