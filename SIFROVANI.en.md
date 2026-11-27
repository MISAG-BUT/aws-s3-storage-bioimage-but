# Encrypting data on the bioimage-but storage

[🇨🇿 Česky](SIFROVANI.md) · 🇬🇧 English

Data is stored **unencrypted** on the server. The storage administrator can see it, and technically so can
the provider (CESNET). Therefore:

- upload **regular data** with the **AWS CLI** (see the [member guide](README.en.md));
- upload **sensitive data** **encrypted with rclone** as described in this guide.

## Ground rules
- **Lost password = lost data.** Neither the administrator nor CESNET can recover it. Keep the password
  in a password manager.
- Encrypted files can only be read through rclone with the same password. Through the AWS CLI you only see
  unreadable content with meaningless names.
- **Shared project:** all members need the same password. Share it securely (password manager, in person,
  never in a plain e-mail). When a member leaves, consider re-encrypting.
- Always encrypt into an `encrypted/` subfolder (e.g. `<project>/encrypted/`) and do not mix unencrypted
  files into it.

## 1. Install rclone
- **Windows:** `winget install -e --id Rclone.Rclone` (then open a new terminal)
- **Linux:** `sudo -v ; curl https://rclone.org/install.sh | sudo bash`, or without sudo download the
  [binary](https://rclone.org/downloads/) to `~/.local/bin`
- **macOS:** `brew install rclone`

Check: `rclone version`

## 2. Connect to S3
This reuses the keys from your AWS profile `<user>`, so you do not enter them again:
```
rclone config create cesnet s3 provider=Ceph env_auth=true profile=<user> endpoint=https://s3.cl4.du.cesnet.cz
```
Check: `rclone lsd cesnet:bioimage-but` lists the folders in the shared bucket.

## 3. Encryption layer
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

## 4. Usage
```
rclone copy ./data <project>-crypt:data          # upload, encrypted
rclone sync ./data <project>-crypt:data          # mirror; deletes on S3 what is missing locally
rclone ls <project>-crypt:                       # listing with decrypted names
rclone copy <project>-crypt:data ./restore       # download and decrypt
rclone cat <project>-crypt:data/file.txt         # print a single file
rclone cryptcheck ./data <project>-crypt:data    # verify that data on S3 matches the local copy
```

## Troubleshooting
- **Empty listing although the data is on S3:** most likely a wrong password. rclone silently skips files
  it cannot decrypt (`Skipping undecryptable file name`) and does not report an error.
- **`403 AccessDenied`:** you do not have access to that folder. Check the project name and the profile,
  or ask the administrator for access.
