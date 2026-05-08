# ERP Discovery — TopM-Termine (NET7)

Dieser Leitfaden strukturiert Discovery-Termine mit TopM zur ERP-Schnittstelle NET7. Er adressiert zwei Personas — **TopM Technical** (Pre-Sales-/Implementation-/Support-Engineering, Hauptfokus) und **TopM Account/Commercial** (Account-/Project-Management, leichter). Zielgruppe: Architektur, Backend-Lead und Procurement-Sponsor auf AOT-Seite. Anwendung: vor jedem TopM-Termin und während der Auswertung. Diese Discovery läuft parallel zur Customer-Discovery (`business-discovery.md`).

---

## 1. Zweck und Anwendung

- **Phase**: Sprint 0 — ERP Discovery & Foundation.
- **Zweck**: Validierung der ERP-Annahmen aus `AGENTS.md` (ERP Ownership, ERP Integration Rules) und `procurement-hypotheses.md` §7; Input für Sprint 2 *ERP Boundary Design*.
- **Charakter**: Leitfaden, **kein Skript**. Capability-Fragen ("welche Mechanismen", "welche Operationen") statt Endpoint-Behauptungen. Follow-ups zulassen, Schweigen aushalten.
- **Dauer**: ca. 60–90 Minuten für Technical-Termine, 30–45 Minuten für Account/Commercial-Termine.

### Vor-Termin-Checkliste

- [ ] NDA-Status zwischen AOT und TopM geklärt — geteilte Inhalte, Umgang mit Notizen.
- [ ] Vorhandene NET7-Dokumentation (Onboarding-Mappen, Schnittstellen-Beschreibungen, frühere E-Mails) gesichtet.
- [ ] Demo- oder Sandbox-Zugang vorab erfragt; Zugangs-Status ins Profil unten übernehmen.
- [ ] AOT-IT vorab informiert (interner Stakeholder, Nachfragen zur bestehenden NET7-Installation möglich).
- [ ] Persona-Modus mental eingestellt: Technical-Vokabular vs. Commercial-Vokabular bewusst trennen.
- [ ] Notiz-Tool offen, Zeitfenster für Nachbereitung im Anschluss reserviert.

---

## 2. Interviewee-Profil

Pro Termin einmal ausfüllen.

- **Firma**: TopM
- **Rolle (offizieller Titel)**: ____________________
- **Persona**:
  - [ ] TopM Technical
  - [ ] TopM Account/Commercial
  - [ ] andere: ____________________
- **Funktion**:
  - [ ] Pre-Sales-Architect
  - [ ] Implementation-Engineer
  - [ ] Account-Manager
  - [ ] Project-Manager
  - [ ] Support-Engineer
  - [ ] andere: ____________________
- **Demo-Zugang verfügbar**:
  - [ ] ja
  - [ ] nein
  - [ ] unklar
- **Datum**: ____________________
- **Interviewer:in**: ____________________

---

## 3. TopM Technical — Hauptsektion

Hauptsektion aller ERP-Discovery-Termine. 9 Themen-Cluster, je 3–5 offene Fragen. Fokus: belastbare Grundlage für das *Sprint-2-Boundary-Design* zwischen AOT-Plattform und NET7. Capability-orientiert formulieren — keine Annahmen über konkrete NET7-Endpunkte oder -Mechanismen einbringen.

### a. Schnittstellen-Inventar & Dokumentation

- Welche maschinenlesbaren Schnittstellen-Klassen bietet NET7 heute (REST, SOAP, OData, File-basierter Export, Datenbank-Zugriff, EDI, weitere), und welche davon sind für Drittsystem-Integration freigegeben?
- In welcher Form liegt die Schnittstellen-Dokumentation typischerweise vor (OpenAPI/Swagger, WSDL, PDF, internes Wiki, …) und wie aktuell ist sie?
- Welche Referenz-Customer betreiben heute eine vergleichbare Web-Procurement- oder Customer-Portal-Anbindung an NET7?
- Welche Schnittstellen-Klassen sind in Ihrer Lifecycle-Strategie als "stable", "deprecated" oder "next-generation" eingestuft?
- Wie wird Breaking-Change-Kommunikation typischerweise an integrierende Drittsysteme adressiert?

**Notizen**:

### b. Authentifizierung & Autorisierung

