# Kapitel 2: Wir haben doch schon...

## Prolog: Die Geister der Entscheidungen

„Eine Entscheidung ist wie ein Stein, den du ins Wasser wirfst. Die Wellen siehst du sofort. Die Veränderung des Flussbetts – die siehst du erst Jahre später, wenn es zu spät ist, den Stein zurückzuholen.“

— Aus den Chroniken des Architektenordens

Der alte Architekt scrollte durch das Git-Log. Der junge Schüler stand neben ihm, las über seine Schulter.

```text
commit a3f8e92
Author: Arik <arik@rebels.dev>
Date: Mon Nov 4 09:23:45 2024

feat: Added API Beta support
- Added if/else for auth switching
- Reused existing upload logic
- Quick win! 🚀
```

"Siehst du?" sagte der Alte. "Hier. Commit a3f8e92. Das ist der Moment."

"Der Moment wovon?"

"Der Moment, in dem aus einem eleganten Service ein Monolith wurde. Der Moment, in dem ‚Wiederverwendung' zu ‚Verkrüppelung' wurde."

Er klickte auf den Commit. Das Diff explodierte über den Bildschirm. 400 Zeilen geändert. In einer einzigen Datei. In einer einzigen Methode.

Der junge Schüler starrte darauf. "Aber... das ist doch nur eine weitere API? Das ist doch Wiederverwendung? Das ist doch gut?"

Der Alte lachte. Es war kein fröhliches Lachen.

"Wiederverwendung," sagte er langsam, "ist wie Feuer. In einem Ofen wärmt es dein Haus. Außer Kontrolle brennt es dein Haus nieder."

Er deutete auf eine Zeile im Code:

```csharp
if (source == "ApiAlpha") {
    // Alpha logic
} else if (source == "ApiBeta") {
    // Beta logic - almost same but different auth
} else if ...
```

"Das ist kein Feuer im Ofen," sagte er. "Das ist Feuer auf dem Teppich."

---

**Drei Monate nach dem ersten Commit...**

## I. Die Nachricht

Montagmorgen, 9:03 Uhr.

Arik Dane öffnete seinen Laptop. Kaffee in der Hand. Gute Laune. Das Wochenende war schön gewesen. Keine Production-Alerts. Keine Midnight-Deployments.

Dann sah er die Slack-Nachricht. Vom Freitag, 16:47 Uhr.

**Product Owner:** "Hey team! 🎉 Great work on the uploader. The client is super happy. Quick question - can we add API Beta? It's basically the same as Alpha, just different auth. Should be quick, right?"

Arik las es zweimal. "Basically the same" war gut. "Quick" war besser.

Er öffnete die `DmsUploader.cs`. 750 Zeilen. Alles in der Run-Methode. Aber es funktionierte. Drei Wochen Production, null Bugs.

**Arik (im Chat):** "Sure! We already have the infrastructure. It's just a new client. I'll add a parameter for the source type. Done by Wednesday 👍"

**Product Owner:** "You're a star! ⭐"

Arik lehnte sich zurück. Fühlte sich gut. Das war, warum er Entwickler war. Probleme lösen. Schnell. Effizient.

Er öffnete eine neue Branch: `feature/api-beta-support`

Die Finger flogen über die Tastatur.

## II. Der erste Riss

Die Logik war einfach. Fast zu einfach.

API Alpha und API Beta waren identisch. Bis auf die Authentifizierung. Alpha nutzte Basic Auth. Beta nutzte OAuth2.

"Kein Problem," murmelte Arik. "Ein Parameter. Ein if-Statement."

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

"Perfekt," sagte er zu sich selbst.

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

Sora Nyra sah die PR-Notification. Klickte sie an. Sah das Diff.

400 Zeilen geändert. Alles in `DmsUploader.cs`.

Sie scrollte durch den Code. Sah die if-Statements. Sah die Duplikation in den Auth-Methoden. Sah die Duplikation in den Patch-Methoden.

Ihre Finger schwebten über dem Kommentar-Feld.

**Der Kommentar, den sie schreiben wollte:**

