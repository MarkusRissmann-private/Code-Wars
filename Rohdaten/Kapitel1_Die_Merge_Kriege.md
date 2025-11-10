# Kapitel 1: Die Merge-Kriege

*Du hast die Geschichten gehört. Vom strahlenden X-Wing, der zu einem Todesstern wurde. Von der Klon-Armee, die aus einer einzigen Function erwuchs. Aber diese Geschichten beginnen nach dem Sieg. Nach dem „Great Split“.*

*Dies ist die Geschichte von davor.*

*Dies ist die Geschichte des Krieges, der diesen Sieg erst notwendig machte.*

## Prolog: Das Relikt

Der alte Jedi-Architekt zeigte auf ein verstaubtes Artefakt im digitalen Archiv. Ein Git-Repository, markiert als `OBSOLETE_DO_NOT_TOUCH`.

„Bevor es zwei Repositories gab – `frontend-app` und `backend-api` – gab es nur dieses eine."

Der junge Padawan beugte sich vor. Der Name des Repos war schlicht: `PhoenixProject`.

„Phoenix?" fragte er. „Sollte es nicht aus der Asche auferstehen?"

Der Alte lachte bitter. „Es ist nie auferstanden. Es hat nur Asche hinterlassen. Wir mussten auf den Trümmern neu bauen."

Er öffnete die Commit-History. Sie war ein Schlachtfeld.

```text
commit 8f3c5b1 (main)
Merge pull request #749 from feature/backend-hotfix-3
- HOTFIX: Critical payment bug
- Reverts frontend team's dependency updates to deploy

commit 2d8e7a4
Merge pull request #748 from feature/frontend-webpack-5-upgrade
- Big upgrade!
- NOTE: Breaks backend pipeline, fix TBD
```

„Siehst du?" sagte der Alte. „Commit 8f3c5b1. Ein Backend-Hotfix, der eine Frontend-Änderung zurücksetzen musste, nur um deployen zu können. Das war nicht der Anfang vom Ende. Das war das Ende."

„Was war der Anfang?"

Der Alte scrollte ganz nach oben. Zum allerersten Commit.

```text
commit 1a2b3c4
Author: Management <cto@empire.com>
Date: Mon Jan 5 10:00:00 2023

Initial commit: The Monorepo Dream
- Frontend (React) and Backend (.NET) in one place
- Simplified tooling and collaboration!
```

„Hier," sagte der Alte. „Mit einer Lüge. ‚Simplified collaboration'. Sie haben das Gegenteil geschaffen."

„Aber ein Monorepo kann doch gut sein?", wandte der Padawan ein.

„Ein Monorepo ist wie ein Lichtschwert," erwiderte der Meister. „In der Hand eines Meisters ist es eine Waffe der Präzision. In den Händen von Unvorbereiteten... schneidet es ihnen die eigenen Gliedmaßen ab. Dieses Team war unvorbereitet."

---

*Zwei Jahre früher...*

---

## I. Der gemeinsame Graben

Es gab zwei Stämme.

Die Frontend-Gilde. Meister von React, Hüter von `node_modules`, Sprecher der heiligen Sprache TypeScript. Ihre Welt war schnell, iterativ, visuell. Ihre Rüstung bestand aus `divs`, ihre Waffe war der `useEffect`-Hook.

Der Backend-Orden. Hüter der APIs, Meister von C#, Priester der Datenbank-Integrität. Ihre Welt war stabil, transaktional, abstrakt. Ihre Rüstung war die `try-catch`-Klausel, ihre Waffe die LINQ-Query.

Und sie lebten alle in einem Haus. Dem `PhoenixProject`-Repository.

Ein typischer Morgen, 10:15 Uhr:

Leya, eine Frontend-Entwicklerin, starrte auf ihre Pipeline. 45 Minuten. Und sie zählte immer noch. Sie hatte nur eine CSS-Farbe geändert.

```text
Slack-Channel: #phoenix-ci
- CI-Bot [10:15 AM]: Build #847 running... (Triggered by Leya)
- Step 1: Installing node modules... (5 min)
- Step 2: Running frontend tests... (8 min)
- Step 3: Building frontend assets... (7 min)
- Step 4: Restoring nuget packages... (6 min)
- Step 5: Building .NET solution... (12 min)
- Step 6: Running backend tests... (7 min)
- Step 7: Deploying... (pending)
```

„Warum," murmelte sie, „muss mein CSS-Fix durch einen C#-Compiler?"

Am Schreibtisch gegenüber seufzte Anakin, ein Backend-Entwickler. Er hatte gerade einen Pull Request gestellt. Ein einfacher Bugfix. Eine Zeile Code.

