# bioimage-but S3 storage: member guide

[🇨🇿 Česky](README.md) · 🇬🇧 English

Group S3 storage at CESNET (`https://s3.cl4.du.cesnet.cz`). You work with it using **rclone**.

**Read the [terms of use](PRAVIDLA.en.md) before you start.**
Encrypt sensitive data as described in the [encryption guide](SIFROVANI.en.md).

In the commands, replace `<user>` with your user name (the same as in your private bucket name,
e.g. `novak`), `<project>` with the project name and `<password>` with your password.

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

## 3. Send your ID to the administrator
In Gatekeeper, the group page shows the **group ID (tenant)** and your **user ID**.
- The group ID must be `94b66cdd_86a7_41a0_9d47_97f7b10fcdb7`. If it is different, you are in a different
  group; go back to step 2.
- Send the administrator your **user ID** and tell them which projects you need access to.

## 4. Install rclone
- **Windows** (PowerShell), then open a new terminal:
  ```powershell
  winget install -e --id Rclone.Rclone
  ```
- **Linux** (with sudo):
  ```bash
  sudo -v ; curl https://rclone.org/install.sh | sudo bash
  ```
  The installation may take a few minutes (it updates the man page index). If it fails because `unzip`
  is missing, run this first:
  ```bash
  sudo apt install unzip
  ```
- **Linux** (without sudo, e.g. on a server or cluster; on ARM replace `amd64` with `arm64`):
  ```bash
  curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
  python3 -m zipfile -e rclone-current-linux-amd64.zip .
  mkdir -p ~/.local/bin && cp rclone-*-linux-amd64/rclone ~/.local/bin/ && chmod +x ~/.local/bin/rclone
  ```
  If the `rclone` command is not found, add `~/.local/bin` to your path:
  ```bash
  echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
  ```
- **macOS:**
  ```bash
  brew install rclone
  ```

Check:
```
rclone version
```

## 5. Encrypt the rclone config (mandatory)
rclone stores your keys in its config file. **The file must always be encrypted with a password**, and
you must do this before you enter any keys:
```
rclone config encryption set
```
Enter a strong config password and keep it in a password manager. From now on rclone asks for it with
every command (`Enter configuration password`).

Check: the following command prints the config file path. The file must start with the line
`# Encrypted rclone configuration File`.
```
rclone config file
```

## 6. Set up the connection
Run:
```
rclone config
```
and answer as follows (for choices, type the text, not the number; numbers differ between versions).
Endpoint to copy:
```
https://s3.cl4.du.cesnet.cz
```

| Prompt | Answer |
|---|---|
| `Enter configuration password` | the config password from step 5 |
| `n) New remote` | `n` |
| `name>` | `cesnet` |
| `Storage>` | `s3` |
| `provider>` | `Ceph` |
| `env_auth>` | Enter |
| `access_key_id>` | your Access key from Gatekeeper |
| `secret_access_key>` | your Secret key from Gatekeeper |
| `region>` | Enter |
| `endpoint>` | `https://s3.cl4.du.cesnet.cz` |
| `location_constraint>`, `acl>`, `server_side_encryption>`, `sse_kms_key_id>`, `bucket_object_lock_enabled>` | Enter |
| `Edit advanced config?` | `n` |
| `Keep this "cesnet" remote?` | `y` |
| `e/n/d/r/c/s/q>` | `q` (quit) |

Check that your keys were saved:
```
rclone config show cesnet
```
The output must contain the lines `access_key_id`, `secret_access_key` and `endpoint = https://s3.cl4.du.cesnet.cz`.
It includes your secret key, so do not send it to anyone. If the keys are missing (you only pressed Enter at the
key prompt), run `rclone config`, choose `e` (edit), remote `cesnet` and enter the keys again.

Never send or commit the config file, not even the encrypted one.

## 7. Check your access (once the administrator confirms the setup)
```
rclone lsd cesnet:bioimage-but                   # folders: common, projects
rclone ls cesnet:bioimage-but/<project>          # contents of your project
rclone ls cesnet:private-<user>                  # your private bucket
```