"This is starting to smell. We now have two sources, and the logic is branching. What happens when API Gamma comes? When OneDrive replaces Drive? This is the moment to stop and refactor to interfaces. Let me sketch what that would look like..."

**Der Kommentar, den sie tatsächlich schrieb:**

Der Linter – unbarmherzig, niemals müde – nickte stumm.

„LGTM! ✅ Nice reuse of the upload logic.“

**Warum?**

Weil es Montag, 14:30 Uhr war. Weil sie selbst drei andere Tickets hatte. Weil ein "Bitte refactor zuerst"-Kommentar zu einem Tag Diskussion führen würde. Weil Arik gute Argumente haben würde: "YAGNI. It's just two sources. When Gamma comes, then we refactor."

Und die Argumente waren nicht falsch.

Sie waren nur... kurzsichtig.

Sora klickte "Approve".

## IV. Die zweite Stimme

Qion Varr sah die PR eine Stunde später.

Er öffnete sie nicht. Er brauchte das Diff nicht zu sehen. Er kannte dieses Drehbuch.

Er ging zu Arik's Desk.

"Hey," sagte er. Leise. Fast entschuldigend.

Arik drehte sich um. "Hey! Hast du meine PR gesehen? Clean, oder? Minimal change, maximum reuse."

"Ja," sagte Qion. "Ich habe eine Frage."

"Shoot."

"Was passiert, wenn API Gamma kommt?"

Arik blinzelte. "Dann füge ich ein weiteres else-if hinzu?"

"Und wenn OneDrive das neue Ziel wird?"

"Dann... wrappen wir den Drive-Upload in eine Abstraktion?"

"Und wenn Delta kommt? Epsilon? Was ist die Grenze?"

Arik lehnte sich zurück. Sein Lächeln wurde schmaler. "Was willst du sagen?"

Qion seufzte. "Ich will sagen: Dies ist der Moment, an dem wir stoppen sollten. Die Function macht noch eine Sache: Dokumente von Quelle zu Ziel bewegen. Aber sie macht es jetzt für zwei Quellen. Bald drei. Dann vier. Wo ist die Grenze?"

"Die Grenze," sagte Arik, und seine Stimme hatte einen Hauch von Gereiztheit, "ist, wenn es nicht mehr wartbar ist. Und das ist es noch. Two sources, same logic. Das ist Wiederverwendung. Das ist DRY."

"DRY," sagte Qion langsam, "bedeutet 'Don't Repeat Yourself'. Es bedeutet nicht 'Stopf alles in eine Datei'."

Die Luft zwischen ihnen wurde kälter.

"Hast du die PR geblockt?" fragte Arik.

"Nein," sagte Qion. "Ich wollte nur fragen."

Er ging zurück zu seinem Desk.

Arik starrte ihm nach. Dann, leise, zu sich selbst: "Old school. Sie verstehen moderne Entwicklung nicht. Agil heißt liefern, nicht endlos diskutieren."

Er klickte "Merge".

## V. Der Fall beschleunigt sich

**Drei Wochen später.**

**Product Owner:** "API Beta is a hit! Client wants API Gamma now. Also different auth (API Key), but otherwise same. Can we add it?"

**Arik:** "On it 👍"

Ein weiteres else-if. 200 weitere Zeilen.

**Vier Wochen später.**

**Product Owner:** "Quick one - can we support OneDrive instead of Google Drive for some clients? They want to choose the target."

**Arik:** "Ugh. That's... more complicated. But doable. Give me a few days."

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

**Sechs Wochen später.**

**Product Owner:** "They want validation before upload. Just check file size and extension. Easy, right?"

**Arik:** "..."

Er starrte auf den Code. 1,400 Zeilen. Verschachtelte if-Statements. Er brauchte drei Minuten, um zu verstehen, wo er das Validation-Logic einfügen sollte.

Vor drei Monaten hätte er die gesamte Datei in fünf Sekunden verstanden.

Er öffnete Slack.

**Arik (an Sora, privat):** "Help. I think we need to refactor."

**Sora:** "Now? With all the Production traffic?"

**Arik:** "..."

## VI. Das Meeting, das alles änderte

Eine Woche später. Der Tech Lead berief ein "Architecture Review" ein.

