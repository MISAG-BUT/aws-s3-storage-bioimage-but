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
- **Windows** (PowerShell): `winget install -e --id Rclone.Rclone`, then open a new terminal
- **Linux** (with sudo):
  ```bash
  sudo -v ; curl https://rclone.org/install.sh | sudo bash
  ```
  The installation may take a few minutes (it updates the man page index). If it fails because `unzip`
  is missing, run `sudo apt install unzip` first.
- **Linux** (without sudo, e.g. on a server or cluster; on ARM replace `amd64` with `arm64`):
  ```bash
  curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
  python3 -m zipfile -e rclone-current-linux-amd64.zip .
  mkdir -p ~/.local/bin && cp rclone-*-linux-amd64/rclone ~/.local/bin/ && chmod +x ~/.local/bin/rclone
  ```
  If the `rclone` command is not found, add `export PATH="$HOME/.local/bin:$PATH"` to your `~/.bashrc`.
- **macOS:** `brew install rclone`

Check: `rclone version`

## 5. Set up the connection
```
rclone config create cesnet s3 provider=Ceph endpoint=https://s3.cl4.du.cesnet.cz access_key_id=<access_key> secret_access_key=<secret_key>
```
The keys are stored in the rclone config file (`rclone config file` prints its location).
They are stored in readable form, so never send or commit the file; optionally protect it with a
password using `rclone config encryption set`. The command with your keys stays in your terminal
history; if you do not want that, use the interactive `rclone config`.

## 6. Check your access (once the administrator confirms the setup)
```
rclone lsd cesnet:bioimage-but                   # folders: common, projects
rclone ls cesnet:bioimage-but/<project>          # contents of your project
rclone ls cesnet:private-<user>                  # your private bucket
```

## Working with data
```
rclone copy ./data cesnet:bioimage-but/<project>/data      # upload (skips identical files)
rclone sync ./data cesnet:bioimage-but/<project>/data      # mirror; deletes on S3 what is missing locally
rclone copy cesnet:bioimage-but/<project>/data ./data      # download
rclone check ./data cesnet:bioimage-but/<project>/data     # compare local data with S3
rclone cat cesnet:bioimage-but/<project>/notes.txt         # print a file
rclone deletefile cesnet:bioimage-but/<project>/file.tif   # delete a file
rclone delete cesnet:bioimage-but/<project>/old            # delete the contents of a folder
rclone link --expire 7d cesnet:bioimage-but/<project>/file.tif   # download link for anyone
```
- Try `sync` and deletions with `--dry-run` first. Deleted files cannot be restored.
- `rclone ls cesnet:bioimage-but` (the whole bucket at once) fails with 403. Always specify the project.
- `403 AccessDenied` means you do not have access to that folder.
- S3 has no empty folders. A folder appears with its first file; `rclone mkdir` does nothing.
- A `rclone link` link works for anyone who has it until it expires (max. 7 days).
- `common/` is shared by the whole group. Anyone can read, overwrite and delete files there.

## Contact

Access, new projects and problems are handled by the storage administrator: **Jiří Chmelík, [chmelikj@vut.cz](mailto:chmelikj@vut.cz)**.

---
<sub>This repository is generated automatically. Send changes to the administrator rather than opening a pull request.</sub>
