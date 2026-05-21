# Lab 22-1d: Inspect ubi9 Remotely with skopeo

**RHCSA EX200 Lab** | Series: Container Management (Lab 22)  
**Prerequisite:** Lab 22-1c complete (`conadm` sudo verified)  
**Time Estimate:** ~5 minutes

---

## 🎯 Objective

Use `skopeo` to inspect the latest ubi9 image from the Red Hat registry **without downloading it**. This lets you verify image metadata — architecture, OS, version, and layer count — before committing to a pull.

---

## 📚 Command Decision Map

| Task | Command |
|---|---|
| Check if skopeo is installed | `skopeo --version` |
| Install skopeo | `sudo dnf install skopeo -y` |
| Inspect remote image (no download) | `skopeo inspect docker://<registry>/<image>:<tag>` |
| Inspect specific version | `skopeo inspect docker://registry.access.redhat.com/ubi9/ubi:9.8` |
| Filter output to key fields | `skopeo inspect ... \| jq '{version: .Labels.version, arch: .Architecture}'` |

---

## 🧠 Concept: What Is skopeo?

`skopeo` is a command-line tool for working with container images and registries — **without a container runtime**. You don't need Docker or podman running to use it.

| Tool | What it does | Downloads image? |
|---|---|---|
| `skopeo inspect` | Reads image metadata from a registry | ❌ No |
| `podman pull` | Downloads image to local storage | ✅ Yes |
| `podman inspect` | Reads metadata of a **locally stored** image | N/A |

> **Analogy:** `skopeo inspect` is like `git log --remote` — you read the history and metadata without fetching the actual content.

On the RHCSA exam, `skopeo` is used specifically when you need to verify an image before pulling — checking architecture compatibility, available tags, or image size without using storage or bandwidth.

---

## 🧠 Concept: What Is ubi9?

**UBI** stands for **Universal Base Image** — Red Hat's minimal, freely distributable container base image.

| Property | Value |
|---|---|
| Registry | `registry.access.redhat.com` |
| Image path | `ubi9/ubi` |
| License | Free to use and redistribute |
| Base OS | RHEL 9 userland |
| Typical use | Base layer for containerized RHEL applications |

> UBI images are intentionally minimal — they contain just enough to run RHEL-based workloads in containers. This is why you'll see only **1 layer** in the inspect output.

---

## 🔧 Steps

### Step 1 — Switch to `conadm`

```bash
su - conadm
```

**Expected output:**

```
Password:
[conadm@ip-172-31-0-157 ~]$
```

> Always use `su -` (with the dash) to load the full login environment. See Lab 22-1c for the full explanation.

---

### Step 2 — Check if skopeo is installed

```bash
skopeo --version
```

**Actual output (first attempt):**

```
-bash: skopeo: command not found
```

#### Output explained

`command not found` means the binary is not in any directory listed in `$PATH`. skopeo is not installed by default on all RHEL configurations — it must be added via `dnf`.

> This is a normal situation. On the exam, if a required tool is missing, install it with `dnf` before proceeding.

---

### Step 3 — Install skopeo

```bash
sudo dnf install skopeo -y
```

**Actual output:**

```
Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered with an entitlement server. You can use "rhc" or
"subscription-manager" to register.

Last metadata expiration check: 1:29:39 ago on Thu May 21 13:43:04 2026.
Dependencies resolved.
=============================================================================
 Package        Arch   Version             Repository                   Size
=============================================================================
Installing:
 skopeo         x86_64 2:1.22.2-1.el10_2   rhel-10-appstream-rhui-rpms 7.9 M
Installing dependencies:
 podman-sequoia x86_64 0.4.0~pqc.2-1.el10  rhel-10-appstream-rhui-rpms 2.5 M

Transaction Summary
=============================================================================
Install  2 Packages

Total download size: 10 M
Installed size: 33 M
...
Complete!
```

#### Output explained — section by section

**Subscription warning block:**

```
Unable to read consumer identity
This system is not registered with an entitlement server.
```

| Line | Meaning |
|---|---|
| `Unable to read consumer identity` | This EC2 instance is not registered with Red Hat's subscription service (RHSM). |
| `You can use "rhc" or "subscription-manager" to register.` | Informational only — suggests registration tools. |

> **This is not an error.** On AWS, RHEL instances pull packages from **RHUI** (Red Hat Update Infrastructure) repos managed by AWS — not from RHSM. The warning is expected and can be ignored. Installation proceeds normally.

**Repository metadata:**

```
Last metadata expiration check: 1:29:39 ago on Thu May 21 13:43:04 2026.
```

