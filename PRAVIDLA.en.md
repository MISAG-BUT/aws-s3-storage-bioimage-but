# Terms of use for the bioimage-but storage

[🇨🇿 Česky](PRAVIDLA.md) · 🇬🇧 English

## 1. Purpose
The storage is for storing and sharing the group's research data within projects. It is not meant for
private data unrelated to work, nor as the only copy of irreplaceable data.

## 2. Access
- Access is granted to VO members based on an invitation. Keys are personal: do not share them and never
  commit them anywhere (git, shared scripts, notebooks).
- The rclone config containing your keys **must always be encrypted with a password** (`rclone config encryption set`).
- If you suspect a key has leaked, delete it in Gatekeeper immediately and tell the administrator.
- The administrator has technical access to all data, including private buckets. This access is used
  only for administration (clean-up, a member leaving, troubleshooting).
- **Leaving the group:** access is revoked on the day you leave. Your private bucket is deleted
  **30 days after you leave** unless you agree otherwise with the administrator. Data in project folders
  stays with the project.

## 3. Data
- **Sensitive and personal data** (patient data, personal data under GDPR) may be stored **only in
  encrypted form** according to the [encryption guide](SIFROVANI.en.md) and **with the project lead's
  approval**.
- Follow the licence and contractual terms of the data (partner data, NDAs).
- Every member can see and modify `common/`. Do not put anything there that not everyone should see,
  or anything you cannot afford to lose.
- Store project data in the project folder, not in the root or in `common/`.
- **Recommended project structure:**
  ```
  <project>/
  ├── README.md      project description, data source, contact person
  ├── raw/           original data, do not modify
  ├── processed/     processed data
  ├── results/       results
  └── encrypted/     encrypted data (rclone crypt)
  ```
  Depending on the project, raw and other data may be kept entirely in `encrypted/` (e.g.
  `encrypted/raw/`, `encrypted/processed/`).

## 4. Capacity and clean-up
- The quota for the whole group is **20 TB and 20 million files**, shared by all members and projects.
- Announce transfers **larger than 1 TB** to the administrator in advance.
- Delete temporary and intermediate files you no longer need. Pack large numbers of small files into an
  archive (e.g. `.tar`); the number of files is limited too.
- The administrator may delete files in `common/` **older than 90 days**.

## 5. Backups and responsibility
- The storage **is not a backup**. A deleted or overwritten file cannot be restored. Keep important data
  elsewhere as well.
- Whoever encrypted the data is responsible for the encryption passwords. A lost password means lost data;
  the administrator cannot recover it.

## 6. Provider terms
The CESNET / e-INFRA CZ terms of service (AUP), which you accepted when signing in to Gatekeeper, also apply.

## 7. Breach of the rules
If the rules are broken, the administrator may restrict or revoke access and remove data that breaks the
rules (e.g. unencrypted sensitive data).

---
Version 1.0, 2026-11-27
