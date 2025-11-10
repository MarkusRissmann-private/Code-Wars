# Kapitel 9: Die Event-Storm

## Prolog: Das Flüstern vor dem Sturm

*„Es gibt drei Stufen des Leidens in der verteilten Architektur. Erste Stufe: Synchrone Kopplung – du merkst sofort, dass etwas falsch ist. Zweite Stufe: Asynchrone Entkopplung – du fühlst dich sicher. Dritte Stufe: Message‑Overwhelm – du merkst erst, wenn es zu spät ist.“*

– Aus den Chroniken des Architektenordens

---

Der alte Architekt des Architektenordens öffnete ein Monitoring‑Dashboard. Timestamp: Vor drei Jahren. Ein anderes Projekt.

```text
╔════════════════════════════════════════════════╗
║   SERVICE BUS METRICS - PROJECT HORIZON        ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Messages/sec: 847                             ║
║  Queue Depth: 23                               ║
║  Processing Time: 1.2 sec avg                  ║
║  Dead Letter Queue: 3 messages                 ║
║                                                 ║
║  Status: ✅ HEALTHY                             ║
║  Last Updated: 2022-03-15 14:23:45             ║
╚════════════════════════════════════════════════╝
```

„Das“, sagte der Alte, „war zwei Wochen vor dem Zusammenbruch.“

Der junge Schüler las die Zahlen. „Aber ... das sieht perfekt aus. Grün. Stabil.“

„Ja.“ Der Alte scrollte weiter. Zwei Wochen später.

```text
╔════════════════════════════════════════════════╗
║   SERVICE BUS METRICS - PROJECT HORIZON        ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Messages/sec: 94,231                          ║
║  Queue Depth: 1,847,291                        ║
║  Processing Time: TIMEOUT                      ║
║  Dead Letter Queue: 847,234 messages           ║
║                                                 ║
║  Status: 🔥 CRITICAL FAILURE                   ║
║  Last Updated: 2022-03-29 03:47:12             ║
╚════════════════════════════════════════════════╝
```

„Von 847 auf 94.000 Messages pro Sekunde“, flüsterte der Schüler. „In zwei Wochen.“

„Nicht in zwei Wochen“, korrigierte der Alte. „In zwei Stunden.“

„Das ist – wie ist das möglich?“

Der Alte zeigte auf eine einzelne Zeile im Code:

```csharp
// DocumentProcessor.cs - Line 89
await _serviceBus.PublishAsync(new DocumentProcessedEvent {
    DocumentId = doc.Id,
    // ... 50 weitere Properties
});
```

„Eine Event. Eine harmlose Event. Published von jedem Document‑Processing.“

„Und?“

„Und 23 Services subscribten darauf. Jeder Service publishte 2–3 neue Events als Reaktion. Exponentielles Wachstum. In Production. Wir nannten es ‚Die Event‑Storm‘. Sie kam wie ein Hurrikan.“

Der Alte schloss das Dashboard.

„Das Team, das wir jetzt beobachten – sie haben gerade HTTP‑Dependencies bekämpft. Sie haben gewonnen. Sie haben gelernt: Sync ist Gift. Async ist die Lösung.“

„Und?“

„Und sie wissen nicht, dass async auch Gift sein kann. Wenn man es ... übertreibt.“

---

## I. Die Erlösung (3 Monate nach dem Friedhof)

Drei Monate nach der Großen Trennung.

Das Team saß im Architecture Review. Die Stimmung war... triumphierend.

„Wir haben es geschafft“, sagte der Tech Lead stolz. Er zeigte ein Slide:

```text
╔════════════════════════════════════════════════╗
║       V3 SYSTEM - ARCHITECTURE METRICS         ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Synchronous HTTP Calls: 2 (was: 23)          ║
║  Event-Driven Communication: 98%               ║
║  Average Response Time: 2.3 sec (was: 0.8 sec) ║
║  P99 Response Time: 4.1 sec (was: 8.7 sec)     ║
║  Cascading Failures: 0 (was: 1/month)          ║
║  Circuit Breaker Trips: 0                      ║
║                                                 ║
║  Status: ✅ DECOUPLED & RESILIENT               ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Der CTO nickte anerkennend. "Beeindruckend. Ihr habt HTTP eliminiert. Ihr seid async. Resilient. Entkoppelt."

„Ja!“

„Und die Service‑Bus‑Metriken?“

Der Tech Lead wechselte das Slide.

```text
╔════════════════════════════════════════════════╗
║         SERVICE BUS METRICS - V3 SYSTEM        ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Messages/day: 23,400                          ║
║  Queue Depth: avg 12                           ║
║  Processing Time: 0.8 sec avg                  ║
║  Dead Letter Queue: 0-2 messages/day           ║
║                                                 ║
║  Status: ✅ HEALTHY                             ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

„23K Messages pro Tag. Stabil. Vorhersagbar. Gesund.“