dnf caches repository metadata locally to speed up package resolution. This line confirms the cache is recent (1 hour 29 minutes old). If it were stale, dnf would refresh it automatically.

**Package resolution table:**

```
 Package        Arch   Version             Repository                   Size
 skopeo         x86_64 2:1.22.2-1.el10_2   rhel-10-appstream-rhui-rpms 7.9 M
 podman-sequoia x86_64 0.4.0~pqc.2-1.el10  rhel-10-appstream-rhui-rpms 2.5 M
```

| Column | Meaning |
|---|---|
| `Package` | Name of the package being installed |
| `Arch` | CPU architecture — `x86_64` matches your EC2 instance |
| `Version` | `2:1.22.2` — the `2:` prefix is the **epoch** (version precedence number, not the major version) |
| `Repository` | `rhel-10-appstream-rhui-rpms` — the AWS-hosted RHUI repo serving this package |
| `Size` | Download size of this package only |

> **Why is `podman-sequoia` being installed?** skopeo depends on `podman-sequoia` for cryptographic signature handling (post-quantum crypto support). dnf resolves and installs dependencies automatically — you don't need to request them explicitly.

**Transaction summary:**

```
Install  2 Packages
Total download size: 10 M
Installed size: 33 M
```

| Line | Meaning |
|---|---|
| `Install 2 Packages` | 1 requested (skopeo) + 1 dependency (podman-sequoia) |
| `Total download size: 10 M` | Compressed size pulled from the network |
| `Installed size: 33 M` | Uncompressed size on disk after installation |

**Transaction phases:**

```
Running transaction check    → Verifies package compatibility
Transaction check succeeded. → No conflicts detected
Running transaction test     → Dry run to catch install errors before writing to disk
Transaction test succeeded.  → Safe to proceed
Running transaction          → Actual installation begins
```

> `Transaction test` is a safety check — dnf simulates the install first. If it would fail, it stops here rather than leaving your system in a broken partial state.

**Final confirmation:**

```
Installed:
  podman-sequoia-0.4.0~pqc.2-1.el10.x86_64  skopeo-2:1.22.2-1.el10_2.x86_64
Complete!
```

Both packages installed successfully. `Complete!` = no errors.

---

### Step 4 — Confirm skopeo is now available

```bash
skopeo --version
```

**Actual output:**

```
skopeo version 1.22.2 commit: 89020f547653ac78655b687cbbb61625594738cc
```

| Field | Meaning |
|---|---|
| `skopeo version 1.22.2` | skopeo is installed and in `$PATH` — ready to use |
| `commit: 89020f54...` | Exact Git commit this binary was built from — useful for bug reports or pinning versions |

---

### Step 5 — Inspect ubi9 remotely

```bash
skopeo inspect docker://registry.access.redhat.com/ubi9/ubi:latest
```

#### Command breakdown

| Part | Meaning |
|---|---|
| `skopeo inspect` | Read image metadata from a registry — no download |
| `docker://` | Transport protocol — tells skopeo to use the Docker/OCI registry API. **Required** even for Red Hat registries. |
| `registry.access.redhat.com` | Red Hat's public container registry (no auth required for UBI images) |
| `ubi9/ubi` | Image name — Universal Base Image 9 |
| `:latest` | Tag — resolves to the most recent stable release |

> ⚠️ **Common mistake:** Do not split the URL. `registry.access.redhat.com` is one hostname — not `registry.access/redhat.com`.

**Actual output (key sections):**

```json
{
    "Name": "registry.access.redhat.com/ubi9/ubi",
    "Digest": "sha256:808f291a8c255bed32c89a170d42cb280843e533b30eefef565e116066d207c2",
    "RepoTags": [
        "9.0.0",
        "9.0.0-1468",
        "9.4",
        "9.5",
        "9.8",
        "latest",
        ...
    ],
    "Created": "2026-04-29T11:00:15.879011974Z",
    "Labels": {
        "architecture": "x86_64",
        "build-date": "2026-04-29T10:59:59Z",
        "com.redhat.component": "ubi9-container",
        "name": "ubi9/ubi",
        "vendor": "Red Hat, Inc.",
        "version": "9.8"
    },
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:c9f6ad4a5dfbfbcdba1424a26bd0755c2dcee28df785e495fffbfc060638474c"
    ],
    "LayersData": [
        {
            "MIMEType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
            "Digest": "sha256:c9f6ad4a5dfb...",
            "Size": 80492959,
            "Annotations": null
        }
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
        "container=oci"
    ]
}
```