Die Pipeline-Vorschau zeigte 45 Minuten.

„Warum," sagte er zu seinem Monitor, „muss mein Datenbank-Fix warten, bis 80.000 JavaScript-Dateien installiert sind?"

Sie waren keine Feinde. Noch nicht. Sie waren Verbündete in einem Krieg gegen einen gemeinsamen Feind: die Pipeline.

Aber in jedem gemeinsamen Graben wächst irgendwann Misstrauen.

## II. Die Schlacht am Pull Request #347

Es war ein Donnerstag. 16:30 Uhr.

Ein kritisches Sicherheits-Ticket kam herein. SQL-Injection-Lücke im Backend. Priorität: Höchste Stufe.

Anakin und sein Team arbeiteten fieberhaft. Um 18:00 Uhr war der Hotfix fertig. Eine kleine, präzise Änderung. Getestet. Bereit zum Deployen.

Er öffnete den Pull Request #347 in `main`.

Und dann sah er es. Den roten Text.

**`1,247 conflicts.`**

Er erstarrte. Was war passiert?

Er öffnete den `main`-Branch. Und sah Pull Request #346. Fünf Minuten zuvor gemerged.

**`feat: Upgrade to Webpack 5 and refactor all frontend build scripts`**

Leya und ihr Team hatten wochenlang daran gearbeitet. Ein riesiges Refactoring. Notwendig. Aber es hatte alles berührt. `package.json`. `webpack.config.js`. Dutzende von Skripten. Und, aus irgendeinem Grund, die `.csproj`-Dateien des Backends, um „Pfade zu vereinheitlichen".

Anakins Hände zitterten leicht. Er rief Leya an.

„Leya, was habt ihr getan? Ich kann den Security-Hotfix nicht mergen!"

„Was? Wir haben nur das Frontend-Build modernisiert! Das sollte das Backend nicht betreffen."

„Es betrifft alles! Die `csproj`-Dateien sind voller Konflikte! Die Pipeline schlägt fehl, weil eure neuen Skripte globale Abhängigkeiten erwarten, die der Docker-Container nicht hat!"

„Das ist nicht unser Problem! Der Code hat auf meinem Rechner funktioniert!"

Das Gespräch wurde lauter. Es war kein Gespräch mehr. Es war eine Anklage.

Um 22:00 Uhr saßen beide Teams in einem Notfall-Meeting. Das Management war zugeschaltet. Der Hotfix war immer noch nicht live. Die Sicherheitslücke war immer noch offen.

Die Merge-Kriege hatten begonnen.

## III. Das Kriegsgericht

Das Meeting war ein Tribunal.

„Warum hat das Frontend-Team eine so große Änderung gemerged, ohne das Backend zu informieren?", fragte ein Manager.

„Warum hat das Backend-Team keine Branch-Policies, um `main` zu schützen?", konterte Leya.

„Warum dauert die Pipeline eine Stunde?", rief Anakin. „Wir könnten zehnmal deployen in dieser Zeit!"

„Warum müsst ihr überhaupt `npm install` ausführen, um eine API zu deployen?", schrie ein Frontend-Entwickler zurück.

Qui-Gon, der leitende Architekt, der bis jetzt geschwiegen hatte, trat vor das Whiteboard.

Er sagte nur ein Wort: „Stopp."

Der Raum wurde still.

„Ihr kämpft gegeneinander," sagte er leise. „Aber ihr habt den falschen Feind im Visier."

Er zeichnete eine einzige große Box.

```text
[PhoenixProject Repository]
- Frontend Lifecycle (schnell, iterativ)
- Backend Lifecycle (stabil, transaktional)
- EINE CI/CD Pipeline
- EINE "main" Branch
- EIN geteiltes Schicksal
```

„Das ist euer Feind," sagte er und tippte auf die Box. „Nicht Leya. Nicht Anakin. Diese Struktur. Ihr versucht, zwei verschiedene Organismen in einem Körper am Leben zu erhalten. Aber sie haben unterschiedliche Herzschläge. Unterschiedliche Atemzüge. Und jedes Mal, wenn einer atmet, erstickt der andere fast."

Stille.

„Ihr habt nicht das falsche Team," fuhr er fort. „Ihr habt das falsche Schlachtfeld."

## IV. Die Doktrin der Trennung

„Was schlägst du vor?", fragte der CTO.

Qui-Gon löschte die Box. Er zeichnete zwei neue, kleinere Boxen, mit einem leeren Raum dazwischen.