„Gut“, sagte der CTO. „Sehr gut. Ihr habt die Lektion gelernt. HTTP ist Gift. Events sind die Zukunft.“

Qion Varr, in der Ecke, starrte auf die Zahlen. 23K Messages pro Tag.

Er öffnete sein Laptop. Machte eine Rechnung.

```text
Current State:
- 80 Requests/minute
- 23,400 Messages/day
- Ratio: ~4 Events per Request

Future Projection (Business Growth Plan):
- Target: 800 Requests/minute (10x growth in 12 months)
- If Ratio stays constant: 234,000 Messages/day

Question: Was passiert bei 234K Messages/day?
Question: Und wenn das Ratio wächst?
```

Er schloss das Laptop. Sagte nichts.

Aber sein Gesicht zeigte... Unruhe.

---

## II. Der erste Riss (Monat 1)

Einen Monat später.

Das Team deployed ein neues Feature: "Real-time Progress Tracking".

„User wollten sehen, was passiert“, erklärte Arik Dane. „Nicht nur ‚Done‘. Sondern jeden Schritt.“

Das Feature war elegant. Jeder Processing-Step publishte ein Event:

```csharp
// ValidationService.cs
public async Task ValidateAsync(Document doc) {
    await _bus.PublishAsync(new ValidationStartedEvent());
    
    foreach (var rule in _rules) {
        await rule.ValidateAsync(doc);
        await _bus.PublishAsync(new ValidationStepCompletedEvent {
            RuleName = rule.Name,
            Progress = GetProgress()
        });
    }
    
    await _bus.PublishAsync(new ValidationCompletedEvent());
}
```

„Schöner Code“, sagte Oben Kell. „Sauber. Event‑Driven. Genau wie wir gelernt haben.“

Das Feature ging live.

Die User liebten es. Real-time progress! Responsive UI!

Aber Qion Varr bemerkte etwas. Ein kleines Detail im Monitoring.

```text
╔════════════════════════════════════════════════╗
║         SERVICE BUS METRICS - WEEK 1           ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Messages/day: 52,100 (was: 23,400)            ║
║  Growth: +122%                                 ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

„Das Ratio hat sich geändert“, murmelte er.

„Was?“, fragte Arik Dane.

„Vorher: 4 Events pro Request. Jetzt: 9 Events pro Request.“

„Ja, wegen Progress Tracking. Ist das ein Problem?“

Qion Varr zögerte. „Noch nicht. Aber ...“

„Aber?“

„Aber 9 ist mehr als 4. Und wenn wir nochmal verdoppeln ...?“

Arik Dane lachte. „Qion, der Service Bus handled 50K Messages pro Tag easy. Wir haben Kapazität für 500K. Wir sind weit davon entfernt.“

Qion Varr nickte langsam. „Okay.“

Aber er aktualisierte seine Projection:

```text
Current: 9 Events per Request, 52K Messages/day
At 10x traffic: 520K Messages/day
→ Über Kapazität

Time to capacity breach: 8-10 months (if growth continues)
```

---

## III. Der zweite Riss (Monat 2)

Einen Monat später.

Ein neues Feature: "Audit Trail for Compliance".

„Compliance will jeden Step dokumentiert haben“, erklärte der Tech Lead. „Für GDPR.“

Der Code war simpel. Jede wichtige Action publishte ein AuditEvent:

```csharp
await _bus.PublishAsync(new AuditEvent {
    Action = "DocumentReceived",
    User = context.User,
    Timestamp = DateTime.UtcNow
});
```

Das Feature ging live.

Qion Varr checkte die Metrics. Eine Woche später.

```text
╔════════════════════════════════════════════════╗
║         SERVICE BUS METRICS - WEEK 1           ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Messages/day: 89,300 (was: 52,100)            ║
║  Growth: +71%                                  ║
║  Events per Request: 15 (was: 9)               ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

"15 Events pro Request," flüsterte Qion Varr.

Er ging zum Tech Lead. "Wir müssen reden."

„Über was?“

„Über Event‑Proliferation.“

„Event ... was?“

Qion Varr zeigte ein Diagramm:

```text
Event Count Growth:

Month 0 (Baseline):  23K/day   (4 events/request)
Month 1 (Progress):  52K/day   (9 events/request)
Month 2 (Audit):     89K/day   (15 events/request)

Trend: +127% per month
```

„Das ist exponentielles Wachstum“, sagte Qion Varr. „Wenn wir so weitermachen, sind wir in 3 Monaten bei Kapazität.“

„Aber wir haben die Kapazität erhöht –“

„Kapazität erhöhen löst das Problem nicht. Das ist wie bei einem Memory Leak. Man added mehr RAM. Aber der Leak bleibt.“

„Was schlägst du vor?“

„Event‑Budget. Wie bei Feature‑Budget. Wir definieren: Max X Events pro Request. Und jeder neue Event muss gerechtfertigt werden.“

