# Kapitel 2: Wir haben doch schon...

## Prolog: Die Geister der Entscheidungen

*„Eine Entscheidung ist wie ein Stein, den du ins Wasser wirfst. Die Wellen siehst du sofort. Die Veränderung des Flussbetts – die siehst du erst Jahre später, wenn es zu spät ist, den Stein zurückzuholen."*

— Aus den Chroniken des Jedi-Ordens der Clean-Code-Architekten

---

Der alte Jedi-Architekt öffnete zwei Fenster nebeneinander.

Links: Das Git-Repository. Zwei Monate nach dem Great Split. Sauber. Getrennt. Backend-only. Keine Frontend-Konflikte mehr.

Rechts: Dasselbe Repository. Vier Monate später.

Der junge Padawan las über seine Schulter:

```text
commit a3f8e92
Author: Anakin <anakin@rebels.dev>
Date: Mon Nov 4 09:23:45 2024

feat: Added API Beta support
- Added if/else for auth switching
- Reused existing upload logic
- Quick win! 🚀
```

„Siehst du?" sagte der Alte. „Hier. Commit a3f8e92. Das ist der Moment."

„Der Moment wovon?"

„Der Moment, in dem aus einem X-Wing ein Todesstern wurde. Der Moment, in dem ‚Wiederverwendung' zu ‚Verkrüppelung' wurde."

Er klickte auf den Commit. Das Diff explodierte über den Bildschirm. 400 Zeilen geändert. In einer einzigen Datei. In einer einzigen Methode.

Der junge Padawan starrte darauf. „Aber... das ist doch nur eine weitere API? Das ist doch Wiederverwendung? Das ist doch gut?"

Der Alte lachte. Es war kein fröhliches Lachen.

„Wiederverwendung," sagte er langsam, „ist wie Feuer. In einem Ofen wärmt es dein Haus. Außer Kontrolle brennt es dein Haus nieder."

Er deutete auf eine Zeile im Code:

```csharp
if (source == "ApiAlpha") {
    // Alpha logic
} else if (source == "ApiBeta") {
    // Beta logic - almost same but different auth
} else if ...
```

„Das ist kein Feuer im Ofen," sagte er. „Das ist Feuer auf dem Teppich."

Der junge Padawan runzelte die Stirn. „Aber sie hatten doch gerade den Great Split gemacht? Die Repos getrennt? Sie hatten doch gelernt?"

Der Alte nickte langsam. „Sie lernten. Die falsche Lektion."

„Was meinst du?"

„Sie lernten: ‚Trenne Frontend und Backend.' Das war richtig. Aber sie lernten nicht: ‚Trenne Verantwortlichkeiten.' Sie dachten, ein sauberes Repo bedeutet saubere Architektur."

Er scrollte weiter nach unten im Diff.

„Sie hatten jetzt ihr eigenes Repository. Keine Merge-Konflikte mehr. Keine Frontend-Entwickler, die ihre Backend-Files anfassen. Sie fühlten sich... unbesiegbar."

„Und das war das Problem?"

„Das war das Problem. Nach einem Sieg – einem echten, sichtbaren Sieg – wird man übermütig. Man denkt: ‚Wir haben es verstanden. Wir können jetzt alles.' Und genau dann begeht man den größten Fehler."

Er zeigte auf die Zeile: `feat: Added API Beta support`

„Das war nicht böswillig. Das war nicht dumm. Das war... menschlich. Sie hatten die Infrastruktur. Sie hatten das Repo. Sie hatten den Momentum."

„Also fügten sie einfach hinzu."

„Genau. Weil sie **doch schon** die Infrastruktur hatten."

---

*Drei Monate nach dem Great Split...*

*Zwei Monate nach dem ersten erfolgreichen Upload...*

*Eine Woche nachdem die letzte Frontend-Merge-Hölle zur Erinnerung wurde...*

---

## I. Die Nachricht

Montagmorgen, 9:03 Uhr.

Anakin öffnete seinen Laptop. Kaffee in der Hand. Gute Laune. Das Wochenende war schön gewesen. Keine Production-Alerts. Keine Midnight-Deployments. Keine Merge-Konflikte.

*Keine Merge-Konflikte.*

Das allein war Grund zur Freude.

Dann sah er die Slack-Nachricht. Vom Freitag Abend. Beim Bier hatte er sie gelesen. Aber nicht geantwortet.

*Montag war früh genug*, hatte er gedacht.

Jetzt war Montag.

```text
**Product Owner:** „Hey team! 🎉 Great work on the split. Everything's running smooth now. Quick question – can we add **API Beta**? It's basically the same as Alpha, just different auth. Should be quick, right?"
```

Anakin las es. Wieder. Wie am Freitag schon.

„Basically the same" war gut. „Quick" war besser.

