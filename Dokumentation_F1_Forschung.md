# SHA-256 Forschung – Dokumentation: Von liesen.txt bis F3

Datum: 13.09.2026
Arbeitsverzeichnis: `C:\Users\walee\Desktop\server\mathe SHA 256`
Ausgangsdatei: `liesen.txt` (arabisch, 1 Zeile)
Constraint: KEINE gute CPU – zuerst theoretisch lösen, dann Code.

---

## 1. Ausgangstext (liesen.txt) – Zusammenfassung

Der Text behauptet:
- Bester Weg für SHA-256-Bruch = Hybrid aus SAT-Solvern + differentieller Kryptoanalyse + Neutral Bits (SAT-aided Differential & Neutral Bits Cryptanalysis).
- Reine Gröbner-Basen scheitern (Degree of Regularity explodiert nach wenigen Runden, Memory).
- Reine manuelle Analyse scheitert (64 Runden, Rotationen, Carry-Ketten).
- Automatisierte differentielle Suche (wie bei MD5/SHA-1) + SAT-Formulierung für Local Collisions + Neutral Bits ist der einzige Weg mit Werkzeugen für kombinatorische Komplexität.
- Generelle Solver (CaDiCaL, Kissat) verstehen keine Krypto → braucht Domain-Heuristiken (Branching auf W_i Message Schedule, Carry-Ketten chronologisch).
- Kein One-Shot 64 Runden, sondern schrittweise: Differential Mitte (z.B. Runden 20–45) + Biclique/MitM an Rändern.
- Bruch beginnt nicht mit Preimage, sondern praktischer Collision für 40–45 Runden.

## 2. Meine Meinung nach Internet-Recherche (2024–2026)

Geprüft mit WebSearch: ePrint 2026/1120, 2026/232, 2026/1080, 2024/349, Alamgir/Nejati/Bright 2024 (SAT+CAS), Khovratovich Biclique 2011, Guo Dual-Syncopation 2026/353.

Ergebnis: Text liegt im Kern auf aktueller Forschungslinie.
- Bis 2013 Stillstand bei 31-Step Collision + 38-Step SFS (Mendel et al. EUROCRYPT 2013), 10+ Jahre.
- Durchbruch 2024 Li et al.: SAT/SMT Sparsity-Control → praktische 39-Step SFS, 31-Step 2^65.5 → 2^49.8 → praktisch.
- 2026 Zhang et al.: erste 37-Step Collision 2^119.1 + 36-Step 2^94.4 (+6 Steps nach 12 Jahren) via automatischer Local-Collision-Suche + exakter IF/MAJ-Modelle.
- 2026 Li et al.: 38-Step Collision 2^104.3, praktisch 35-Step, 39-Step SHA-512 2^178, 39-Step SHA-256 Collision scheitert (low-probability uncontrolled part).
- Pure SAT nur ~28 Steps (Prokop 24, Nejati 25, plain CaDiCaL 28). SAT+CAS mit IPASIR-UP (wordwise Propagation, inconsistency blocking) → 38-Step SFS. Bestätigt These zu Domain-Heuristiken.
- Preimage: 45 Runden (2011 Biclique) → erst 2026 46/47-Step (Dual-Syncopation), 51-Step SHA-512. Alles ~2^255, nur theoretisch.
- Kritik/Ergänzung: Neutral Bits allein nicht Schlüssel (heute implizit via Solver + Message Modification). Wichtiger: Wahl der Local Collision + exakte IF/MAJ-Modellierung + Encoding (MILP Liu 2023 → SAT/SMT), nicht nur Branching.

Fazit damals: Hybrid-Pfad ist Konsens seit 2024. Volle 64 sicher. Praktischer Break käme als 40–45 Runden Collision.

## 3. Phase 1 – 4 Forschungs-Subagenten (mit CPU-Annahme, verworfen)