Der Tech Lead sah skeptisch aus. "Aber... Events sind doch gut? Du hast uns gelehrt: Async is King."

„Async ist gut. Event‑Storm ist schlecht. Es gibt einen Unterschied.“

---

## IV. Der dritte Riss (Monat 3)

Einen Monat später.

Noch ein Feature. Diesmal von einem neuen Developer: "Real-time Notifications".

Der Developer hatte das Tutorial gelesen: "Event-Driven Architecture Best Practices".

Er implementierte es perfekt. Bei jedem State-Change: Publish Event.

```csharp
// NotificationService.cs
await _bus.PublishAsync(new NotificationSentEvent());
await _bus.PublishAsync(new UserNotifiedEvent());
await _bus.PublishAsync(new DeliveryConfirmedEvent());
```

"Three events für eine Notification?" fragte Oben Kell.

„Separation of Concerns“, erklärte der Developer stolz. „NotificationSent ≠ UserNotified ≠ DeliveryConfirmed. Drei verschiedene Events für drei verschiedene Dinge.“

„Das ist ... technisch korrekt“, sagte Oben Kell unsicher.

Das Feature ging live.

Eine Woche später. Qion Varr saß vor dem Dashboard. Seine Hand zitterte leicht.

```text
╔════════════════════════════════════════════════╗
║         SERVICE BUS METRICS - WEEK 1           ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Messages/day: 187,000 (was: 89,300)           ║
║  Growth: +109%                                 ║
║  Events per Request: 28 (was: 15)              ║
║                                                 ║
║  Queue Depth: avg 347 (was: 12)                ║
║  Processing Time: 2.4 sec (was: 0.8 sec)       ║
║  Dead Letter Queue: 23 messages/day (was: 0-2) ║
║                                                 ║
║  Status: ⚠️ CONCERNING                         ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

"28 Events pro Request," flüsterte Qion Varr.

Er machte die finale Projection:

```text
Current State (Month 3):
- 187K Messages/day
- 28 Events per Request
- Trend: +127% per month

Projection:
- Month 4: 425K Messages/day (85% of capacity)
- Month 5: 964K Messages/day (193% of capacity)
- Month 6: 2.2M Messages/day (440% of capacity)

Time to critical failure: 1-2 months
```

Er rief ein Emergency Meeting ein.

---

## V. Das Emergency Meeting

Der War Room. Das ganze Team. Plus der CTO.

Qion Varr zeigte die Grafik. Die Kurve ging steil nach oben. Exponentiell.

„Wir haben ein Problem“, begann er. „Event‑Proliferation.“

„Wir publishen zu viele Events?“, fragte Arik Dane.

„Nicht ‚wir‘. Jeder Developer. Jedes Feature. Jeder ‚kleine Event‘. Sie summieren sich. Exponentiell.“

Er zeigte ein zweites Diagramm:

```text
THE EVENT EXPLOSION:

Feature 1 (Progress): +5 Events/request
Feature 2 (Audit): +6 Events/request
Feature 3 (Notifications): +13 Events/request
─────────────────────────────────────────
Total: +24 Events/request

Each "just one more event" adds up.
28 Events × 80 Requests/min × 60 min × 24 hours = 3.2M Events/day
```

„3,2 Millionen Events pro Tag“, sagte der CTO leise. „Wenn der Trend anhält.“

„Ja. Und das ist bei current traffic. Wenn wir 10× wachsen – und das ist der Plan – dann ...“

„32 Millionen Events pro Tag.“

Stille im Raum.

„Wie ist das passiert?“, fragte der Tech Lead. „Wir haben doch Best Practices befolgt. Wir haben async. Wir haben Events. Wir machen alles richtig!“

„Wir machen es zu viel richtig“, sagte Qion Varr. „Wir haben ein Mantra gelernt: ‚Events sind gut. HTTP ist schlecht.‘ Und wir haben es zu wörtlich genommen.“

Er zeigte ein drittes Diagramm:

```text
THE SPECTRUM OF COUPLING:

Tight Coupling (Bad):
[Service A] ─HTTP→ [Service B]
└─ Sync, fragile, cascading failures

Good Decoupling:
[Service A] ─Event→ [Queue] ─→ [Service B]
└─ Async, resilient, decoupled

