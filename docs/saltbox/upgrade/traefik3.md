# Traefik 3.0 Upgrade

Saltbox has undergone some major breaking changes which land with the release and integration of Traefik 3.0.

!!! tip "Important!"
    Due to the amount of changes related to Traefik we recommend that all containers are redeployed by running their respective tags.

These changes include:

1. Upgrade Traefik to version 3.0
    1. HTTP validation for certificates is no longer enabled by default.
        1. Enable it in adv_settings.yml if you need it while using Cloudflare with Saltbox.

2. Remote mount changes - **Breaking Changes**
    1. Add support for specifying multiple remote mounts using different predefined templates (Google, Dropbox, SFTP).
    2. Moved remote mounts from `/mnt/remote` to `/mnt/remote/<remote_name>`
    3. Changed behavior to overwrite mount service files by default.
        1. The previous default was to not touch existing services. This was a necessary change to allow the addition and removal of extra mounts.
        2. As a result the old rclone_vfs.service will get removed so preserve a copy if you want to keep any tweaks you made to it.

    ??? note "What Does this mean for me?"
    
        If you have custom mount services and mergerfs changes to support your multiple remotes [maybe you have google and dropbox both configured, for example] saltbox will now manage that for you.

        IF you set up Dropbox or Box or some other non-Google cloud storage using guides from this wiki or the Discord, you have custom mount services.

        You will define all your mounts in the settings.yml file, and Saltbox will create and manage all the mount services and the mergerfs config.  The cloudplow role will look at the same information to build its config file.

        If you have custom mount services:

        1. stop all containers
        2. stop and disable those mount services; MAKE COPIES IF YOU WANT TO SAVE THEM
        3. remove any mergerfs- or mount-related changes you made to the [inventory](../inventory/index.md)
        4. define your rclone remotes in `settings.yml` as described on the install page or the config file page.
        5. run `sb install mounts` to build the new service files and start the mounts.

4. Database role changes - **Breaking Changes**
    1. Added multi-instance support to database roles.
    2. Moved roles requiring databases to provision a unique database instance for each app instance.
        1. This is still being worked on and most of this was moved to a separate branch for now.

5. Authelia changes
    1. Added greater configurability to Authelia using the [inventory](../inventory/index.md).
    2. Added LDAP backend to Authelia as an option.

6. Add support to restoring the appdata of a single app from backup
    1. `sb install restore -e restore_tar=plex.tar` but it assumes you are past any steps restore would require.

7. Changed default torrent client to qBittorrent

8. Changed default usenet client to SABnzbd

    ??? note "What if I want to keep using nzbget and/or rutorrent?"
    
        You can override this with a setting in the [inventory](../inventory/index.md):

        ```
        download_clients_enabled: ["qbittorrent", "sabnzbd"]
        ```

10. Add new custom container (ddns role) for keeping a dynamic IP on Cloudflare in sync with all containers using Traefik (not just Saltbox installed ones).

11. Changed the rutorrent image since the previously used one was no longer getting updates.
    1. No longer includes autodl

12. Docker volumes such as /data, /tv and /movies are no longer mounted into relevant containers by default.
    1. Restore the old behavior by setting `docker_legacy_volume: true` using the [inventory](../inventory/index.md), then running the relevant tags [**typically** `plex, radarr, sonarr` but your setup may differ].

As with any major update double check your [inventory](../inventory/index.md) edits are in line with any changes made to the roles. Ask on our discord server if in doubt.