Das Team saß im Meetingraum. Auf dem Bildschirm: Die DmsUploader.cs.

**Tech Lead:** "Wir haben ein Problem. Die Uploader-Function ist nicht mehr wartbar. Jede Änderung dauert Tage. Bugs sind schwer zu finden. Wer kann mir erklären, wie wir hier gelandet sind?"

Stille.

Dann sprach Qion. Ruhig. Ohne Vorwurf.

"Ich kann es erklären. Es begann bei Commit a3f8e92. API Beta. Wir haben eine Entscheidung getroffen: Wiederverwendung über Verantwortlichkeit. DRY über SRP. Geschwindigkeit über Architektur."

"War die Entscheidung falsch?" fragte der Tech Lead.

"Nein," sagte Qion. "Die Entscheidung war... menschlich. Aber sie hatte Konsequenzen. Und wir haben die Konsequenzen nicht bedacht."

"Was hätten wir tun sollen?"

Qion öffnete seinen Laptop. Zeigte ein Diagramm.

"Hier. Das ist, was wir hätten tun sollen. Bei API Beta. Nicht bei Epsilon."

## VII. Das Diagramm, das zu spät kam

Das Diagramm zeigte eine einfache Architektur:

```text
┌─────────────────────────────────────────────┐
│           DmsUploader (Orchestrator)        │
│  - Nimmt source/target Parameter            │
│  - Delegiert an Interfaces                  │
│  - Keine Business Logic                     │
└─────────────────────────────────────────────┘
                      │
        ┌─────────────┴─────────────┐
        │                           │
        ▼                           ▼
┌──────────────────┐      ┌──────────────────┐
│ IDocumentSource  │      │ IDocumentTarget  │
└──────────────────┘      └──────────────────┘
        │                           │
    ┌───┴───┬───────┬────────┐     │
    ▼       ▼       ▼        ▼     ▼
 Alpha   Beta   Gamma   Delta  GDrive / OneDrive
```

"Jede Quelle," erklärte Qion, "ist eine eigene Klasse. Jedes Ziel ist eine eigene Klasse. Die Function orchestriert nur. Sie mischt keine Verantwortlichkeiten."

**Der Tech Lead nickte.** "Wie lange würde das dauern? Jetzt zu refactoren?"

Qion seufzte. "Vier Wochen. Vielleicht sechs. Wir müssen es neu schreiben. Die bestehende Logik ist zu verwoben."

"Und wenn wir es nicht machen?"

"Dann," sagte Qion, "wird jede neue Änderung länger dauern. Jeder Bug schwerer zu finden. Bis das System zusammenbricht."

Der Tech Lead sah zur Decke. Dann zu Arik.

"Arik. Du hast das gebaut. Was denkst du?"

Arik schluckte. "Ich... ich glaube, Qion hat recht. Ich habe einen Fehler gemacht. Nicht bei Epsilon. Bei Beta. Ich hätte auf ihn hören sollen."

## VIII. Der Code, der zu einem Monster wurde

Sechs Wochen später hatte sich die Situation nicht verbessert. Sie hatte sich verschlimmert.

API Delta. API Epsilon. Zeta. SharePoint als neues Ziel. Validation Rules. Retry Logic. Logging.

Die Run-Methode sah aus wie ein Kriegsschauplatz:

