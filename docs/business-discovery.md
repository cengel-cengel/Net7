# Business Discovery — AOT-Kunden-Interviews

Dieser Leitfaden strukturiert Discovery-Interviews mit AOT-Kunden in Sprint 0. Er adressiert die drei Personas der Plattform — **Buyer (Strategic Procurement)** als Hauptfokus sowie **Product Developer** und **Quality Manager** in geringerer Tiefe. Zielgruppe: Interviewer:innen aus Produkt, Architektur und Vertrieb. Anwendung: vor jedem Kunden-Discovery-Call und während der Auswertung.

---

## 1. Zweck und Anwendung

- **Phase**: Sprint 0 — Discovery & Foundation.
- **Zweck**: Validierung der Hypothesen aus `AGENTS.md` zu Personas, Beschaffungsfriktion und ERP-Integration; Input für `docs/erp-discovery.md` und nachfolgende Sprint-Planung.
- **Charakter**: Leitfaden, **kein Skript**. Reihenfolge anpassen, Follow-ups zulassen, Schweigen aushalten. Offene Fragen bevorzugen ("wie", "warum", "erzählen Sie").
- **Dauer**: ca. 60 Minuten pro Buyer-Interview, 30–45 Minuten für Product Developer und Quality Manager.

### Vor-Interview-Checkliste

- [ ] Interviewee-Profil (§2) vorab so weit wie möglich befüllt (Firma, Rolle, Persona).
- [ ] Aufzeichnung und DSGVO-Einwilligung geklärt.
- [ ] Nicht-AOT-Materialien des Kunden gesichtet (Website, Pressemitteilungen, Zertifikate, falls öffentlich).
- [ ] Bekannte AOT-Berührungspunkte aus NET7 vorab vom Vertrieb erfragt (bestehender Kunde? aktive Kontrakte? offene Samples?).
- [ ] Interviewer:in im Persona-Modus: Buyer-Vokabular vs. Quality-Manager-Vokabular bewusst trennen.
- [ ] Notiz-Tool offen, Persona-Schalter im Kopf gesetzt.

---

## 2. Interviewee-Profil

Pro Interview einmal ausfüllen.

- **Firma**: ____________________
- **Rolle (offizieller Titel)**: ____________________
- **Persona**:
  - [ ] Buyer (Strategic Procurement)
  - [ ] Product Developer
  - [ ] Quality Manager
  - [ ] andere: ____________________
- **Buyer Type** *(nur ausfüllen, wenn Persona = Buyer; sonst n.a.)*:
  - [ ] Strategic
  - [ ] Tactical
  - [ ] Mixed
- **Region**:
  - [ ] DACH
  - [ ] EU
  - [ ] UK
  - [ ] US
  - [ ] APAC
- **Business Segment**:
  - [ ] Food
  - [ ] Cosmetics
  - [ ] Both
- **Unternehmensgröße** (Mitarbeiterzahl / Jahresumsatz, Größenordnung reicht): ____________________
- **Datum**: ____________________
- **Interviewer:in**: ____________________

---

## 3. Buyer (Strategic Procurement) — Hauptsektion

Hauptfokus aller Discovery-Aktivitäten. 9 Themen-Cluster, je 3–5 offene Fragen. Cluster d–f sind die wichtigsten Stellschrauben für die Plattform.

### a. Unternehmensprofil und Volumen

- Erzählen Sie kurz, wofür Ihre Beschaffungsabteilung verantwortlich ist und welchen Anteil organische Rohstoffe daran haben.
- Wie groß ist Ihr typisches Bestellvolumen für einen Wirkstoff oder ein Ingredient — Kilo, Tonnen, oder ganz anders?
- Wie viele aktive Lieferanten für organische Rohstoffe haben Sie typischerweise im Portfolio?
- Wie unterscheidet sich Ihr Einkauf zwischen Kosmetik- und Food-Anwendungen — falls beides?
- Welche Beschaffungs-KPIs misst Ihr Haus aktiv (Lead-Time, Ausschuss, Lieferantenrisiko, Total Cost)?

**Notizen**:

### b. Aktueller Beschaffungsprozess (heutige Tools, Kanäle)

- Beschreiben Sie den typischen Weg von "Bedarf identifiziert" bis "Bestellung ausgelöst" Schritt für Schritt.
- Welche Systeme nutzen Sie heute aktiv (ERP, e-Procurement, Excel, E-Mail, Portale der Lieferanten)?
- Wie kommen Sie heute an Preise, Verfügbarkeit und Lead-Times bei AOT und Wettbewerbern?
- Über welchen Kanal sprechen Sie heute am häufigsten mit AOT — Telefon, Mail, Vertriebsbesuch, Portal?
- Welche Schritte würden Sie als "doppelt" oder "manuell, sollte automatisch" beschreiben?

**Notizen**:

### c. Schmerzpunkte und Reibungsverluste