Auf Wunsch Subagenten gestartet:
1. **SAT-Differential:** Top 5 Papers, Encoding (signed differences, Sparsity Objective, Contradiction Monitoring), Heuristik-Rezept, Prognose (40-Step SFS machbar, 39-Step Collision >2^128 derzeit infeasible).
2. **Algebra + MitM:** Pure Gröbner verliert (>2000 Variablen bei 20 Runden, >18000 bei 64, Carry-Grad explodiert). Hybrid nur als Propagator (CryptoMiniSat XOR, CDCL(Crypto), SAT+CAS 28→38, Autoguess, Triangulation). MitM/Biclique stark an Rändern, Differential stark in Mitte → 3-Zonen-Split Vorschlag.
3. **Neutral Bits / Local Collisions:** Geschichte Biham-Chen → Wang Message Modification → Mendel GnD. Objective für LC-Suche (wenige aktive W, wenige Cancellations, freie Prefix-W n≥8 direkt / n≥5 2-Block / n=0 nur SFS). Workflow A-B-C (Wort-Ebene → Bit-Ebene → Verifikation).
4. **Praxis-Roadmap:** Toolchain STP + CaDiCaL/IPASIR-UP + Python-Verifikation. Meilensteine M0–M5 (24-Step → 28-Step → 31-Step → 38/39 SFS → eigene LC). Abgelehnt vom Nutzer wegen schwacher CPU.

## 4. Phase 2 – 4 Theorie-Subagenten (Low-CPU, Denken vor Rechnen)

Prinzip: Download statt Suchen, Toy statt Full, Papier statt Cluster.

A. **Mathe-Struktur:** Expansion wegen SHR nicht rotationsinvariant (Unterschied zu SHA-1). Ghost Differences Typ 1+2 (Theorem 1 Mendel 2008/130). Kette GH 9-Step 2^-66 → Hawkes modular 2^-39 → Sanadhya 16 saubere Locals 2^-42–2^-54 → Mendel t=11/14/18 → Li 2024. State: Ch/Maj per 8-Zeilen-Tafel zählbar (IF (0,1,1)→1 immer, Kosten 1/2 pro Bit). Sigma bijektiv, keine iterativen Invarianten. ADD: MSB gratis, LSB 1/2 pro Carry. 3 Hypothesen H1–H3 für Sekunden-Tests.

B. **Literatur-Synthese:** Ideen-Geschichte (Wang Modification, Biham Neutral Bits, Mendel Starting Points, Liu MILP, Zhang exakte Modelle, Biclique, Dual-Syncopation). Wiederverwendbare Tabellen: 2021/292 Table 2/4 (31+38-Step), 2406.20072 Table 4/5 + 6–9 (38-Step SFS + Starting Points), 2011/037 Table 2/3 (Second-Order 1–46), 2024/349 Sec 4–5 (39-Step SFS), 2026/232 Table 3 (37-Step Locals), 2026/1120+1080 (Limit-Analyse). 3 Forschungsfragen F1/F2/F3 formuliert.

C. **Low-Resource Methodik:** Linear-SHA, Mini-SHA 8-bit, 1 LC isoliert, Carry mit 2-bit Beispiel, Micro-SAT (nur W16..W23, nur 1x Ch). Gratis-Ressourcen: Colab, Kaggle, Codespaces, Narval. Repos nur zum Lesen: cadical-sha256, cryptanalysis, sha_2_attack, cryptosym, CDCL-Crypto. 5 Mini-Experimente <1 Min.

D. **Alternative Theorie:** Rotational tot für volles SHA-256 (>300 ADDs, 64 Kt), Rebound/Integral/Zero-Sum nie über Distinguisher, Second-Order/Boomerang einzige CPU-arme Erfolgsgeschichte (47 Runden 2^46). MITM-Baukasten auf Papier (Splice-and-Cut, neutrale Wörter + Compensation, Initial Structure, Partial Matching, Biclique, Triangulation 2026/598). 3 Richtungen A/B/C mit Papier-Hypothesen.

## 5. F1 – IF/MAJ Bedingungszählung

Frage: Werden IF/MAJ-Bedingungen in publizierten Trails über- oder unterschätzt?

### 5.1 F1-Subagent Ergebnis (Behauptung)
- IF/Ch y = g ^ (e&(f^g)): max 1 Bedingung, nie 2. (-,x,x)→x forciert Kosten 0. (x,-,-) 1 Joint f^g=c statt 2 Fixes.
- MAJ: w=1/w=2 je 1 Joint, nicht 2. (x,x,x)→x forciert 0, →- unmöglich.
- Netto 2–4 Bits = Faktor 4–16 pro Trail nur Ch/Maj. Erklärt 31→37 Sprung + 2 zusätzliche Null-Diffs bei Alamgir.
- Modell-Vorschlag: Single-Bit-Projektion (min-Control) + Two-Bit-Layer (x^y=z Graph, ungerader Zyklus = Widerspruch).
- Limitation: keine bitgenauen Step-Nummern (Mendel paywalled, PDFs binär).

