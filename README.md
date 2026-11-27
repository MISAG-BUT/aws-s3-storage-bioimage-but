# S3 úložiště bioimage-but: návod pro členy

🇨🇿 Česky · [🇬🇧 English](README.en.md)

Skupinové S3 úložiště na CESNET (`https://s3.cl4.du.cesnet.cz`). Pracuje se s ním nástrojem **rclone**.

**Před použitím si přečti [pravidla používání](PRAVIDLA.md).**
Citlivá data šifruj podle [návodu k šifrování](SIFROVANI.md).

V příkazech nahraď `<uzivatel>` svým uživatelským jménem (stejným jako v názvu soukromého bucketu,
např. `novak`), `<projekt>` názvem projektu a `<heslo>` svým heslem.

## Struktura

```
private-<uzivatel>/   tvůj soukromý bucket: vidíš ho jen ty a správce
bioimage-but/         sdílený bucket
├── common/           čtení i zápis pro všechny členy skupiny
└── <projekt>/        jen členové projektu a správce
```

- V kořeni `bioimage-but` vidíš názvy všech projektů, dovnitř ale jen do těch svých.
- Přístupy nastavuje správce. O přidání do projektu nebo nový projekt požádej jeho.
- Správce má technický přístup ke všem datům včetně soukromých bucketů.

## 1. Přijmi pozvánku do VO
Správce tě pozve do virtuální organizace (VO) přes e-INFRA CZ (Perun). Pozvánku přijmi
a přihlas se svou domovskou institucí.

> **Počkej.** Přiřazení do VO a skupiny se do Gatekeeperu propisuje se zpožděním, které
> trvá **desítky minut až hodiny** (CESNET uvádí nejméně 30 minut). Dokud se skupina
> v Gatekeeperu neobjeví, klíče vygenerovat nejde. Zkus to později, nic nerozbiješ.

## 2. Vygeneruj si S3 klíče v Gatekeeperu
1. Otevři **https://gatekeeper.du.cesnet.cz** (dříve `access.du.cesnet.cz`) a klikni na **LOG IN**.
2. Vyber svou domovskou organizaci (např. VUT) a přihlas se.
3. V levém menu vyber naši **VO** a v ní **skupinu**, tedy tenant, kde klíče vzniknou.
4. Klikni na **New key**, zadej název (např. `notebook`) a klíč vytvoř.
5. **Hned si ulož Access key i Secret key.** Secret key se zobrazí jen jednou. Když ho ztratíš,
   vygeneruj nový klíč a starý v Gatekeeperu smaž.

## 3. Pošli správci své ID
V Gatekeeperu u skupiny najdeš **ID skupiny (tenant)** a **ID uživatele**.
- ID skupiny musí být `94b66cdd_86a7_41a0_9d47_97f7b10fcdb7`. Pokud je jiné, jsi v jiné skupině;
  vrať se ke kroku 2.
- Správci pošli **ID uživatele** a napiš, do kterých projektů potřebuješ přístup.

## 4. Nainstaluj rclone
- **Windows** (PowerShell), pak otevři nový terminál:
  ```powershell
  winget install -e --id Rclone.Rclone
  ```
- **Linux** (se sudo):
  ```bash
  sudo -v ; curl https://rclone.org/install.sh | sudo bash
  ```
  Instalace může trvat několik minut (aktualizuje se rejstřík manuálových stránek). Pokud skončí
  chybou kvůli chybějícímu `unzip`, spusť nejdřív:
  ```bash
  sudo apt install unzip
  ```
- **Linux** (bez sudo, např. na serveru nebo clusteru; na ARM nahraď `amd64` za `arm64`):
  ```bash
  curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
  python3 -m zipfile -e rclone-current-linux-amd64.zip .
  mkdir -p ~/.local/bin && cp rclone-*-linux-amd64/rclone ~/.local/bin/ && chmod +x ~/.local/bin/rclone
  ```
  Pokud příkaz `rclone` nejde najít, přidej `~/.local/bin` do cesty:
  ```bash
  echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
  ```
- **macOS:**
  ```bash
  brew install rclone
  ```

Ověření:
```
rclone version
```

## 5. Zašifruj konfiguraci rclone (povinné)
rclone si ukládá klíče do konfiguračního souboru. **Ten musí být vždy zašifrovaný heslem**, a to ještě
předtím, než do něj klíče zadáš:
```
rclone config encryption set
```
Zadej silné heslo ke konfiguraci a ulož si ho do správce hesel. Od teď se na něj rclone zeptá při
každém příkazu (`Enter configuration password`).

Kontrola: následující příkaz vypíše cestu ke konfiguraci. Soubor musí začínat řádkem
`# Encrypted rclone configuration File`.
```
rclone config file
```

