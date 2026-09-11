# madomori 窓守

**High-rise / façade window-cleaning robotics — coverage path + wind/sway envelope + adhesion.**
Tier-B actor · ADR-2606142020 · 🟡 R0 (design + sim) · Clojure-first.

madomori **removes a human from the rope/cradle** on a high-rise façade — the GAP the
robotics remote-work survey (ADR-2606073001 §4) named the **highest unmet remote value**
("高所・façade window cleaning — fall fatality; current GAP"). kiyome 清め is indoor/ground
only; madomori covers fall-risk façade work.

It mirrors the **kuramori 倉守 reference Clojure-first idiom** (ADR-2606142000): pure methods,
no deps → runnable under both `bb` and the kotoba pywasm runtime.

## Run

```bash
kbb run_tests.cljk   # 46 tests / 185 assertions
kbb --classpath . -m madomori.methods.analyze                            # → façade R0 report
kbb --classpath . -m madomori.methods.datom-emit                         # → kotoba EAVT Datom log
kbb --classpath . -m madomori.methods.citations                          # → provenance for the ★ gate constants
```

## What it does

| Method | Role |
|---|---|
| `facade_path.clj`   | boustrophedon (S-shape) coverage path over a pane grid · path length · per-pane water + agent budget (G2) · coverage completeness (every pane once) |
| `wind_envelope.clj` | rope/BMU pendulum sway model · **wind work-stop** + **≥2 fall-arrest anchors** (★ G5 — both RAISE on violation) |
| `adhesion.clj`      | suction adhesion force vs payload × surface factor-of-safety · **adhesion-safe? RAISES below the required margin** (★ G7) |
| `analyze.clj`       | end-to-end: load seed → coverage + budget → wind/sway envelope → adhesion FoS → **GO?** (both safety gates pass) |
| `datom_emit.clj`    | kotoba EAVT projection (`:mado.*` GROUND + `:bond/*` DERIVED transient; G3 structural — only the on-device imagery flag is emittable) |

## Provenance of the ★ gate constants

Both ★ gates REFUSE work by comparing against a number, so `data/citations.edn`
records where each number comes from — every entry quotes text fetched from the
cited instrument (verified 2026-08-30, HTTP 200), and says what it does **not**
establish.

| Constant | Status |
|---|---|
| wind work-stop *exists* | **law** — ゴンドラ安全規則 第十九条「当該作業を行なつてはならない」(this is why G5 raises) |
| wind work-stop = `10.0` m/s | **operator-set, not law** — neither ゴンドラ則 第十九条 nor クレーン則 第三十一条の二 states a figure; 気象庁 treats 強風 as a general term and puts 10–15 m/s at 「やや強い風」 |
| anchors `:independent` | **law** — OSHA 1926.502(d)(15) "independent of any anchorage being used to support or suspend platforms" |
| anchors `≥2` | **design decision** — the regulation requires independence, not a count |
| `required-fos` 2.5 | **uncited** — ゴンドラ則 has no 安全係数 clause; OSHA's factor of 2 governs fall-arrest systems, not suction |
| `surface-efficiency` | **uncited** — only the ordering glass > metal > stone is defensible at R0 |

`citations/cited?` is pessimistic: an unrecorded constant reads as **uncited**,
never as fine. Do not attach a source to an uncited number without fetching and
reading the instrument first.

## Gates

R0 design+sim only (G1, no-server-key) · water + chemical minimization (G2) · ★
privacy-by-construction — on-device imagery only, no person/interior recognition (G3) ·
Displacement-Dividend-coupled (G4) · ★ wind work-stop + fall-arrest redundancy raise (G5) ·
Murakumo-only (G6) · ★ adhesion factor-of-safety raises (G7) · tazuna-teleoperable (G8).
See `CLAUDE.md` for the full text.

Apache 2.0 + etzhayyim Charter Compliance Rider v3.1.
