# Encrypting data on the bioimage-but storage

[🇨🇿 Česky](SIFROVANI.md) · 🇬🇧 English

Data is stored **unencrypted** on the server. The storage administrator can see it, and technically so can
the provider (CESNET). **Upload sensitive data encrypted** using the rclone encryption layer (`crypt`) as
described in this guide.

Prerequisite: rclone is installed, its config is encrypted and the `cesnet:` connection is set up as
described in the [member guide](README.en.md).

## Ground rules
- **Lost password = lost data.** Neither the administrator nor CESNET can recover it. Keep the password
  in a password manager.
- Encrypted files can only be read through the encryption layer with the same password. Through `cesnet:`
  you only see unreadable content with meaningless names.
- **Shared project:** all members need the same password. Share it securely (password manager, in person,
  never in a plain e-mail). When a member leaves, consider re-encrypting.
- Always encrypt into an `encrypted/` subfolder (e.g. `<project>/encrypted/`) and do not mix unencrypted
  files into it.

## 1. Create the encryption layer
Run:
```
rclone config
```
and answer according to the table. Values for `remote>` to copy (replace `<project>` / `<user>`):
```
cesnet:bioimage-but/<project>/encrypted
```
```
cesnet:private-<user>/encrypted
```

| Prompt | Answer |
|---|---|
| `Enter configuration password` | your rclone config password |
| `e/n/d/r/c/s/q>` | `n` (new remote) |
| `name>` | project: `<project>-crypt`, private data: `private` |
| `Storage>` | `crypt` |
| `remote>` | project: `cesnet:bioimage-but/<project>/encrypted`<br>private data: `cesnet:private-<user>/encrypted` |
| `filename_encryption>` | Enter (`standard`) |
| `directory_name_encryption>` | Enter (`true`) |
| `password` → `y/g>` | `y`, then `<password>` twice (for a project, the shared project password) |
| `password2` → `y/g/n>` | `n` |
| `Edit advanced config?` | `n` |
| `Keep this ... remote?` | `y` |
| `e/n/d/r/c/s/q>` | `q` |

The encryption layer `<password>` is different from your rclone config password. Only you know the config
password; the project password is shared by all project members.

## 2. Usage
The same commands as in the [member guide](README.en.md), just use the encryption layer instead of `cesnet:…`:
```
rclone copy ./data <project>-crypt:data          # upload, encrypted
rclone sync ./data <project>-crypt:data          # S3 = copy of the local folder; deletes on S3 what is not local
rclone sync <project>-crypt:data ./data          # reverse: deletes LOCALLY what is not on S3
rclone ls <project>-crypt:                       # listing with decrypted names
rclone copy <project>-crypt:data ./restore       # download and decrypt
rclone cat <project>-crypt:data/file.txt         # print a single file
rclone cryptcheck ./data <project>-crypt:data    # verify that data on S3 matches the local copy
```

## Troubleshooting
- **Empty listing although the data is on S3:** most likely a wrong password. rclone silently skips files
  it cannot decrypt (`Skipping undecryptable file name`) and does not report an error.
- **`403 AccessDenied`:** you do not have access to that folder. Check the project name, or ask the
  administrator for access.