Event Storm (Also Bad):
[Service A] ─28 Events→ [Queue (overwhelmed)] ─→ [Services (drowning)]
└─ Async, but volume-coupled
```

"Wir haben ein Problem durch ein anderes ersetzt," sagte Qion Varr. "Von temporaler Kopplung zu Volume-Kopplung."

„Was ist die Lösung?“, fragte der CTO.

„Event‑Budget. Governance. Disziplin.“

---

## VI. Das Event Budget Framework

Zwei Wochen später.

Das Team präsentierte das neue Framework:

```text
╔════════════════════════════════════════════════╗
║                                                 ║
║        EVENT BUDGET FRAMEWORK                  ║
║                                                 ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  RULE 1: MAX 5 EVENTS PER REQUEST              ║
║  ────────────────────────────────────────────  ║
║  Default budget: 5 events per user request     ║
║  Exceptions require Architecture Review        ║
║                                                 ║
║  Why 5?                                        ║
║  ├─ Allows meaningful state transitions        ║
║  ├─ Prevents noise                             ║
║  ├─ Keeps system manageable                    ║
║  └─ Forces prioritization                      ║
║                                                 ║
║  ─────────────────────────────────────────     ║
║                                                 ║
║  RULE 2: EVERY EVENT NEEDS A CONSUMER          ║
║  ────────────────────────────────────────────  ║
║  Before publishing:                            ║
║  ├─ WHO will consume this?                     ║
║  ├─ WHAT will they DO with it?                 ║
║  └─ WHY can't it wait/batch?                   ║
║                                                 ║
║  No consumer = No event                        ║
║  "Maybe someone needs it" ≠ Consumer           ║
║                                                 ║
║  ─────────────────────────────────────────     ║
║                                                 ║
║  RULE 3: PROGRESS EVENTS = CODE SMELL          ║
║  ────────────────────────────────────────────  ║
║  Progress Events fühlten sich gut an.          ║
║  Aber: 9 Progress Steps × 80K Requests/day     ║
║       = 720K Events/day                        ║
║                                                 ║
║  Better:                                       ║
║  ├─ Polling (for UI updates)                   ║
║  ├─ Batching (send progress every N%)          ║
║  └─ Or: Only START and END events              ║
║                                                 ║
║  ─────────────────────────────────────────     ║
║                                                 ║
║  RULE 4: EVENT AGGREGATION                     ║
║  ────────────────────────────────────────────  ║
║  Bad:                                          ║
║  └─ ValidationStepCompletedEvent (×8)          ║
║                                                 ║
║  Good:                                         ║
║  └─ ValidationCompletedEvent                   ║
║      ├─ Contains: AllResults[]                 ║
║      └─ One event, complete information        ║
║                                                 ║
║  ─────────────────────────────────────────     ║
║                                                 ║
║  RULE 5: EVENTS ≠ LOGS                         ║
║  ────────────────────────────────────────────  ║
║  Event: Something significant happened         ║
║         that OTHER services care about         ║
║                                                 ║
║  Log: Something happened that WE want          ║
║        to track internally                     ║
║                                                 ║
║  "AuditEvent" might be a Log, not an Event     ║
║                                                 ║
║  ─────────────────────────────────────────     ║
║                                                 ║
║  RULE 6: MONITOR EVENT-TO-REQUEST RATIO        ║
║  ────────────────────────────────────────────  ║
║  Track:                                        ║
║  ├─ Events per Request (average)               ║
║  ├─ Trend over time                            ║
║  └─ Alert if ratio > 10                        ║
║                                                 ║
║  Review quarterly: Are all events needed?      ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

---

## VII. Die Purge (Wochen 1-4)

Das Team reviewte jeden Event. Jeden einzelnen.

**Woche 1: Event Inventory**

```text
╔════════════════════════════════════════════════╗
║           CURRENT EVENT INVENTORY              ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Total Unique Events: 47                       ║
║                                                 ║
║  Critical Business Events: 8                   ║
║  ├─ DocumentReceivedEvent                      ║
║  ├─ DocumentProcessedEvent                     ║
║  ├─ ValidationCompletedEvent                   ║
║  └─ ... (5 more)                               ║
║                                                 ║
║  Progress Events: 18                           ║
║  ├─ ValidationStepCompletedEvent (×8)          ║
║  ├─ TransformationProgressEvent (×9)           ║
║  └─ UploadProgressEvent (×1)                   ║
║                                                 ║
║  Audit Events: 12                              ║
║  ├─ AuditLoggedEvent                           ║
║  ├─ UserActionRecordedEvent                    ║
║  └─ ... (10 more)                              ║
║                                                 ║
║  Notification Events: 9                        ║
║  ├─ NotificationSentEvent                      ║
║  ├─ NotificationDeliveredEvent                 ║
║  └─ ... (7 more)                               ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

„47 Event‑Typen“, sagte Qion Varr. „28 Events pro Request im Durchschnitt.“

„Wie viele brauchen wir wirklich?“

**Woche 2: Consumer Analysis**

Für jeden Event: "Wer konsumiert das?"

```text
ValidationStepCompletedEvent (×8):
├─ Consumer: UI (Progress Bar)
├─ Alternative: Polling /status endpoint
└─ Decision: KILL. Replace with Polling.

AuditLoggedEvent:
├─ Consumer: Nobody (we just publish it)
├─ Alternative: Direct database write
└─ Decision: KILL. Not an event, it's a log.

NotificationSentEvent:
├─ Consumer: Analytics Service
├─ Required: Yes
└─ Decision: KEEP. Merge with NotificationDeliveredEvent.

