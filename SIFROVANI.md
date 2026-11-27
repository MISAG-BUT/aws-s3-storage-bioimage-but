# Šifrování dat na úložišti bioimage-but

🇨🇿 Česky · [🇬🇧 English](SIFROVANI.en.md)

Data jsou na serveru uložená **nešifrovaně**. Vidí je správce úložiště a technicky i provozovatel (CESNET).
**Citlivá data proto nahrávej zašifrovaná** přes šifrovanou vrstvu rclone (`crypt`) podle tohoto návodu.

Předpoklad: máš rclone nainstalovaný a nastavené připojení `cesnet:` podle [návodu pro členy](README.md).

## Zásady
- **Ztracené heslo = ztracená data.** Správce ani CESNET je neobnoví. Ulož si heslo do správce hesel.
- Zašifrované soubory jdou číst jen přes šifrovanou vrstvu se stejným heslem. Přes `cesnet:` uvidíš jen
  nečitelný obsah s nesmyslnými názvy.
- **Sdílený projekt:** všichni členové potřebují stejné heslo. Předávej ho bezpečně (správce hesel,
  osobně, ne e-mailem v čisté podobě). Po odchodu člena zvaž přešifrování.
- Šifruj vždy do podsložky `encrypted/` (např. `<projekt>/encrypted/`) a nemíchej do ní nešifrované soubory.

## 1. Vytvoř šifrovanou vrstvu
Pro projekt (heslo společné pro celý projekt):
```
rclone config create <projekt>-crypt crypt remote=cesnet:bioimage-but/<projekt>/encrypted password=<heslo> --obscure
```
Pro soukromá data:
```
rclone config create soukrome crypt remote=cesnet:private-<uzivatel>/encrypted password=<heslo> --obscure
```
`--obscure` heslo v konfiguraci rclone jen zamaskuje, nešifruje. Kdo získá tvůj konfigurační soubor,
získá i heslo. Chraň ho stejně jako klíče; volitelně ho zašifruj příkazem `rclone config encryption set`.

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