Er öffnete die `DmsUploader.cs`. 850 Zeilen. Alles in der Run-Methode. Aber es funktionierte. Der Great Split war erfolgreich. Das Repo war sauber. Und – wichtigster Punkt – keine Frontend-Dependencies mehr.

*Wir haben unser eigenes Repo jetzt. Wir können tun, was wir wollen.*

```text
**Anakin (im Chat):** „Sure! We already have the infrastructure. It's just a new client. I'll add a parameter for the source type. Done by Wednesday 👍"

**Product Owner:** „You're a star! ⭐"
```

Anakin lehnte sich zurück. Fühlte sich gut. Das war, warum er Entwickler war. Probleme lösen. Schnell. Effizient.

Und diesmal – diesmal ohne die Frontend-Hölle.

Er öffnete eine neue Branch: `feature/api-beta-support`

Die Finger flogen über die Tastatur.

---

*Und so begann der zweite Krieg.*

*Nicht der Krieg der Repos.*

*Der Krieg der Verantwortlichkeiten.*

---

## II. Der erste Riss

Die Logik war einfach. Fast zu einfach.

API Alpha und API Beta waren identisch. Bis auf die Authentifizierung. Alpha nutzte Basic Auth. Beta nutzte OAuth2.

„Kein Problem," murmelte Anakin. „Ein Parameter. Ein if-Statement."

```csharp
public async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    ILogger log)
{
    string source = req.Query["source"]; // NEW: "alpha" or "beta"
    
    // Fetch document
    Document doc;
    if (source == "alpha") {
        doc = await FetchFromAlpha(req);
    } else if (source == "beta") {
        doc = await FetchFromBeta(req);
    } else {
        return new BadRequestResult();
    }
    
    // Upload to Drive (same for both)
    var driveFileId = await UploadToDrive(doc);
    
    // Patch back
    if (source == "alpha") {
        await PatchToAlpha(doc.Id, driveFileId);
    } else if (source == "beta") {
        await PatchToBeta(doc.Id, driveFileId);
    }
    
    return new OkResult();
}
```

Er betrachtete den Code. Sauber. DRY. Der Upload-Teil war shared. Nur Auth und Patch waren unterschiedlich.

„Perfekt," sagte er zu sich selbst.

Er committete. Pushed. Öffnete einen Pull Request.

**PR Title:** `feat: Add API Beta support - Reuse existing infrastructure`

**Description:**

```text
Quick win! 🚀
- Reused upload logic (DRY principle)
- Added source parameter
- Minimal changes, maximum value
```

## III. Das Review, das nicht stattfand

Obi-Wan sah die PR-Notification. Klickte sie an. Sah das Diff.

400 Zeilen geändert. Alles in `DmsUploader.cs`.

Er scrollte durch den Code. Sah die if-Statements. Sah die Duplikation in den Auth-Methoden. Sah die Duplikation in den Patch-Methoden.

Seine Finger schwebten über dem Kommentar-Feld.

*Der Kommentar, den er schreiben wollte:*

```text
"This is starting to smell. We now have two sources, and the logic is 
branching. What happens when API Gamma comes? When OneDrive replaces 
Drive? This is the moment to stop and refactor to interfaces. 
Let me sketch what that would look like..."
```

*Der Kommentar, den er tatsächlich schrieb:*

```text
"LGTM! ✅ Nice reuse of the upload logic."
```

Warum?

Weil es Montag, 14:30 Uhr war. Weil er selbst drei andere Tickets hatte. Weil ein „Bitte refactor zuerst"-Kommentar zu einem Tag Diskussion führen würde. Weil Anakin gute Argumente haben würde: „YAGNI. It's just two sources. When Gamma comes, then we refactor."

Und die Argumente waren nicht falsch.

Sie waren nur... kurzsichtig.

Obi-Wan klickte „Approve".

## IV. Die zweite Stimme

Qui-Gon sah die PR eine Stunde später.

Er öffnete sie nicht. Er brauchte das Diff nicht zu sehen. Er kannte dieses Drehbuch.

Er ging zu Anakin's Desk.

„Hey," sagte er. Leise. Fast entschuldigend.

Anakin drehte sich um. „Hey! Hast du meine PR gesehen? Clean, oder? Minimal change, maximum reuse."

„Ja," sagte Qui-Gon. „Ich habe eine Frage."

„Shoot."

„Was passiert, wenn API Gamma kommt?"

Anakin blinzelte. „Dann füge ich ein weiteres else-if hinzu?"

„Und wenn OneDrive das neue Ziel wird?"

„Dann... wrappen wir den Drive-Upload in eine Abstraktion?"

„Und wenn Delta kommt? Epsilon? Was ist die Grenze?"

Anakin lehnte sich zurück. Sein Lächeln wurde schmaler. „Was willst du sagen?"