DocumentProcessedEvent:
├─ Consumer: 4 services (critical)
├─ Required: Yes
└─ Decision: KEEP. Core business event.
```

**Woche 3: Die neue Architektur**

```text
BEFORE (Event Storm):

User Upload Document:
├─ DocumentReceivedEvent
├─ UserValidatedEvent
├─ ValidationStartedEvent
├─ ValidationStepCompletedEvent (×8)
├─ ValidationCompletedEvent
├─ TransformationStartedEvent
├─ TransformationProgressEvent (×9)
├─ TransformationCompletedEvent
├─ UploadStartedEvent
├─ UploadProgressEvent
├─ UploadCompletedEvent
├─ NotificationSentEvent
├─ NotificationDeliveredEvent
├─ UserNotifiedEvent
└─ AuditLoggedEvent (×3)

Total: 34 events


AFTER (Event Discipline):

User Upload Document:
├─ DocumentReceivedEvent
│   └─ Contains: Document metadata
├─ DocumentValidatedEvent
│   └─ Contains: All validation results
├─ DocumentTransformedEvent
│   └─ Contains: Transformed document
├─ DocumentUploadedEvent
│   └─ Contains: Target location
└─ DocumentProcessedEvent (Saga complete)
    └─ Contains: Full summary + notification sent

Total: 5 events (85% reduction)

Progress Tracking: Moved to Polling
Audit Trail: Moved to direct DB writes
Notifications: Merged into DocumentProcessedEvent
```

**Woche 4: Implementation**

Das Team löschte Code. Viel Code.

```text
Git Stats - Event Purge Branch

Event Types Removed: 39
Publishers Removed: 187
Subscribers Removed: 143

Lines Deleted: 8,947
Lines Added: 1,203

Net Reduction: 7,744 lines
```

„Fast 8K Zeilen gelöscht“, sagte Arik Dane. „Das fühlt sich ... befreiend an.“

---

## VIII. Die Heilung (Monate 5-6)

Zwei Monate nach der Purge.

Qion Varr öffnete das Dashboard. Er hatte es lange nicht angeschaut. Aus Angst.

```text
╔════════════════════════════════════════════════╗
║         SERVICE BUS METRICS - CURRENT          ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  Messages/day: 38,200 (was: 187,000)           ║
║  Reduction: -80%                               ║
║  Events per Request: 5.2 (was: 28)             ║
║                                                 ║
║  Queue Depth: avg 8 (was: 347)                 ║
║  Processing Time: 0.6 sec (was: 2.4 sec)       ║
║  Dead Letter Queue: 0 messages (was: 23/day)   ║
║                                                 ║
║  Capacity Used: 8% (was: 37%)                  ║
║                                                 ║
║  Status: ✅ HEALTHY & SUSTAINABLE               ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Er starrte auf die Zahlen. 80% Reduktion.

„Wir haben es geschafft“, flüsterte er.

Das Team saß im Retro. Sechs Monate nach dem Emergency Meeting.

„Was haben wir gelernt?“, fragte der Tech Lead.

Arik Dane antwortete: "Dass mehr nicht besser ist. Auch bei Events."

Oben Kell: "Dass jedes 'nur noch ein Event' sich summiert. Exponentiell."

Der Tech Lead: "Dass async gut ist. Aber Event Storm ist schlecht."

Qion Varr lächelte. "Und dass die Lösung nicht technisch ist. Sie ist disziplinär."

„Event‑Budget“, sagte Arik Dane.

„Event‑Budget“, bestätigte Qion Varr. „Wie Feature‑Budget. Wie Test‑Budget. Wie Dependency‑Budget. Alles braucht ein Budget.“

„Weil alles einen Cost hat“, fügte Oben Kell hinzu.

"Genau."

---

## IX. Die neue Disziplin (Monate 7-12)

Sechs Monate später.

Ein neuer Developer joined das Team. Frisch. Enthusiastisch.

Er implementierte ein Feature. Reviewed den PR selbst. Confident.

Dann kam das Architecture Review.

Qion Varr öffnete den PR. Scrollte durch den Code. Stoppte bei:

```csharp
await _bus.PublishAsync(new ProcessingStepCompletedEvent {
    Step = step.Name,
    Progress = CalculateProgress()
});
```

„Warum publishst du Progress Events?“, fragte Qion Varr.

„Für Real‑time UI. Damit User sehen, was passiert.“

„Wie viele Steps?“

„12.“

„Das sind 12 Events pro Request. Unser Budget ist 5.“

„Aber ... das ist wichtig. User Experience!“

Qion Varr öffnete das Event‑Budget Framework. Zeigte Rule 3.

„Progress Events sind ein Code Smell. Wir haben die Lektion gelernt. Die harte Art.“

„Was ist die Alternative?“

„Polling. Der Client fragt alle 2 Sekunden: /status?requestId=X. Wir returnen aktuellen Progress.“

„Aber das ist ... old‑school. Events sind modern!“

