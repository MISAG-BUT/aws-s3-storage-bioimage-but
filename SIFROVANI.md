# Šifrování dat na úložišti bioimage-but

🇨🇿 Česky · [🇬🇧 English](SIFROVANI.en.md)

## Co umí server (ověřeno 2026-11-26 na s3.cl4.du.cesnet.cz)

| Varianta | Stav |
|---|---|
| SSE-S3: server šifruje vlastním klíčem | ❌ nepodporováno (`InvalidArgument`) |
| SSE-C: server šifruje klíčem, který posílá klient | ❌ nepodporováno (`InvalidRequest`) |
| Výchozí šifrování bucketu | ❌ nenastaveno |

Přenos probíhá přes HTTPS (šifrovaný po cestě), ale **data jsou na serveru uložena nešifrovaně**.
Vidí je správce úložiště a technicky i provozovatel (CESNET).
Pro citlivá data proto použij **šifrování na straně klienta**: soubory zašifruješ dřív, než je nahraješ.

## Možnosti

| Nástroj | Vhodné pro | Poznámka |
|---|---|---|
| **rclone crypt** (doporučeno, ✅ otestováno) | průběžná práce, sync celých složek | open source, Win/Linux/macOS, šifruje obsah i názvy souborů |
| **restic** | zálohy | šifrované deduplikované zálohy přímo do S3 |
| **age** / **gpg** | jednotlivé soubory a archivy | jednoduché, ruční |

### Zásady
- **Ztracené heslo = ztracená data.** Správce ani CESNET je neobnoví. Ulož si heslo do správce hesel.
- Zašifrované soubory jdou číst jen přes rclone se stejným heslem. `aws s3 cp` nebo prohlížeč dají
  jen nečitelný obsah s nesmyslnými názvy.
- **Sdílený projekt:** všichni členové potřebují stejné heslo. Předávej ho bezpečně (správce hesel,
  osobně, ne e-mailem v čisté podobě). Po odchodu člena zvaž přešifrování.
- Šifruj vždy do vlastní podsložky (např. `<projekt>/encrypted/`) a nemíchej do ní nešifrované soubory.

## rclone crypt: návod

### 1. Instalace
- **Windows:** `winget install -e --id Rclone.Rclone` (pak otevři nový terminál)
- **Linux:** `sudo -v ; curl https://rclone.org/install.sh | sudo bash`, nebo bez sudo stáhni
  [binárku](https://rclone.org/downloads/) do `~/.local/bin`
- **macOS:** `brew install rclone`

Ověření: `rclone version`

### 2. Připojení k S3
Použije klíče z AWS profilu `bioimage` (viz [návod pro členy](README.md)), takže je nezadáváš znovu:
```
rclone config create cesnet s3 provider=Ceph env_auth=true profile=bioimage endpoint=https://s3.cl4.du.cesnet.cz
```
Kontrola: `rclone lsd cesnet:bioimage-but` vypíše složky ve sdíleném bucketu.

### 3. Šifrovaná vrstva
Pro projekt (heslo společné pro celý projekt):
```
rclone config create <projekt>-crypt crypt remote=cesnet:bioimage-but/<projekt>/encrypted password=<heslo> --obscure
```
Pro soukromá data:
```
rclone config create soukrome crypt remote=cesnet:private-<jméno>/encrypted password=<heslo> --obscure
```
`--obscure` heslo v `rclone.conf` jen zamaskuje, nešifruje. Kdo získá tvůj `rclone.conf`, získá i heslo.
Chraň ho stejně jako AWS klíče; volitelně ho zašifruj příkazem `rclone config encryption set`.
Heslo v příkazu zůstane v historii shellu; bezpečnější je interaktivní `rclone config`.

### 4. Používání
```
rclone copy ./data <projekt>-crypt:data          # nahraje zašifrovaně
rclone sync ./data <projekt>-crypt:data          # zrcadlí, na S3 smaže, co lokálně chybí
rclone ls <projekt>-crypt:                       # výpis s rozšifrovanými názvy
rclone copy <projekt>-crypt:data ./obnova        # stáhne a rozšifruje
rclone cat <projekt>-crypt:data/soubor.txt       # vypíše jeden soubor
rclone cryptcheck ./data <projekt>-crypt:data    # ověří, že data na S3 odpovídají lokálním
```
Připojení jako disk (`rclone mount <projekt>-crypt: X:`) potřebuje na Windows
[WinFsp](https://winfsp.dev/) a na macOS [macFUSE](https://osxfuse.github.io/). Tuto možnost jsme netestovali.

### Co jsme ověřili (2026-11-27)
- Nahrání, stažení a `cryptcheck`: soubory jsou po stažení shodné.
- V S3 jsou šifrované obsah i názvy souborů a složek (`aws s3 ls` ukáže např. `gskmalbnrqag3dhgev13d1ui6o/…`).
- Druhý uživatel se stejným heslem data přečte.
- Se špatným heslem rclone nevypíše nic. Soubory přeskočí s hláškou `Skipping undecryptable file name`,
  ale **neohlásí chybu**. Prázdný výpis tedy může znamenat špatné heslo.
- Bez přístupu ke složce vrátí `403 AccessDenied`.

## age: jednotlivé soubory
Instalace: `winget install -e --id FiloSottile.age` / `apt install age` / `brew install age`
```
age -p -o data.tar.age data.tar                 # zašifruje heslem
aws s3 cp data.tar.age s3://private-<jméno>/ --profile bioimage
age -d -o data.tar data.tar.age                 # rozšifruje
```
