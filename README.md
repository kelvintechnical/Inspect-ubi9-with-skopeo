# Lab 22-1d: Inspect ubi9 Remotely with skopeo

**RHCSA EX200 Lab** | Series: Container Management (Lab 22)  
**Prerequisite:** Lab 22-1c complete (`conadm` sudo verified)  
**Time Estimate:** 5 minutes

---

## 🎯 Objective

Use `skopeo` to inspect the latest ubi9 image from the Red Hat registry **without downloading it**. This lets you verify image metadata, architecture, size, and available tags before committing to a pull.

---

## 📚 Command Decision Map

| Task | Command |
|---|---|
| Check skopeo is installed | `skopeo --version` |
| Inspect remote image (no download) | `skopeo inspect docker://<registry>/<image>:<tag>` |
| View image metadata | Look for `Labels`, `Architecture`, `Os`, `Layers` in output |

---

## 🔧 Steps

### Step 1 — Switch to `conadm`

```bash
su - conadm
```

**Expected output:**

```
Password:
Last login: Wed May 20 15:57:42 EDT 2026 on pts/1
[conadm@ip-172-31-10-183 ~]$
```

---

### Step 2 — Verify skopeo is installed

```bash
skopeo --version
```

**Actual output:**

```
skopeo version 1.22.2 commit: 4db1c56e8003ad07759a4d899254643d445dbe04
```

**Output explained:**

| Field | Meaning |
|---|---|
| `skopeo version 1.22.2` | skopeo is installed and ready |
| `commit: 4db1c56e...` | Exact build commit — useful for bug reports |

---

### Step 3 — Inspect ubi9 remotely

```bash
skopeo inspect docker://registry.access.redhat.com/ubi9/ubi:latest
```

> ⚠️ **Common mistake:** Do not split the URL. It must be `registry.access.redhat.com` — not `registry.access/redhat.com`.

**Actual output (key section):**

```json
{
    "Created": "2026-04-29T11:00:15.879011974Z",
    "Labels": {
        "architecture": "x86_64",
        "build-date": "2026-04-29T10:59:59Z",
        "com.redhat.component": "ubi9-container",
        "description": "The Universal Base Image is designed and engineered to be the base layer for all of your containerized applications...",
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

**Output explained:**

| Field | Value | Meaning |
|---|---|---|
| `"version": "9.8"` | 9.8 | Current ubi9 release |
| `"Architecture": "amd64"` | amd64 | Built for x86_64 systems |
| `"Os": "linux"` | linux | Target operating system |
| `"Created"` | 2026-04-29 | When this image was built by Red Hat |
| `"Layers"` | 1 layer | Minimal base image — only 1 filesystem layer |
| `"Size": 80492959` | ~80MB | Compressed download size |
| `"name": "ubi9/ubi"` | ubi9/ubi | Confirms you inspected the correct image |
| `sha256-...` entries | hundreds | Every version/signature ever published — normal, ignore them |
| `"Env"` | PATH + container=oci | Default environment variables baked into the image |

> 📌 The long list of `sha256` hashes in the full output = all available tags and signatures in the registry. This is expected — not an error.

---

## ✅ Lab Checklist

- [ ] `skopeo --version` returns a version number
- [ ] `skopeo inspect` runs without error
- [ ] Output shows `"version": "9.8"` and `"Architecture": "amd64"`
- [ ] Only 1 layer confirmed (minimal base image)
- [ ] No image was downloaded to disk

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---|---|
| Split URL (`registry.access/redhat.com`) | Must be `registry.access.redhat.com` |
| `skopeo: command not found` | Install with `sudo dnf install skopeo -y` |
| `FATA: Error pinging container registry` | Check network/DNS — registry unreachable |
| Confusing inspect with pull | `skopeo inspect` = read only; `podman pull` = download |

---

## 📌 Exam Tips

- `skopeo inspect` is used to **read image metadata without pulling** — important on the exam when you need to verify an image before downloading.
- Always use the full registry path: `registry.access.redhat.com/ubi9/ubi:latest`
- `docker://` prefix tells skopeo to use the Docker registry protocol — required even for Red Hat registries.

---

## ➡️ Next Lab

**[Lab 22-1e: Pull ubi9 image with podman](./lab-22-1e-podman-pull.md)**

---

## 🔗 Series Index

- ✅ [Lab 22-1a: Create conadm user](./lab-22-1a-create-conadm.md)
- ✅ [Lab 22-1b: Grant sudo rights](./lab-22-1b-sudo-conadm.md)
- ✅ [Lab 22-1c: Verify sudo access](./lab-22-1c-verify-sudo.md)
- 👉 **Lab 22-1d: Inspect ubi9 with skopeo** ← you are here
- [Lab 22-1e: Pull ubi9 image](./lab-22-1e-podman-pull.md)
- [Lab 22-1f: Launch container with port mapping](./lab-22-1f-launch-container.md)
- [Lab 22-1g: Run commands inside container](./lab-22-1g-container-commands.md)
- [Lab 22-1h: Verify port mapping from host](./lab-22-1h-verify-port.md)

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