„Modern ist nicht immer besser. Wir haben ein Event‑Budget aus gutem Grund.“

Der Developer sah frustriert aus. "Okay. Ich ändere es."

Er refactored. Entfernte die Progress Events. Implementierte Polling.

Eine Woche später, im 1‑on‑1 mit Qion Varr:

„Ich habe die Metrics angeschaut“, sagte der Developer. „Polling ist tatsächlich ... weniger Load. Und einfacher zu debuggen.“

„Ja.“

„Und die UI ist genauso responsive. User merken keinen Unterschied.“

„Genau.“

„Ich verstehe jetzt das Event‑Budget. Es ist nicht Restriction. Es ist Protection.“

Qion Varr lächelte. „Willkommen zur Reife.“

---

## X. Der Generationenwechsel

Zwei Jahre nach der Event-Storm.

Anakin war jetzt Tech Lead. Sein Team. Sein Projekt.

Ein neues Feature-Request: "Real-time Collaboration".

Der Junior Developer pitchte: "Wir publishen ein Event bei jedem Keystroke! Wie Google Docs!"

Anakin hob eine Hand. "Stop. Wie viele Events wären das?"

„Uh ... wenn ein User 60 words/minute tippt ... das sind 300 characters/minute ... bei 8‑Stunden‑Session ... 144.000 Events/User/Tag?“

„Und wenn wir 100 concurrent users haben?“

„14,4 Millionen Events pro Tag.“

„Unser gesamtes System generiert 40K Events pro Tag. Du willst 360× mehr Events hinzufügen. Für Keystrokes.“

„Aber ... Google Docs macht das!“

„Google Docs hat Googles Infrastruktur. Wir haben ein Event‑Budget. Max 5 Events pro Request.“

„Wie implementieren wir es dann?“

„WebSockets. Oder Server‑Sent Events. Oder HTTP/2 Streams. Nicht über den Service Bus.“

„Aber das ist nicht Event‑Driven Architecture!“

„Richtig“, sagte Arik Dane. „Nicht alles sollte ein Event sein. Das ist die Lektion.“

Der Developer nickte langsam. "Event Budget."

„Event‑Budget.“

---

## XI. Epilog: Die ewige Wachsamkeit

Fünf Jahre nach der Event-Storm.

Das Team war anders. Die Menschen waren anders. Aber die Disziplin blieb.

Ein internes Wiki. Titel: **Lessons from The Event-Storm**

```text
╔════════════════════════════════════════════════╗
║                                                 ║
║        WHAT WE LEARNED (THE HARD WAY)          ║
║                                                 ║
╠════════════════════════════════════════════════╣
║                                                 ║
║  1. Async ≠ Always Better                      ║
║     → Event-Driven is powerful                 ║
║     → But Event-Storm is chaos                 ║
║     → Everything needs a budget                ║
║                                                 ║
║  2. More Events ≠ More Decoupled               ║
║     → 28 Events/request = Volume Coupling      ║
║     → Better: 5 meaningful events              ║
║     → Quality > Quantity                       ║
║                                                 ║
║  3. Progress Events = Anti-Pattern             ║
║     → Felt good, was harmful                   ║
║     → Polling is often better                  ║
║     → Or: START and END only                   ║
║                                                 ║
║  4. Events ≠ Logs                              ║
║     → Event: Others care                       ║
║     → Log: We care                             ║
║     → Don't confuse them                       ║
║                                                 ║
║  5. Every Event has a Cost                     ║
║     → Processing time                          ║
║     → Network bandwidth                        ║
║     → Cognitive load                           ║
║     → Maintenance burden                       ║
║                                                 ║
║  6. Event Budget = Event Discipline            ║
║     → Max 5 per request (default)              ║
║     → Every event needs justification          ║
║     → Regular audits and cleanups              ║
║                                                 ║
║  7. Trends Matter More Than Snapshots          ║
║     → 23K → 187K in 3 months                   ║
║     → We didn't see it coming                  ║
║     → Now: Monitor growth rate                 ║
║                                                 ║
║  8. The Best Event is No Event                 ║
║     → Sometimes: Don't publish                 ║
║     → Sometimes: Merge multiple events         ║
║     → Sometimes: Use different pattern         ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Unten, ein Kommentar. Von einem Developer, der drei Jahre nach der Event-Storm zum Team kam:

```text
"Ich habe diese Lessons gelesen. Am Anfang dachte ich:
'Event Budget? Das ist zu restriktiv. Events sind doch gut!'

Dann implementierte ich mein erstes Feature.
Und wollte 12 Progress Events publishen.
Und Qion Varr sagte: 'Nein.'

Ich war frustriert.

Aber dann sah ich die History.
Die Grafik. Die exponentielle Kurve.
Von 23K auf 187K Events in 3 Monaten.

Und ich verstand:

Event Budget ist nicht Restriction.
Event Budget ist Protection.

