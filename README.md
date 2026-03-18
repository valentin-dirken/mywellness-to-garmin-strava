# Technogym to Garmin + Strava Converter

**Convert MyWellness (Technogym) workout data to native Garmin FIT files.**
Single HTML file. No server. No install. Runs entirely in your browser.

Created by [Valentin Dirken](https://github.com/valentindirken)

[English](#english) | [Francais](#francais) | [Nederlands](#nederlands) | [Deutsch](#deutsch)

---

## English

### What it does

Converts your Technogym MyWellness data export (JSON/ZIP) into binary FIT files that import directly into Garmin Connect and Strava. Built on the official [Garmin FIT JavaScript SDK v21.195](https://github.com/garmin/fit-javascript-sdk).

### Why it exists

Technogym equipment (Excite Climb, Artis Strength, treadmills, rowers) logs workouts to MyWellness Cloud. MyWellness syncs **to** Strava but **not to** Garmin Connect. Garmin Connect syncs **to** MyWellness but not the reverse. This tool closes the gap.

### Activity type mapping

**Indoor equipment (classified by MyWellness JSON field signature):**

| Technogym Equipment | Detection Field(s) | FIT sport / subSport | Garmin Connect | Strava |
|---|---|---|---|---|
| Excite Climb, Excite+ Climb | `Floors` | fitnessEquipment / stairClimbing | Stair Climbing | Stair Stepper |
| Selection, Biostrength, Pure Strength, Artis (strength) | `TotalIsoWeight` or `Rm1` | training / strengthTraining | Strength Training | Weight Training |
| Skillrow | `RowingDistance` | fitnessEquipment / indoorRowing | Indoor Rowing | Row |
| Excite Bike, Skillbike, Technogym Ride, Group Cycle | `AvgRpm` | cycling / indoorCycling | Indoor Cycling | Ride |
| Excite Synchro, Excite Vario | `AvgSpm` | fitnessEquipment / elliptical | Elliptical | Elliptical |
| Excite Run, Skillrun, Technogym Run | `HDistance` + `AvgSpeed` + (`AvgGrade` or `Elevation`) | running / treadmill | Treadmill Running | Run |
| Excite Recline, Excite Bike (no RPM) | `HDistance` + `AvgSpeed` (no grade) | cycling / indoorCycling | Indoor Cycling | Ride |
| Biocircuit, OMNIA, Kinesis, manual entries | Duration + Calories only | training / cardioTraining | Cardio | Workout |

**Outdoor activities (classified by MyWellness `activityName`):**

| Activity Name (contains) | FIT sport / subSport | Garmin Connect | Strava |
|---|---|---|---|
| run, course, jog | running / generic | Running | Run |
| walk, march | walking / generic | Walking | Walk |
| hik, rando | hiking / generic | Hiking | Hike |
| cycl, bik, velo, indoor cycling | cycling / generic | Cycling | Ride |
| row | rowing / generic | Rowing | Row |
| swim, nage | swimming / openWater | Open Water Swimming | Swim |
| surf | surfing / generic | Surfing | Surf |
| yoga | training / yoga | Yoga | Yoga |
| hiit, hyrox, burn, crossfit | hiit / generic | HIIT | Workout |
| box | boxing / generic | Boxing | Workout |
| stair, stairstepper | fitnessEquipment / stairClimbing | Stair Climbing | Stair Stepper |
| pilate | training / pilates | Pilates | Workout |
| (fallback) | training / cardioTraining | Cardio | Workout |

### FIT Protocol compliance

All output files pass the official SDK `checkIntegrity()` validation. Verified against the [FIT Protocol specification](https://developer.garmin.com/fit/protocol/):

- 14-byte header with computed CRC
- `.FIT` magic bytes at offset 8-11
- file_id message (type=activity) as first data record
- Sport message (mesg 12) for device compatibility
- Event start/stopAll pair (mesg 21)
- Record messages every 60 seconds with HR, distance, power (mesg 20)
- Lap message (mesg 19)
- Session message with sport/subSport, calories, HR, power (mesg 18)
- Activity message (mesg 34)
- File CRC (16-bit, little-endian)
- Scale/offset handled by SDK (distance x100, time x1000)
- Little-endian architecture

### Data preserved per activity

**In the FIT file** (visible in Garmin Connect and Strava):
- Duration, Calories, Avg/Max Heart Rate, Avg Power, Distance

**In `_notes_technogym.txt`** (included in the output ZIP):
Technogym-specific fields have no native FIT equivalent. They are preserved as structured plain text, one line per activity, cross-referenced by filename:

```
2025-03-15_09-00-00_stair_climbing.fit | Technogym Stair Climbing | Floors: 42 | Power: 120W | Level: 8.5 | VO2: 32.1
2025-03-15_10-30-00_strength_training.fit | Technogym Strength Training | Weight: 1250kg
2025-03-15_11-15-00_indoor_rowing.fit | Technogym Indoor Rowing | Power: 180W | Dist: 5.00km
```

This file is your reference for machine-specific data that Garmin and Strava cannot display. Keep it alongside your FIT files for complete records.

### How to use

1. **Export** your data from [mywellness.com](https://www.mywellness.com) (Settings > Data Export > Download). You get a ZIP containing JSON files.
2. **Open** `technogym-to-garmin.html` in any modern browser.
3. **Drop** the ZIP file onto the drop zone (or click to select).
4. **Filter** activity types using the checkboxes. Outdoor is unchecked by default (avoids duplicates if you already record outdoor activities with your Garmin watch).
5. **Download** the FIT ZIP.
6. **Import** into Garmin Connect: [connect.garmin.com](https://connect.garmin.com) > Import Data > drop the .fit files.
7. **Import** into Strava (option A): if your Garmin and Strava accounts are linked, activities auto-sync. (Option B): upload directly at [strava.com/upload/select](https://www.strava.com/upload/select).

### Training metrics

The 60-second record interval with heart rate data enables:
- **Garmin**: Training Effect (aerobic/anaerobic), Training Load, Recovery Time
- **Strava**: Relative Effort, Fitness & Freshness

### Technical details

- Single self-contained HTML file, no build step, no dependencies to install
- Uses `@garmin/fitsdk@21.195.0` via CDN (ESM import from jsdelivr)
- Uses `JSZip` via CDN for ZIP handling
- All processing is client-side (your data never leaves your browser)
- ES module (`<script type="module">`)

### Requirements

- A modern browser with ES module support (Chrome, Firefox, Safari, Edge)
- A MyWellness data export (ZIP or individual JSON files)

### License

MIT

### Disclaimer

This project is an **independent, community-built tool**. It is **not affiliated with, endorsed by, or supported by Technogym, Garmin, or Strava**.

- **No warranty.** This software is provided "as is", without warranty of any kind, express or implied. Use it at your own risk.
- **No data guarantee.** The converter performs a best-effort mapping from MyWellness JSON to Garmin FIT format. Heart rate, calories, power, and distance values are transferred as reported by MyWellness. The author does not guarantee the accuracy, completeness, or correctness of the converted data.
- **Training metrics.** Garmin Training Effect, Training Load, and Strava Relative Effort are computed from the imported heart rate records. These are approximations based on summary data (average HR), not second-by-second sensor recordings. They should not be used for medical decisions or precise training prescription.
- **Duplicate activities.** Importing FIT files into Garmin Connect or Strava may create duplicate entries if the same activities already exist from another source (e.g., Fenix watch, Strava-MyWellness sync). Use the filter checkboxes to avoid this.
- **Trademarks.** Technogym, MyWellness, Garmin, Garmin Connect, Fenix, FIT SDK, Strava, and all related logos are trademarks of their respective owners.
- **No liability.** The author shall not be held liable for any damages, data loss, or issues arising from the use of this tool.

By using this tool, you acknowledge that you have read and understood this disclaimer.

---

## Francais

### Ce que ca fait

Convertit l'export de donnees MyWellness (Technogym) en fichiers FIT binaires natifs, importables directement dans Garmin Connect et Strava. Construit sur le [SDK JavaScript FIT officiel de Garmin v21.195](https://github.com/garmin/fit-javascript-sdk).

### Pourquoi ca existe

Les equipements Technogym (Excite Climb, Artis Strength, tapis de course, rameurs) enregistrent les seances dans MyWellness Cloud. MyWellness synchronise **vers** Strava mais **pas vers** Garmin Connect. Garmin Connect synchronise **vers** MyWellness mais pas l'inverse. Cet outil comble le vide.

### Mapping des types d'activite

**Equipements indoor (classifies par champs JSON MyWellness) :**

| Equipement Technogym | Champ(s) de detection | FIT sport / subSport | Garmin Connect | Strava |
|---|---|---|---|---|
| Excite Climb, Excite+ Climb | `Floors` | fitnessEquipment / stairClimbing | Stair Climbing | Stair Stepper |
| Selection, Biostrength, Pure Strength, Artis (musculation) | `TotalIsoWeight` ou `Rm1` | training / strengthTraining | Strength Training | Weight Training |
| Skillrow | `RowingDistance` | fitnessEquipment / indoorRowing | Indoor Rowing | Row |
| Excite Bike, Skillbike, Technogym Ride, Group Cycle | `AvgRpm` | cycling / indoorCycling | Indoor Cycling | Ride |
| Excite Synchro, Excite Vario | `AvgSpm` | fitnessEquipment / elliptical | Elliptical | Elliptical |
| Excite Run, Skillrun, Technogym Run | `HDistance` + `AvgSpeed` + (`AvgGrade` ou `Elevation`) | running / treadmill | Treadmill Running | Run |
| Excite Recline, Excite Bike (sans RPM) | `HDistance` + `AvgSpeed` (sans grade) | cycling / indoorCycling | Indoor Cycling | Ride |
| Biocircuit, OMNIA, Kinesis, entrees manuelles | Duration + Calories uniquement | training / cardioTraining | Cardio | Workout |

**Activites outdoor (classifiees par `activityName` MyWellness) :**

| Nom de l'activite (contient) | FIT sport / subSport | Garmin Connect | Strava |
|---|---|---|---|
| run, course, jog | running / generic | Running | Run |
| walk, march | walking / generic | Walking | Walk |
| hik, rando | hiking / generic | Hiking | Hike |
| cycl, bik, velo | cycling / generic | Cycling | Ride |
| row | rowing / generic | Rowing | Row |
| swim, nage | swimming / openWater | Open Water Swimming | Swim |
| surf | surfing / generic | Surfing | Surf |
| yoga | training / yoga | Yoga | Yoga |
| hiit, hyrox, burn, crossfit | hiit / generic | HIIT | Workout |
| box | boxing / generic | Boxing | Workout |
| stair, stairstepper | fitnessEquipment / stairClimbing | Stair Climbing | Stair Stepper |
| pilate | training / pilates | Pilates | Workout |
| (defaut) | training / cardioTraining | Cardio | Workout |

### Conformite au protocole FIT

Tous les fichiers produits passent la validation `checkIntegrity()` du SDK officiel. Verification complete contre la [specification du protocole FIT](https://developer.garmin.com/fit/protocol/) : header 14 octets avec CRC, magic bytes `.FIT`, file_id en premier record, Sport message (mesg 12), paire Event start/stopAll, Records toutes les 60s, Lap, Session, Activity, CRC de fin de fichier.

### Comment utiliser

1. **Exporter** depuis [mywellness.com](https://www.mywellness.com) (Settings > Data Export > Download).
2. **Ouvrir** `technogym-to-garmin.html` dans un navigateur.
3. **Deposer** le ZIP sur la zone de drop.
4. **Filtrer** les types via les checkboxes. Outdoor est decoche par defaut (evite les doublons avec ta montre Garmin).
5. **Telecharger** le ZIP FIT.
6. **Importer** dans Garmin Connect : [connect.garmin.com](https://connect.garmin.com) > Import Data.
7. **Importer** dans Strava : auto-sync via Garmin ou upload direct sur [strava.com/upload/select](https://www.strava.com/upload/select).

### Metriques d'entrainement

L'intervalle de 60 secondes avec donnees HR active :
- **Garmin** : Training Effect, Training Load, temps de recuperation
- **Strava** : Relative Effort, Fitness & Freshness

### Donnees conservees par activite

**Dans le fichier FIT** (visible dans Garmin Connect et Strava) :
- Duree, Calories, FC moyenne/max, Puissance moyenne, Distance

**Dans `_notes_technogym.txt`** (inclus dans le ZIP de sortie) :
Les champs specifiques Technogym n'ont pas d'equivalent natif dans le format FIT. Ils sont conserves en texte structure, une ligne par activite :

```
2025-03-15_09-00-00_stair_climbing.fit | Technogym Stair Climbing | Floors: 42 | Power: 120W | Level: 8.5 | VO2: 32.1
```

Ce fichier est votre reference pour les donnees machine que Garmin et Strava ne peuvent pas afficher.

### Licence

MIT

### Avertissement

Ce projet est un **outil independant, developpe par la communaute**. Il n'est **ni affilie, ni approuve, ni soutenu par Technogym, Garmin ou Strava**.

- **Aucune garantie.** Ce logiciel est fourni "tel quel", sans garantie d'aucune sorte. Utilisation a vos propres risques.
- **Aucune garantie sur les donnees.** Le convertisseur effectue un mapping au mieux depuis le JSON MyWellness vers le format FIT Garmin. L'auteur ne garantit pas l'exactitude, l'exhaustivite ou la correction des donnees converties.
- **Metriques d'entrainement.** Le Training Effect Garmin et le Relative Effort Strava sont des approximations basees sur la FC moyenne, pas sur des enregistrements capteur seconde par seconde. Ne pas utiliser pour des decisions medicales.
- **Doublons.** L'import peut creer des doublons si les memes activites existent deja dans Garmin Connect ou Strava. Utilisez les filtres pour eviter cela.
- **Marques.** Technogym, MyWellness, Garmin, Garmin Connect, Fenix, FIT SDK, Strava sont des marques de leurs proprietaires respectifs.
- **Aucune responsabilite.** L'auteur ne saurait etre tenu responsable de tout dommage, perte de donnees ou probleme resultant de l'utilisation de cet outil.

En utilisant cet outil, vous reconnaissez avoir lu et compris cet avertissement.

---

## Nederlands

### Wat het doet

Converteert je MyWellness (Technogym) data-export (JSON/ZIP) naar native Garmin FIT-bestanden die rechtstreeks importeerbaar zijn in Garmin Connect en Strava. Gebouwd op de officiele [Garmin FIT JavaScript SDK v21.195](https://github.com/garmin/fit-javascript-sdk).

### Waarom het bestaat

Technogym-apparatuur (Excite Climb, Artis Strength, loopbanden, roeiers) slaat trainingen op in MyWellness Cloud. MyWellness synchroniseert **naar** Strava maar **niet naar** Garmin Connect. Garmin Connect synchroniseert **naar** MyWellness maar niet andersom. Deze tool overbrugt de kloof.

### Activiteitstypes mapping

**Indoor apparatuur (geclassificeerd op MyWellness JSON-velden):**

| Technogym apparaat | Detectieveld(en) | FIT sport / subSport | Garmin Connect | Strava |
|---|---|---|---|---|
| Excite Climb, Excite+ Climb | `Floors` | fitnessEquipment / stairClimbing | Stair Climbing | Stair Stepper |
| Selection, Biostrength, Pure Strength, Artis (kracht) | `TotalIsoWeight` of `Rm1` | training / strengthTraining | Strength Training | Weight Training |
| Skillrow | `RowingDistance` | fitnessEquipment / indoorRowing | Indoor Rowing | Row |
| Excite Bike, Skillbike, Technogym Ride, Group Cycle | `AvgRpm` | cycling / indoorCycling | Indoor Cycling | Ride |
| Excite Synchro, Excite Vario | `AvgSpm` | fitnessEquipment / elliptical | Elliptical | Elliptical |
| Excite Run, Skillrun, Technogym Run | `HDistance` + `AvgSpeed` + (`AvgGrade` of `Elevation`) | running / treadmill | Treadmill Running | Run |
| Excite Recline, Excite Bike (zonder RPM) | `HDistance` + `AvgSpeed` (zonder grade) | cycling / indoorCycling | Indoor Cycling | Ride |
| Biocircuit, OMNIA, Kinesis, handmatige invoer | Alleen Duration + Calories | training / cardioTraining | Cardio | Workout |

**Buitenactiviteiten (geclassificeerd op MyWellness `activityName`):**

| Activiteitsnaam (bevat) | FIT sport / subSport | Garmin Connect | Strava |
|---|---|---|---|
| run, course, jog | running / generic | Running | Run |
| walk, march | walking / generic | Walking | Walk |
| hik, rando | hiking / generic | Hiking | Hike |
| cycl, bik, velo | cycling / generic | Cycling | Ride |
| row | rowing / generic | Rowing | Row |
| swim, nage, zwem | swimming / openWater | Open Water Swimming | Swim |
| surf | surfing / generic | Surfing | Surf |
| yoga | training / yoga | Yoga | Yoga |
| hiit, hyrox, burn, crossfit | hiit / generic | HIIT | Workout |
| box | boxing / generic | Boxing | Workout |
| stair, stairstepper | fitnessEquipment / stairClimbing | Stair Climbing | Stair Stepper |
| pilate | training / pilates | Pilates | Workout |
| (standaard) | training / cardioTraining | Cardio | Workout |

### FIT Protocol conformiteit

Alle outputbestanden slagen voor de officiele SDK `checkIntegrity()` validatie. Geverifieerd tegen de [FIT Protocol specificatie](https://developer.garmin.com/fit/protocol/): 14-byte header met CRC, `.FIT` magic bytes, file_id als eerste datarecord, Sport message (mesg 12), Event start/stopAll paar, Records elke 60 seconden, Lap, Session, Activity, file CRC.

### Gebruik

1. **Exporteer** je data van [mywellness.com](https://www.mywellness.com) (Settings > Data Export > Download).
2. **Open** `technogym-to-garmin.html` in een moderne browser.
3. **Sleep** het ZIP-bestand naar de dropzone.
4. **Filter** activiteitstypes met de selectievakjes. Outdoor is standaard uitgeschakeld (voorkomt duplicaten als je buitenactiviteiten al met je Garmin-horloge registreert).
5. **Download** de FIT ZIP.
6. **Importeer** in Garmin Connect: [connect.garmin.com](https://connect.garmin.com) > Import Data.
7. **Importeer** in Strava: auto-sync via Garmin of direct uploaden op [strava.com/upload/select](https://www.strava.com/upload/select).

### Trainingsmetrieken

Het 60-seconden interval met hartslagdata activeert:
- **Garmin**: Training Effect, Training Load, hersteltijd
- **Strava**: Relative Effort, Fitness & Freshness

### Bewaarde gegevens per activiteit

**In het FIT-bestand** (zichtbaar in Garmin Connect en Strava):
- Duur, Calorieen, Gem./Max hartslag, Gem. vermogen, Afstand

**In `_notes_technogym.txt`** (opgenomen in de output-ZIP):
Technogym-specifieke velden hebben geen native FIT-equivalent. Ze worden bewaard als gestructureerde tekst, een regel per activiteit:

```
2025-03-15_09-00-00_stair_climbing.fit | Technogym Stair Climbing | Floors: 42 | Power: 120W | Level: 8.5 | VO2: 32.1
```

### Licentie

MIT

### Disclaimer

Dit project is een **onafhankelijke tool, ontwikkeld door de community**. Het is **niet gelieerd aan, goedgekeurd door, of ondersteund door Technogym, Garmin of Strava**.

- **Geen garantie.** Deze software wordt geleverd "zoals het is", zonder enige garantie. Gebruik op eigen risico.
- **Geen gegevensgarantie.** De converter voert een best-effort mapping uit van MyWellness JSON naar Garmin FIT-formaat. De auteur garandeert niet de nauwkeurigheid, volledigheid of juistheid van de geconverteerde gegevens.
- **Trainingsmetrieken.** Garmin Training Effect en Strava Relative Effort zijn benaderingen gebaseerd op gemiddelde hartslag, niet op seconde-per-seconde sensorregistraties. Niet gebruiken voor medische beslissingen.
- **Duplicaten.** Import kan duplicaten creeren als dezelfde activiteiten al bestaan in Garmin Connect of Strava. Gebruik de filters om dit te voorkomen.
- **Merken.** Technogym, MyWellness, Garmin, Garmin Connect, Fenix, FIT SDK, Strava zijn merken van hun respectieve eigenaren.
- **Geen aansprakelijkheid.** De auteur kan niet aansprakelijk worden gesteld voor schade, gegevensverlies of problemen die voortvloeien uit het gebruik van deze tool.

Door deze tool te gebruiken, erkent u dat u deze disclaimer hebt gelezen en begrepen.

---

## Deutsch

### Was es macht

Konvertiert deinen MyWellness (Technogym) Datenexport (JSON/ZIP) in native Garmin FIT-Dateien, die direkt in Garmin Connect und Strava importiert werden konnen. Basiert auf dem offiziellen [Garmin FIT JavaScript SDK v21.195](https://github.com/garmin/fit-javascript-sdk).

### Warum es existiert

Technogym-Gerate (Excite Climb, Artis Strength, Laufbander, Rudermaschinen) speichern Trainingseinheiten in der MyWellness Cloud. MyWellness synchronisiert **zu** Strava aber **nicht zu** Garmin Connect. Garmin Connect synchronisiert **zu** MyWellness aber nicht umgekehrt. Dieses Tool schliesst die Lucke.

### Aktivitatstypen-Zuordnung

**Indoor-Gerate (klassifiziert nach MyWellness JSON-Feldern):**

| Technogym Gerat | Erkennungsfeld(er) | FIT sport / subSport | Garmin Connect | Strava |
|---|---|---|---|---|
| Excite Climb, Excite+ Climb | `Floors` | fitnessEquipment / stairClimbing | Stair Climbing | Stair Stepper |
| Selection, Biostrength, Pure Strength, Artis (Kraft) | `TotalIsoWeight` oder `Rm1` | training / strengthTraining | Strength Training | Weight Training |
| Skillrow | `RowingDistance` | fitnessEquipment / indoorRowing | Indoor Rowing | Row |
| Excite Bike, Skillbike, Technogym Ride, Group Cycle | `AvgRpm` | cycling / indoorCycling | Indoor Cycling | Ride |
| Excite Synchro, Excite Vario | `AvgSpm` | fitnessEquipment / elliptical | Elliptical | Elliptical |
| Excite Run, Skillrun, Technogym Run | `HDistance` + `AvgSpeed` + (`AvgGrade` oder `Elevation`) | running / treadmill | Treadmill Running | Run |
| Excite Recline, Excite Bike (ohne RPM) | `HDistance` + `AvgSpeed` (ohne Grade) | cycling / indoorCycling | Indoor Cycling | Ride |
| Biocircuit, OMNIA, Kinesis, manuelle Eingaben | Nur Duration + Calories | training / cardioTraining | Cardio | Workout |

**Outdoor-Aktivitaten (klassifiziert nach MyWellness `activityName`):**

| Aktivitatsname (enthalt) | FIT sport / subSport | Garmin Connect | Strava |
|---|---|---|---|
| run, course, jog | running / generic | Running | Run |
| walk, march | walking / generic | Walking | Walk |
| hik, rando | hiking / generic | Hiking | Hike |
| cycl, bik, velo | cycling / generic | Cycling | Ride |
| row | rowing / generic | Rowing | Row |
| swim, nage | swimming / openWater | Open Water Swimming | Swim |
| surf | surfing / generic | Surfing | Surf |
| yoga | training / yoga | Yoga | Yoga |
| hiit, hyrox, burn, crossfit | hiit / generic | HIIT | Workout |
| box | boxing / generic | Boxing | Workout |
| stair, stairstepper | fitnessEquipment / stairClimbing | Stair Climbing | Stair Stepper |
| pilate | training / pilates | Pilates | Workout |
| (Standard) | training / cardioTraining | Cardio | Workout |

### FIT-Protokoll Konformitat

Alle Ausgabedateien bestehen die offizielle SDK `checkIntegrity()` Validierung. Verifiziert gegen die [FIT-Protokoll-Spezifikation](https://developer.garmin.com/fit/protocol/): 14-Byte-Header mit CRC, `.FIT` Magic Bytes, file_id als erster Datensatz, Sport Message (mesg 12), Event start/stopAll Paar, Records alle 60 Sekunden, Lap, Session, Activity, Datei-CRC.

### Verwendung

1. **Exportiere** deine Daten von [mywellness.com](https://www.mywellness.com) (Settings > Data Export > Download).
2. **Offne** `technogym-to-garmin.html` in einem modernen Browser.
3. **Ziehe** die ZIP-Datei auf die Dropzone.
4. **Filtere** Aktivitatstypen mit den Kontrollkastchen. Outdoor ist standardmassig deaktiviert (vermeidet Duplikate, wenn du Outdoor-Aktivitaten bereits mit deiner Garmin-Uhr aufzeichnest).
5. **Lade** die FIT-ZIP herunter.
6. **Importiere** in Garmin Connect: [connect.garmin.com](https://connect.garmin.com) > Import Data.
7. **Importiere** in Strava: Auto-Sync uber Garmin oder direkt hochladen auf [strava.com/upload/select](https://www.strava.com/upload/select).

### Trainingsmetriken

Das 60-Sekunden-Intervall mit Herzfrequenzdaten aktiviert:
- **Garmin**: Training Effect, Training Load, Erholungszeit
- **Strava**: Relative Effort, Fitness & Freshness

### Gespeicherte Daten pro Aktivitat

**In der FIT-Datei** (sichtbar in Garmin Connect und Strava):
- Dauer, Kalorien, Durchschn./Max Herzfrequenz, Durchschn. Leistung, Distanz

**In `_notes_technogym.txt`** (in der Ausgabe-ZIP enthalten):
Technogym-spezifische Felder haben kein natives FIT-Aquivalent. Sie werden als strukturierter Text gespeichert, eine Zeile pro Aktivitat:

```
2025-03-15_09-00-00_stair_climbing.fit | Technogym Stair Climbing | Floors: 42 | Power: 120W | Level: 8.5 | VO2: 32.1
```

### Lizenz

MIT

### Haftungsausschluss

Dieses Projekt ist ein **unabhangiges, von der Community entwickeltes Tool**. Es ist **weder mit Technogym, Garmin oder Strava verbunden, noch von diesen genehmigt oder unterstutzt**.

- **Keine Garantie.** Diese Software wird "wie besehen" bereitgestellt, ohne jegliche ausdruckliche oder stillschweigende Garantie. Nutzung auf eigene Gefahr.
- **Keine Datengarantie.** Der Converter fuhrt eine Best-Effort-Zuordnung von MyWellness-JSON zum Garmin-FIT-Format durch. Der Autor garantiert nicht die Genauigkeit, Vollstandigkeit oder Korrektheit der konvertierten Daten.
- **Trainingsmetriken.** Garmin Training Effect und Strava Relative Effort sind Naherungswerte basierend auf der durchschnittlichen Herzfrequenz, nicht auf sekundengenauen Sensoraufzeichnungen. Nicht fur medizinische Entscheidungen verwenden.
- **Duplikate.** Der Import kann Duplikate erzeugen, wenn dieselben Aktivitaten bereits in Garmin Connect oder Strava vorhanden sind. Verwenden Sie die Filter, um dies zu vermeiden.
- **Marken.** Technogym, MyWellness, Garmin, Garmin Connect, Fenix, FIT SDK, Strava sind Marken ihrer jeweiligen Inhaber.
- **Keine Haftung.** Der Autor ubernimmt keine Haftung fur Schaden, Datenverlust oder Probleme, die aus der Nutzung dieses Tools entstehen.

Durch die Nutzung dieses Tools bestatigen Sie, dass Sie diesen Haftungsausschluss gelesen und verstanden haben.

---

## Project structure

```
technogym-to-garmin.html    # Single-file converter (drop in browser, done)
README.md                   # This file
LICENSE                     # MIT License
```

## Dependencies (loaded via CDN at runtime)

| Library | Version | Purpose |
|---|---|---|
| [@garmin/fitsdk](https://www.npmjs.com/package/@garmin/fitsdk) | 21.195.0 | Official Garmin FIT file encoding |
| [JSZip](https://stuk.github.io/jszip/) | 3.10.1 | ZIP file reading and writing |

## Contributing

Issues and pull requests welcome. The converter handles the 10 most common Technogym activity types. If you encounter equipment with different field signatures in the MyWellness JSON, open an issue with a sample (anonymized) JSON entry.

## Acknowledgments

- [Garmin FIT SDK](https://developer.garmin.com/fit/overview/) for the official JavaScript encoder
- [Garmin FIT Protocol specification](https://developer.garmin.com/fit/protocol/) for the binary format documentation
- [Strava file upload documentation](https://developers.strava.com/docs/uploads/) for FIT Activity File compatibility requirements