## Working with data
```
rclone copy ./data cesnet:bioimage-but/<project>/data      # upload (skips identical files)
rclone sync ./data cesnet:bioimage-but/<project>/data      # S3 = copy of the local folder; deletes on S3 what is not local
rclone sync cesnet:bioimage-but/<project>/data ./data      # reverse: local folder = copy of S3; deletes LOCALLY what is not on S3
rclone copy cesnet:bioimage-but/<project>/data ./data      # download
rclone check ./data cesnet:bioimage-but/<project>/data     # compare local data with S3
rclone cat cesnet:bioimage-but/<project>/notes.txt         # print a file
rclone deletefile cesnet:bioimage-but/<project>/file.tif   # delete a file
rclone delete cesnet:bioimage-but/<project>/old            # delete a whole folder incl. subfolders
rclone link --expire 7d cesnet:bioimage-but/<project>/file.tif   # download link for anyone
```
- **`rclone sync SOURCE DEST`** makes the destination an exact copy of the source: it **deletes** everything
  in the destination that is not in the source. The order sets the direction. A `sync` from S3 to your
  computer therefore deletes **local** files that are not on S3. If you do not want to delete, use `rclone copy`.
- Try `sync` and deletions with `--dry-run` first. Deleted files cannot be restored, neither on S3 nor locally.
- `rclone ls cesnet:bioimage-but` (the whole bucket at once) fails with 403. Always specify the project.
- `403 AccessDenied` means you do not have access to that folder.
- `directory not found` for `cesnet:bioimage-but` means the server does not know your keys: they are missing
  from the config (check as in step 6), or they come from a different group in Gatekeeper (group ID, see step 3).
- S3 does not really have folders. A folder appears with its first file; `rclone mkdir` does nothing.
  Some tools, however, create empty "folder markers". Such a folder does not disappear after its files
  are deleted, see below.
- A `rclone link` link works for anyone who has it until it expires (max. 7 days).
- `common/` is shared by the whole group. Anyone can read, overwrite and delete files there.

### Deleting a non-empty folder
First check what will be deleted, then delete:
```
rclone delete cesnet:bioimage-but/<project>/folder --dry-run
```
```
rclone delete cesnet:bioimage-but/<project>/folder
```
This deletes all files in the folder and its subfolders. If the folder is still visible afterwards (empty
folder markers are left), remove them:
```
rclone rmdirs cesnet:bioimage-but/<project>/folder --s3-directory-markers
```
Do not run this on the whole project folder (`cesnet:bioimage-but/<project>`); it would delete the project
itself.

Delete encrypted data through the encryption
layer (`rclone delete <project>-crypt:folder`). `rclone purge` also deletes the folder, but on our storage
it prints a harmless `GetBucketVersioning ... 403` error, so use `delete`.

## Scripts and automation
rclone asks for the config password with every command. In a script, pass it via the
`RCLONE_CONFIG_PASS` variable. **Never put the password directly in a script** or on the command line:
it would stay in the file or in your history. You enter the password once in the terminal and it lasts
until you close it.

**Windows:** first check your PowerShell version:
```powershell
$PSVersionTable.PSVersion
```

1. Enter the password (PowerShell asks for it, nothing is shown):
   - **PowerShell 7** (version 7.x):
     ```powershell
     $env:RCLONE_CONFIG_PASS = Read-Host -MaskInput "rclone config password"
     ```
   - **PowerShell 5.1** (the default "Windows PowerShell", lacks `-MaskInput`):
     ```powershell
     $env:RCLONE_CONFIG_PASS = (New-Object PSCredential "x", (Read-Host -AsSecureString "rclone config password")).GetNetworkCredential().Password
     ```
     In PowerShell 5.1, pasting the password with Ctrl+V does not work (an empty password is stored).
     Paste it with the **right mouse button** or type it by hand.
2. Check: prints the length of the password (not the password itself). If it prints nothing or `0`, the
   password was not stored; repeat step 1.
   ```powershell
   $env:RCLONE_CONFIG_PASS.Length
   ```
3. Run your script in the same terminal. rclone inside no longer asks for the password:
   ```powershell
   .\my_script.ps1
   ```
4. When done, remove the password from the terminal (or close the terminal):
   ```powershell
   Remove-Item Env:\RCLONE_CONFIG_PASS
   ```

**Linux / macOS (bash, zsh):**
1. Enter the password (nothing is shown):
   ```bash
   read -rsp "rclone config password: " RCLONE_CONFIG_PASS && echo && export RCLONE_CONFIG_PASS
   ```