Qui-Gon seufzte. „Ich will sagen: Dies ist der Moment, an dem wir stoppen sollten. Die Function macht noch eine Sache: Dokumente von Quelle zu Ziel bewegen. Aber sie macht es jetzt für zwei Quellen. Bald drei. Dann vier. Wo ist die Grenze?"

„Die Grenze," sagte Anakin, und seine Stimme hatte einen Hauch von Gereiztheit, „ist, wenn es nicht mehr wartbar ist. Und das ist es noch. Two sources, same logic. Das ist Wiederverwendung. Das ist DRY."

„DRY," sagte Qui-Gon langsam, „bedeutet ‚Don't Repeat Yourself'. Es bedeutet nicht ‚Stopfe alles in eine Datei'."

Die Luft zwischen ihnen wurde kälter.

„Hast du die PR geblockt?" fragte Anakin.

„Nein," sagte Qui-Gon. „Ich wollte nur fragen."

Er ging zurück zu seinem Desk.

Anakin starrte ihm nach. Dann, leise, zu sich selbst: „Old school. Sie verstehen moderne Entwicklung nicht. Agil heißt liefern, nicht endlos diskutieren."

Er klickte „Merge".

## V. Der Fall beschleunigt sich

Drei Wochen später.

```text
**Product Owner:** „API Beta is a hit! Client wants API Gamma now. Also different auth (API Key), but otherwise same. Can we add it?"

**Anakin:** „On it 👍"
```

Ein weiteres else-if. 200 weitere Zeilen.

---

Vier Wochen später.

```text
**Product Owner:** „Quick one – can we support OneDrive instead of Google Drive for some clients? They want to choose the target."

**Anakin:** „Ugh. That's... more complicated. But doable. Give me a few days."
```

Jetzt brauchte es zwei Parameter: `source` und `target`.

```csharp
if (source == "alpha") {
    doc = await FetchFromAlpha(req);
} else if (source == "beta") {
    ...
}

if (target == "gdrive") {
    driveFileId = await UploadToGoogleDrive(doc);
} else if (target == "onedrive") {
    driveFileId = await UploadToOneDrive(doc);
}

if (source == "alpha" && target == "gdrive") {
    await PatchToAlpha(doc.Id, driveFileId);
} else if (source == "alpha" && target == "onedrive") {
    await PatchToAlphaOneDrive(doc.Id, driveFileId);
} else if ...
```

Die Run-Methode war jetzt 1,400 Zeilen lang.

---

Sechs Wochen später.

```text
**Product Owner:** „They want validation before upload. Just check file size and extension. Easy, right?"

**Anakin:** „..."
```

Er starrte auf den Code. 1,400 Zeilen. Verschachtelte if-Statements. Er brauchte drei Minuten, um zu verstehen, wo er das Validation-Logic einfügen sollte.

Vor drei Monaten hätte er die gesamte Datei in fünf Sekunden verstanden.

Er öffnete Slack.

```text
**Anakin (an Obi-Wan, privat):** „Help. I think we need to refactor."

**Obi-Wan:** „Now? With all the Production traffic?"

**Anakin:** „..."
```

Er schloss Slack. Fügte das Validation-Logic ein. Weitere 200 Zeilen. Mehr if-Statements.

Der PR-Titel war nicht mehr optimistisch:

```text
feat: Add validation (TODO: Refactor needed)
```

## VI. Die Archäologie des Scheiterns

Drei Monate nach dem ersten „API Beta"-Commit saß das Team in einer Retrospektive.

Die `DmsUploader.cs` war 2,100 Zeilen lang.

Sie unterstützte:

- 4 APIs (Alpha, Beta, Gamma, Delta)
- 3 Targets (Google Drive, OneDrive, SharePoint)
- 2 Validierungs-Modi
- 1 experimentelles Feature für „Dokumenten-Transformation"

Das Product Backlog hatte noch 5 weitere Requests:

- API Epsilon
- Azure Blob Storage als Target
- OCR-Validierung
- Multi-Upload (mehrere Dokumente gleichzeitig)
- Retry-Logic

„Okay," sagte der Tech Lead. „Lasst uns ehrlich sein. Wie fühlt sich der Code an?"

Stille.

Dann, Obi-Wan: „Ich habe Angst, ihn anzufassen."

Anakin nickte. „Ich auch. Und ich habe ihn geschrieben."

Qui-Gon sagte nichts. Aber sein Gesicht sagte alles.

Der Tech Lead: „Was ist passiert?"

Anakin: „Wir haben wiederverwendet. DRY. Effizienz."

Obi-Wan: „Wir haben nie gestoppt. Jede Anforderung war ‚nur noch eine kleine Änderung'."

Qui-Gon, schließlich: „Wir haben die erste Regel gebrochen."

„Welche Regel?"