- Was war der frustrierendste Beschaffungs-Vorgang in den letzten 6 Monaten — und warum?
- An welchen Stellen verlieren Sie heute am meisten Zeit?
- Wo haben Sie das Gefühl, mit unvollständigen Informationen entscheiden zu müssen?
- Welche Fehler passieren regelmäßig und sind teuer (falsche Spezifikation, falsche Charge, fehlende Dokumente)?
- Was würden Sie in Ihrem heutigen Prozess als Erstes abschaffen, wenn Sie könnten?

**Notizen**:

### d. Sample-Request-Workflow heute

- Wie lösen Sie heute eine Musterbestellung bei AOT oder anderen Lieferanten aus?
- Wie lange dauert es typischerweise von Anfrage bis physischem Muster im Labor?
- Wer in Ihrem Haus arbeitet mit dem Muster — Sie, Entwicklung, Qualität, mehrere?
- Welche Informationen müssen ein Muster zwingend begleiten (COA, SDS, Spezifikation, Allergeninfos, Bio-Zertifikat)?
- Was passiert nach dem Test — wie wird die Entscheidung dokumentiert und an den Lieferanten zurückgespielt?

**Notizen**:

### e. RFQ-Workflow heute

- Wie sieht eine typische RFQ bei Ihnen aus — Formular, Tabelle, freier Mailtext?
- Wie viele Lieferanten fragen Sie parallel an, und wie vergleichen Sie deren Angebote?
- Welche Felder einer RFQ sind für Sie verhandlungsrelevant (Preis, Menge, Incoterm, Lead-Time, Zahlungsziel, Spezifikation)?
- Wie lange dauert üblicherweise eine RFQ-Runde von Versand bis vergleichbarem Angebot?
- Was würde es bei Ihnen einfacher machen, bei AOT eine RFQ auszulösen statt bei einem Wettbewerber?

**Notizen**:

### f. Dokumentations-Bedürfnisse (SDS / COA / TDS / Zertifikate)

- Welche Dokumente brauchen Sie pro Rohstoff zwingend, bevor er ins Lager darf (SDS, COA, TDS, Bio-Zertifikat, Allergen-Statement, Kosher/Halal, weitere)?
- Wo liegen diese Dokumente heute — Mail-Postfach, Lieferanten-Portal, eigenes DMS, ERP-Anhang?
- Wie gehen Sie mit Dokument-Versionierung um — wenn ein COA pro Charge neu kommt, wie verwalten Sie das?
- Welche regulatorischen Anforderungen treiben Ihre Dokumentenliste (Bio-Verordnung, LFGB, EU-Kosmetik-VO 1223/2009, REACH, sonstige)?
- Was würde es Ihnen ersparen, wenn AOT diese Dokumente versionsgenau pro Charge per Self-Service bereitstellt?

**Notizen**:

### g. Entscheidungs-Treiber (Preis, Lead-Time, Qualität, Zertifizierung, Nachhaltigkeit)

- Wenn Sie zwischen zwei vergleichbaren Lieferanten wählen, was gibt am Ende den Ausschlag?
- Wie gewichten Sie Preis vs. Lead-Time vs. Qualität — und ändert sich das je nach Anwendung Kosmetik/Food?
- Welche Rolle spielen Bio-, Fair-Trade- oder Nachhaltigkeits-Zertifizierungen in Ihrer Lieferantenauswahl?
- Wie gehen Sie mit Preisschwankungen bei Naturprodukten um (Ernten, Saisonalität, Verfügbarkeit)?
- Gibt es "K.-o.-Kriterien", die ein Lieferant zwingend erfüllen muss, sonst kommt er gar nicht erst in Betracht?

**Notizen**:

### h. Lieferanten-Beziehungen

- Wie würden Sie Ihre heutige Beziehung zu AOT in einem Satz beschreiben?
- Was unterscheidet einen Lieferanten, mit dem Sie gern arbeiten, von einem, mit dem Sie nur arbeiten müssen?
- Wie wichtig ist Ihnen ein persönlicher Ansprechpartner gegenüber Self-Service-Tools?
- Wie messen oder bewerten Sie Lieferanten-Performance über die Zeit?
- Wann zuletzt haben Sie einen Lieferanten ausgetauscht — und was war der Auslöser?

**Notizen**:

### i. Tech-Readiness (e-Procurement, ERP-Integration, API-Erfahrung)

- Welches ERP setzen Sie ein, und wie tief ist Ihre Beschaffung darin abgebildet?
- Haben Sie Erfahrung mit Lieferanten-Portalen oder e-Procurement-Plattformen — welche, und wie waren die Erfahrungen?
- Gibt es bei Ihnen Schnittstellen-Anforderungen (Punch-out, OCI, EDI, API), die ein Lieferantenportal erfüllen müsste?
- Wie nimmt Ihre IT-Abteilung neue Lieferanten-Tools ab (Security-Review, SSO, DSGVO)?
- Wie hoch ist Ihre Bereitschaft, einen Teil des Beschaffungsprozesses ausschließlich digital über ein Lieferanten-Portal abzuwickeln — und welche Voraussetzungen müssten dafür erfüllt sein?

**Notizen**:

---

