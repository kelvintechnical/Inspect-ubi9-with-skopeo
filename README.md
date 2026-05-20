# Lab 22-1e: Pull ubi9 Image with podman

**RHCSA EX200 Lab** | Series: Container Management (Lab 22)  
**Prerequisite:** Lab 22-1d complete (ubi9 inspected with skopeo)  
**Time Estimate:** 5 minutes

---

## 🎯 Objective

Download the ubi9 image from the Red Hat registry to your local machine using `podman pull`, then verify it is stored locally.

---

## 📚 Command Decision Map

| Task | Command |
|---|---|
| Check podman is installed | `podman --version` |
| Pull an image | `sudo podman pull <registry>/<image>:<tag>` |
| List local images | `sudo podman images` |
| Remove an image | `sudo podman rmi <image>` |

---

## 🔧 Steps

### Step 1 — Verify podman is installed

```bash
podman --version
```

**Actual output:**

```
podman version 5.6.0
```

**Output explained:**

| Field | Meaning |
|---|---|
| `podman version 5.6.0` | podman is installed and ready to use |

---

### Step 2 — Pull ubi9 image

```bash
sudo podman pull registry.access.redhat.com/ubi9/ubi:latest
```

**Actual output:**

```
Trying to pull registry.access.redhat.com/ubi9/ubi:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob c9f6ad4a5dfb done   |
Copying config 829e2dc9f6 done   |
Writing manifest to image destination
Storing signatures
829e2dc9f679020f0c3a2740b743c174de84b3de729ad6b00de19d95c8652e2c
```

**Output explained:**

| Line | Meaning |
|---|---|
| `Trying to pull...` | Connecting to the Red Hat registry |
| `Getting image source signatures` | Fetching cryptographic signatures to verify authenticity |
| `Checking if image destination supports signatures` | Verifying local storage can handle signed images |
| `Copying blob c9f6ad4a5dfb done` | Downloading the single filesystem layer (~80MB compressed) |
| `Copying config 829e2dc9f6 done` | Downloading image configuration/metadata |
| `Writing manifest to image destination` | Saving the image index (manifest) locally |
| `Storing signatures` | Saving cryptographic signatures for verification |
| `829e2dc9f679...` | The image's unique SHA256 ID — your local copy fingerprint |

> 📌 The SHA256 at the end matches the `IMAGE ID` you'll see in `podman images` — confirms integrity.

---

### Step 3 — Verify image is stored locally

```bash
sudo podman images
```

**Actual output:**

```
REPOSITORY                           TAG         IMAGE ID      CREATED      SIZE
registry.access.redhat.com/ubi9/ubi  latest      829e2dc9f679  3 weeks ago  220 MB
```

**Output explained:**

| Field | Value | Meaning |
|---|---|---|
| `REPOSITORY` | `registry.access.redhat.com/ubi9/ubi` | Full registry path of the image |
| `TAG` | `latest` | Most current published version |
| `IMAGE ID` | `829e2dc9f679` | Matches the SHA256 from the pull — confirms correct image ✅ |
| `CREATED` | 3 weeks ago | Image build date (April 29, 2026) |
| `SIZE` | 220 MB | Uncompressed size on disk (was ~80MB compressed during pull) |

---

## ✅ Lab Checklist

- [ ] `podman --version` returns a version number
- [ ] `sudo podman pull` completes without error
- [ ] SHA256 at end of pull matches `IMAGE ID` in `podman images`
- [ ] `sudo podman images` shows ubi9 with `latest` tag
- [ ] Size shows ~220 MB on disk

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---|---|
| `Error: image not known` | Check registry URL spelling |
| Pull hangs or times out | Check network connectivity |
| `permission denied` | Use `sudo podman pull` for rootful operations |
| Image shows wrong size | Normal — compressed (~80MB) vs uncompressed (~220MB) |
| Forgot `sudo` | Rootful containers require sudo |

---

## 📌 Exam Tips

- `skopeo inspect` = read metadata only (no download)
- `podman pull` = actually download the image
- Always verify with `podman images` after pulling — don't assume success.
- `IMAGE ID` is a shortened version of the full SHA256 — both refer to the same image.

---

## ➡️ Next Lab

**[Lab 22-1f: Launch container with port mapping](./lab-22-1f-launch-container.md)**

---

## 🔗 Series Index

- ✅ [Lab 22-1a: Create conadm user](./lab-22-1a-create-conadm.md)
- ✅ [Lab 22-1b: Grant sudo rights](./lab-22-1b-sudo-conadm.md)
- ✅ [Lab 22-1c: Verify sudo access](./lab-22-1c-verify-sudo.md)
- ✅ [Lab 22-1d: Inspect ubi9 with skopeo](./lab-22-1d-skopeo-inspect.md)
- 👉 **Lab 22-1e: Pull ubi9 image** ← you are here
- [Lab 22-1f: Launch container with port mapping](./lab-22-1f-launch-container.md)
- [Lab 22-1g: Run commands inside container](./lab-22-1g-container-commands.md)
- [Lab 22-1h: Verify port mapping from host](./lab-22-1h-verify-port.md)

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
