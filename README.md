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
rclone sync ./data cesnet:bioimage-but/<projekt>/data      # zrcadlí, na S3 smaže, co lokálně chybí
rclone copy cesnet:bioimage-but/<projekt>/data ./data      # stáhne
rclone check ./data cesnet:bioimage-but/<projekt>/data     # porovná lokální data s S3
rclone cat cesnet:bioimage-but/<projekt>/poznamka.txt      # vypíše soubor
rclone deletefile cesnet:bioimage-but/<projekt>/soubor.tif # smaže soubor
rclone delete cesnet:bioimage-but/<projekt>/stare          # smaže obsah složky
rclone link --expire 7d cesnet:bioimage-but/<projekt>/soubor.tif   # odkaz ke stažení pro kohokoli
```
- Před `sync` a mazáním si to vyzkoušej s `--dry-run`. Smazané soubory nejde obnovit.
- `rclone ls cesnet:bioimage-but` (celý bucket najednou) skončí chybou 403. Vždy zadej projekt.
- `403 AccessDenied` znamená, že do dané složky nemáš přístup.
- S3 nemá prázdné složky. Složka vznikne až s prvním souborem, `rclone mkdir` nic neudělá.
- Odkaz z `rclone link` funguje pro kohokoli, kdo ho má, až do vypršení (max. 7 dní).
- `common/` je pro všechny ve skupině. Každý tam může soubory číst, přepsat i smazat.

## Skripty a automatizace
rclone se na heslo ke konfiguraci ptá při každém příkazu. Ve skriptu mu ho předáš proměnnou
`RCLONE_CONFIG_PASS`. **Heslo nikdy nepiš přímo do skriptu** ani na příkazovou řádku: zůstalo by
v souboru nebo v historii.

Zadej heslo jednou na začátku práce v terminálu; platí jen do jeho zavření:

**Linux / macOS (bash, zsh):**
```bash
read -rs RCLONE_CONFIG_PASS && export RCLONE_CONFIG_PASS   # zeptá se na heslo, nevypisuje ho
./muj_skript.sh                                            # rclone uvnitř se už neptá
unset RCLONE_CONFIG_PASS                                   # po skončení zapomene
```

**Windows (PowerShell 7):**
```powershell
$env:RCLONE_CONFIG_PASS = Read-Host -MaskInput "Heslo ke konfiguraci rclone"
.\muj_skript.ps1
Remove-Item Env:\RCLONE_CONFIG_PASS
```

**Windows (PowerShell 5.1)**, výchozí „Windows PowerShell“, který nezná `-MaskInput`:
```powershell
$s = Read-Host -AsSecureString "Heslo ke konfiguraci rclone"
$env:RCLONE_CONFIG_PASS = (New-Object PSCredential "x", $s).GetNetworkCredential().Password
.\muj_skript.ps1
Remove-Item Env:\RCLONE_CONFIG_PASS
```
Verzi zjistíš příkazem:
```powershell
$PSVersionTable.PSVersion
```

**Úlohy bez obsluhy** (cron, úlohy na clusteru), kde heslo nemá kdo zadat: ulož ho do souboru čitelného
jen pro tebe a předej ho přes `RCLONE_PASSWORD_COMMAND`:
```bash
read -rs P && printf '%s\n' "$P" > ~/.config/rclone/pass && unset P
chmod 600 ~/.config/rclone/pass
export RCLONE_PASSWORD_COMMAND="cat $HOME/.config/rclone/pass"   # do skriptu úlohy
```
Kdo má přístup k tvému účtu, má tím i přístup ke klíčům. Používej to jen na stroji a účtu, kterým věříš.

## Kontakt

Přístupy, nové projekty a problémy řeší správce úložiště: **Jiří Chmelík, [chmelikj@vut.cz](mailto:chmelikj@vut.cz)**.

---
<sub>Tento repozitář se generuje automaticky. Změny posílej správci, ne pull requestem.</sub>
