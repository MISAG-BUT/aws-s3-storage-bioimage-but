# Šifrování dat na úložišti bioimage-but

🇨🇿 Česky · [🇬🇧 English](SIFROVANI.en.md)

Data jsou na serveru uložená **nešifrovaně**. Vidí je správce úložiště a technicky i provozovatel (CESNET).
**Citlivá data proto nahrávej zašifrovaná** přes šifrovanou vrstvu rclone (`crypt`) podle tohoto návodu.

Předpoklad: máš rclone nainstalovaný, konfiguraci zašifrovanou a nastavené připojení `cesnet:` podle
[návodu pro členy](README.md).

## Zásady
- **Ztracené heslo = ztracená data.** Správce ani CESNET je neobnoví. Ulož si heslo do správce hesel.
- Zašifrované soubory jdou číst jen přes šifrovanou vrstvu se stejným heslem. Přes `cesnet:` uvidíš jen
  nečitelný obsah s nesmyslnými názvy.
- **Sdílený projekt:** všichni členové potřebují stejné heslo. Předávej ho bezpečně (správce hesel,
  osobně, ne e-mailem v čisté podobě). Po odchodu člena zvaž přešifrování.
- Šifruj vždy do podsložky `encrypted/` (např. `<projekt>/encrypted/`) a nemíchej do ní nešifrované soubory.

## 1. Vytvoř šifrovanou vrstvu
Spusť:
```
rclone config
```
a odpovídej podle tabulky. Hodnoty pro `remote>` ke zkopírování (nahraď `<projekt>` / `<uzivatel>`):
```
cesnet:bioimage-but/<projekt>/encrypted
```
```
cesnet:private-<uzivatel>/encrypted
```

| Otázka | Odpověď |
|---|---|
| `Enter configuration password` | heslo ke konfiguraci rclone |
| `e/n/d/r/c/s/q>` | `n` (nový remote) |
| `name>` | projekt: `<projekt>-crypt`, soukromá data: `soukrome` |
| `Storage>` | `crypt` |
| `remote>` | projekt: `cesnet:bioimage-but/<projekt>/encrypted`<br>soukromá data: `cesnet:private-<uzivatel>/encrypted` |
| `filename_encryption>` | Enter (`standard`) |
| `directory_name_encryption>` | Enter (`true`) |
| `password` → `y/g>` | `y`, pak 2× `<heslo>` (u projektu společné heslo projektu) |
| `password2` → `y/g/n>` | `n` |
| `Edit advanced config?` | `n` |
| `Keep this ... remote?` | `y` |
| `e/n/d/r/c/s/q>` | `q` |

`<heslo>` šifrovací vrstvy je jiné než heslo ke konfiguraci rclone. Heslo ke konfiguraci máš jen ty,
heslo projektu sdílí všichni jeho členové.

## 2. Používání
Stejné příkazy jako v [návodu pro členy](README.md), jen místo `cesnet:…` použiješ šifrovanou vrstvu:
```
rclone copy ./data <projekt>-crypt:data          # nahraje zašifrovaně
rclone sync ./data <projekt>-crypt:data          # zrcadlí, na S3 smaže, co lokálně chybí
rclone ls <projekt>-crypt:                       # výpis s rozšifrovanými názvy
rclone copy <projekt>-crypt:data ./obnova        # stáhne a rozšifruje
rclone cat <projekt>-crypt:data/soubor.txt       # vypíše jeden soubor
rclone cryptcheck ./data <projekt>-crypt:data    # ověří, že data na S3 odpovídají lokálním
```

## Řešení problémů
- **Prázdný výpis, i když data na S3 jsou:** nejspíš máš špatné heslo. rclone soubory, které nejde
  rozšifrovat, tiše přeskočí (`Skipping undecryptable file name`) a chybu neohlásí.
- **`403 AccessDenied`:** nemáš přístup do dané složky. Zkontroluj název projektu, případně
  požádej správce o přístup.