- Welche Authentifizierungs-Mechanismen unterstützt NET7 für Drittsystem-Integration (OAuth 2.0, API-Key, Client-Certificate, mTLS, IP-Whitelisting, weitere)?
- Wie wird Berechtigungs-Granularität typischerweise modelliert — pro Endpoint, pro Entity, pro Feld, pro Customer-Scope?
- Wie sieht der Credential-Lifecycle aus (Beantragung, Rotation, Sperrung, Wiederherstellung)?
- Wie unterscheidet NET7 zwischen System-zu-System-Identität und delegierter End-Customer-Identität?
- Welche Audit-Spuren entstehen bei Authentifizierungs- und Autorisierungs-Vorgängen aus Sicht des integrierenden Drittsystems?

**Notizen**:

### c. Datenmodell & Entities

- Welche fachlichen Entities sind über die Schnittstelle exposed (Produkte, Kunden, Kontrakte, Bestellungen, Lieferungen, Chargen, Dokumente, Preise, weitere)?
- In welcher Form liegt das Datenmodell-Schema vor und wie wird es an Integrations-Partner verteilt?
- Wie stabil sind technische IDs gegenüber fachlichen Änderungen (Umbenennen, Mergen von Stammdaten, Auslaufmodelle)?
- Welche Cardinalitäten und Beziehungen zwischen den exposed Entities sollten Drittsysteme zwingend kennen, bevor sie ihr eigenes Datenmodell entwerfen?
- An welchen Stellen weicht das exposed Datenmodell vom internen NET7-Datenmodell ab?

**Notizen**:

### d. Lese-Operationen

- Welche Lese-Operationen stehen für Produktkatalog, Preise, Verfügbarkeit, Dokumente, Kontrakte und Order-History heute zur Verfügung?
- Welche Filter-, Such- und Pagination-Mechanismen werden auf Listen-Operationen unterstützt?
- Wie wird Customer-Scope in Lese-Operationen üblicherweise durchgesetzt (z.B. nur die Kontrakt-Preise des authentifizierten Kunden)?
- Welche Lese-Operationen liefern Live-Daten gegenüber asynchron aufbereiteten Snapshots — und wie wird das gekennzeichnet?
- Welche Felder oder Entities sind aus Schutzgründen explizit nicht oder nur eingeschränkt lesbar?

**Notizen**:

### e. Schreib-Operationen

- Welche Schreib-Operationen sind heute über die API möglich, welche nur über die Stamm-Anwendung?
- Wie werden RFQs, Sample-Anfragen und Order-Auslösungen aus Drittsystemen typischerweise eingebracht?
- Welche Validierungs-Regeln greifen serverseitig auf Schreib-Operationen, und welche Fehlerklassen kommen zurück?
- Wie wird Idempotenz auf Schreib-Operationen unterstützt — über Client-Tokens, natürliche Keys, beides, gar nicht?
- Welche Schreib-Operationen stoßen Folgeprozesse innerhalb NET7 an, und wie sichtbar sind diese Folgeprozesse für das Drittsystem?

**Notizen**:

### f. Sync-Strategie

- Welche Synchronisations-Muster sind in NET7-Integrationen etabliert (Real-Time-Pull, Event-Push/Webhook, Polling, periodischer Bulk-Export)?
- Welche Daten lassen sich als Delta seit einem Zeitpunkt abrufen, welche nur als Full-Set?
- In welcher Frequenz aktualisieren sich für Procurement relevante Daten typischerweise (Stammdaten, Preise, Verfügbarkeiten, Dokumente)?
- Welche Konsistenz-Garantien gelten zwischen mehreren Operationen (z.B. ist ein neu angelegter Datensatz sofort über die Lese-Schnittstelle sichtbar)?
- Welche Sync-Topologie empfehlen Sie für ein Customer-Portal mit mehreren tausend Stammdaten, kontinuierlich wechselnden Verfügbarkeiten und versionierten Dokumenten?

**Notizen**:

### g. Failure-Modes & Resilience

- Wie häufig sind Schnittstellen-Ausfälle oder Degradationen typischerweise, und wie werden sie an integrierende Drittsysteme kommuniziert?
- Welche Retry- und Backoff-Strategien empfehlen Sie integrierenden Drittsystemen?
- Welche Fehler-Codes oder -Klassen sollten Drittsysteme für sauberes Error-Handling unbedingt unterscheiden können?
- Wie verhält sich die Schnittstelle bei partiellen Ausfällen einzelner Backend-Module (z.B. Dokumenten-Service erreichbar, aber Preis-Service nicht)?
- Wie werden Transaktions-Recovery-Patterns unterstützt, wenn ein mehrstufiger Schreib-Vorgang mittendrin abbricht?

**Notizen**:

### h. Performance, Limits, Versioning