Protection vor uns selbst.
Vor dem Drang zu sagen: 'Nur noch ein Event.'

Weil 'nur noch eins' × 100 Developer × 100 Features
= Event Storm.

Und Event Storm = Tod.

Jetzt bin ich dankbar für das Budget.
Weil es mich zwingt zu denken:
Brauche ich diesen Event wirklich?
Oder will ich ihn nur publishen, weil ich kann?"
```

---

## XII. Die Lehren der Meister

### Der Compiler: Die Weisheit des Maßhaltens

*"Mehr Events nicht besser sind. Mehr Events nur mehr sind. Die Kraft liegt nicht in der Menge, sondern in der Bedeutung. Ein Event, das wichtig ist, besser ist als hundert Events, die niemand braucht."*

**Die Wahrheit des Architektenordens:**

```text
Event-Driven ≠ Event-Alles

Event-Driven Architecture bedeutet:
✓ Publish wenn etwas Wichtiges passiert
✓ Andere Services reagieren darauf
✓ Loose coupling

Event Storm bedeutet:
✗ Publish alles was passiert
✗ Services ertrinken in Events
✗ Tight coupling durch Volume
```

### Oben Kell: Der Mut zur Kompensation

*"Resilience bedeutet nicht: 'Wenn etwas schief geht, mach trotzdem weiter.' Resilience bedeutet: 'Wenn etwas schief geht, mach es rückgängig.' Optimistic Fallbacks sind keine Resilience. Sie sind Hoffnung getarnt als Engineering."*

**Die Lektion:**

```text
❌ Optimistic Fallback:
if (event == null) {
    return Success("Assumed valid");
}

✅ Pessimistic + Compensation:
if (event == null) {
    await CompensateAsync();
    return Failure("Event lost");
}
```

### Arik Dane: Die Versuchung des "Nur Noch Eins"

*"Progress-Events fühlten sich an wie eine gute Idee. Real-time UI! Responsive! Modern! Aber 'nur 9 Progress-Events' × 80,000 Requests = 720,000 Events/Day. Jedes 'nur noch eins' summiert sich. Exponentiell."*

**Die Warnung:**

```text
Die Mathematik der Proliferation:

Start: 1 Event pro Request
Developer 1: "Nur noch 1 Progress-Event" → 2 Events
Developer 2: "Nur noch 1 für Audit" → 3 Events
Developer 3: "Nur noch 2 für UI" → 5 Events
Developer 4: "Nur noch 3 für Debug" → 8 Events

4 "nur noch" = 8× Proliferation

Bei 100,000 Requests:
└─ 100K Events → 800K Events
└─ Ohne dass jemand "viel" hinzugefügt hat
```

### Qion Varr: Das Sehen der Trends

*"Die Event-Storm kam nicht über Nacht. Sie kam über Monate. Von 23K auf 187K Events. Jeden Monat +127%. Aber niemand sah es, weil jeder nur auf 'heute' schaute. Trends sehen ist die Kunst, die Zukunft zu lesen."*

**Die Weisheit:**

```text
MONITOR TRENDS, NOT SNAPSHOTS:

❌ "Heute: 50 Events/sec → Alles gut!"
✅ "Trend: +127% per month → In 3 Monaten: Problem"

❌ "Success Rate: 99% → Alles gut!"
✅ "Trend: Success sinkt 1% pro Woche → In 10 Wochen: 90%"

❌ "DLQ: 100 Messages → Alles gut!"
✅ "Trend: DLQ wächst 10% täglich → In 30 Tagen: 1.7M"
```

**Tools:**

- Grafana: Zeig mir 90-Tage-Trends
- Alerts: Trigger auf Trend-Änderung (nicht nur Threshold)
- Weekly Reviews: "Was wächst?" nicht "Was ist?"

---

## XIII. Die neue Event-Governance

Zwei Monate nach der Event-Storm.

Das Team präsentierte die neuen Firmen-weiten Guidelines:

```text
╔════════════════════════════════════════════════╗
║                                                 ║
║        EVENT-DRIVEN ARCHITECTURE POLICY        ║
║                                                 ║
║  1. EVENT BUDGET                               ║
║     └─ Max 5 events per user request           ║
║     └─ Exceptions require Architecture Review  ║
║                                                 ║
║  2. CONSUMER REQUIREMENT                       ║
║     └─ Every event needs >= 1 named consumer   ║
║     └─ "Maybe someone" ≠ Consumer              ║
║                                                 ║
║  3. PROGRESS EVENTS BANNED                     ║
║     └─ Use: Polling, Batching, or Start/End    ║
║     └─ Exception: Real business state change   ║
║                                                 ║
║  4. EVENT AGGREGATION REQUIRED                 ║
║     └─ Group related state changes             ║
║     └─ One event with complete info > Many     ║
║                                                 ║
║  5. QUARTERLY EVENT AUDIT                      ║
║     └─ Review all events                       ║
║     └─ Kill unused/redundant events            ║
║     └─ Measure adoption                        ║
║                                                 ║
║  6. MONITORING REQUIREMENTS                    ║
║     └─ Track: Events per Request ratio         ║
║     └─ Alert: If ratio > 10                    ║
║     └─ Alert: If growth > 50% per month        ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