```csharp
public async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    ILogger log)
{
    try {
        string source = req.Query["source"];
        string target = req.Query["target"];
        bool validate = bool.Parse(req.Query["validate"] ?? "false");
        int retries = int.Parse(req.Query["retries"] ?? "3");
        
        log.LogInformation($"Processing: {source} -> {target}");
        
        Document doc;
        
        // FETCH LOGIC (400 lines of if/else)
        if (source == "alpha") {
            if (validate) {
                if (!await ValidateAlphaAuth(req)) {
                    log.LogError("Alpha auth failed");
                    return new UnauthorizedResult();
                }
            }
            doc = await FetchFromAlpha(req);
            if (doc == null) {
                if (retries > 0) {
                    await Task.Delay(1000);
                    // Retry logic...
                }
            }
        } else if (source == "beta") {
            // 300 lines...
        } else if (source == "gamma") {
            // 250 lines...
        } else if (source == "delta") {
            // 280 lines...
        } else if (source == "epsilon") {
            // 290 lines...
        } else if (source == "zeta") {
            // 300 lines...
        } else {
            log.LogError($"Unknown source: {source}");
            return new BadRequestResult();
        }
        
        // VALIDATION LOGIC (200 lines)
        if (validate) {
            if (doc.Size > MAX_SIZE) {
                log.LogError("File too large");
                return new BadRequestResult();
            }
            if (!ALLOWED_EXTENSIONS.Contains(doc.Extension)) {
                log.LogError("Invalid extension");
                return new BadRequestResult();
            }
            // More validation...
        }
        
        // UPLOAD LOGIC (500 lines of if/else)
        string uploadedLink;
        if (target == "gdrive") {
            if (validate) {
                // Pre-upload validation...
            }
            uploadedLink = await UploadToGoogleDrive(doc);
            if (uploadedLink == null && retries > 0) {
                // Retry logic...
            }
        } else if (target == "onedrive") {
            // 350 lines...
        } else if (target == "sharepoint") {
            // 400 lines...
        } else {
            log.LogError($"Unknown target: {target}");
            return new BadRequestResult();
        }
        
        // PATCH LOGIC (600 lines of nested if/else)
        if (source == "alpha" && target == "gdrive") {
            await PatchToAlpha(doc.Id, uploadedLink);
        } else if (source == "alpha" && target == "onedrive") {
            await PatchToAlphaOneDrive(doc.Id, uploadedLink);
        } else if (source == "alpha" && target == "sharepoint") {
            await PatchToAlphaSharepoint(doc.Id, uploadedLink);
        } else if (source == "beta" && target == "gdrive") {
            // ...
        } else if (source == "beta" && target == "onedrive") {
            // ...
        }
        // ... 50 more combinations ...
        
        log.LogInformation("Success");
        return new OkResult();
    }
    catch (Exception ex) {
        log.LogError(ex, "Failed to process document");
        return new StatusCodeResult(500);
    }
}
```

**2,100 Zeilen. Eine Methode. Eine Datei.**

**Cognitive Complexity: 847.**

Für Kontext: Eine Complexity über 15 gilt als "schwer zu warten". Über 25 als "sehr riskant".

847 ist... jenseits der Skala.

## IX. Die Lehren der Meister

### Die Weisheit der Grenze

**Aus den Chroniken des Architektenordens:**

"Wiederverwendung gut ist. Aber Wiederverwendung ohne Grenze, zur dunklen Seite führt. Wissen musst du, wann zu teilen und wann zu trennen. Die Macht der Verantwortlichkeit, in der Klarheit liegt sie."

**Die Wahrheit:**