- Welche typischen Latenzen sehen integrierende Drittsysteme auf Lese- und Schreib-Operationen?
- Welche Rate-Limits, Concurrency-Limits oder Quotas existieren, und wie werden sie an integrierende Drittsysteme kommuniziert?
- Wie wird API-Versioning umgesetzt (URL, Header, Query-Parameter, parallele Endpoint-Generationen)?
- Wie lange werden alte API-Versionen typischerweise parallel zu neuen weitergeführt?
- Welche Vorlauf-Zeit gibt es vor Breaking Changes, und in welcher Form wird informiert?

**Notizen**:

### i. Test-Sandbox & Observability

- Steht eine dedizierte Sandbox- oder Test-Instanz mit realistischen Seed-Daten für Drittsystem-Entwicklung zur Verfügung?
- Welche Logs, Tracing-Daten oder Metriken kann ein integrierendes Drittsystem aus NET7-Sicht einsehen, wenn es die eigene Integration debuggen muss?
- Wie wird die Datenbasis in der Sandbox typischerweise aktuell gehalten (Refresh-Intervall, manuelles Seeding, Production-Klone)?
- Existiert eine öffentliche oder gemeinsame Status-Page für Schnittstellen-Verfügbarkeit?
- Welche Audit-Trail-Sicht hat das integrierende Drittsystem auf seine eigenen API-Calls (wer hat wann welchen Call abgesetzt, mit welchem Ergebnis)?

**Notizen**:

---

## 4. TopM Account/Commercial — leichter

Sekundäre Sektion. 4 Cluster, je 2–3 Fragen. Fokus: kommerzielle Rahmenbedingungen, Onboarding-Aufwand, Support-Realität und mittelfristige Roadmap der Schnittstelle.

### a. Vertrag & Lizenz

- Ist API-Zugang im Standard-Lizenzumfang von NET7 enthalten oder erfordert er eine separate Lizenz bzw. Lizenz-Erweiterung?
- Wie wird die API-Nutzung kommerziell modelliert — pro User, pro Connection, pro Call-Volumen, pauschal?
- Welche vertraglichen Anpassungen wären nötig, um eine Customer-Portal-Integration mit perspektivisch wachsendem Datenverkehr abzudecken?

**Notizen**:

### b. Projekt-Setup & Onboarding

- Wie sieht der Standard-Prozess für die Beantragung von API-Credentials für ein Customer-Projekt aus?
- Mit welcher Lead-Time von Beauftragung bis funktionaler Sandbox sollten wir kalkulieren?
- Welche Sign-offs sind kunden- bzw. anbieterseitig vor Produktiv-Schaltung üblicherweise erforderlich?

**Notizen**:

### c. Support-Modell & SLAs

- Welche Reaktions- und Lösungszeiten sind für Schnittstellen-Probleme im Standard-Support vorgesehen?
- Wie sieht der Eskalationsweg bei produktionskritischen Schnittstellen-Störungen aus?
- Welche Verfügbarkeits-Garantien werden für die produktive Schnittstelle zugesichert?

**Notizen**:

### d. Roadmap & Strategische Ausrichtung

- Welche strategische Richtung verfolgt TopM für die NET7-Schnittstellen in den nächsten 12–24 Monaten (Modernisierung, Cloud, Event-Driven, weitere)?
- Welche aktuellen Schnittstellen-Versionen sind als langfristig stabil eingeplant, welche werden in absehbarer Zeit abgelöst?
- Welche Beta- oder Early-Access-Programme gibt es heute, an denen ein anspruchsvoller Integrations-Customer teilnehmen kann?

**Notizen**:

---

## 5. Notizen / Offene Fragen / Architektur-Implikationen

Freie Capture-Sektion. Pro Termin innerhalb von 24 Stunden nachbereiten, solange der Eindruck frisch ist.

- **Überraschende Erkenntnisse**:
  - …

- **Offene Fragen für Follow-up**:
  - …

- **Architektur-Implikationen** (welche Regel aus `AGENTS.md` ERP Ownership/Integration oder welche Annahme aus `procurement-hypotheses.md` §7 wird durch die Findings bestätigt / widerlegt / verschoben?):
  - …

---

## 6. Quellen / Cross-Refs

- [`AGENTS.md`](../AGENTS.md) — Personas, ERP Ownership Rules, ERP Integration Rules.
- [`docs/adr/0001-stack-and-deployment.md`](./adr/0001-stack-and-deployment.md) — §2.5 ERP-Adapter-Topologie.
- [`business-discovery.md`](./business-discovery.md) — parallele Customer-Discovery.
- [`procurement-hypotheses.md`](./procurement-hypotheses.md) — §7 ERP Assumptions.
- [`ROADMAP.md`](../ROADMAP.md) — Sprint 2 ERP Boundary Design.