#### Output explained — field by field

| Field | Value | Meaning |
|---|---|---|
| `"Name"` | `registry.access.redhat.com/ubi9/ubi` | Confirms you inspected the correct image and registry |
| `"Digest"` | `sha256:808f291a...` | Content-addressable hash of this exact image manifest — unique fingerprint |
| `"RepoTags"` | Long list of versions | Every tag ever published for this image (`9.0.0`, `9.4`, `9.5`, `9.8`, `latest`, etc.) |
| `"Created"` | `2026-04-29T11:00:15Z` | When Red Hat built and published this image |
| `"version": "9.8"` | `9.8` | Current ubi9 release — the actual RHEL minor version |
| `"Architecture"` | `amd64` | Built for x86_64 — matches your EC2 instance ✅ |
| `"Os"` | `linux` | Target operating system ✅ |
| `"Layers"` | 1 entry | Only **1 filesystem layer** — confirms this is a minimal base image |
| `"Size": 80492959` | ~80 MB | Compressed layer size on disk after pull |
| `"Env"` | PATH + `container=oci` | Default environment variables baked into the image at build time |

> **The long `RepoTags` list** is expected — it shows every version tag ever published for this image in the registry. It is not an error. When you only need specific fields, pipe through `jq`:

```bash
skopeo inspect docker://registry.access.redhat.com/ubi9/ubi:latest \
  | jq '{version: .Labels.version, arch: .Architecture, os: .Os, layers: (.Layers | length)}'
```

**Expected filtered output:**

```json
{
  "version": "9.8",
  "arch": "amd64",
  "os": "linux",
  "layers": 1
}
```

> `jq` must be installed separately if not present: `sudo dnf install jq -y`

---

## ✅ Lab Checklist

- [ ] `skopeo --version` returns a version number
- [ ] `skopeo inspect` runs without error
- [ ] Output shows `"Architecture": "amd64"` ✅
- [ ] Output shows `"Os": "linux"` ✅
- [ ] Output shows `"version": "9.8"` (or current) ✅
- [ ] Only 1 layer confirmed (minimal base image) ✅
- [ ] No image downloaded to disk ✅

---

## ⚠️ Common Pitfalls

| Mistake | Symptom | Fix |
|---|---|---|
| skopeo not installed | `command not found` | `sudo dnf install skopeo -y` |
| Split URL in command | `Error: ... no such host` | Must be `registry.access.redhat.com` (one hostname) |
| Missing `docker://` prefix | `Error: unsupported transport` | Always prefix with `docker://` for registry inspection |
| `FATA: Error pinging container registry` | Connection refused or DNS failure | Check network connectivity — registry unreachable |
| Confusing inspect with pull | Unexpected disk usage | `skopeo inspect` = zero download; `podman pull` = full download |

---

## 📌 RHCSA Exam Strategy

- `skopeo inspect` is your **pre-pull verification tool** — use it to confirm architecture and version before pulling.
- The `docker://` transport prefix is **always required** for registry inspection, even on non-Docker registries.
- A long `RepoTags` list is normal output — don't mistake it for an error.
- If a tool is missing on the exam, `sudo dnf install <package> -y` is the correct fix — always verify with `--version` after.
- `jq` can filter noisy JSON output into just the fields you need — worth knowing for the exam and production work.

---

## ➡️ Next Lab

**[Lab 22-1e: Pull ubi9 image with podman](https://github.com/kelvintechnical/Pull-ubi9-Image-with-podman)**

---

## 🔗 Series Index

| Lab | Topic |
|---|---|
| ✅ [22-1a](https://github.com/kelvintechnical/Create-User-Account-Conadm) | Create user `conadm` |
| ✅ [22-1b](https://github.com/kelvintechnical/Grant-Conadm-Full-Rights) | Grant `conadm` full sudo rights |
| ✅ [22-1c](https://github.com/kelvintechnical/Verify-Sudo-Access-Conadm) | Verify sudo access |
| 👉 **22-1d** | Inspect ubi9 image with skopeo ← *you are here* |
| [22-1e](https://github.com/kelvintechnical/Pull-ubi9-Image-with-podman) | Pull ubi9 image with podman |
| [22-1f](https://github.com/kelvintechnical/launch-container-interative-terminal) | Launch container with port mapping |
| [22-1g](https://github.com/kelvintechnical/run-commands-inside-terminal) | Run commands inside container |
| [22-1h](https://github.com/kelvintechnical/verify-port-mapping-from-host) | Verify port mapping from host |

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
