# SHA-256 Local Collisions unter Low-CPU-Bedingungen: F1-Zählung, F3-Tail-Schranke und Bewertung des Prefix-8-Kandidaten K1

Datum: 13.09.2026. Constraint: Laptop ohne starke CPU — Theorie vor Rechnen, Sekunden-Tests statt Cluster.

## Abstract

Wir untersuchen, ob eine direkte 6-Wort Local Collision mit Null-Prefix W0..W7=0 (V>=8) den Stand (37-Step Collision 2^119.1, 38-Step 2^104.3) verbessern kann. Ergebnisse: (1) Exakte Ch/Maj-Zählung spart 2–4 Bit pro Trail (lokal verifiziert, global Rang 3 hinter LC-Wahl und Propagation). (2) Der Tail 24–37 ist zu ~80% strukturelle Schranke (+20–30 Bit pro Extra-Step, ~25–35 Bit fehlen für 39-Step). (3) Von zwei neuen Shift-Kandidaten ist K2 linear widerlegt (null=0), K1 lösbar aber schwächer als Zhang37 (+4 bis +8 Bit) und als 39-Step-Kandidat beerdigt. MSB-exakt auf dW24/dW25 ist linear unmöglich, MSB-set möglich aber roh teurer. Alle Aussagen sind auf Laptop in Sekunden reproduzierbar (GF(2)-Gauss über 512 Variablen).

## 1. Stand

Bis 2013 Stillstand (31-Step Collision, 38-Step SFS, Mendel et al.). 2024 Li et al.: SAT/SMT Sparsity-Control → 39-Step SFS praktisch. 2026 Zhang et al.: 37-Step Collision 2^119.1 + 36-Step 2^94.4 via automatischer LC-Suche + exakter IF/MAJ-Modelle. 2026 Li et al.: 38-Step Collision 2^104.3, 39-Step SHA-512 2^178, 39-Step SHA-256 scheitert (low-probability uncontrolled part). Pure SAT nur ~28 Steps; SAT+CAS (IPASIR-UP, wordwise Propagation) → 38-Step SFS. Konsens: Hybrid SAT + Differential + Message Modification; Bruch kommt als 40–45-Runden-Collision, nicht 64-Runden-One-Shot.

## 2. F1: Exakte IF/MAJ-Zählung (lokal wahr, global Rang 3)

Modell: Ch(e,f,g) = g ^ (e&(f^g)), max 1 Bedingung (nie 2). (-,x,x)->x forciert Kosten 0. (x,-,-) kostet 1 Joint (f^g=c) statt 2 Fixes. Maj w=1/w=2 je 1 Joint statt 2; (x,x,x)->x forciert 0, ->- unmöglich. Verifikation per eigenem Test (alle 64 Paare): Ch(0,1,1) N-=0 Nx=8, Maj(1,1,1) N-=0 Nx=8, alle anderen aktiven 4/4 → Kosten 1; kein Fall kostet 2. Netto 2–4 Bit (Faktor 4–16) pro Trail als obere Schranke pro Muster. Gegenprüfung: Joint notwendig, nicht hinreichend — Odd-Cycles (Sigma vs Maj/Ch 4er-Zyklen), Propagation >> Blocking, Kostenaufteilung 2^-66 = 2^-48 ADD + 2^-18 Ch/Maj (73% ADD). Praktisch Rang 3 nach (1) LC-Wahl/Sparsity und (2) Propagation/Modification. Auf K1 angewandt: -3 bis -4 Bit realistisch, Tail-Verlust +6 bis +12 wird nur zu ~1/3 kompensiert.

## 3. F3: Tail-Schranke (80% strukturell) + E-Schwerpunkt-Idee

Kontrolliert = Steps 6–23 (LC, per Modification/SAT lösbar); unkontrolliert = Head 0–5 (IV fix) + Tail 24–37 (alle W fix). Ab i>=16 keine Löschungs-Freiheit: sigma expandiert ~3-fach (Single-Bit über 64 Steps → Gewicht 467, optimiert 356). A-Pfad dominiert E-Pfad (Maj symmetrisch + Sigma0 + 4-Step-Persistenz vs Ch-MUX mit Selektor-Trick, E-Abklingen in ~4 Steps). Pro Tail-Step ~3–6 Bit; +1 Step hinten ≈ +20–30 Bit (2^58 / 2^104.3 / 2^119.1 / 2^178). 2^104 × 2^25 > 2^128 Birthday → 39-Step SHA-256 nur SFS. V-Shift V→V+1: vorne ~3–5 gespart, hinten ~8–12 bezahlt, netto -5 bis -8. Idee für 2–4 Bit: Tail-W dünn + alle Tail-ΔE auf MSB Bit 31 (kein Carry) + Ch-Selektor e=0 mit Δf=Δg (gratis). Reicht für 38-Step 2^104→~2^100, nicht 39-Step. MSB-Rechnung: L0(0x80000000)={24,13,28}, L1(0x80000000)={14,12,21} (j=31−n; eine anderslautende Korrektur {25,14,28} ist Off-by-one und wird zurückgewiesen). MSB kostet +1 Expansion, spart ~27 Carry.

## 4. F2/K1: Shift-Kandidaten und Mini-Tests

Definition (2026/232): V erstes aktives W, E letztes, S aktive Menge, t=E−V+1; Conversion V>=8 direkt, V>=5–6 2-Block (X>128), V=0 nur SFS. Kandidaten mit Zhang-Gaps, +2 geshiftet: K1={8,9,11,16,24,25} (E=25), K2={8,10,11,16,24,25}. Methode: GF(2)-Gauss über 16×32=512 Variablen, Expansion linearisiert, Nullen für i∉S bis E, Tail frei. Kontrollen lösbar (Mendel28 null=96, Mendel31 null=160, Zhang37 null=64, Zhang36 null=96). Ergebnis: K1 LÖSBAR null=32 (kleinste Freiheit, Erst-Lösung Kosten 18 vs Zhang 10), K2 UNLÖSBAR null=0 → verworfen. Modifizierter MSB-Test: exakt W24=W25=0x80000000 UNLÖSBAR (inkonsistent); nur Bit31=1 LÖSBAR (null 30) aber roh teurer (OBJ 78 vs frei 33–49) — rohes HW darf nicht entscheiden, Carry-Ersparnis fehlt darin. Hill+Stern auf rohem HW kann K1 daher weder retten noch beerdigen.

## 5. Urteil

K1 lebt als direkter Prefix-8-Kandidat (lösbar), liegt aber +4 bis +8 Bit über Zhang37 und wird als 39-Step-Kandidat aus strukturellen Gründen beerdigt (Tail +2 Steps, dünnste Freiheit, MSB-exakt unmöglich) — „Shift allein reicht nicht" ist selbst das Ergebnis. Offen bleibt K1 als direkter 35/36-Step mit gekürztem Tail oder SFS-Startpunkt. Empfohlene nächste Schritte erst nach Schreiben: neue Gap-Familie (kein Shift) oder CPU-armer Boomerang-Distinguisher statt Collision.

Quellen: FIPS 180-4; ePrint 2008/130, 2008/142, 2024/349, 2026/232, 2026/1080, 2026/1120, 2026/1505, 2026/353, 2011/286; Mendel et al. EUROCRYPT 2013; Hawkes et al. 2004/207; Sanadhya 2007/352; Gilbert-Handschuh SAC 2003/2004; arXiv 2406.20072.
