# S3 úložiště bioimage-but: návod pro členy

Skupinové S3 úložiště na CESNET (`https://s3.cl4.du.cesnet.cz`).

**Před použitím si přečti [pravidla používání](PRAVIDLA.md).**
Citlivá data šifruj podle [návodu k šifrování](SIFROVANI.md).

## Struktura

```
private-<jméno>/      tvůj soukromý bucket: vidíš ho jen ty a správce
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

## 3. Nastav AWS profil
Potřebuješ [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
ve verzi **2.13 nebo novější** (`aws --version`). Příklad s profilem `bioimage`:

```
aws configure --profile bioimage
aws configure set endpoint_url https://s3.cl4.du.cesnet.cz --profile bioimage
```
`aws configure` se zeptá na Access Key ID a Secret Access Key. Region a výstupní formát nech
prázdné (Enter).

Výsledek v `~/.aws/credentials` a `~/.aws/config`:
```ini
# ~/.aws/credentials
[bioimage]
aws_access_key_id = …
aws_secret_access_key = …

# ~/.aws/config
[profile bioimage]
endpoint_url = https://s3.cl4.du.cesnet.cz
```
Na starší verzi CLI, která `endpoint_url` v profilu nezná, přidávej ke každému příkazu
`--endpoint-url https://s3.cl4.du.cesnet.cz`.

## 4. Pošli správci své ID
```
aws s3api list-buckets --query Owner.ID --output text --profile bioimage
```
Vypíše `94b66cdd_86a7_41a0_9d47_97f7b10fcdb7$<tvoje_id>`. Část před `$` musí být přesně takhle;
jinak máš klíče z jiné skupiny a vrať se ke kroku 2. Správci pošli **část za `$`** a napiš,
do kterých projektů potřebuješ přístup.

## 5. Ověř přístup (až ti správce potvrdí nastavení)
```
aws s3 ls s3://bioimage-but/ --profile bioimage                  # složky: common/, projekty
aws s3 ls s3://bioimage-but/<projekt>/ --profile bioimage        # obsah tvého projektu
aws s3 ls s3://private-<jméno>/ --profile bioimage               # tvůj soukromý bucket
```

## Práce s daty
```
aws s3 cp soubor.tif s3://bioimage-but/<projekt>/ --profile bioimage
aws s3 sync ./data s3://bioimage-but/<projekt>/data --profile bioimage
aws s3 sync ./data s3://private-<jméno>/data --profile bioimage
```
- `aws s3 ls` bez bucketu nic neukáže. Sdílené buckety nejsou vidět ve výpisu, musíš je zadat jménem.
- Chyba `argument of type 'NoneType' is not a container or iterable` znamená **přístup odepřen**.
  Zkontroluj endpoint, profil a jestli jsi v projektu, kam zapisuješ.
- `common/` je pro všechny ve skupině. Každý tam může soubory číst, přepsat i smazat.

## Kontakt

Přístupy, nové projekty a problémy řeší správce úložiště: **TODO: doplnit kontakt**.

---
<sub>Tento repozitář se generuje automaticky. Změny posílej správci, ne pull requestem.</sub>