## 6. Nastav připojení
Spusť:
```
rclone config
```
a odpovídej takto (u výběru napiš text, ne číslo; čísla se mezi verzemi liší). Endpoint ke zkopírování:
```
https://s3.cl4.du.cesnet.cz
```

| Otázka | Odpověď |
|---|---|
| `Enter configuration password` | heslo ke konfiguraci z kroku 5 |
| `n) New remote` | `n` |
| `name>` | `cesnet` |
| `Storage>` | `s3` |
| `provider>` | `Ceph` |
| `env_auth>` | Enter |
| `access_key_id>` | tvůj Access key z Gatekeeperu |
| `secret_access_key>` | tvůj Secret key z Gatekeeperu |
| `region>` | Enter |
| `endpoint>` | `https://s3.cl4.du.cesnet.cz` |
| `location_constraint>`, `acl>`, `server_side_encryption>`, `sse_kms_key_id>`, `bucket_object_lock_enabled>` | Enter |
| `Edit advanced config?` | `n` |
| `Keep this "cesnet" remote?` | `y` |
| `e/n/d/r/c/s/q>` | `q` (konec) |

Konfiguraci nikomu neposílej a necommituj, ani zašifrovanou.

## 7. Ověř přístup (až ti správce potvrdí nastavení)
```
rclone lsd cesnet:bioimage-but                   # složky: common, projekty
rclone ls cesnet:bioimage-but/<projekt>          # obsah tvého projektu
rclone ls cesnet:private-<uzivatel>              # tvůj soukromý bucket
```

## Práce s daty
```
rclone copy ./data cesnet:bioimage-but/<projekt>/data      # nahraje (přeskočí shodné soubory)
rclone sync ./data cesnet:bioimage-but/<projekt>/data      # S3 = kopie lokální složky; na S3 smaže, co lokálně není
rclone sync cesnet:bioimage-but/<projekt>/data ./data      # opačně: lokální složka = kopie S3; LOKÁLNĚ smaže, co na S3 není
rclone copy cesnet:bioimage-but/<projekt>/data ./data      # stáhne
rclone check ./data cesnet:bioimage-but/<projekt>/data     # porovná lokální data s S3
rclone cat cesnet:bioimage-but/<projekt>/poznamka.txt      # vypíše soubor
rclone deletefile cesnet:bioimage-but/<projekt>/soubor.tif # smaže soubor
rclone delete cesnet:bioimage-but/<projekt>/stare          # smaže celou složku i s podsložkami
rclone link --expire 7d cesnet:bioimage-but/<projekt>/soubor.tif   # odkaz ke stažení pro kohokoli
```
- **`rclone sync ZDROJ CÍL`** udělá z cíle přesnou kopii zdroje: v cíli **smaže** vše, co ve zdroji není.
  Směr určuje pořadí. Při `sync` z S3 do počítače tak přijdeš o **lokální** soubory, které na S3 nejsou.
  Pokud nechceš mazat, použij `rclone copy`.
- Před `sync` a mazáním si to vyzkoušej s `--dry-run`. Smazané soubory nejde obnovit, ani na S3, ani lokálně.
- `rclone ls cesnet:bioimage-but` (celý bucket najednou) skončí chybou 403. Vždy zadej projekt.
- `403 AccessDenied` znamená, že do dané složky nemáš přístup.
- S3 ve skutečnosti nemá složky. Složka vznikne s prvním souborem; `rclone mkdir` nic neudělá.
  Některé nástroje ale vytvářejí prázdné „značky složek“. Taková složka po smazání souborů nezmizí,
  viz níže.
- Odkaz z `rclone link` funguje pro kohokoli, kdo ho má, až do vypršení (max. 7 dní).
- `common/` je pro všechny ve skupině. Každý tam může soubory číst, přepsat i smazat.

### Smazání neprázdné složky
Nejdřív se podívej, co se smaže, a pak smaž:
```
rclone delete cesnet:bioimage-but/<projekt>/slozka --dry-run
```
```
rclone delete cesnet:bioimage-but/<projekt>/slozka
```
Tím se smažou všechny soubory ve složce i v podsložkách. Pokud je složka pak pořád vidět (zůstaly
prázdné značky složek), smaž je:
```
rclone rmdirs cesnet:bioimage-but/<projekt>/slozka --s3-directory-markers
```
Nespouštěj to na celou složku projektu (`cesnet:bioimage-but/<projekt>`), smazal bys i samotný projekt.

Šifrovaná data mažeš přes šifrovanou vrstvu
(`rclone delete <projekt>-crypt:slozka`). `rclone purge` složku smaže také, ale na našem úložišti vypíše
neškodnou chybu `GetBucketVersioning ... 403`, proto používej `delete`.

