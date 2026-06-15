# VideoWare for Unraid

Unraid Community Applications template for **VideoWare** — a self-hosted media upload and
processing platform. Upload media, get fast previews (thumbnails & sprites), edit and
render clips on a timeline, and run AI video analysis — all from one container.

> **One container, one port, one data path.** This template runs the *monolithic*
> VideoWare image, which bundles the web app, database, background worker, and its job
> broker together. There are no sidecar containers to set up and no external Redis or
> database to manage.

---

## ✨ Features

- 📤 **Upload media** with progress tracking and validation
- ⚡ **Fast previews** — thumbnails and hover sprites generated in the background
- ✂️ **Clip editing & timeline rendering** — compose and export final videos
- 🤖 **AI video analysis** (optional, via Google Cloud) — object/face/person detection,
  object tracking, shot-change detection, and speech transcription
- 🎬 **FFmpeg-powered** transcoding and rendering, built in
- 💾 **Flexible storage** — local appdata by default, or any S3-compatible bucket
- 🔐 **Built-in auth & API** via PocketBase, with a realtime database and admin UI

## 📦 What's inside the container

The monolithic image is fully self-contained. A single container runs:

| Service        | Role                                                          |
| -------------- | ------------------------------------------------------------- |
| **Next.js**    | The web application (upload, browse, edit, render)            |
| **PocketBase** | Database, auth, realtime updates, and REST API                |
| **Worker**     | NestJS + FFmpeg background processor (transcode, sprites, render, labels) |
| **Redis**      | Internal BullMQ job broker — `localhost` only, never exposed  |
| **Nginx**      | Reverse proxy fronting everything on a single port            |

You only ever map **one port** and **one volume**.

---

## 🚀 Installation (Community Applications)

1. In Unraid, open the **Apps** tab (Community Applications).
2. Search for **VideoWare** and click **Install**.
3. Set a strong **Admin Password** (and optionally change the **Admin Email**). The
   PocketBase superuser is created automatically on first boot from these values —
   leaving the password blank/default skips superuser creation.
4. Adjust the **WebUI Port** and **App Data** path if you like (defaults are fine for
   most setups).
5. Click **Apply** and wait for the container to start.

When it's up, click the container's **WebUI** button, or browse to:

- **Web app:** `http://<UNRAID-IP>:<PORT>/`
- **PocketBase admin:** `http://<UNRAID-IP>:<PORT>/_/`
- **PocketBase API:** `http://<UNRAID-IP>:<PORT>/api/`

> First boot takes a few moments while the database initializes. If the web app shows a
> connection error, give it ~30 seconds and refresh.

### Manual template install (without CA)

If you aren't using Community Applications, you can add the template directly. In the
Unraid **Docker** tab → **Add Container** → **Template** field, paste:

```
https://raw.githubusercontent.com/make-ware/video-ware-ca/main/templates/video-ware.xml
```

---

## ⚙️ Configuration

These map to the fields shown in the Unraid template:

| Setting            | Variable / Target          | Default                          | Notes                                                                 |
| ------------------ | -------------------------- | -------------------------------- | --------------------------------------------------------------------- |
| **WebUI Port**     | container port `80`        | `8888`                           | Host port for the web app, API, and admin (all proxied by nginx).     |
| **App Data**       | `/data`                    | `/mnt/user/appdata/video-ware`   | Persistent storage — database, uploaded media, and the Redis queue.   |
| **Admin Email**    | `POCKETBASE_ADMIN_EMAIL`   | `admin@example.com`              | Email for the auto-created PocketBase superuser.                      |
| **Admin Password** | `POCKETBASE_ADMIN_PASSWORD`| _(required)_                     | **Must be set.** Password for the superuser.                          |
| **Storage Type**   | `STORAGE_TYPE`             | `local`                          | `local` (files under `/data`) or `s3`. *(advanced)*                   |
| **Log Level**      | `LOG_LEVEL`                | `warn`                           | `error`, `warn`, `info`, `debug`, or `verbose`. *(advanced)*          |

### Optional: S3-compatible storage

Set **Storage Type** to `s3` and add these variables (Add another Path, Port, Variable…):

| Variable                      | Required | Notes                                                          |
| ----------------------------- | -------- | -------------------------------------------------------------- |
| `STORAGE_S3_BUCKET`           | ✅       | Bucket name                                                    |
| `STORAGE_S3_REGION`           | ✅       | e.g. `us-east-1` (use `garage` for Garage)                     |
| `STORAGE_S3_ACCESS_KEY_ID`    | ✅       | Access key ID                                                  |
| `STORAGE_S3_SECRET_ACCESS_KEY`| ✅       | Secret access key                                              |
| `STORAGE_S3_ENDPOINT`         |          | Defaults to `https://s3.<region>.amazonaws.com`                |
| `STORAGE_S3_FORCE_PATH_STYLE` |          | `true` for MinIO/Garage-style endpoints                        |

### Optional: AI video analysis (Google Cloud)

Add `GOOGLE_PROJECT_ID`, `GOOGLE_CLOUD_CREDENTIALS` (inline service-account JSON), and
`GCS_BUCKET` to enable Video Intelligence features. See the project README for details.

---

## 💾 Data, backups & updates

All persistent state lives under the single **App Data** mount (`/data`):

```
/mnt/user/appdata/video-ware/
├── pb_data/    # PocketBase database
├── storage/    # Uploaded media and derivatives (when STORAGE_TYPE=local)
└── redis/      # BullMQ job queue (AOF persistence)
```

- **Back up:** include `/mnt/user/appdata/video-ware` in your appdata backup (e.g. the
  *CA Appdata Backup/Restore* plugin). That single folder is the whole app.
- **Update:** when a new image is published, Unraid's Docker tab shows an update — click
  **Apply Update**. Your data persists across updates.

---

## 🆘 Support & links

- **Issues / support:** https://github.com/make-ware/video-ware/issues
- **Application source:** https://github.com/make-ware/video-ware
- **Container image:** `ghcr.io/make-ware/video-ware:latest`

## 📁 This repository

This repo holds only the Unraid Community Applications metadata:

```
video-ware-ca/
├── ca_profile.xml            # CA repository profile
├── icon.png                  # Repository / app icon
├── README.md                 # This file
└── templates/
    └── video-ware.xml        # Docker app template (monolithic image)
```

The application itself is built and published from
[make-ware/video-ware](https://github.com/make-ware/video-ware).
