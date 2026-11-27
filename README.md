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
- **Windows** (PowerShell): `winget install -e --id Rclone.Rclone`, pak otevři nový terminál
- **Linux:** `sudo -v ; curl https://rclone.org/install.sh | sudo bash`
  (bez sudo, např. na clusteru: stáhni [binárku](https://rclone.org/downloads/) do `~/.local/bin`)
- **macOS:** `brew install rclone`

Ověření: `rclone version`

## 5. Nastav připojení
```
rclone config create cesnet s3 provider=Ceph endpoint=https://s3.cl4.du.cesnet.cz access_key_id=<access_key> secret_access_key=<secret_key>
```
Klíče se uloží do konfiguračního souboru rclone (umístění vypíše `rclone config file`).
Klíče jsou v něm uložené čitelně, takže soubor nikomu neposílej a necommituj; volitelně ho zašifruj heslem příkazem
`rclone config encryption set`. Příkaz s klíči zůstane v historii terminálu; pokud to nechceš, použij
interaktivní `rclone config`.

## 6. Ověř přístup (až ti správce potvrdí nastavení)
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

## Kontakt

Přístupy, nové projekty a problémy řeší správce úložiště: **Jiří Chmelík, [chmelikj@vut.cz](mailto:chmelikj@vut.cz)**.

---
<sub>Tento repozitář se generuje automaticky. Změny posílej správci, ne pull requestem.</sub>
