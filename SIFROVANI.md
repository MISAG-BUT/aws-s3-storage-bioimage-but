# Šifrování dat na úložišti bioimage-but

## Co umí server (ověřeno 2026-11-26 na s3.cl4.du.cesnet.cz)

| Varianta | Stav |
|---|---|
| SSE-S3: server šifruje vlastním klíčem | ❌ nepodporováno (`InvalidArgument`) |
| SSE-C: server šifruje klíčem, který posílá klient | ❌ nepodporováno (`InvalidRequest`) |
| Výchozí šifrování bucketu | ❌ nenastaveno |

Přenos probíhá přes HTTPS (šifrovaný po cestě), ale **data jsou na serveru uložena nešifrovaně**.
Vidí je správce úložiště a technicky i provozovatel (CESNET).
Pro citlivá data proto použij **šifrování na straně klienta**: soubory zašifruješ dřív, než je nahraješ.

## Možnosti šifrování na straně klienta

| Nástroj | Vhodné pro | Poznámka |
|---|---|---|
| **rclone crypt** (doporučeno) | průběžná práce s daty, sync celých složek | open source, Win/Linux/macOS, šifruje obsah i názvy souborů, umí připojit bucket jako disk |
| **restic** | zálohy | šifrované deduplikované zálohy přímo do S3 |
| **age** / **gpg** | jednotlivé soubory a archivy | jednoduché, ruční |
| AWS S3 Encryption Client | vlastní programy | jen pro některé jazyky (Java, .NET, C++, Go), ne pro Python |

### Zásady
- **Ztracený klíč nebo heslo = ztracená data.** Správce ani CESNET je neobnoví. Ulož si heslo do
  správce hesel.
- Šifrované soubory nejdou otevřít přímo z S3 (prohlížeč, `aws s3 cp` dá jen zašifrovaný obsah).
- **Sdílené projekty:** všichni členové potřebují stejný klíč. Předávej ho bezpečně (ne e-mailem
  v čisté podobě) a po odchodu člena zvaž přešifrování.
- Pro soukromá data stačí vlastní heslo a soukromý bucket `private-<jméno>`.

## rclone crypt: nastavení

1. Nainstaluj [rclone](https://rclone.org/downloads/) (`winget install Rclone.Rclone`, `apt install rclone`, `brew install rclone`).
2. Spoj rclone s S3 (použije klíče z AWS profilu `bioimage`):
   ```
   rclone config create cesnet s3 provider=Ceph env_auth=true endpoint=https://s3.cl4.du.cesnet.cz
   ```
   a nastav proměnnou `AWS_PROFILE=bioimage` (nebo v `rclone config` zadej klíče přímo).
3. Vytvoř šifrovanou vrstvu nad složkou v bucketu:
   ```
   rclone config create tajne crypt remote=cesnet:private-<jméno>/encrypted password=<heslo> --obscure
   ```
   Pro projekt: `remote=cesnet:bioimage-but/<projekt>/encrypted` a heslo sdílené v rámci projektu.
4. Používání:
   ```
   rclone copy ./data tajne:data          # nahraje zašifrovaně
   rclone ls tajne:                       # rozšifrovaný výpis
   rclone copy tajne:data ./obnova        # stáhne a rozšifruje
   rclone mount tajne: X:                 # připojí jako disk (Windows vyžaduje WinFsp)
   ```
   Přímo v S3 (`aws s3 ls`) uvidíš jen zašifrované názvy a obsah.

> rclone crypt jsme na tomto úložišti zatím neotestovali. Pokud narazíš na problém, dej vědět správci.

## age: jednotlivé soubory
```
age -p -o data.tar.age data.tar                 # zašifruje heslem
aws s3 cp data.tar.age s3://private-<jméno>/ --profile bioimage
age -d -o data.tar data.tar.age                 # rozšifruje
```