### 5.2 F1-Verifikation (2 Gegen-Agenten + eigener Test)
Eigener Python-Test (alle 64 (v,v') Paare):
- Ch (0,1,1): N-=0 Nx=8 → bestätigt 0 Kosten.
- Maj (1,1,1): N-=0 Nx=8 → bestätigt 0 Kosten.
- Alle anderen mit Diff: 4/4 → Kosten 1.
→ Kein XOR-Fall kostet 2. In Sekunden auf Laptop reproduzierbar.

Verifikations-Agent: BESTÄTIGT mit Präzisierung.
- Ch -xx: -2, Ch x-x/xx-: -1 (braucht 3er-Joint f^g^e), Ch xxx: -2, Maj w2: -1, Maj xxx: -3.
- x-- numerisch gleich (1=1) aber qualitativ Joint statt Fix.
- Netto 2–4 Bits nur als obere Schranke pro Muster, nicht pro Trail garantiert ohne geparsten Trail.
- Literatur bestätigt Muster (CRYPTREC DDT, Mendel Obs.1+2, Alamgir Two-Bit-Blocking, Table 5 sparsamer).

Gegenprüfungs-Agent (Widerlegungsversuch): Joint notwendig, nicht hinreichend.
1. Minefield: Mehrheit Trails bleibt nach 2-Bit-Gauss ungültig (Mendel 2011 Fig.3, Sigma vs Maj/Ch 4er-Zyklen a=b=c≠a).
2. Alamgir: Blocking im Mittel negativ, Propagation >> Blocking (28→38 kommt von Propagation).
3. Feng/Gao 2026/1505 Disclaimer: neither speedup nor new attack, CaDiCaL ja / Kissat nein.
4. Kostenaufteilung: Local Collision 2^-66 = 2^-48 ADD + 2^-18 Ch/Maj (73% ADD). Tail 38-Step 2^104.3 dominiert. 39-Step SHA-256 unmöglich wegen uncontrolled part.
5. Zhang: joint (x,y,z) zählt zu viel, korrekt ist variabel-spezifisch weniger.

### 5.3 F1-Fazit
Lokal wahr, global begrenzt. Saubere Cofaktor-Charakterisierung, nützlich als redundante BCP-Verstärkung. Praktisch Rang 3 nach (1) Sparsity/LC-Wahl und (2) Propagation/MITM-Modification. Kein 64-Runden-Break, aber Tool-Rangfolge-Dreher.

Mini-Prüfung für Laptop (<10s, nur Zählen): alle 64 Paare pro F enumerieren, pro (d,Ausgang) minimale Bedingung log2(8/4)=1 über Kandidaten e=0/1, f^g=0/1 berechnen, mit naiv vergleichen.

## 6. F3 – Uncontrolled Part: strukturell oder Pech?

Frage: Ist der low-probability uncontrolled part strukturell oder nur Pech des gewählten Trails?
Urteil F3-Subagent: überwiegend strukturell (80/20). Kein Retuning innerhalb bekannter W-Familien macht aus 39-Step SHA-256 >2^128 ein <2^128. Es fehlen ~25–35 Bit.

### 6.1 Definition kontrolliert vs unkontrolliert
- State: T1 = H + Sigma1(E) + Ch(E,F,G) + K_i + W_i, T2 = Sigma0(A) + Maj, E' = D + T1, A' = T1+T2.
- Expansion: W_i = sigma1(W_{i-2}) + W_{i-7} + sigma0(W_{i-15}) + W_{i-16}, i>=16.
- Kontrolliert = Starting Point, Mitte Steps 6–23 (LC, dicht, per Message Modification / MITM / SAT deterministisch gelöst). Braucht freie W-Bits.
- Unkontrolliert = Head 0–5 (IV fix, keine Freiheit vorwärts) + Tail 24–37 (alle W fix, muss probabilistisch abklingen).
- Trail-Wahrscheinlichkeit = Hamming weight + Bit-Conditions (Zhang 2026/232).
- SFS→Collision: erste 8 (min 5–6) W differenzlos nötig, sonst kein Collision unter Birthday. Für SHA-256 X>128 freie Bits nötig (Mendel 2-Block).
- Quellen: ePrint 2026/1120 (38-Step 2^104.3, 39-Step SHA-512 2^178, 39-Step SHA-256 unmöglich), 2026/232 (37-Step LC 6–23, W6,W7,W9,W14,W22,W23), 2021/292 (Mendel-Framework X>128), Rechberger PreliminaryAnalysis (Single-Bit Expansion), RFC6234 (Ch/Maj/Sigma), arXiv 2406.20072 (Starting Points).

### 6.2 Tail-Analyse
Teilung: 0–5 Head dünn (W0..W5=0 für Conversion), 6–23 kontrolliert (LC), 24–37 Tail dicht.
1. Tail dicht weil ab i>=16 keine Löschungs-Freiheit mehr: sigma expandiert jedes Rest-Bit ~3-fach. Single-Bit über 64 Steps → Gewicht 467 minimal, 356 optimiert (Rechberger). Pro Tail-Step wächst HW(ΔW) um mehrere Bits.
2. A-Pfad dominiert E-Pfad: Ch ist MUX (mit Δe=0 billig), klingt in ~4 Steps ab (E→F→G→H). Maj symmetrisch nicht wegmuxbar + Sigma0 3 Rotationen + A persistiert 4 Steps als B,C,D → hallt länger. Hawkes-Anker: eine 9-Step LC 2^-39..2^-42. Tail = Kette erzwungener Mini-LCs, ~3–6 Bit pro Step. 8–10 Steps = 2^-50..2^-90 allein. Darum 36-Step 2^58, 37-Step 2^119, 38-Step 2^104.3 (besseres Modeling/MITM, nicht leichterer Trail).
3. 39-Step kippt: +1 Step hinten = dichter Expansionsschritt + Abkling-Step gegen Maj/Sigma. +20–30 Bit. 2^104 * 2^25 ≈ 2^129 > 2^128 Birthday → nur noch SFS.
- V-Shift V→V+1: vorne ~3–5 Bit gespart (dünn), hinten ~8–12 Bit bezahlt (dicht). Netto -5 bis -8. V-1 zerstört W0..W5=0. Zhang hat alle V exhaustiv gesucht ohne 38/39-Fenster → Strukturschranke.
- i=3 Shift (Li 2026/1080) derselbe Effekt in Message-Richtung: Tail-Prob zu niedrig.

### 6.3 Urteil: 80% strukturell, 20% Muster-Pech
Strukturell: exponentielle Tail-Steuer (sigma + Maj/Sigma0 + A-Persistenz), Birthday-Mauer 2^128, W0..W5=0 fixiert Fenster vorne, zwei Teams scheitern unabhängig an selber Stelle.
Muster-Anteil: alle LCs aus selber Mendel-Idee (Span t=14..25, XOR-Gewicht + nachträgliche Conditions). Neuer LC-Typ (modulare Carry-Tricks, Single-Bit Startwort Gewicht 356) könnte 2–4 Bit holen. Exakte Ch/Maj-Modelle erst seit 2026/232 → 1–3 Bit Suchluft. Aber für 39-Step fehlen 25–35 Bit. Drei Ziele (längerer Span + dünnerer Tail + W0..W5=0) widersprechen sich. Quasi-prinzipielle Schranke für Single-LC-Trails; nur Multi-LC / anderer X-Tradeoff könnte umgehen.

### 6.4 Idee für 2–4 Bit Tail-Gewinn (ungetestet)
E-Schwerpunkt-Trail mit Tail-W_{n-1}=W_n=0 + alle Tail-ΔE auf MSB Bit 31 + Selektor e=0 ausnutzen (ΔCh=0 gratis wenn Δf=Δg). Begründung: Ch mit e differenzlos kontrollierbar, MSB minimiert ADD-Carrys (Hawkes 2^-66→2^-39). Erwartung 1–2 Bit aus Null-W + 1–2 aus Ch/MSB. Nicht genug für 39-Step, könnte 38-Step 2^104→2^100 drücken. Variante: V+1-Shift kombiniert mit dünnerem W4+i-Muster (allein negativ, als Paar neutral/leicht positiv).
Limitation: keine bitgenauen Tabellen verifiziert, Tail-Zahlen Größenordnungen aus 2^58/2^104/2^119/2^178, IF-vs-MAJ ohne Li-Tabelle nicht belegbar, K/Rotationsunterschiede SHA-256/512 als zweitrangig eingestuft, Idee ungetestet ob W_tail=0 + MSB-Ch + W0..W5=0 gleichzeitig erfüllbar.

### 6.5 SHA-256 vs SHA-512 Asymmetrie
Gleiche Tail-Steuer ~2^130–150 ist über 2^128 (SHA-256 tot) aber unter 2^256 (SHA-512 mit 2^178 gültig). Mehr Freiheit: 1024-Bit Block / 64-Bit Worte vs 512/32 → MITM hat doppelt Material, Kosten skalieren nicht doppelt.

## 7. F2 – 6-Wort Local Collision mit Null-Prefix W0..W7=0

Frage: Welche 6-Wort LC erlaubt V>=8 (direkte Collision) statt V=5/6 (2-Block/SFS)?
Ergebnis F2-Subagent: Einzige bekannte 6-Wort mit W0..W7=0 ist Mendel 28-Step (V=8). Alle 31+ Collision-LCs haben V=5/6. Zwei neue Shift-Kandidaten vorgeschlagen (ungetestet).

### 7.1 Expansion + Definition
W_i = m_i (i<16), W_i = sigma1(W_{i-2}) + W_{i-7} + sigma0(W_{i-15}) + W_{i-16} (i>=16).
Linearisiert: dW_i = L1(dW_{i-2}) XOR dW_{i-7} XOR L0(dW_{i-15}) XOR dW_{i-16}.
Def 2026/232: V = erstes i mit dW!=0, E = letztes, S = aktive Menge, t = E-V+1. Cancellation = i>=16, i not in S, dW=0 gefordert.
Conversion: V>=8 direkt, V>=5..6 2-Block (X>128), V=0 nur SFS.

### 7.2 Tabelle bekannter LCs
- Mendel 28-Step: W8,W9,W13,W16,W18 (5), V=8 E=18 t=11, direkt, 3–4 Cancellations.
- Mendel 31-Step: W5..W9,W16,W18 (7), V=5 t=14, 2-Block 2^65.5→2^49.8.
- Mendel 38-SFS: W7,W8,W10,W15,W23,W24 (6), V=7 t=18, nur SFS.
- Zhang 37-Step: W6,W7,W9,W14,W22,W23 (6), V=6 t=18, 2-Block 2^119.1, 6 Cancellations 16,21,24,25,29,30.
- Zhang 36-Step: W5,W7,W8,W13,W21,W22 (6), V=5 t=18, 2^58/2^94, 5 Cancellations.
- Li 2026/1080 Familie: (W4+i..W8+i,W12+i,W13+i,W20+i,W22+i) mit 9 aktiven (!) i=0..3 → 35 prakt i=0, 36/37 theor i=1/2, 38 fail i=3. Vorteil MITM-Form nicht Sparsity.
Korrektur: Li hat 9 aktive, nicht 6.

### 7.3 Warum V>=8 härter ist (Papier-Zählung)
Mit dW0..7=0 gilt für 16–22: dW16 = L1(dW14) XOR dW9, dW17 = L1(dW15) XOR dW10, … Erst ab i=23 kommt L0(dW8) dazu. Bei V=6 schon ab i=21 (L0(dW6)). V=8 hat 2 zusätzliche reine 2-Term-Gleichungen → restriktiver, ~2 Bit Mehrgewicht.
Pro Bit: L1/L0 1→~3 Positionen (SHR verliert Top-Bits → MSB Bit 31 minimal). Loch-Gleichung 0 = L1(a) XOR b: b folgt, Gewicht ~1+3=4. 6 Worte = 192 Positions-Variablen, 6×32 Constraints Rang <192 → Wort-Lösungen existieren (Zhang OBJ = sum flag). Bit-Gewicht entscheidet danach.

### 7.4 V-Shift + i-Shift
V→V+1: vorne ~3–5 gespart, hinten ~8–12 bezahlt (sigma 3-fach + Maj-Nachhall). Netto -5 bis -8. i=3 V=7 E~25 probiert → uncontrolled zu niedrig. Zhang 14≤t≤25 V≥0 exhaustiv ohne 38/39-Fenster. X>128 braucht V≥5–6 (5×32=160 roh). V=8 X~256 conversion-leicht aber Tail schwer. Drei Ziele (längerer Span + dünnerer Tail + W0..W7=0) widersprechen sich → 80/20 strukturell.

### 7.5 Zwei neue Kandidaten (Gap-erhaltend, ungetestet)
Prinzip: Zhang-Gaps erhalten, +2/+3 shiften.
- K1 geshiftete 37er direkt: S={8,9,11,16,24,25}, V=8 E=25 t=17 (Gaps 1,2,5,8,1). Prefix 8 JA direkt. 6 aktiv, 6 Cancellations erwartet 18,23,26,27,31,32. Preis: Tail endet 25 statt 23 → 2 Steps länger, 31/32 dicht.
- K2 geshiftete 36er direkt: S={8,10,11,16,24,25}, V=8 E=25 t=18 (Gaps 2,1,5,8,1). Prefix 8 direkt. 6 aktiv 5–6 Cancellations. Preis: Gleichung 21/22 ohne L0.
Beide einzige 6-Wort-Shifts mit Prefix-8 + identischer Gap-Topologie. Jeder andere Gap-Bruch braucht neues Tool.
Objective: OBJ = |S| + C + lambda*HW_lin + mu*max(0,8-V) + nu*TailWeight. Zhang nur |S|+C. Gut = klein.
Limitation: XOR ignoriert Carrys (2^-48 ADD-Anteil), Ch/Maj 2^-18 ignoriert, SHR nur MSB-Heuristik, keine Odd-Cycle-Prüfung, keine 39-Step Aussage (zielt 37/38 direkt).

### 7.6 Mini-Test Ergebnis (13.09.2026, Laptop, <10s, nur XOR-Expansion)
Methode: GF(2)-Gauss über 16×32=512 Message-Bit-Variablen, Expansion L0/L1 linearisiert, Nullen für alle i∉S bis E erzwungen, Tail nach E frei. Kontrolle: bekannte Muster müssen lösbar sein.
- Zu streng zuerst (Nullen bis 32): alle UNLÖSBAR inkl. Mendel/Zhang → Test falsch, korrigiert auf N=E+1.
- Korrekt bis E:
  - Mendel28-V8: lösbar null=96, alle 5 aktiv ≠0 möglich.
  - Mendel31-V5: lösbar null=160.
  - Zhang37-V6: lösbar null=64.
  - Zhang36-V5: lösbar null=96.
  - K1-V8 {8,9,11,16,24,25} E=25: LÖSBAR null=32, alle 6 aktiv ≠0 möglich. Erste Nullraum-Lösung Kosten 18 (Worte z.B. 0x1, 0x2004000, 0x28008150, 0x2004000, 0x10440801, 0x2004000). Schwerer als Zhang (Kosten 10 in erster Lösung, null=64) → plausibel aber schlechter, Tail endet 25.
  - K2-V8 {8,10,11,16,24,25} E=25: UNLÖSBAR null=0, kein Wort kann ≠0 sein → VERWORFEN.
- Random-Sparse-Suche (nur 1–2 Bit MSB-Werte, 30000 Trials): keine Lösung für Zhang/K1/K2 → Lösungen brauchen dichtere Worte, kein Gegenbeweis (Nullraum-Beweis zählt).
Fazit: K1 überlebt Wort-Ebene knapp (null=32 kleinste Freiheit), K2 tot. K1 ist direkter Collision-Kandidat (Prefix 8) aber mit schwererem Tail als Zhang37.

### 7.7 K1-Optimierung Ergebnis (13.09.2026, 3 Subagenten)
K1-F1 (exakte Zählung): Bereiche Anfang 8–12 (2×w2, 1×xxx-Kandidat), Mitte 16–20 (1×x-- Selektor-Joint, 1×w2), Ende 24–29 (1×x--, 3×w2/Decay). Naiv ~44 vs exakt ~20, max -24, realistisch nach Zhang-Korrektur (kein Doppelzählen von -xx, kein xxx das via Odd-Cycle invalid wird): -3 bis -4 Bit. Tail-Verlust +2 Steps = +6 bis +12 Bit, F1 holt ~1/3 zurück. Netto K1 vs Zhang37 nach F1 allein: +5 bis +11 schwerer. F1 drückt K1 nicht unter Zhang.
K1-F3 (Tail + MSB): dW26 = L1(dW24) XOR L0(dW11), dW27 = L1(dW25) XOR dW11, 28–30 dicht, 31/32 am dichtesten (Rückkopplung W24/25/16 via W_{i-7}/W_{i-16}). A-Pfad dominiert (Maj + Sigma0 + 4-Step-Persistenz), E/Ch billig via Selektor (Δe=0, Δf=Δg gratis). W_tail=0 exakt mit null=32 fast sicher unverträglich (64 Bit-Bedingungen); dünn (HW 1–3) stattdessen. MSB-Rechnung von Hand: L0(0x80000000) = {24,13,28} Gewicht 3, L1(0x80000000) = {14,12,21} Gewicht 3. MSB gewinnt auf ADD-Seite (kein Carry, spart ~27 Bit Carry vs ~1 Bit Extra-Expansion). Vorschlag: 24/25 auf 1-Bit-MSB abspecken, 26+ E-schwer mit Δe=0 + Δf=Δg, ~2–4 Bit erhofft (38-Step 2^104→2^100, nicht 39-Step).
K1-Gewicht (Low-CPU Plan <60s): null=32 = 2^32 zu groß für brute force. Plan: PRE Basis (5s) + Greedy (2s) + Hill-Climb p=1,2 (10s, 18→13/14) + Stern-ISD p=2 l=10 (30s, →11/12) + SA-Polish (10s). Schätzung: 18→14 fast sicher, 14→12 realistisch, 12→10 ~20%, <10 unwahrscheinlich weil TailWeight (E=25) in OBJ dominiert. Kriterium: lebt wenn HW≤12 + TailWeight<20 in <60s; pari bei HW=10; beerdigt wenn 3×60s kein HW<15 oder Tail≥22 bleibt.
Gesamt-Urteil: K1 lebt als Theorie-Kandidat weiter (direkter Prefix-8, lösbar), aber schwächer als Zhang37 (+4 bis +8 Bit nach F1+F3). Kein 39-Step, allenfalls direkter 35/36 mit gekürztem Tail oder SFS-Startpunkt. Nächster Schritt nach Plan oben 60s Hill+Stern: fällt nicht unter 14 → keine CPU mehr verbrennen, neue Gap-Familie statt Shift.

### 7.8 Modifizierter MSB-Test (13.09.2026, Vorgabe aus liesen 2)
Frage: hilft MSB-Restriktion auf dW24/dW25 mit OBJ = F1+Tail statt rohem XOR-Gewicht? Ergebnis Laptop (<60s, Seeds 1–2, 30–40k Trials):
- MSB-exakt (W24=W25=0x80000000, 64 affine Gleichungen): UNLÖSBAR (Gauss inkonsistent) → F3-Forderung „exakt dünn" tot, wie in 7.7 vermutet.
- MSB-set (nur Bit31=1 auf 24+25, 2 affine Gleichungen): LÖSBAR, null 32→30. Aber rohes OBJ (HW_S + Tail 26–32) schlechter: frei ~33–49 vs MSB-set ~78 (Seed 2). MSB kostet auf XOR-Seite, spart erst auf ADD/Carry-Seite (~27 Bit laut Hawkes) → rohes HW darf nicht entscheiden (liesen-2-Kritik bestätigt).
- K1 vs Zhang roh (Seed 1, ungleiche Tail-Fenster 7 vs 9 Worte): K1 33 (18+15) vs Zhang 40 (18+22) → kein fairer Vergleich, keine Aussage.
- MSB-Korrektur aus liesen 2 ({25,14,28}) zurückgewiesen: ROTR-Regel j=31−n gibt {24,13,28} für L0, {14,12,21} für L1 bleibt. Off-by-one in liesen 2, Gewicht 3 und Schluss unverändert.
Fazit: Hill+Stern auf rohem HW kann K1 weder retten noch beerdigen. Beerdigungs-Kriterium umgestellt: K1 scheitert strukturell (Tail +2 Steps, null=32 dünnste Freiheit, MSB-exakt unmöglich), nicht an HW<14. Als 39-Step-Kandidat beerdigt, als 35/36-Edge oder SFS-Startpunkt offen.

### 7.9 K1 als 35/36-Edge: liesen-2-Zahlen + 3 Subagenten + fairer Lauf (13.09.2026)
liesen 2 neu: K1 18+25=43 (36-Step) vs Zhang36 10+161=171 / Zhang37 31+105=136, K1-Tail 26–35 ~2.5/Step.
Agent 1 (Verifikation): null 32/96/64 plausibel; HW/Tail-Zahlen sind Einzel-Lösungen kein Mustervergleich (10 vs 31 = dünnste vs dickste Lösung, Doku 7.6 sagte Zhang37-Erst-Kosten 10 → Label/Seed-Wechsel); 2.5/Step fast cancellations-frei über 10 Steps widerspricht Ghost-Theorem + F3-Steuer → Rekord braucht Log; Fenster ungleich (10 vs 12 vs 13 Worte) + nur W ohne State/BD + Conversion-Prämie ignoriert → Vergleich unfair.
Agent 2 (Vollkosten): Formel -log2P = H_word + C_F1 + C_ADD + C_EXP + C_TAIL − R_MOD + C_CONV. Zhang 171→94.4 = 77 Bit Rabatt (SFS 2^12.6 + Conversion +82; Li-Pipeline sogar →2^57, Rabatt 114). R_MOD braucht Freiheit: null=96 schluckt ~60–80 Prefix-Conditions, null=32 nur ~20–30. Conversion-Vorteil K1 direkt ~+60–80 Bit wiegt +8 HW (18 vs 10) bei weitem auf, aber Freiheits-Nachteil ~−50 frisst ihn wieder. Schätzung K1_real ~65–90 vs Zhang 94.4/57 → gleiche Liga wie Zhang-Original, hinter Li-Pipeline.
Agent 3 (Urteil): OFFEN Tendenz negativ. Lebt als SFS-Start/35-Edge, tot als bewiesener 36-Sieger. Sieg-Kriterium: C_K1(0–35, direkt) < 2^94.4/2^57 bei gleichem Zählmodell + Odd-Cycle-frei + konkreter Lösung. Roh 43<171 gilt nicht.
Eigener fairer Lauf (gleiches Fenster W0–W35, gleiche 30k Trials, Seed 7): K1 total 66 (S=30, 1.83/Step) vs Zhang37 124 (S=15, 3.44) vs Zhang36 140 (S=9, 3.89). Roh gewinnt K1 fair — bestätigt dünnen Tail, aber ohne ADD/F1/R_MOD/Conversion keine Angriffs-Aussage. Nächster Mini-Test (A Gauss fair + B F1-Zählung + C Odd-Cycle) steht aus; ohne Voll-OBJ keine CPU mehr verbrennen.

## 8. Offene Fragen / Stand
- F1 erledigt + verifiziert (lokal wahr, global Rang 3).
- F3 erledigt (80% strukturell, Idee E-Schwerpunkt + MSB für 2–4 Bit).
- F2 erledigt + Mini-Test: K1 LÖSBAR (null=32, Kosten 18), K2 VERWORFEN (null=0).
- K1-Optimierung erledigt (3 Agenten): F1 -3/-4, F3 +2/-4 Idee, Gewicht-Plan <60s. Urteil: K1 lebt schwach, +4 bis +8 über Zhang.

Nächster Schritt: schreiben (Paper-Dokument aus F1/F3/K1/K2), Boomerang/Gap-Familie erst danach.

---
Quellen (Auswahl): ePrint 2015/350, 2021/292, 2024/349, 2026/232, 2026/1080, 2026/1120, 2026/1505, 2026/353, 2026/598, 2011/286, 2008/130, 2008/142, 2007/352, arXiv 2406.20072, FIPS 180-4, Gilbert-Handschuh SAC 2003/2004, Hawkes et al. 2004/207, Biham-Chen 2004/146, Wang 2005/400.