## 4. Product Developer — leichter

Sekundäre Persona. 4 Cluster, je 2–3 Fragen. Fokus: Wie beeinflusst Rohstoff-Sourcing die Formulierungs- und Entwicklungsarbeit?

### a. Rolle bei Lieferantenauswahl

- Wie sind Sie als Entwickler:in heute in die Auswahl neuer Rohstoffe eingebunden — beratend, entscheidend, gar nicht?
- An welcher Stelle im Beschaffungsprozess kommt Ihr Input typischerweise rein?
- Welche Lieferanten-Eigenschaften sind aus Ihrer Entwickler-Perspektive nicht verhandelbar?

**Notizen**:

### b. Sample-Testing-Prozess

- Beschreiben Sie, was mit einem AOT-Muster ab dem Moment passiert, in dem es bei Ihnen ankommt.
- Wie viele Iterationen Muster → Test → Feedback → neues Muster sind typisch, bis Sie eine Freigabe geben?
- Welche Informationen fehlen Ihnen am häufigsten zu einem Muster — und welche Folge hat das?

**Notizen**:

### c. Technische Dokumente, die wirklich gelesen werden

- Welche Dokumente lesen Sie tatsächlich vor einem Test (TDS, Spezifikation, COA, Application Notes)?
- In welchem Format brauchen Sie diese Dokumente, damit Sie damit produktiv arbeiten können?
- Welche Information vermissen Sie regelmäßig in den heute gelieferten Dokumenten?

**Notizen**:

### d. Was bremst Formulierungs-/Entwicklungsarbeit?

- Was sind die größten externen Bremsen (Lieferanten-bezogen) in Ihrer Entwicklungsarbeit?
- Wo erleben Sie Reibung, die nichts mit der eigentlichen Formulierung zu tun hat (Beschaffung, Doku, Kommunikation)?
- Was würden Sie an einem Lieferantenportal als Erstes nutzen, wenn AOT eines hätte?

**Notizen**:

---

## 5. Quality Manager — leichter

Sekundäre Persona. 4 Cluster, je 2–3 Fragen. Fokus: Compliance, Audit, Traceability.

### a. Compliance & Regulatorik (Bio, Lebensmittel-/Kosmetikrecht)

- Welche regulatorischen Rahmen sind für Sie bei organischen Rohstoffen die wichtigsten (EU-Bio-VO 2018/848, LFGB, EU-Kosmetik-VO 1223/2009, REACH, FDA, weitere)?
- Wie unterscheidet sich Ihre Compliance-Prüfung zwischen Kosmetik- und Food-Anwendungen?
- Welche Compliance-Schritte müssen abgeschlossen sein, bevor ein neuer Lieferant qualifiziert ist?

**Notizen**:

### b. Audit-Trail-Bedürfnisse

- Welche Vorgänge müssen Sie heute lückenlos nachvollziehen können (Bestellung, Wareneingang, Freigabe, Reklamation)?
- Wie führen Sie heute Nachweise gegenüber Auditoren — Papier, ERP-Export, separates QM-System?
- Welche Audit-Fragen kommen am häufigsten und sind heute am schwersten zu beantworten?

**Notizen**:

### c. Zertifikate-Management (Empfang, Versionierung, Audit)

- Wie kommen Lieferanten-Zertifikate heute zu Ihnen, und wie werden sie abgelegt?
- Wie stellen Sie sicher, dass ein Zertifikat zum Zeitpunkt der Verarbeitung gültig war (Versionierung, Ablauf-Datum, Charge-Bezug)?
- Wie viel Zeit verlieren Sie pro Monat allein durch Zertifikat-Suchen oder -Nachforderungen?

**Notizen**:

### d. Traceability / Batch-Rückverfolgbarkeit

- Bis auf welche Granularität müssen Sie eine Charge zurückverfolgen können (Lieferant, Sub-Lieferant, Erntejahr, Region)?
- Wie häufig haben Sie reale Rückverfolgungs-Fälle pro Jahr (Reklamation, Rückruf, Audit-Stichprobe)?
- Welche Informationen vom Lieferanten sind für eine saubere Rückverfolgung heute kritisch — und welche fehlen Ihnen am häufigsten?

**Notizen**:

---

## 6. Notizen / Offene Fragen / Emerging Insights

Freie Capture-Sektion. Pro Interview kurz nachbereiten — innerhalb von 24 Stunden, solange der Eindruck frisch ist.

- **Überraschende Erkenntnisse**:
  - …

- **Offene Fragen für Follow-up**:
  - …

- **Hypothesen-Updates** (welche Annahme aus `AGENTS.md` bestätigt / widerlegt / verschoben?):
  - …

---

## 7. Quellen / Cross-Refs

- [`AGENTS.md`](../AGENTS.md) — Personas, Project Context, ERP Ownership Rules.
- [`docs/workflow.md`](./workflow.md) — Collaboration Protocol, KOMPAKT-Stil.
- [`ROADMAP.md`](../ROADMAP.md) — Sprint 0 Discovery-Deliverables.