„Die Regel der Verantwortung. Diese Function hatte eine Verantwortung: Dokumente von Alpha zu Google Drive bewegen. Jetzt hat sie zwölf Verantwortungen. Und jede neue Verantwortung macht alle anderen brüchiger."

Der Tech Lead lehnte sich zurück. „Okay. Was machen wir?"

Anakin: „Refactoring. Wir brauchen Interfaces. `IDocumentSource`. `IDocumentTarget`. Dependency Injection. Das volle Programm."

Obi-Wan: „Das dauert drei Sprints."

Tech Lead: „Wir haben nicht drei Sprints. API Epsilon kommt nächste Woche."

Qui-Gon: „Dann haben wir zwei Optionen. Wir stoppen jetzt und refactoren. Oder wir bauen weiter auf einem Fundament, das bereits bricht."

Stille.

Tech Lead: „Wir können nicht stoppen. Der Client zahlt. Der Vertrag läuft."

Qui-Gon: „Dann wird es schlimmer."

„Wie viel schlimmer?"

Qui-Gon öffnete seinen Laptop. Zeigte ein Diagramm. Ein anderes Projekt. Drei Jahre alt. 47 Azure Functions. Alle miteinander verbunden. Niemand wusste mehr, was wovon abhing.

„So schlimmer," sagte er.

Der Raum war still.

Tech Lead, schließlich: „Okay. Wir machen beides. Epsilon geht in die alte Function. Parallel bauen wir eine neue Architektur. Service Bus. Proper separation. Und dann migrieren wir."

Anakin: „Wann?"

„Sobald wir Zeit haben."

Qui-Gon schloss seinen Laptop. Stand auf. Ging zur Tür.

„Was ist los?" fragte der Tech Lead.

„Ich kenne diese Geschichte," sagte Qui-Gon. „Ich habe sie schon erlebt. ‚Sobald wir Zeit haben' heißt ‚niemals'."

„Du glaubst nicht, dass wir es schaffen?"

Qui-Gon drehte sich um. Seine Augen waren nicht wütend. Nur müde.

„Ich glaube, dass ihr es schaffen wollt. Aber ich habe gelernt: Die dunkle Seite gewinnt nicht, weil sie stärker ist. Sie gewinnt, weil wir zu beschäftigt sind, um sie zu stoppen."

Die Tür schloss sich.

## VII. Der innere Krieg

Anakin saß allein in seinem Büro. 23:47 Uhr. Die Anforderung für API Gamma lag vor ihm.

Er öffnete die `DmsUploader.cs`. 1,200 Zeilen. Beta war vor zwei Wochen gemerged. Es funktionierte.

Seine Finger schwebten über der Tastatur.

*Zwei Wege lagen vor ihm. Er sah sie beide.*

**Der erste Weg:** Drei Tage nehmen. Stoppen. Refactoren. Interfaces bauen. Alles sauber machen, bevor Gamma kommt.

Drei Tage Arbeit. Drei Tage, in denen er nichts liefert. Drei Tage, in denen der Product Owner fragt: „Warum dauert das so lang?"

Drei Tage, die sich anfühlen wie... Versagen. Wie Verschwendung. Wie das Eingeständnis: „Ich habe Alpha und Beta falsch gebaut."

*„Ich habe so viel Zeit investiert," dachte er. „850 Zeilen. Zwei Sprints. Es funktioniert. Es ist production-ready."*

*„Warum sollte ich das alles wegwerfen?"*

**Der zweite Weg:** Drei Stunden. Ein else-if. Copy-paste von Beta. Minimal change. Quick win.

Die Infrastruktur war da. Der Upload-Code war da. Die Tests waren da.

*„Wir haben doch schon..."*

Die Worte formten sich in seinem Kopf wie ein Mantra. Wie eine Rechtfertigung. Wie ein... Fluch.

*„Wiederverwendung. DRY. Effizienz."*

Seine Hand bewegte sich. Öffnete eine neue Branch: `feature/api-gamma-support`

Copy. Paste. Rename.

```csharp
} else if (source == "gamma") {
    doc = await FetchFromGamma(req);
    ...
}
```

Er drückte commit.

*Er spürte es. Dieses Gefühl. Wie ein Riss. Wie eine dunkle Stimme, die flüsterte: „Du hättest stoppen sollen."*

Aber der Commit war durch.

Und mit jedem Commit wurde es schwerer, zurückzugehen. Weil jedes else-if mehr Investment war. Mehr Sunk Cost. Mehr Grund, weiterzumachen.

*X wurde für X gebaut. Nicht für X+Y.*

*Aber er presste Y hinein. Dann Z. Dann W.*

*Und mit jedem Druck verformte sich X. Nicht sichtbar. Nicht sofort. Aber die Risse wuchsen. Im Inneren. Wo niemand sie sah.*

*Bis es zu spät war.*

---

*Kennst du diesen Moment? Sitzt du jetzt in deinem Büro? Zwei Wege vor dir?*

