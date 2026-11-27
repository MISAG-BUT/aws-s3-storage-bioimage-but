# Pravidla a podmínky používání úložiště bioimage-but

🇨🇿 Česky · [🇬🇧 English](PRAVIDLA.en.md)

## 1. Účel
Úložiště slouží k ukládání a sdílení výzkumných dat skupiny v rámci projektů. Není určené pro
soukromá data nesouvisející s prací ani jako jediné úložiště nenahraditelných dat.

## 2. Přístup
- Přístup dostávají členové VO na základě pozvánky. Klíče jsou osobní, nikomu je nepředávej
  a nikam je necommituj (git, sdílené skripty, notebooky).
- Konfigurace rclone s klíči **musí být vždy zašifrovaná heslem** (`rclone config encryption set`).
- Při podezření na únik klíče ho okamžitě smaž v Gatekeeperu a dej vědět správci.
- Správce má technický přístup ke všem datům včetně soukromých bucketů. Využije ho jen
  pro správu (úklid, odchod člena, řešení problémů).
- **Odchod ze skupiny:** přístup se ruší ke dni odchodu. Soukromý bucket se smaže **30 dní po odchodu**,
  pokud se se správcem nedohodneš jinak. Data v projektových složkách zůstávají projektu.

## 3. Data
- **Citlivá a osobní data** (pacientská data, osobní údaje podle GDPR) smí být na úložišti **jen
  zašifrovaná** podle [návodu k šifrování](SIFROVANI.md) a **se souhlasem vedoucího projektu**.
- Dodržuj licenční a smluvní podmínky dat (data od partnerů, NDA).
- `common/` vidí a může měnit každý člen. Nepatří tam nic, co by neměli vidět všichni,
  a nic, o co nesmíš přijít.
- Projektová data ukládej do složky projektu, ne do kořene ani do `common/`.
- **Doporučená struktura projektu:**
  ```
  <projekt>/
  ├── README.md      popis projektu, zdroj dat, kontaktní osoba
  ├── raw/           původní data, neměnit
  ├── processed/     zpracovaná data
  ├── results/       výsledky
  └── encrypted/     šifrovaná data (rclone crypt)
  ```
  Podle potřeb projektu mohou být surová i další data celá v `encrypted/` (např. `encrypted/raw/`,
  `encrypted/processed/`).

## 4. Kapacita a úklid
- Kvóta celé skupiny je **20 TB a 20 milionů souborů**, sdílená všemi členy a projekty.
- Přenosy **větší než 1 TB** předem ohlas správci.
- Mazej dočasné a mezivýsledkové soubory, které už nepotřebuješ. Velké množství malých souborů raději
  zabal do archivu (např. `.tar`), počet souborů je také omezený.
- Soubory v `common/` **starší než 90 dní** může správce smazat.

## 5. Zálohy a odpovědnost
- Úložiště **není záloha**. Smazaný nebo přepsaný soubor nejde obnovit. Důležitá data měj uložená
  i jinde.
- Za šifrovací hesla odpovídá ten, kdo data šifroval. Ztracené heslo = ztracená data; správce je
  obnovit nedokáže.

## 6. Podmínky poskytovatele
Platí i podmínky služby CESNET / e-INFRA CZ (AUP), které jsi odsouhlasil při přihlášení do Gatekeeperu.

## 7. Porušení pravidel
Při porušení pravidel může správce omezit nebo zrušit přístup a odstranit data, která pravidla porušují
(např. nešifrovaná citlivá data).

---
Verze 1.0, 2026-11-27
