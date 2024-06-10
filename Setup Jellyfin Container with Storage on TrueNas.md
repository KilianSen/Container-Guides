This guide demonstrates how to run a Jellyfin instance on a Debian machine inside a Docker container, with storage located on a different machine using a TrueNAS CIFS share. Use this as a reference and adjust it to suit your needs.

# Setup

### TrueNAS
1. **Create a new local user for Jellyfin:**
    - Activate Samba authentication.

2. **Create a new dataset for your Jellyfin data.**

3. **Create folders for media, config, and cache in the Jellyfin dataset root:**
    - Use Windows Explorer to mount the share on your Linux machine temporarily.

4. **Create an SMB/CIFS share pointing to the Jellyfin dataset.**

5. **Set ACL rules:**
    - Ensure the Jellyfin user can read, execute, and write within the entire dataset.

### Jellyfin Machine

Ensure Docker is installed, and install `cifs-utils`:

```bash
sudo apt install cifs-utils
```

Create a Docker Compose file `compose.yml` in its own directory, e.g., `jellyfin-truenas`:

```yaml
services:
    jellyfin:
        image: jellyfin/jellyfin:latest
        container_name: jellyfin
        restart: unless-stopped
        volumes:
            - config:/config
            - cache:/cache
            - media:/media
        network_mode: host

volumes:
    cache:
        driver: local
        driver_opts:
            type: cifs
            o: username={TRUENAS_JELLYFIN_USERNAME},password={TRUENAS_JELLYFIN_PASSWORD},uid={TRUENAS_JELLYFIN_UID},gid={TRUENAS_JELLYFIN_GID},vers=3.0,nobrl
            device: //{DEVICE_ADDRESS}/{SHARE_NAME}/cache
    config:
        driver: local
        driver_opts:
            type: cifs
            o: username={TRUENAS_JELLYFIN_USERNAME},password={TRUENAS_JELLYFIN_PASSWORD},uid={TRUENAS_JELLYFIN_UID},gid={TRUENAS_JELLYFIN_GID},vers=3.0,nobrl
            device: //{DEVICE_ADDRESS}/{SHARE_NAME}/config
    media:
        driver: local
        driver_opts:
            type: cifs
            o: username={TRUENAS_JELLYFIN_USERNAME},password={TRUENAS_JELLYFIN_PASSWORD},uid={TRUENAS_JELLYFIN_UID},gid={TRUENAS_JELLYFIN_GID},vers=3.0,nobrl
            device: //{DEVICE_ADDRESS}/{SHARE_NAME}/media
```

This setup is based on the [Jellyfin Docker installation command](https://jellyfin.org/downloads/docker), with local volumes replaced by CIFS volumes on TrueNAS.

- **`TRUENAS_JELLYFIN_UID` and `TRUENAS_JELLYFIN_GID`** can be found on the TrueNAS Local Users page.
- The **`nobrl`** option is crucial as it disables locking, detailed in [this Jellyfin storage guide](https://jellyfin.org/docs/general/administration/storage/#nfs).

I hope this guide helps you configure your setup.