*Der eine: drei Tage, sauber, richtig.*

*Der andere: drei Stunden, schnell, „wir haben doch schon"?*

*Welchen Weg wählst du?*

*Und wenn du den zweiten wählst – wann merkst du, dass du nicht mehr zurückkommst?*

## VIII. Die Anatomie der Katastrophe

Drei Monate später. Der Tech Lead öffnete die `DmsUploader.cs`.

Er wollte verstehen. Wollte sehen, wo es schiefgelaufen war. Wann aus „einfach" „unmöglich" wurde.

Er scrollte nicht durch den Code. Er betrachtete nur die Struktur. Das Skelett. Die Form des Monsters.

```csharp
public class DmsUploader
{
    public async Task<IActionResult> Run(HttpRequest req)
    {
        // 1. Parse parameters (50 lines)
        string source = req.Query["source"];
        string target = req.Query["target"];
        bool validate = bool.Parse(req.Query["validate"] ?? "false");
        bool transform = bool.Parse(req.Query["transform"] ?? "false");
        
        // 2. Fetch document (400 lines)
        Document doc;
        if (source == "alpha") {
            // Alpha auth (50 lines)
            // Alpha fetch (80 lines)
        } else if (source == "beta") {
            // Beta auth (60 lines)
            // Beta fetch (80 lines)
        } else if (source == "gamma") {
            // Gamma auth (40 lines)
            // Gamma fetch (90 lines)
        } else if (source == "delta") {
            // Delta auth (70 lines)
            // Delta fetch (30 lines)
        }
        
        // 3. Validate (300 lines)
        if (validate) {
            // File size validation (50 lines)
            // Extension validation (40 lines)
            // Content validation (210 lines)
        }
        
        // 4. Transform (200 lines)
        if (transform) {
            // PDF transformation (150 lines)
            // Image transformation (50 lines)
        }
        
        // 5. Upload (500 lines)
        string uploadedId;
        if (target == "gdrive") {
            // Google Drive upload (180 lines)
        } else if (target == "onedrive") {
            // OneDrive upload (160 lines)
        } else if (target == "sharepoint") {
            // SharePoint upload (160 lines)
        }
        
        // 6. Patch back (650 lines)
        // This is where it gets insane
        if (source == "alpha" && target == "gdrive") {
            // 80 lines
        } else if (source == "alpha" && target == "onedrive") {
            // 90 lines
        } else if (source == "alpha" && target == "sharepoint") {
            // 85 lines
        } else if (source == "beta" && target == "gdrive") {
            // 75 lines
        } 
        // ... 12 combinations total
        
        return new OkResult();
    }
}
```

2,100 Zeilen. Eine Methode. Eine Datei.

Cognitive Complexity: 847.

Für Kontext: Eine Complexity über 15 gilt als „schwer zu warten". Über 25 als „sehr riskant".

847 ist... jenseits der Skala.

## IX. Die Lehren der Meister

### Yoda: Die Weisheit der Grenze

„Wiederverwendung gut ist. Aber Wiederverwendung ohne Grenze, zur dunklen Seite führt. Wissen musst du, wann zu teilen und wann zu trennen. Die Macht der Verantwortung, in der Klarheit liegt sie."

**Die Jedi-Wahrheit:**