2. Check: prints the length of the password (not the password itself). If it prints `0`, repeat step 1.
   ```bash
   echo ${#RCLONE_CONFIG_PASS}
   ```
3. Run your script in the same terminal. rclone inside no longer asks for the password:
   ```bash
   ./my_script.sh
   ```
4. When done, remove the password from the terminal (or close the terminal):
   ```bash
   unset RCLONE_CONFIG_PASS
   ```

### Unattended jobs
For jobs where nobody can type the password (scheduled jobs, cron, cluster jobs), store the config password
in a file only you can read and pass it via `RCLONE_PASSWORD_COMMAND`. Anyone with access to your account
then also has access to your keys. Only do this on a machine and account you trust.

**Windows:** the password is stored encrypted by Windows (DPAPI), so only you can read it
on this computer.

1. Save the password (once; works in PowerShell 5.1 and 7):
   ```powershell
   Read-Host -AsSecureString "rclone config password" | ConvertFrom-SecureString | Set-Content "$env:APPDATA\rclone\pass.dpapi"
   ```
   In PowerShell 5.1, paste the password with the right mouse button or type it by hand; Ctrl+V does not work.
2. Create a helper script that decrypts the password for rclone (once, paste the whole block):
   ```powershell
   @'
   Add-Type -AssemblyName System.Security
   $h = [IO.File]::ReadAllText("$PSScriptRoot\pass.dpapi").Trim()
   $b = [byte[]]::new($h.Length / 2)
   for ($i = 0; $i -lt $b.Length; $i++) { $b[$i] = [Convert]::ToByte($h.Substring(2 * $i, 2), 16) }
   [Console]::Out.Write([Text.Encoding]::Unicode.GetString([Security.Cryptography.ProtectedData]::Unprotect($b, $null, 'CurrentUser')))
   '@ | Set-Content "$env:APPDATA\rclone\rclone-pass.ps1"
   ```
3. Set the variable: run this line in the terminal and also put it at the start of the script that runs
   rclone (e.g. `my_script.ps1`):
   ```powershell
   $env:RCLONE_PASSWORD_COMMAND = "powershell -NoProfile -ExecutionPolicy Bypass -File `"$env:APPDATA\rclone\rclone-pass.ps1`""
   ```
4. Check: in the same terminal, rclone must not ask for the password:
   ```powershell
   $null | rclone lsd cesnet:bioimage-but
   ```
   The variable only applies in the terminal where you set it. Run your script there as `.\my_script.ps1`;
   with `powershell -File my_script.ps1` or "Run with PowerShell" it runs in a new process, which has to set
   it itself (that is why the line is also in the script). Mind the `$env:` prefix: without it rclone does not
   see the variable. To check whether it is set, run `$env:RCLONE_PASSWORD_COMMAND`.
5. Optional: to stop rclone asking for the password in any terminal, set the variable permanently for your
   account (applies to newly opened terminals):
   ```powershell
   [Environment]::SetEnvironmentVariable("RCLONE_PASSWORD_COMMAND", "powershell -NoProfile -ExecutionPolicy Bypass -File `"$env:APPDATA\rclone\rclone-pass.ps1`"", "User")
   ```
   To remove it:
   ```powershell
   [Environment]::SetEnvironmentVariable("RCLONE_PASSWORD_COMMAND", $null, "User")
   ```

**Linux / macOS (cron, cluster):**
```bash
read -rsp "rclone config password: " P && echo && printf '%s\n' "$P" > ~/.config/rclone/pass && unset P
chmod 600 ~/.config/rclone/pass
```
In the job script:
```bash
export RCLONE_PASSWORD_COMMAND="cat $HOME/.config/rclone/pass"
```
Check (rclone must not ask for the password):
```bash
RCLONE_PASSWORD_COMMAND="cat $HOME/.config/rclone/pass" rclone lsd cesnet:bioimage-but < /dev/null
```

## Contact

Access, new projects and problems are handled by the storage administrator: **Jiří Chmelík, [chmelikj@vut.cz](mailto:chmelikj@vut.cz)**.

---
<sub>This repository is generated automatically. Send changes to the administrator rather than opening a pull request.</sub>
