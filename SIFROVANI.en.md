# Encrypting data on the bioimage-but storage

[🇨🇿 Česky](SIFROVANI.md) · 🇬🇧 English

## What the server supports (tested 2026-11-26 on s3.cl4.du.cesnet.cz)

| Option | Status |
|---|---|
| SSE-S3: server encrypts with its own key | ❌ not supported (`InvalidArgument`) |
| SSE-C: server encrypts with a key sent by the client | ❌ not supported (`InvalidRequest`) |
| Default bucket encryption | ❌ not configured |

Transfers use HTTPS (encrypted in transit), but **data is stored unencrypted on the server**.
The storage administrator can see it, and technically so can the provider (CESNET).
For sensitive data, use **client-side encryption**: encrypt files before you upload them.

## Options

| Tool | Best for | Notes |
|---|---|---|
| **rclone crypt** (recommended, ✅ tested) | day-to-day work, syncing whole folders | open source, Win/Linux/macOS, encrypts file contents and names |
| **restic** | backups | encrypted, deduplicated backups straight to S3 |
| **age** / **gpg** | single files and archives | simple, manual |

### Ground rules
- **Lost password = lost data.** Neither the administrator nor CESNET can recover it. Keep the password
  in a password manager.
- Encrypted files can only be read through rclone with the same password. `aws s3 cp` or a browser
  only give you unreadable content with meaningless names.
- **Shared project:** all members need the same password. Share it securely (password manager, in person,
  never in a plain e-mail). When a member leaves, consider re-encrypting.
- Always encrypt into a dedicated subfolder (e.g. `<project>/encrypted/`) and do not mix unencrypted files
  into it.

## rclone crypt: how-to

### 1. Install
- **Windows:** `winget install -e --id Rclone.Rclone` (then open a new terminal)
- **Linux:** `sudo -v ; curl https://rclone.org/install.sh | sudo bash`, or without sudo download the
  [binary](https://rclone.org/downloads/) to `~/.local/bin`
- **macOS:** `brew install rclone`

Check: `rclone version`

### 2. Connect to S3
This reuses the keys from your AWS profile `<user>` (see the [member guide](README.en.md)), so you do not
enter them again:
```
rclone config create cesnet s3 provider=Ceph env_auth=true profile=<user> endpoint=https://s3.cl4.du.cesnet.cz
```
Check: `rclone lsd cesnet:bioimage-but` lists the folders in the shared bucket.

### 3. Encryption layer
For a project (one password shared by the whole project):
```
rclone config create <project>-crypt crypt remote=cesnet:bioimage-but/<project>/encrypted password=<password> --obscure
```
For private data:
```
rclone config create private crypt remote=cesnet:private-<user>/encrypted password=<password> --obscure
```
`--obscure` only masks the password in `rclone.conf`, it does not encrypt it. Anyone who gets your
`rclone.conf` also gets the password. Protect it like your AWS keys; optionally encrypt it with
`rclone config encryption set`. A password typed on the command line stays in your shell history; the
interactive `rclone config` is safer.

### 4. Usage
```
rclone copy ./data <project>-crypt:data          # upload, encrypted
rclone sync ./data <project>-crypt:data          # mirror; deletes on S3 what is missing locally
rclone ls <project>-crypt:                       # listing with decrypted names
rclone copy <project>-crypt:data ./restore       # download and decrypt
rclone cat <project>-crypt:data/file.txt         # print a single file
rclone cryptcheck ./data <project>-crypt:data    # verify that data on S3 matches the local copy
```
Mounting as a drive (`rclone mount <project>-crypt: X:`) needs [WinFsp](https://winfsp.dev/) on Windows and
[macFUSE](https://osxfuse.github.io/) on macOS. We have not tested this option.

### What we verified (2026-11-27)
- Upload, download and `cryptcheck`: files are identical after download.
- On S3 both contents and file/folder names are encrypted (`aws s3 ls` shows e.g. `gskmalbnrqag3dhgev13d1ui6o/…`).
- A second user with the same password can read the data.
- With a wrong password, rclone lists nothing. It skips the files with `Skipping undecryptable file name`
  but **does not report an error**, so an empty listing may mean a wrong password.
- Without access to the folder you get `403 AccessDenied`.

## age: single files
Install: `winget install -e --id FiloSottile.age` / `apt install age` / `brew install age`
```
age -p -o data.tar.age data.tar                 # encrypt with a password
aws s3 cp data.tar.age s3://private-<user>/ --profile <user>
age -d -o data.tar data.tar.age                 # decrypt
```
