# Terms of use for the bioimage-but storage

[🇨🇿 Česky](PRAVIDLA.md) · 🇬🇧 English

> **TODO / DRAFT:** working version to be completed and approved by the administrator. Items marked ❓ still need a decision.

## 1. Purpose
The storage is for storing and sharing the group's research data within projects. It is not meant for
private data unrelated to work, nor as the only copy of irreplaceable data.

## 2. Access
- Access is granted to VO members based on an invitation. Keys are personal: do not share them and never
  commit them anywhere (git, shared scripts, notebooks).
- The rclone config containing your keys **must always be encrypted with a password** (`rclone config encryption set`).
- If you suspect a key has leaked, delete it in Gatekeeper immediately and tell the administrator.
- The administrator has technical access to all data, including private buckets. This access is used
  only for administration (backups, clean-up, a member leaving, troubleshooting).
- ❓ When leaving the group: access is revoked on [date]; data in the private bucket will be deleted after
  [N] days unless agreed otherwise.

## 3. Data
- **Sensitive and personal data** (patient data, GDPR-covered data) ❓ [not allowed / anonymized only /
  only encrypted according to the [guide](SIFROVANI.en.md) and with the project lead's approval].
- Follow the licence and contractual terms of the data (partner data, NDAs).
- Every member can see and modify `common/`. Do not put anything there that not everyone should see,
  or anything you cannot afford to lose.
- Store project data in the project folder, not in the root or in `common/`.
- ❓ Recommended project structure (e.g. `raw/`, `processed/`, `results/`) and file naming.

## 4. Capacity and clean-up
- ❓ The group quota is [X TB]. Announce large transfers (>[Y] TB) to the administrator in advance.
- Delete temporary and intermediate files you no longer need.
- ❓ The administrator may delete content of `common/` older than [N] days.

## 5. Backups and responsibility
- The storage **is not a backup**. A deleted or overwritten file cannot be restored (versioning is not
  enabled). ❓ Enable versioning for projects?
- Whoever encrypted the data is responsible for the encryption keys. A lost key means lost data; the
  administrator cannot recover it.

## 6. Provider terms
The CESNET / e-INFRA CZ terms of service (AUP), which you accepted when signing in to Gatekeeper, also apply.

## 7. Breach of the rules
❓ The administrator may restrict or revoke access if the rules are broken.

---
Version: draft, 2026-11-27
