# S3 úložiště bioimage-but: návod pro členy

🇨🇿 Česky · [🇬🇧 English](README.en.md)

Skupinové S3 úložiště na CESNET (`https://s3.cl4.du.cesnet.cz`).

**Před použitím si přečti [pravidla používání](PRAVIDLA.md).**
Citlivá data šifruj podle [návodu k šifrování](SIFROVANI.md).

## Struktura

```
private-<uzivatel>/      tvůj soukromý bucket: vidíš ho jen ty a správce
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

V Gatekeeperu je vidět i S3 endpoint a tvoje ID uživatele.

## 3. Nainstaluj AWS CLI a nastav profil

### Instalace AWS CLI v2
Potřebuješ **AWS CLI v2 ve verzi 2.13 nebo novější**. Balíček `awscli` z `apt`/`pip` bývá stará
verze 1, ta nestačí. Oficiální návod:
[Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

**Windows** (PowerShell):
```powershell
winget install -e --id Amazon.AWSCLI
# nebo bez winget:
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```
Po instalaci otevři nové okno terminálu.

**Linux** (x86_64; na ARM nahraď `x86_64` za `aarch64`):
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install                     # aktualizace: sudo ./aws/install --update
# bez sudo (např. na serveru/clusteru):
./aws/install -i ~/.local/aws-cli -b ~/.local/bin
```

**macOS**:
```bash
brew install awscli
# nebo oficiální balíček:
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o AWSCLIV2.pkg
sudo installer -pkg AWSCLIV2.pkg -target /
```

Ověření: `aws --version` musí vypsat `aws-cli/2.13` nebo vyšší.

### Nastavení profilu
Za `<uzivatel>` všude dosaď své uživatelské jméno, stejné jako v názvu soukromého bucketu
`private-<uzivatel>` (např. `novak`). Stejně tak `<projekt>` a `<heslo>` nahraď skutečnými hodnotami.
Profil pojmenuj stejně jako svého uživatele (příkazy jsou stejné na všech systémech):

```
aws configure --profile <uzivatel>
aws configure set endpoint_url https://s3.cl4.du.cesnet.cz --profile <uzivatel>
```
`aws configure` se zeptá na Access Key ID a Secret Access Key. Region a výstupní formát nech
prázdné (Enter).

Výsledek v `~/.aws/credentials` a `~/.aws/config` (na Windows `%USERPROFILE%\.aws\`):
```ini
# ~/.aws/credentials
[<uzivatel>]
aws_access_key_id = …
aws_secret_access_key = …

# ~/.aws/config
[profile <uzivatel>]
endpoint_url = https://s3.cl4.du.cesnet.cz
```
Na starší verzi CLI, která `endpoint_url` v profilu nezná, přidávej ke každému příkazu
`--endpoint-url https://s3.cl4.du.cesnet.cz`.

## 4. Pošli správci své ID
```
aws s3api list-buckets --query Owner.ID --output text --profile <uzivatel>
```
Vypíše `94b66cdd_86a7_41a0_9d47_97f7b10fcdb7$<tvoje_id>`. Část před `$` musí být přesně takhle;
jinak máš klíče z jiné skupiny a vrať se ke kroku 2. Správci pošli **část za `$`** a napiš,
do kterých projektů potřebuješ přístup.

## 5. Ověř přístup (až ti správce potvrdí nastavení)
```
aws s3 ls s3://bioimage-but/ --profile <uzivatel>                  # složky: common/, projekty
aws s3 ls s3://bioimage-but/<projekt>/ --profile <uzivatel>        # obsah tvého projektu
aws s3 ls s3://private-<uzivatel>/ --profile <uzivatel>               # tvůj soukromý bucket
```

## Práce s daty
```
aws s3 cp soubor.tif s3://bioimage-but/<projekt>/ --profile <uzivatel>
aws s3 sync ./data s3://bioimage-but/<projekt>/data --profile <uzivatel>
aws s3 sync ./data s3://private-<uzivatel>/data --profile <uzivatel>
```
- `aws s3 ls` bez bucketu nic neukáže. Sdílené buckety nejsou vidět ve výpisu, musíš je zadat jménem.
- Chyba `argument of type 'NoneType' is not a container or iterable` znamená **přístup odepřen**.
  Zkontroluj endpoint, profil a jestli jsi v projektu, kam zapisuješ.
- `common/` je pro všechny ve skupině. Každý tam může soubory číst, přepsat i smazat.

## Kontakt

Přístupy, nové projekty a problémy řeší správce úložiště: **TODO: doplnit kontakt**.

---
<sub>Tento repozitář se generuje automaticky. Změny posílej správci, ne pull requestem.</sub>