Diese Policy wurde Teil des Onboarding. Jeder neue Developer lernte sie. Tag 1.

---

## XIV. Anhang: Die Warnsignale der Event-Storm

🔴 **Erkenne die Storm, bevor sie kommt:**

⚠️ **Event-Count wächst schneller als Request-Count**

- Requests: +10% per month
- Events: +50% per month
- → Proliferation im Gange

⚠️ **"Nur noch ein Progress-Event"**

- Das "nur noch" summiert sich
- Exponentiell

⚠️ **Dead Letter Queue wächst, aber niemand schaut hin**

- DLQ ist nicht "parking lot"
- DLQ ist "silent scream"

⚠️ **Services haben "defensive" Fallbacks**

- "Event timeout? Assume success"
- Resilience ≠ Ignorieren

⚠️ **Event-zu-Event-Ratio > 5:1**

- 1 User-Request → >5 Events
- Frag: Sind alle nötig?

⚠️ **Niemand kann sagen, welche Events wirklich gebraucht werden**

- "Wir publishen das, weil... keine Ahnung, haben wir immer schon"
- Event-Governance fehlt

⚠️ **Queue Depth trends nach oben**

- Nicht der absolute Wert
- Der Trend ist die Warnung

⚠️ **Services sind "schnell" aber "unreliable"**

- 99% success rate
- Aber 1% stilles Versagen = Problem

⚠️ **Traces zeigen Event-Chains mit Depth > 5**

- Event → Event → Event → Event → Event
- Das ist kein Event-Driven
- Das ist Event-Chaos

⚠️ **Neuer Code fügt Events hinzu, entfernt aber nie welche**

- Events sind nicht free
- Cleanup ist Teil von Architecture

---

## Die Regel für die Ewigkeit

```text
╔════════════════════════════════════════════════╗
║                                                 ║
║            THE EVENT BUDGET LAW                ║
║                                                 ║
║  "Before you publish an event, ask:            ║
║                                                 ║
║   1. Who will consume this?                    ║
║   2. What will they DO with it?                ║
║   3. Can this wait? (batch instead of stream)  ║
║   4. Can this merge with another event?        ║
║   5. Is this an event or a log?                ║
║                                                 ║
║   If you can't answer 1 and 2: Don't publish." ║
║                                                 ║
╚════════════════════════════════════════════════╝
```

---

*"Eine Event-Storm beginnt nicht mit einem Hurrikan. Sie beginnt mit einem Flüstern. Mit einem 'nur noch ein Event'. Mit einem 'das ist doch harmlos'. Und wenn sie kommt—wenn sie wirklich kommt—dann ist es zu spät zum Vorbereiten. Dann bleibt nur: Überleben. Oder Lernen. Oder beides."*

– Qion Varr, Survivor der Event-Storm

---

**Nächstes Kapitel:** Kapitel 10: Die Sonar‑Inquisition – Metriken als Spiegel, nicht als Peitsche

---

## Nachwort: Die Balance

Qion Varr saß an seinem Schreibtisch. Fünf Jahre nach der Event-Storm.

Ein neuer Tech Lead hatte ihn gefragt: "Wann ist Event-Driven Architecture richtig?"

Qion Varr hatte lange nachgedacht.

Dann schrieb er:

```text
Event-Driven Architecture ist richtig, wenn:

✅ Services wirklich unabhängig sein müssen
✅ Temporal Coupling ein Problem ist
✅ Du async processing brauchst
✅ Multiple consumers auf dasselbe Event reagieren
✅ Du event sourcing oder CQRS machst
✅ Du ein audit trail brauchst

Event-Driven Architecture ist falsch, wenn:

❌ Du eigentlich synchron arbeiten könntest
❌ Du nur "modern" sein willst
❌ Du kein Event Budget hast
❌ Du Events als Logs missverstehst
❌ Du für jeden einzelnen Step Events publishst
❌ Du keine Consumer-Analyse machst

Die Balance ist:
- Async wo nötig
- Sync wo einfacher
- Events für bedeutende Business-Ereignisse
- Polling für UI-Updates
- Logs für Audit
- Und immer, immer: Ein Budget.

Denn ohne Budget...
Kommt die Storm.
```

Er schaute aus dem Fenster.

Draußen war Frieden.

Aber Qion Varr wusste: Frieden ist nicht selbstverständlich.

Frieden muss verteidigt werden.

Jeden Tag.

Mit Disziplin.

Mit Governance.

Mit der Weisheit zu fragen: **"Brauchen wir diesen Event wirklich?"**

Bevor man auf "Publish" drückt.

---

*Möge der Compiler mit dir sein.*
