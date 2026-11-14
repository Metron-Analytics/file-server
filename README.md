# File server

> **Coming soon.** The server image isn't publicly available yet, so the install steps below don't work for everyone yet; the licence is being finalised. Questions and ideas are welcome in [Discussions](../../discussions), and app bugs in [Issues](../../issues/new/choose). To hear when it's ready, choose Watch > Custom > Releases at the top of this page.

The server for **File**, an iPhone app for a private document archive: scan paper, find any document by the text on its pages, and keep notes and reminders with your documents, on your own server instead of someone else's cloud.

- **The app:** [File on the App Store](https://apps.apple.com/app/id6826832579) (free, iPhone with iOS 26 or later)
- **Website and user guide:** [file.metron-apps.eu](https://file.metron-apps.eu)
- **This repository:** the releases, the setup files, and the place for bug reports, ideas and questions. The source code is not here.

The server runs in Docker on a NAS, a small server or a Raspberry Pi (64-bit Intel/AMD or ARM). It keeps your PDFs in an ordinary folder tree, and the details, notes and reminders in its own database. The app does nothing without a server.

## Install

You need Docker with Docker Compose, 2 GB of memory, disk space for your PDFs and the database, and for access from outside your home: your own domain name with ports 80 and 443 open to the server.

**1. Choose how you reach it**

| Profile | For |
|---|---|
| `lan` | Only on your home network, over plain HTTP on port 8080. Never forward this port on your router. |
| `public` | From anywhere, over HTTPS on your own domain, with a free certificate that renews by itself. |

**2. Back up your documents.** If you already have a folder of PDFs, make a backup or snapshot first. The first import renames files whose names don't fit the title rules.

**3. Get the setup files** from the [latest release](../../releases/latest): `compose.yaml`, `Caddyfile`, `Caddyfile.lan` and `env.example`. Put them in a `deploy` folder next to the data folders:

```bash
mkdir -p data/pgdata data/inbox deploy
cd deploy
cp env.example .env
```

**4. Fill in the settings.** Open `.env` and fill in at least these, with full paths:

```bash
ARCHIVE_PATH=/volume1/archive      # your PDFs
DATA_PATH=/volume1/docker/archive  # database and uploads
PUID=1026                          # owner of the archive folder: id <user>
PGID=100
POSTGRES_PASSWORD=                 # a long random password
JWT_SECRET=                        # openssl rand -base64 48
ADMIN_USERNAME=                    # your first account
ADMIN_PASSWORD=                    # at least 10 characters
COMPOSE_PROFILES=lan               # or public
IMAGE=ghcr.io/metron-analytics/file-server
IMAGE_TAG=                         # the release, for example 1.0.8
```

For the `public` profile, also fill in `PUBLIC_DOMAIN` and `ACME_EMAIL`, and point your domain's A record to the server.

**5. Start it.** On a Synology NAS, put `sudo` before each command.

```bash
docker compose pull
docker compose up -d --no-build
docker compose logs -f worker
```

The worker reads your PDFs. Files with text go quickly; scanned pages without text take longer, because the server recognises the text. Press Ctrl+C to stop watching.

**6. Check it.** `http://<server address>:8080/api/health` (or `https://<your domain>/api/health`) answers `{"ok":true}` and a version.

**7. Sign in** with the app, using the address and the admin account from `.env`. Then remove `ADMIN_PASSWORD` from `.env` and run `docker compose up -d --no-build` again. Create accounts for other people on the website, under Admin. The [user guide](https://file.metron-apps.eu/guide/) explains the rest.

## Updating

Read the release notes first, then set the new release in `IMAGE_TAG` and run:

```bash
docker compose pull
docker compose up -d --no-build
```

The database updates itself when the server starts. Going back to an older release isn't always possible, so make a backup first. Releases are listed under [Releases](../../releases); to hear about new ones, choose Watch > Custom > Releases at the top of this page.

## Backups

Back up the archive folder and the `data` folder. The PDFs are in the archive folder; notes, reminders, tags and accounts are only in the database.

## Push notifications

Push notifications need a key from Apple that belongs to the app, so they don't work on your own server yet. The app sets its own reminders on the iPhone instead.

## Help, bugs and ideas

- **Questions and ideas:** [Discussions](../../discussions)
- **Bugs:** [Issues](../../issues/new/choose)
- **Security problems:** not in public; see [SECURITY.md](SECURITY.md)
- **Your account or documents:** ask the administrator of your server. We can't see or reach anything on it.

Everything here is public. **Never post documents, screenshots of real documents, passwords, server addresses or anything else personal.** Use invented examples.

## Licence

The server software is free for personal use: for yourself, your family or your household, also for the paperwork of your own job or one-person business. It will be licensed under the [PolyForm Noncommercial License 1.0.0](https://polyformproject.org/licenses/noncommercial/1.0.0/), with an extra permission for individuals and households. Selling it, offering it as a hosted service, or using it for a company where several people work with it needs a written agreement: archive@metron-apps.eu.

The iPhone app is free on the App Store, under Apple's standard terms.

Metron-Apps.eu, part of Metron Labs.
