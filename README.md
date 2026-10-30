# SpaleHub

The hub for the Spale programs: it holds your account and your licence,
installs SpaleVoice and SpaleStudio, keeps them up to date, and activates them
on the computers you use.

**[Download the newest SpaleHub](https://spale.pages.dev/download/spalehub)**:
that link always hands over the latest installer. Every version is under
[Releases](../../releases). This repository holds only the installers; the
source is private.

## What a release holds

- `SpaleHub_<version>_x64-setup.exe`: the installer
- `latest.yml`: how installed copies learn that a release exists
- `latest.signed.json`: the same facts, signed. SpaleHub 0.5.1 and later
  install an update only when this signature checks out.

Beside the releases, this repository serves `update-policy.json` (what changed
in each version), `hub-manifest.json` (what SpaleStudio installs when SpaleHub
is missing) and the add-on catalogue.

## What it is for

Your licence belongs to your account, not to a computer. Sign in here once and
the programs on that machine unlock; sign in on another machine and they unlock
there too, up to the number of computers your licence covers.

Adding a computer asks for a six-digit code sent to your email, so somebody who
learns your password still cannot take a seat.

## Requirements

Windows 10 or later, 64-bit.
