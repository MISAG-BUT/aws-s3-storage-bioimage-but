# Pravidla a podmínky používání úložiště bioimage-but

> **TODO / NÁVRH:** pracovní verze k doplnění a schválení správcem. Body označené ❓ je potřeba rozhodnout.

## 1. Účel
Úložiště slouží k ukládání a sdílení výzkumných dat skupiny v rámci projektů. Není určené pro
soukromá data nesouvisející s prací ani jako jediné úložiště nenahraditelných dat.

## 2. Přístup
- Přístup dostávají členové VO na základě pozvánky. Klíče jsou osobní, nikomu je nepředávej
  a nikam je necommituj (git, sdílené skripty, notebooky).
- Při podezření na únik klíče ho okamžitě smaž v Gatekeeperu a dej vědět správci.
- Správce má technický přístup ke všem datům včetně soukromých bucketů. Využije ho jen
  pro správu (zálohy, úklid, odchod člena, řešení problémů).
- ❓ Po odchodu ze skupiny: přístup se ruší k [datu]; data v soukromém bucketu budou smazána
  po [N] dnech, pokud se nedohodne jinak.

## 3. Data
- **Citlivá a osobní data** (pacientská data, údaje podle GDPR) ❓ [zakázána / jen anonymizovaná /
  jen šifrovaná podle [návodu](SIFROVANI.md) a se souhlasem vedoucího projektu].
- Dodržuj licenční a smluvní podmínky dat (data od partnerů, NDA).
- `common/` vidí a může měnit každý člen. Nepatří tam nic, co by neměli vidět všichni,
  a nic, o co nesmíš přijít.
- Projektová data ukládej do složky projektu, ne do kořene ani do `common/`.
- ❓ Doporučená struktura projektu (např. `raw/`, `processed/`, `results/`) a pojmenování souborů.

## 4. Kapacita a úklid
- ❓ Kvóta skupiny je [X TB]. Velké přesuny (>[Y] TB) předem ohlas správci.
- Mazej dočasné a mezivýsledkové soubory, které už nepotřebuješ.
- ❓ Obsah `common/` starší než [N] dní může správce smazat.

## 5. Zálohy a odpovědnost
- Úložiště **není záloha**. Smazaný nebo přepsaný soubor nejde obnovit (verzování není zapnuté).
  ❓ Zapnout verzování pro projekty?
- Za šifrovací klíče odpovídá ten, kdo data šifroval. Ztracený klíč = ztracená data; správce je
  obnovit nedokáže.

## 6. Podmínky poskytovatele
Platí i podmínky služby CESNET / e-INFRA CZ (AUP), které jsi odsouhlasil při přihlášení do Gatekeeperu.

## 7. Porušení pravidel
❓ Správce může při porušení pravidel omezit nebo zrušit přístup.

---
Verze: návrh, 2026-11-26