DRY (Don't Repeat Yourself) ist ein gutes Prinzip. Aber es ist nicht das einzige Prinzip.

Es gibt etwas Wichtigeres: **SRP (Single Responsibility Principle)**.

Wenn du Code wiederverwendest, der verschiedene Verantwortlichkeiten mischt, machst du nicht DRY. Du machst nur **Mud**.

Die Frage ist nie: *"Können wir das wiederverwenden?"*

Die Frage ist: *"Sollten wir?"*

### Der Moment des Nein

**Sora Nyra, Jahre später in einem Interview:**

"Der schwierigste Moment im Leben eines Architekten ist nicht, wenn er kämpfen muss. Es ist, wenn er Nein sagen muss—zu einem Freund, zu einem Team, zu sich selbst."

**Die Lektion:**

Sora sah die PR. Sie wusste, dass es falsch war. Aber sie sagte nicht Nein.

**Warum?**

Weil Nein schwer ist. Weil Nein zu Diskussion führt. Weil Nein bedeutet, dass du der Bremser bist, der Dogmatiker, der Purist.

Aber hier ist die Wahrheit: **Jetzt Nein zu sagen, spart später tausend Ja.**

Wenn Sora bei API Beta Nein gesagt hätte—wenn sie gesagt hätte: "Stopp. Lass uns refactoren, bevor wir weitermachen"—dann hätten sie drei Tage verloren.

Stattdessen verloren sie drei Monate.

**Ein Nein am Anfang ist billiger als hundert Ja am Ende.**

### Die Weitsicht

**Qion Varr, in seinem persönlichen Journal:**

"Die Zukunft ist immer in Bewegung. Aber manche Pfade sind klarer als andere. Wenn du den Schatten siehst, bevor er da ist—warne. Auch wenn niemand hört."

**Die Lektion:**

Qion kannte dieses Drehbuch. Er hatte es schon erlebt.

Er hätte eskalieren können. Zum Tech Lead. Zur Architektur-Gilde. Er hätte sagen können: "Stoppt. Jetzt. Oder es wird eine Katastrophe."

Aber er tat es nicht. Weil er müde war. Weil er dachte: *"Vielleicht ist dieses Mal anders."*

Es war nicht anders.

**Die Lektion: Wenn du den Schatten siehst, rede nicht nur. Handle.**

## X. Die Warnsignale

Das Notizbuch des Architekten lag offen. Kaffeeflecken. Oder Tränen. Am Rand stand, krakelig: „Wenn du diese Zeichen siehst, halt an.“

### 🔴 Erkenne den Fall

Hier sind die Momente, in denen "Wir haben doch schon..." zur dunklen Seite wird:

⚠️ **"Wir haben doch schon die Infrastruktur. Es ist nur ein Parameter mehr."**

Ein Parameter heute. Zehn Parameter morgen. Hundert if-Statements übermorgen.

⚠️ **"Es ist basically dasselbe. Nur ein bisschen anders."**

"Ein bisschen anders" ist der gefährlichste Satz. Weil "ein bisschen" sich akkumuliert.

⚠️ **"Refactoring können wir später machen. Erst liefern wir."**

Später ist nie. Später ist, wenn Production brennt und niemand den Code mehr versteht.

⚠️ **"Die Datei ist schon 1,000 Zeilen. Was sind noch 200?"**

Boiling Frog. Du merkst nicht, wann du vom Teich zum Ozean wechselst.

⚠️ **Code Reviews werden zu "LGTM" ohne echtes Lesen.**

Wenn niemand mehr den Mut hat, Nein zu sagen, ist das System bereits gebrochen.

⚠️ **"Wir brauchen keinen Architekten. Das ist Over-Engineering."**

Architektur ist nicht Over-Engineering. Architektur ist: **Nachdenken, bevor du bauest.**

Er schlug das Notizbuch zu. Die Seite blieb im Kopf des Lesers offen.

## XI. Die Rettungsstrategie (Die zu spät kam)

Hier ist, was das Team hätte tun sollen. Nicht bei API Epsilon. **Bei API Beta.**

### Der Moment des Stopps

Bei der API Beta PR hätte Sora schreiben sollen:

"Stop. Ich sehe, wohin das führt. Lass uns zwei Tage nehmen und es richtig machen. Hier ist der Plan:"

**Tag 1: Interfaces definieren**

```csharp
public interface IDocumentSource
{
    Task<Document> FetchAsync(string documentId);
    Task PatchLinkAsync(string documentId, string uploadedLink);
}

public interface IDocumentTarget
{
    Task<string> UploadAsync(Document document);
}
```

**Tag 2: Implementierungen**

```csharp
public class ApiAlphaSource : IDocumentSource { ... }
public class ApiBetaSource : IDocumentSource { ... }

public class GoogleDriveTarget : IDocumentTarget { ... }
```

**Tag 3: Refactor the Function**

```csharp
public class DmsUploader
{
    private readonly Dictionary<string, IDocumentSource> _sources;
    private readonly Dictionary<string, IDocumentTarget> _targets;
    
    public async Task<IActionResult> Run(HttpRequest req)
    {
        string sourceType = req.Query["source"];
        string targetType = req.Query["target"];
        
        var source = _sources[sourceType];
        var target = _targets[targetType];
        
        var doc = await source.FetchAsync(req.Query["docId"]);
        var uploadedLink = await target.UploadAsync(doc);
        await source.PatchLinkAsync(req.Query["docId"], uploadedLink);
        
        return new OkResult();
    }
}
```

**Drei Tage. 40 Zeilen statt 2,100.**

Und wenn API Gamma kommt? Eine neue Klasse. 50 Zeilen. **Null Änderungen am Core.**

Aber es passierte nicht.

Weil drei Tage zu lang fühlten. Weil drei Stunden attraktiver waren.

## Epilog: Der Point of No Return

Sechs Monate nach dem ersten Commit.

Das Team saß in einem War Room. Production war instabil. Ein Bug in der Delta-API-Integration. Niemand wusste, wo genau.

"Wer hat Delta implementiert?" fragte der Tech Lead.

"Ich," sagte Arik. "Vor drei Monaten."

"Kannst du den Bug finden?"

Arik öffnete die `DmsUploader.cs`. Scrollte. Suchte. Die Methode war 2,400 Zeilen lang.

"Ich... ich weiß nicht mehr, wo Delta anfängt und wo es endet."

Der Tech Lead drehte sich zu Qion. "Du hast gewarnt."

"Ja."

"Was machen wir jetzt?"

Qion seufzte. Öffnete seinen Laptop. Zeigte ein Dokument.

**Titel: "DmsUploader v2 - Service Bus Architecture"**

"Wir bauen es neu. Von Grund auf. Aber diesmal richtig."

"Wie lange?"

"Drei Monate. Vielleicht vier."

"Und in der Zwischenzeit?"

"In der Zwischenzeit," sagte Qion, "beten wir, dass Production nicht brennt."

---

Eine Woche später brannte Production.

Der Bug war in Zeile 1,847. Ein Tippfehler. Ein `&&` statt `||`.

Er war drei Monate alt. Niemand hatte ihn bemerkt, weil niemand mehr den Code verstand.

Das Team begann mit dem Rewrite.

Es dauerte sechs Monate.

Es kostete das Doppelte des ursprünglichen Budgets.

Und es hätte verhindert werden können—**mit drei Tagen und einem Nein.**

---

## Anhang: Der Brief, den Arik schrieb

Drei Jahre später, als das Projekt endlich stabilisiert war, schrieb Arik einen Blog-Post. Er veröffentlichte ihn nie. Aber er behielt ihn in seinem persönlichen Wiki.

**Titel: "I Built a Frankenstein, and I'm Sorry"**

"Wenn ich zurückblicke, sehe ich den Moment.

Es war nicht bei API Epsilon. Nicht bei OneDrive. Es war bei API Beta.

Ich hätte Nein sagen sollen.

Nicht Nein zu dem Feature. Nein zu dem Weg.

Ich hätte sagen sollen: 'Gebt mir drei Tage. Nicht um Nein zu sagen, sondern um Ja richtig zu sagen.'

Aber ich dachte, ich sei schnell. Ich dachte, ich sei effizient. Ich dachte, Wiederverwendung sei immer gut.

Ich lag falsch.

**Wiederverwendung ohne Architektur ist keine Effizienz. Es ist nur verzögerte Komplexität.**

Und Komplexität verzinst sich. Mit Compound-Interest.

Wenn du dieses liest und du stehst vor einer 'Wir haben doch schon...'-Entscheidung: **Halt inne.**

Frag nicht: *'Können wir das wiederverwenden?'*

Frag: *'Sollten wir? Und was ist die Grenze?'*

Denn die Grenze kommt. Immer.

Die Frage ist nur: Definierst du sie? Oder lässt du das Chaos sie definieren?"

---

Der Post endete dort.

Ein junger Entwickler fand ihn Jahre später. Er stand gerade vor der Entscheidung: API Beta in eine bestehende Function einbauen oder neu strukturieren.

Er las Arik's Worte.

Dann öffnete er eine neue Branch: `refactor/introduce-interfaces`

Es dauerte drei Tage.

Es rettete drei Jahre.

---

> "Der zweite Fehler ist immer teurer als der erste. Aber der erste Fehler ist immer vermeidbarer. Die Frage ist nur: Wann hörst du auf zu bauen—und beginnst zu denken?"
>
> — Aus den Chroniken des Architektenordens
>