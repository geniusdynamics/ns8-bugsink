# Bugsink for NethServer 8 (NS8)

Self-hosted error tracking for your applications, packaged as a NethServer 8 module.

- Upstream app: https://github.com/bugsink/bugsink
- NS8 project: https://github.com/NethServer/ns8-core
- Container image: ghcr.io/geniusdynamics/bugsink:latest
- Category: collaboration

Contents
- Features
- Requirements
- Quick start
- Configure
- Manage (status, update, uninstall)
- Networking and TLS
- Smarthost autodiscovery
- Debug and troubleshooting
- Testing
- Development (UI and images)
- Internationalization
- License

Features
- One-command install on NS8
- Integrated reverse-proxy routing via Traefik
- Optional automatic TLS with Let’s Encrypt
- MariaDB/MySQL backend pre-wired
- Minimal, focused UI bundled in the module

Requirements
- A running NethServer 8 cluster (node with Traefik)
- A DNS-resolvable FQDN for the Bugsink web UI
- Internet connectivity to pull container images

Quick start
1) Install the module

```bash
add-module ghcr.io/geniusdynamics/bugsink:latest 1
```

The command returns the instance name, e.g. {"module_id": "bugsink1", ...}.

2) Configure the instance (replace bugsink1 with your instance)

```bash
api-cli run configure-module --agent module/bugsink1 --data - <<EOF
{
  "host": "bugsink.example.com",
  "http2https": true,
  "lets_encrypt": true
}
EOF
```

This will start the instance and create the Traefik route for the given host.

Open https://bugsink.example.com to access the app.

Configure
Parameters accepted by configure-module:
- host: Fully Qualified Domain Name for the public endpoint
- http2https: Force HTTP to HTTPS redirection (true/false)
- lets_encrypt: Request a Let’s Encrypt certificate (true/false)

Read back the configuration

```bash
api-cli run get-configuration --agent module/bugsink1
```

Manage
Status (from the UI, navigate to the module card and open Status)
- Shows services, images, volumes and instance metadata.

Update the module to a new image

```bash
api-cli run update-module --data '{
  "module_url": "ghcr.io/geniusdynamics/bugsink:latest",
  "instances": ["bugsink1"],
  "force": true
}'
```

Uninstall

```bash
remove-module --no-preserve bugsink1
```

Networking and TLS
- TCP ports: The module reserves one TCP port and is published through Traefik using the host value you set at configuration time.
- TLS: When lets_encrypt=true, a certificate is automatically requested and renewed. If false, you can manage certificates externally and point your DNS accordingly.

Smarthost autodiscovery
Some mail-related settings (like smarthost) are discovered dynamically from the NS8 cluster and are not part of the configure-module payload.
- On each start, bin/discover-smarthost refreshes state/smarthost.env from Redis
- If the centralized smarthost configuration changes while the module is running, the handler events/smarthost-changed/10reload_services restarts the main service
See also systemd/user/bugsink.service.

Debug and troubleshooting
Run as the module agent to get the correct environment (state is under /home/<instance>/.config/state):

- Print environment

```bash
runagent -m bugsink1 env
```

- Enter an agent shell and verify PATH

```bash
runagent -m bugsink1
```

```bash
echo $PATH
```

Inspect containers

- List containers

```bash
podman ps --format "table {{.ID}}\t{{.Image}}\t{{.Names}}\t{{.Ports}}\t{{.Status}}"
```

- Inspect environment of the app container

```bash
podman exec bugsink-app env
```

- Open a shell inside the app container

```bash
podman exec -ti bugsink-app sh
```

Expected database-related environment inside the app container (values can vary):
- MARIADB_DB_HOST=127.0.0.1
- MARIADB_DB_PORT=3306
- MARIADB_DB_NAME=bugsink
- MARIADB_DB_USER=bugsink
- MARIADB_DB_PASSWORD=bugsink

Testing
Robot Framework tests are provided. You can use the helper script:

```bash
./test-module.sh <NODE_ADDR> ghcr.io/geniusdynamics/bugsink:latest
```

Development
Container images
- The build script builds the UI and assembles the module image:

```bash
./build-images.sh
```

Notes:
- UI is built with Node LTS (yarn build) and copied into the image
- The image carries labels for NS8 (authorizations, tcp port demand, rootless)
- External images used by the module include:
  - docker.io/mysql:9.4.0
  - docker.io/bugsink/bugsink:1.7.6
- Images are committed as ghcr.io/geniusdynamics/bugsink:latest by default

UI development
See ui/README.md for the NS8 UI development workflow and refer to the NS8 Developer manual: https://nethserver.github.io/ns8-core/ui/modules/#module-ui-development

Module metadata
See ui/public/metadata.json for name, description, author and links displayed in the NS8 UI.

Internationalization
UI strings are translated with Weblate: https://hosted.weblate.org/projects/ns8/
To set up translation:
- Add the GitHub Weblate app to your repository
- Add your repository to hosted.weblate.org (or ask a NethServer developer to include it in the NS8 Weblate project)

License
This project is released under the GPL-3.0-or-later license. See LICENSE for details.