## Skripty a automatizace
rclone se na heslo ke konfiguraci ptá při každém příkazu. Ve skriptu mu ho předáš proměnnou
`RCLONE_CONFIG_PASS`. **Heslo nikdy nepiš přímo do skriptu** ani na příkazovou řádku: zůstalo by
v souboru nebo v historii.

Zadej heslo jednou na začátku práce v terminálu; platí jen do jeho zavření:

**Windows:** nejdřív zjisti verzi PowerShellu:
```powershell
$PSVersionTable.PSVersion
```

**PowerShell 7** (verze 7.x):
```powershell
$env:RCLONE_CONFIG_PASS = Read-Host -MaskInput "Heslo ke konfiguraci rclone"
.\muj_skript.ps1
Remove-Item Env:\RCLONE_CONFIG_PASS
```

**PowerShell 5.1** (výchozí „Windows PowerShell“, nezná `-MaskInput`):
```powershell
$s = Read-Host -AsSecureString "Heslo ke konfiguraci rclone"
$env:RCLONE_CONFIG_PASS = (New-Object PSCredential "x", $s).GetNetworkCredential().Password
.\muj_skript.ps1
Remove-Item Env:\RCLONE_CONFIG_PASS
```

**Linux / macOS (bash, zsh):**
```bash
read -rsp "Heslo ke konfiguraci rclone: " RCLONE_CONFIG_PASS && echo && export RCLONE_CONFIG_PASS
./muj_skript.sh                                            # rclone uvnitř se už neptá
unset RCLONE_CONFIG_PASS                                   # po skončení zapomene
```

### Úlohy bez obsluhy
Pro úlohy, kde heslo nemá kdo zadat (naplánované úlohy, cron, úlohy na clusteru), ulož heslo ke konfiguraci
do souboru, který můžeš číst jen ty, a předej ho přes `RCLONE_PASSWORD_COMMAND`. Kdo má přístup k tvému
účtu, má tím i přístup ke klíčům. Používej to jen na stroji a účtu, kterým věříš.

**Windows:** heslo se uloží zašifrované přes Windows (DPAPI), takže ho přečteš jen ty
na tomto počítači.

1. Ulož heslo (jednou; funguje v PowerShellu 5.1 i 7):
   ```powershell
   Read-Host -AsSecureString "Heslo ke konfiguraci rclone" | ConvertFrom-SecureString | Set-Content "$env:APPDATA\rclone\pass.dpapi"
   ```
2. Vytvoř pomocný skript, který heslo pro rclone rozšifruje (jednou, celý blok najednou):
   ```powershell
   @'
   Add-Type -AssemblyName System.Security
   $h = [IO.File]::ReadAllText("$PSScriptRoot\pass.dpapi").Trim()
   $b = [byte[]]::new($h.Length / 2)
   for ($i = 0; $i -lt $b.Length; $i++) { $b[$i] = [Convert]::ToByte($h.Substring(2 * $i, 2), 16) }
   [Console]::Out.Write([Text.Encoding]::Unicode.GetString([Security.Cryptography.ProtectedData]::Unprotect($b, $null, 'CurrentUser')))
   '@ | Set-Content "$env:APPDATA\rclone\rclone-pass.ps1"
   ```
3. Na začátek skriptu, který spouští rclone (např. `muj_skript.ps1`), dej:
   ```powershell
   $env:RCLONE_PASSWORD_COMMAND = "powershell -NoProfile -ExecutionPolicy Bypass -File `"$env:APPDATA\rclone\rclone-pass.ps1`""
   ```

**Linux / macOS (cron, cluster):**
```bash
read -rsp "Heslo ke konfiguraci rclone: " P && echo && printf '%s\n' "$P" > ~/.config/rclone/pass && unset P
chmod 600 ~/.config/rclone/pass
```
Do skriptu úlohy dej:
```bash
export RCLONE_PASSWORD_COMMAND="cat $HOME/.config/rclone/pass"
```

**Kontrola** (rclone se nesmí ptát na heslo): na Windows spusť skript úlohy ručně, na Linuxu / macOS:
```bash
RCLONE_PASSWORD_COMMAND="cat $HOME/.config/rclone/pass" rclone lsd cesnet:bioimage-but < /dev/null
```

## Kontakt

Přístupy, nové projekty a problémy řeší správce úložiště: **Jiří Chmelík, [chmelikj@vut.cz](mailto:chmelikj@vut.cz)**.

---
<sub>Tento repozitář se generuje automaticky. Změny posílej správci, ne pull requestem.</sub>