DRY („Don't Repeat Yourself") ist ein gutes Prinzip. Aber es ist nicht das einzige Prinzip.

Es gibt etwas Wichtigeres: SRP (Single Responsibility Principle).

Wenn du Code wiederverwendest, der verschiedene Verantwortlichkeiten mischt, machst du nicht DRY. Du machst nur Mud.

Die Frage ist nie: „Können wir das wiederverwenden?"

Die Frage ist: „Sollten wir?"

### Obi-Wan: Der Moment des Nein

„Der schwierigste Moment im Leben eines Jedi ist nicht, wenn er kämpfen muss. Es ist, wenn er Nein sagen muss – zu einem Freund, zu einem Team, zu sich selbst."

**Die Lektion:**

Obi-Wan sah die PR. Er wusste, dass es falsch war. Aber er sagte nicht Nein.

Warum?

Weil Nein schwer ist. Weil Nein zu Diskussion führt. Weil Nein bedeutet, dass du der Bremser bist, der Dogmatiker, der Purist.

Aber hier ist die Wahrheit: Jetzt Nein zu sagen, spart später tausend Ja.

Wenn Obi-Wan bei API Beta Nein gesagt hätte – wenn er gesagt hätte: „Stopp. Lass uns refactoren, bevor wir weitermachen" – dann hätten sie drei Tage verloren.

Stattdessen verloren sie drei Monate.

Ein Nein am Anfang ist billiger als hundert Ja am Ende.

### Qui-Gon: Die Weitsicht

„Die Zukunft ist immer in Bewegung. Aber manche Pfade sind klarer als andere. Wenn du den Schatten siehst, bevor er da ist – warne. Auch wenn niemand hört."

**Die Lektion:**

Qui-Gon kannte dieses Drehbuch. Er hatte es schon erlebt.

Er hätte eskalieren können. Zum Tech Lead. Zum Architekten. Er hätte sagen können: „Stoppt. Jetzt. Oder es wird eine Katastrophe."

Aber er tat es nicht. Weil er müde war. Weil er dachte: „Vielleicht ist dieses Mal anders."

Es war nicht anders.

Die Lektion: Wenn du den Schatten siehst, rede nicht nur. Handle.

## X. Das vergilbte Notizbuch

Drei Jahre später. Der junge Padawan durchsuchte die Archive.

Er fand ein kleines Notizbuch. Verstaubt. Zwischen alten Ausdrucken und vergessenen USB-Sticks. Die Handschrift war zittrig. Kaffeeflecken auf den Seiten. Oder Tränen?

Der Name auf der ersten Seite: *Qui-Gon Jinn*

Darunter, in verblasster Tinte: *„Die Sätze, die den Fall ankündigen – geschrieben in der Nacht, als ich wusste, dass ich sie verloren hatte."*

Der Padawan schlug die erste Seite auf. Las.

---

*3:22 Uhr. Kann nicht schlafen. Habe heute Anakins PR gesehen. API Gamma. Er hat nicht gestoppt.*

*Ich kenne diese Sätze. Ich habe sie hundertmal gehört. In hundert gescheiterten Projekten.*

*Wenn du diese Sätze hörst – erkenne sie. Denn sie kündigen das Ende an.*

---

**Erster Fluch:**

*„Wir haben doch schon die Infrastruktur. Es ist nur ein Parameter mehr."*

Ein Parameter heute. Zehn Parameter morgen. Hundert if-Statements übermorgen.

Sie merken es nicht. Ein Parameter fühlt sich harmlos an. Aber Parameter akkumulieren. Wie Schulden. Mit Zinseszins.

---

**Zweiter Fluch:**

*„Es ist basically dasselbe. Nur ein bisschen anders."*

„Ein bisschen anders" ist der gefährlichste Satz in der Software-Entwicklung.

Weil „ein bisschen" sich summiert. Weil du denkst, es ist 90% gleich. Aber die 10% Unterschied sind es, die dich umbringen.

Wenn etwas „basically dasselbe" ist, aber „ein bisschen anders" – dann ist es nicht dasselbe. Es ist ein neues Problem. Und es verdient eine neue Lösung.

---

**Dritter Fluch:**

*„Refactoring können wir später machen. Erst liefern wir."*

Später ist nie.

Später ist, wenn Production brennt. Später ist, wenn der Tech Debt so hoch ist, dass niemand mehr weiß, wo anfangen. Später ist, wenn du sagst: „Wir müssen neu bauen."

Wenn du Refactoring später machst, machst du es nicht. Du lebst mit den Konsequenzen.

---

**Vierter Fluch:**

*„Die Datei ist schon 1,000 Zeilen. Was sind noch 200?"*

Boiling Frog.

Die Datei war mal 100 Zeilen. Dann 300. Dann 600. Dann 1,000.

Und bei jedem Schritt dachtest du: „Noch okay. Noch manageable."

Aber du merkst nicht, wann du vom Teich zum Ozean wechselst. Bis du ertrinkst.

---

**Fünfter Fluch:**

*Code Reviews, die zu „LGTM" werden, ohne echtes Lesen.*

Das ist das Ende.

Wenn niemand mehr den Mut hat, Nein zu sagen. Wenn niemand mehr das Review ernst nimmt. Wenn „LGTM" bedeutet: „Ich habe keine Zeit, das zu verstehen."

Dann ist das System bereits gebrochen. Nicht technisch. Kulturell.

---

**Sechster Fluch:**

*„Wir brauchen keinen Architekten. Das ist Over-Engineering."*

Sie verwechseln Nachdenken mit Überdenken.

Architektur ist nicht Over-Engineering. Architektur ist: Dreißig Minuten nehmen, bevor du drei Monate verlierst.

Aber sie sehen die dreißig Minuten. Sie sehen nicht die drei Monate.

Bis es zu spät ist.

---

*Meine Hand schmerzt. Draußen wird es hell. Ich habe diese Sätze aufgeschrieben, damit jemand, irgendwann, sie liest.*

*Damit jemand erkennt: Ich habe diese Sätze auch gehört. Und ich habe nicht gehandelt.*

*Wirst du handeln?*

*Oder wirst du auch nur aufschreiben, was du hättest tun sollen?*

---

Der junge Padawan schloss das Notizbuch.

Seine Hände zitterten.

*Er hatte diese Sätze heute Morgen gehört. In seinem eigenen Stand-Up.*

*„Wir haben doch schon..."*

*Was würde er tun?*

## XI. Die Skizze, die niemand las

Drei Jahre später. Der alte Architekt durchsuchte die Git-History. Nicht nach Code. Nach Kommentaren. Nach gelöschten Branches. Nach den Schatten dessen, was hätte sein können.

Dann fand er es.

Eine Branch. Erstellt in jener Nacht, als API Beta gemerged wurde. Erstellt um 2:17 Uhr. Von Obi-Wan.

Branch-Name: `refactor/interface-based-architecture`

Er klickte darauf.

Ein einziger Commit. Nie gepusht. Nie geteilt. Nie diskutiert.

Commit-Message: *„Was wir hätten tun sollen – für wenn jemand später fragt"*

Der Architekt öffnete die Dateien.

---

### Die ungelesene Weisheit

**Datei:** `ARCHITECTURE_PROPOSAL.md`

```text
Geschrieben: 2:17 AM, in der Nacht nach API Beta

Ich sehe, wohin das führt.

Anakin hat Beta in die bestehende Function gepackt. Ein else-if. 
Es funktioniert. Aber es ist der Anfang vom Ende.

Hier ist, was wir tun sollten. Nicht morgen. Nicht „wenn wir Zeit haben". 
JETZT. Bevor Gamma kommt.

Drei Tage. Das ist alles, was wir brauchen.
```

**Tag 1: Die Interfaces**

```csharp
// IDocumentSource.cs
public interface IDocumentSource
{
    Task<Document> FetchAsync(string documentId);
    Task PatchLinkAsync(string documentId, string uploadedLink);
}

// IDocumentTarget.cs
public interface IDocumentTarget
{
    Task<string> UploadAsync(Document document);
}
```

*Zwei Interfaces. Zehn Zeilen. Eine Stunde Arbeit.*

*Aber sie definieren Grenzen. Sie sagen: „Hier endet eine Quelle. Hier beginnt ein Ziel."*

*Grenzen sind nicht Einschränkungen. Grenzen sind Klarheit.*

---

**Tag 2: Die Implementierungen**

```csharp
// ApiAlphaSource.cs
public class ApiAlphaSource : IDocumentSource 
{
    // 80 Zeilen, fokussiert, klar
}

// ApiBetaSource.cs  
public class ApiBetaSource : IDocumentSource 
{
    // 85 Zeilen, OAuth2, unabhängig
}

// GoogleDriveTarget.cs
public class GoogleDriveTarget : IDocumentTarget 
{
    // 120 Zeilen, nur Upload, nichts mehr
}
```

*Jede Klasse: eine Verantwortung. Eine Quelle. Ein Ziel. Ein Grund zu existieren.*

*Wenn Gamma kommt: eine neue Klasse. Nicht ein neues else-if.*

---

**Tag 3: Der Core**

```csharp
public class DmsUploader
{
    private readonly Dictionary<string, IDocumentSource> _sources;
    private readonly Dictionary<string, IDocumentTarget> _targets;
    
    public async Task<IActionResult> Run(HttpRequest req)
    {
        var source = _sources[req.Query["source"]];
        var target = _targets[req.Query["target"]];
        
        var doc = await source.FetchAsync(req.Query["docId"]);
        var link = await target.UploadAsync(doc);
        await source.PatchLinkAsync(req.Query["docId"], link);
        
        return new OkResult();
    }
}
```

*40 Zeilen. Das ist alles.*

*Keine if-Statements. Keine Branching-Logik. Keine Complexity.*

*Wenn API Gamma kommt: eine neue Klasse registrieren. Null Zeilen im Core.*

*Wenn API Zeta kommt: dasselbe.*

*Wenn wir zwölf APIs haben: der Core bleibt 40 Zeilen.*

---

```text
Drei Tage.

Aber ich weiß, was passieren wird.

Ich werde Anakin diesen Branch zeigen. Er wird sagen: 
„Das ist Over-Engineering. Beta funktioniert doch."

Ich werde zum Tech Lead gehen. Er wird sagen: 
„Wir haben keine drei Tage. Gamma kommt nächste Woche."

Und so wird diese Branch hier liegen. Ungemerged. Ungelesen. Vergessen.

Bis jemand, drei Jahre später, sie findet und denkt: 
„Warum haben sie das nicht gemacht?"

Wenn du das liest – wenn du diese Zeilen findest – dann weißt du:

Wir hatten die Lösung. Wir sahen den Weg.

Wir gingen ihn nur nicht.

Weil drei Tage zu lang waren. Weil drei Stunden attraktiver waren.

Weil „es funktioniert doch" immer lauter ist als „es wird brechen".

— Obi-Wan, 2:47 AM
```

---

Der alte Architekt schloss die Datei.

Seine Hände zitterten.

*Die Lösung war da. Die ganze Zeit. Drei Tage. Vierzig Zeilen.*

*Aber niemand las sie. Bis es zu spät war.*

*Bis 2,100 Zeilen da waren. Bis sechs Monate Rewrite nötig waren. Bis das Doppelte des Budgets verbrannt war.*

*Alles, weil drei Tage zu lang fühlten.*

## Epilog: Der Point of No Return

Sechs Monate nach dem ersten Commit.

Das Team saß in einem War Room. Production war instabil. Ein Bug in der Delta-API-Integration. Niemand wusste, wo genau.

„Wer hat Delta implementiert?" fragte der Tech Lead.

„Ich," sagte Anakin. „Vor drei Monaten."

„Kannst du den Bug finden?"

Anakin öffnete die `DmsUploader.cs`. Scrollte. Suchte. Die Methode war 2,400 Zeilen lang.

„Ich... ich weiß nicht mehr, wo Delta anfängt und wo es endet."

Der Tech Lead drehte sich zu Qui-Gon. „Du hast gewarnt."

„Ja."

„Was machen wir jetzt?"

Qui-Gon seufzte. Öffnete seinen Laptop. Zeigte ein Dokument.

**Titel:** „DmsUploader v2 – Service Bus Architecture"

„Wir bauen es neu. Von Grund auf. Aber diesmal richtig."

„Wie lange?"

„Drei Monate. Vielleicht vier."

„Und in der Zwischenzeit?"

„In der Zwischenzeit," sagte Qui-Gon, „beten wir, dass Production nicht brennt."

Eine Woche später brannte Production.

Der Bug war in Zeile 1,847. Ein Tippfehler. Ein `&&` statt `||`.

Er war drei Monate alt. Niemand hatte ihn bemerkt, weil niemand mehr den Code verstand.

Das Team begann mit dem Rewrite.

Es dauerte sechs Monate.

Es kostete das Doppelte des ursprünglichen Budgets.

Und es hätte verhindert werden können – mit drei Tagen und einem Nein.

---

**Nächstes Kapitel:** „Die Clone Wars beginnen" – Wie eine Function zu einer Function App zu... was genau?

## Anhang: Der Brief, den Anakin schrieb

Drei Jahre später, als das Projekt endlich stabilisiert war, schrieb Anakin einen Blog-Post. Er veröffentlichte ihn nie. Aber er behielt ihn in seinem persönlichen Wiki.

**Titel:** „I Built a Frankenstein, and I'm Sorry"

```text
Wenn ich zurückblicke, sehe ich den Moment.

Es war nicht bei API Epsilon. Nicht bei OneDrive. Es war bei API Beta.

Ich hätte Nein sagen sollen.

Nicht Nein zu dem Feature. Nein zu dem Weg.

Ich hätte sagen sollen: „Gebt mir drei Tage. Nicht um Nein zu sagen, 
sondern um Ja richtig zu sagen."

Aber ich dachte, ich sei schnell. Ich dachte, ich sei effizient. 
Ich dachte, Wiederverwendung sei immer gut.

Ich lag falsch.

Wiederverwendung ohne Architektur ist keine Effizienz. 
Es ist nur verzögerte Komplexität.

Und Komplexität verzinst sich. Mit Compound-Interest.

Wenn du dieses liest und du stehst vor einer „Wir haben doch schon..."-
Entscheidung: Halt inne.

Frag nicht: „Können wir das wiederverwenden?"

Frag: „Sollten wir? Und was ist die Grenze?"

Denn die Grenze kommt. Immer.

Die Frage ist nur: Definierst du sie? Oder lässt du das Chaos sie definieren?
```

Der Post endete dort.

Ein junger Entwickler fand ihn Jahre später. Er stand gerade vor der Entscheidung: API Beta in eine bestehende Function einbauen oder neu strukturieren.

Er las Anakin's Worte.

Dann öffnete er eine neue Branch: `refactor/introduce-interfaces`

Es dauerte drei Tage.

Es rettete drei Jahre.

---

*Du sitzt jetzt vor deinem Screen. Dein Projekt läuft. Es ist stabil. Du hast gerade einen Sieg gefeiert.*

*Und jetzt kommt die nächste Anforderung. Klein. Harmlos. „Should be quick, right?"*

*Du hast die Infrastruktur. Du hast das Repo. Du hast die Methode.*

*„Wir haben doch schon..."*

*Die Worte formen sich in deinem Kopf.*

*Was wirst du sagen?*

*„Nur ein Parameter mehr"?*

*Oder „Stopp. Lasst uns drei Tage nehmen und es richtig machen"?*

---

„Der zweite Fehler ist immer teurer als der erste. Aber der erste Fehler ist immer vermeidbarer. Die Frage ist nur: Wann hörst du auf zu bauen – und beginnst zu denken?"

— Der alte Architekt, Survivor der Code-Kriege
