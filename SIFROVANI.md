# Šifrování dat na úložišti bioimage-but

🇨🇿 Česky · [🇬🇧 English](SIFROVANI.en.md)

Data jsou na serveru uložená **nešifrovaně**. Vidí je správce úložiště a technicky i provozovatel (CESNET).
Proto platí:

- **běžná data** nahrávej přes **AWS CLI** (viz [návod pro členy](README.md));
- **citlivá data** nahrávej **zašifrovaná přes rclone** podle tohoto návodu.

## Zásady
- **Ztracené heslo = ztracená data.** Správce ani CESNET je neobnoví. Ulož si heslo do správce hesel.
- Zašifrované soubory jdou číst jen přes rclone se stejným heslem. Přes AWS CLI uvidíš jen nečitelný obsah
  s nesmyslnými názvy.
- **Sdílený projekt:** všichni členové potřebují stejné heslo. Předávej ho bezpečně (správce hesel,
  osobně, ne e-mailem v čisté podobě). Po odchodu člena zvaž přešifrování.
- Šifruj vždy do podsložky `encrypted/` (např. `<projekt>/encrypted/`) a nemíchej do ní nešifrované soubory.

## 1. Instalace rclone
- **Windows:** `winget install -e --id Rclone.Rclone` (pak otevři nový terminál)
- **Linux:** `sudo -v ; curl https://rclone.org/install.sh | sudo bash`, nebo bez sudo stáhni
  [binárku](https://rclone.org/downloads/) do `~/.local/bin`
- **macOS:** `brew install rclone`

Ověření: `rclone version`

## 2. Připojení k S3
Použije klíče z tvého AWS profilu `<uzivatel>`, takže je nezadáváš znovu:
```
rclone config create cesnet s3 provider=Ceph env_auth=true profile=<uzivatel> endpoint=https://s3.cl4.du.cesnet.cz
```
Kontrola: `rclone lsd cesnet:bioimage-but` vypíše složky ve sdíleném bucketu.

## 3. Šifrovaná vrstva
Pro projekt (heslo společné pro celý projekt):
```
rclone config create <projekt>-crypt crypt remote=cesnet:bioimage-but/<projekt>/encrypted password=<heslo> --obscure
```
Pro soukromá data:
```
rclone config create soukrome crypt remote=cesnet:private-<uzivatel>/encrypted password=<heslo> --obscure
```
`--obscure` heslo v `rclone.conf` jen zamaskuje, nešifruje. Kdo získá tvůj `rclone.conf`, získá i heslo.
Chraň ho stejně jako AWS klíče; volitelně ho zašifruj příkazem `rclone config encryption set`.
Heslo v příkazu zůstane v historii shellu; bezpečnější je interaktivní `rclone config`.

## 4. Používání
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
- **`403 AccessDenied`:** nemáš přístup do dané složky. Zkontroluj název projektu a profil, případně
  požádej správce o přístup.