```text
[Frontend Repository]
- Eigener Lifecycle
- Eigene Pipeline (10 min)
- Eigene "main" Branch
- Deployt unabhängig

[Backend Repository]
- Eigener Lifecycle
- Eigene Pipeline (12 min)
- Eigene "main" Branch
- Deployt unabhängig
```

„Der große Schnitt," sagte er. „Wir trennen die Repositories. Wir trennen die Pipelines. Wir trennen das Schicksal."

Ein Raunen ging durch den Raum.

„Das ist ein riesiger Aufwand!", sagte ein Manager. „Das wird Wochen dauern!"

„Ja," sagte Qui-Gon. „Und der Hotfix, der seit sechs Stunden blockiert ist? Wie viel kostet uns das? Wie viel kostet uns der nächste blockierte Hotfix? Und der danach?"

Er sah Anakin und Leya an.

„Stellt euch vor," sagte er zu Leya, „du könntest eine CSS-Änderung in fünf Minuten live haben."

Ihre Augen weiteten sich.

„Stellt euch vor," sagte er zu Anakin, „du könntest einen Datenbank-Fix deployen, ohne jemals wieder `node_modules` zu sehen."

Ein schwaches Lächeln huschte über Anakins Gesicht.

„Der Schmerz, den wir jetzt investieren," schloss Qui-Gon, „wird den Schmerz von hunderten zukünftigen Schlachten verhindern. Wir beenden nicht nur diesen Kampf. Wir beenden den Krieg."

## V. Der große Schnitt

Es war kein glorreicher Moment. Es war ein Sumpf.

Drei Wochen lang arbeiteten die Teams nicht an Features. Sie arbeiteten an der Scheidung.

- Sie benutzten `git filter-branch`, um die Historie aufzuteilen, ein gefährlicher und fehleranfälliger Prozess.
- Sie schrieben die CI/CD-Pipelines von Grund auf neu.
- Sie entwirrten Konfigurationen, die sich über beide Domänen erstreckten.
- Sie stritten sich über die letzte gemeinsame `README.md`.

Es war mühsam. Es war frustrierend. Es fühlte sich an wie ein Rückschritt.

Aber eines Morgens, drei Wochen später, kam Anakin zur Arbeit. Er änderte eine Zeile Code im neuen `backend-api`-Repository. Erstellte einen Pull Request.

Die Pipeline lief. In 12 Minuten war sie grün.

Er klickte auf „Merge". Fünf Minuten später war der Code in Produktion.

Zur gleichen Zeit änderte Leya eine Komponente im `frontend-app`-Repository. Ihre Pipeline lief in 8 Minuten. Ihr Code war live.

Keine Konflikte. Keine Blockaden. Kein Krieg.

An diesem Nachmittag gab es eine Feier.

## VI. Der falsche Frieden

Das Team saß im Konferenzraum. Pizza. Bier. Die Stimmung war euphorisch.

„Keine Merge-Konflikte mehr!", rief Anakin und hob seine Flasche.

„Auf getrennte Pipelines!", antwortete Leya.

Sie lachten. Sie waren keine verfeindeten Stämme mehr. Sie waren wieder Verbündete.

Der Tech Lead lächelte. „Seht ihr? Wir haben es geschafft. Wir haben das Architekturproblem gelöst. Das war die Lektion."

Qui-Gon, in der Ecke, sagte nichts. Er sah die feiernden Gesichter. Er sah den Triumph in ihren Augen.

Und er sah die neue Gefahr, die niemand sonst sah.

Die Gefahr der halb gelernten Lektion.

Sie dachten, sie hätten „Architektur“ gelernt. Aber sie hatten nur gelernt, eine Mauer zu bauen. Sie hatten ein Symptom behandelt, nicht die Krankheit. Sie hatten die Repo-Struktur gelöst, aber nicht die Service-Struktur.

Sie fühlten sich unbesiegbar. Kompetent. Weise.

Und genau in diesem Moment des Triumphs wurde der Same für den nächsten Krieg gepflanzt. Der Krieg, der nicht zwischen Frontend und Backend stattfinden würde, sondern im Herzen des Backends selbst.

---

*Du hast gerade einen großen Sieg errungen. Du hast ein schreckliches, technisches Problem gelöst. Ein Legacy-System migriert. Eine Hölle von Abhängigkeiten entwirrt. Dein Team feiert dich. Du fühlst dich stark.*

*Genau jetzt, in diesem Moment des Triumphs, bist du am verletzlichsten.*

*Welche neue Wahrheit übersiehst du, geblendet vom Licht deines eigenen Erfolgs?*

*Welche Lektion hast du nur zur Hälfte gelernt?*
