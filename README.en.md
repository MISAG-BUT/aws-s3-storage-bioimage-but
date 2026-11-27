# bioimage-but S3 storage: member guide

[🇨🇿 Česky](README.md) · 🇬🇧 English

Group S3 storage at CESNET (`https://s3.cl4.du.cesnet.cz`).

**Read the [terms of use](PRAVIDLA.en.md) before you start.**
Encrypt sensitive data as described in the [encryption guide](SIFROVANI.en.md).

## Structure

```
private-<user>/       your private bucket: only you and the administrator can access it
bioimage-but/         shared bucket
├── common/           read/write for all group members
└── <project>/        project members and the administrator only
```

- In the root of `bioimage-but` you can see the names of all projects, but you can only open your own.
- Access is managed by the administrator. Ask them to add you to a project or create a new one.
- The administrator has technical access to all data, including private buckets.

## 1. Accept the VO invitation
The administrator invites you to the virtual organization (VO) via e-INFRA CZ (Perun). Accept the
invitation and sign in with your home institution.

> **Be patient.** Your VO and group membership reaches Gatekeeper with a delay of
> **tens of minutes to hours** (CESNET states at least 30 minutes). You cannot create keys until the
> group shows up in Gatekeeper. Just try again later; nothing will break.

## 2. Create S3 keys in Gatekeeper
1. Open **https://gatekeeper.du.cesnet.cz** (formerly `access.du.cesnet.cz`) and click **LOG IN**.
2. Choose your home organization (e.g. Brno University of Technology) and sign in.
3. In the left menu, select our **VO** and then the **group** (the tenant where the keys will be created).
4. Click **New key**, enter a name (e.g. `laptop`) and create the key.
5. **Save the Access key and the Secret key right away.** The secret key is shown only once. If you
   lose it, create a new key and delete the old one in Gatekeeper.

Gatekeeper also shows the S3 endpoint and your user ID.

## 3. Install the AWS CLI and set up a profile

### Installing AWS CLI v2
You need **AWS CLI v2, version 2.13 or newer**. The `awscli` package from `apt`/`pip` is often the old
version 1, which is not enough. Official guide:
[Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

**Windows** (PowerShell):
```powershell
winget install -e --id Amazon.AWSCLI
# or without winget:
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```
Open a new terminal window after installing.

**Linux** (x86_64; on ARM replace `x86_64` with `aarch64`):
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install                     # update: sudo ./aws/install --update
# without sudo (e.g. on a server/cluster):
./aws/install -i ~/.local/aws-cli -b ~/.local/bin
```

**macOS**:
```bash
brew install awscli
# or the official package:
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o AWSCLIV2.pkg
sudo installer -pkg AWSCLIV2.pkg -target /
```

Check: `aws --version` must print `aws-cli/2.13` or newer.

### Setting up the profile
Replace `<user>` everywhere with your user name, the same as in your private bucket name
`private-<user>` (e.g. `novak`). Likewise, replace `<project>` and `<password>` with real values.
Name the profile after your user (the commands are the same on all systems):

```
aws configure --profile <user>
aws configure set endpoint_url https://s3.cl4.du.cesnet.cz --profile <user>
```
`aws configure` asks for the Access Key ID and Secret Access Key. Leave the region and output format
empty (press Enter).

Result in `~/.aws/credentials` and `~/.aws/config` (on Windows `%USERPROFILE%\.aws\`):
```ini
# ~/.aws/credentials
[<user>]
aws_access_key_id = …
aws_secret_access_key = …

# ~/.aws/config
[profile <user>]
endpoint_url = https://s3.cl4.du.cesnet.cz
```
With an older CLI that does not support `endpoint_url` in the profile, add
`--endpoint-url https://s3.cl4.du.cesnet.cz` to every command.

## 4. Send your ID to the administrator
```
aws s3api list-buckets --query Owner.ID --output text --profile <user>
```
This prints `94b66cdd_86a7_41a0_9d47_97f7b10fcdb7$<your_id>`. The part before `$` must be exactly this;
otherwise your keys belong to a different group, so go back to step 2. Send the administrator
**the part after `$`** and tell them which projects you need access to.

## 5. Check your access (once the administrator confirms the setup)
```
aws s3 ls s3://bioimage-but/ --profile <user>                  # folders: common/, projects
aws s3 ls s3://bioimage-but/<project>/ --profile <user>        # contents of your project
aws s3 ls s3://private-<user>/ --profile <user>                # your private bucket
```

## Working with data
```
aws s3 cp image.tif s3://bioimage-but/<project>/ --profile <user>
aws s3 sync ./data s3://bioimage-but/<project>/data --profile <user>
aws s3 sync ./data s3://private-<user>/data --profile <user>
```
- `aws s3 ls` without a bucket shows nothing. Shared buckets do not appear in the listing; you have to
  name them.
- The error `argument of type 'NoneType' is not a container or iterable` means **access denied**.
  Check the endpoint, the profile, and whether you are a member of the project you are writing to.
- `common/` is shared by the whole group. Anyone can read, overwrite and delete files there.

## Contact

Access, new projects and problems are handled by the storage administrator: **Jiří Chmelík, [chmelikj@vut.cz](mailto:chmelikj@vut.cz)**.

---
<sub>This repository is generated automatically. Send changes to the administrator rather than opening a pull request.</sub>
