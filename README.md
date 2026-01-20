Tdarr Podman Quadlet Deployment
===============================

**Disclaimer:** I am not affiliated with, associated, authorized, endorsed by, or in any way officially connected with the Tdarr project. This should be a drop in replacement but please back up all config files before making the switch.

This guide demonstrates how to deploy **Tdarr (Server + Internal Node)** as a System Podman Quadlet on Ubuntu 25.10. Tdarr is an automated library transcoder that benefits greatly from GPU acceleration via Intel QuickSync (QSV).

System Information
------------------

*   **Tested Environment:** Ubuntu 25.10
*   **Podman Version:** 5.4.x
*   **Configuration Type:** System-wide Quadlet (Root)

1\. The Quadlet File (tdarr.container)
--------------------------------------

Download or create the `tdarr.container` file. This configuration includes both the server and the processing node in a single container for easier management.

2\. Installation & Setup
------------------------

### Step A: Stop Existing Containers

Stop any Tdarr instances currently running in Docker or manual Podman.

### Step B: Hardware Permissions

Tdarr needs to access `/dev/dri` for transcoding. Ensure your user is in the correct groups (typically `video` and `render`). On Ubuntu, these are often GIDs 44 and 992, which are explicitly mapped in the Quadlet file's `PodmanArgs`.

`sudo usermod -aG video,render $USER`

### Step C: Identity Verification

Check your **UID** and **GID**. Update the `PUID` and `PGID` environment variables in the `.container` file if they differ from 1000.

`id`

### Step D: Move and Edit

Move the file to the systemd directory and update your media paths and timezone:

`sudo mkdir -p /etc/containers/systemd`
`sudo mv tdarr.container /etc/containers/systemd/`
`sudo nano /etc/containers/systemd/tdarr.container` to edit the container file.

**Pro Tip:** Ensure your `/temp` volume (the cache) is on a fast drive (SSD/NVMe) as Tdarr performs heavy I/O here during transcoding.

3\. Activation
--------------

Trigger the Quadlet generator and start the service:

`sudo systemctl daemon-reload`
`sudo systemctl start tdarr-server-node`

4\. Post-Install Verification
-----------------------------

### Check GPU Access

Verify that the Tdarr node can see the Intel render nodes:

`podman exec -it tdarr-server-node ls -l /dev/dri`

You should see `renderD128` and `card0`. If the ownership shows "nobody," check your `UserNS` and `group-add` settings.

### Verify FFmpeg QSV Support

You can check if FFmpeg inside the container recognizes Intel QSV:

`podman exec -it tdarr-server-node ffmpeg -encoders | grep qsv`

### Check Logs

`sudo journalctl -u tdarr-server-node -f`

5\. Updating
------------

Update Tdarr along with your other services:

`sudo podman auto-update`

_Note: This is a system quadlet; Tdarr will start automatically every time your Ubuntu system starts._
