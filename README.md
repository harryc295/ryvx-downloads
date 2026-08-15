# Ryvx downloads

Public download host for [Ryvx](https://github.com/harryc295/ryvx) — the
desktop installer, and the prebuilt reverse-engineering sandbox image.

Nothing here is source code. The application source lives in the main
repository.

---

## Desktop app

**[Download the Windows installer](../../releases/latest)**

Windows 10 and 11, 64-bit.

Two things to expect on first run, neither of which means anything is
broken:

- **Windows will warn about an "unknown publisher"** and try to block the
  installer. Choose *More info → Run anyway*. The installer is not
  code-signed yet; a signing certificate costs money and hasn't been
  bought. Nothing about that warning is specific to this app — it's what
  Windows says about any unsigned installer.
- **Most screens will be empty until you run a scan.** That's the app
  working, not failing. An empty findings page means nothing has been
  tested yet, not that your systems are clean.

You'll be asked to sign in. Scanning needs a model — see *What you
actually need* below.

---

## RE sandbox guest image

Only needed if you want **malware analysis / reverse engineering**.
Ordinary web and source scanning does not use it and never downloads it.

You should not need to touch these files by hand. Ryvx downloads and
verifies them for you (`ryvx/vm_image_fetch.py`), checking a SHA-256 for
every part against a signed manifest before installing anything.

The alternative is building the image yourself: a Linux host with root,
`debootstrap` and `podman`, an 8 GB rootfs, and up to 90 minutes. This
release exists so you don't have to.

**What's in it:** a Debian bookworm root filesystem containing the static
analysis battery (`file`, `strings`, `radare2`, `capa`, FLOSS), Ghidra for
decompilation, and angr for symbolic solving. Plus a Linux kernel and
initrd to boot it.

**What's not in it:** any malware. This is the analysis environment, not
samples. Analysis runs inside a QEMU microVM with no network interface at
all, and inside that, a container with every capability dropped.

| File | Size | What it is |
|---|---|---|
| `vmlinux` | ~8 MB | Linux kernel the microVM boots |
| `initrd.img` | ~35 MB | Initial ramdisk |
| `rootfs.ext4.gz` | ~1.5 GB | The analysis environment (≈8 GB uncompressed) |
| `manifest.json` | ~1 KB | Sizes and SHA-256 hashes Ryvx verifies against |

---

## What you actually need

Ryvx installs nothing you haven't asked for. Each capability is separate,
and the app's **Setup** screen shows exactly what's present and what's
missing, with an *Install* button where installation can be automated.

| To do this | You need | Automated? |
|---|---|---|
| Scan source code or a live web app | A model (API key, or a local one) | Key: you paste it. Local model: yes — installs Ollama and pulls a model |
| Run scans in an isolated container | Docker | Yes — one click |
| Reverse-engineer malware | QEMU + the guest image above | Yes — one click each |
| Look up file hashes against threat intel | A VirusTotal / MalwareBazaar / YARAify key | Optional; the rest works without them |

**A model is the only hard requirement.** Everything else is per-feature,
and skipping it costs you that feature and nothing else.

On the model: a frontier model via API finds materially more than a small
local one, and for a security tool that gap matters — a scanner that
misses things hands you false confidence. Local models are supported and
genuinely useful when source code can't leave the machine, which is a real
constraint in plenty of places.

---

## Verifying a download yourself

Every file in a guest-image release is listed in `manifest.json` with its
SHA-256. Ryvx checks these automatically. To check by hand:

```bash
sha256sum rootfs.ext4.gz     # Linux/macOS
certutil -hashfile rootfs.ext4.gz SHA256    # Windows
```

and compare against the value in `manifest.json`.
