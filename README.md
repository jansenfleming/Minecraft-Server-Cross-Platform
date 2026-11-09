# Minecraft Server Setup (PaperMC + Java/Bedrock Crossplay)

A step-by-step guide to running your own Minecraft Java server with **PaperMC**, plus **Geyser**, **Floodgate**, and **ViaVersion** so Bedrock players (phone, console, Windows Bedrock) and players on other Java versions can join.

This guide covers two options:

1. **Self-hosting** on your own PC or a spare machine (Linux, tested on Ubuntu Server)
2. **Using a hosting provider** (rented server or tunneling service)

---

## Table of Contents

- [Requirements](#requirements)
- [Part 1: Install Java](#part-1-install-java)
- [Part 2: Set Up the PaperMC Server](#part-2-set-up-the-papermc-server)
- [Part 3: Keep the Server Running with screen](#part-3-keep-the-server-running-with-screen)
- [Part 4: Add Crossplay Plugins (Geyser, Floodgate, ViaVersion)](#part-4-add-crossplay-plugins-geyser-floodgate-viaversion)
- [Part 5: Let Players Connect](#part-5-let-players-connect)
- [Option B: Using a Hosting Provider](#option-b-using-a-hosting-provider)
- [Updating the Server](#updating-the-server)
- [Backups](#backups)
- [Troubleshooting](#troubleshooting)
- [Useful Links](#useful-links)

---

## Requirements

| Item | Recommendation |
|------|----------------|
| OS | Linux (Ubuntu Server works great); Windows/macOS also work with small changes |
| RAM | 4 GB minimum for the server itself, more for bigger worlds or many players |
| Storage | 10 GB+ on an SSD |
| Java | **Java 25 or newer** (required for Minecraft 26.1+) |
| Ports | `25565/TCP` (Java), `19132/UDP` (Bedrock via Geyser) |

> **Note:** Minecraft switched to a new version format in 2026 (e.g. `26.1.2` instead of `1.21.x`). Starting with 26.1, the server **will not start on Java 21**. You need Java 25+.

---

## Part 1: Install Java

Check what you have:

```bash
java -version
```

If it's below 25, install Java 25. On Ubuntu/Debian:

```bash
sudo apt update
sudo apt install openjdk-25-jre-headless
```

If your distro doesn't have that package yet, follow PaperMC's Java install guide (uses Eclipse Temurin / Adoptium):
https://docs.papermc.io/misc/java-install

Then confirm:

```bash
java -version
```

---

## Part 2: Set Up the PaperMC Server

### 1. Create a folder for the server

```bash
mkdir papermc-server
cd papermc-server
```

### 2. Download PaperMC

Go to https://papermc.io/downloads/paper, right-click the latest download button, and copy the link. Then:

```bash
wget <paste-the-paper-download-url-here>
```

Example (this was the latest when the guide was written; get the current one from the site):

```bash
wget https://fill-data.papermc.io/v1/objects/6c59eefe2752f97ee79f83ad0a61fe14865a0976e4ac06597f53de6c44afd6c5/paper-26.1.2-70.jar
```

### 3. Create a startup script

```bash
nano start.sh
```

Paste this in, and **change the jar name** to match the file you downloaded. Set `-Xms` / `-Xmx` to how much RAM you want to give the server (here, 4 GB):

```bash
#!/bin/bash
java -Xms4G -Xmx4G \
  -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 \
  -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch \
  -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M \
  -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 \
  -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 \
  -XX:G1RSetUpdatingPauseTimeLimit=0 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem \
  -XX:MaxTenuringThreshold=1 \
  -jar paper-26.1.2-70.jar --nogui
```

> Tip: PaperMC has a start script generator that builds this for you: https://docs.papermc.io/misc/tools/start-script-gen

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`), then make it executable:

```bash
chmod +x start.sh
```

### 4. First start (this will fail — that's expected)

```bash
./start.sh
```

The server will stop right away because you haven't accepted the Minecraft EULA yet.

### 5. Accept the EULA

```bash
nano eula.txt
```

Change:

```
eula=false
```

to:

```
eula=true
```

Save and exit.

---

## Part 3: Keep the Server Running with screen

If you start the server in a normal SSH session, it dies when you disconnect. `screen` keeps it running in the background.

Install it if needed:

```bash
sudo apt install screen
```

Create a named session (name it whatever you want):

```bash
screen -S minecraft
```

Inside the screen session, start the server:

```bash
./start.sh
```

Useful screen commands:

| Action | Command |
|--------|---------|
| Detach (leave it running) | `Ctrl+A`, then `D` |
| List sessions | `screen -ls` |
| Reattach | `screen -r minecraft` |

To stop the server cleanly, reattach and type `stop` in the server console.

---

## Part 4: Add Crossplay Plugins (Geyser, Floodgate, ViaVersion)

| Plugin | What it does |
|--------|--------------|
| **Geyser-Spigot** | Lets Bedrock Edition players join your Java server |
| **Floodgate** | Lets Bedrock players join **without** owning a Java account |
| **ViaVersion** | Lets Java players on newer client versions join |

### 1. Stop the server and download the plugins

```bash
cd plugins

# Geyser (Spigot/Paper version)
wget -O Geyser-Spigot.jar https://download.geysermc.org/v2/projects/geyser/versions/latest/builds/latest/downloads/spigot

# Floodgate (Spigot/Paper version)
wget -O floodgate-spigot.jar https://download.geysermc.org/v2/projects/floodgate/versions/latest/builds/latest/downloads/spigot

cd ..
```

For **ViaVersion**, grab the latest jar link from Hangar or GitHub releases and `wget` it into the `plugins` folder:

- https://hangar.papermc.io/ViaVersion/ViaVersion
- https://github.com/ViaVersion/ViaVersion/releases

```bash
cd plugins
wget <paste-the-viaversion-jar-url-here>
cd ..
```

### 2. Start, then stop the server

This generates the config folders for each plugin.

```bash
./start.sh
# wait for it to fully load, then type:
stop
```

### 3. Check the Geyser config

Open `plugins/Geyser-Spigot/config.yml` and make sure:

- `auth-type` is `floodgate` (it's usually detected automatically when Floodgate is installed)
- The Bedrock port is `19132` (the default)

### 4. Floodgate key (only if needed)

When Geyser and Floodgate are on the **same** Paper server, Geyser normally picks up Floodgate's key automatically.

If Geyser logs an error about the Floodgate key, copy `key.pem` from the Floodgate folder into the Geyser folder:

```bash
cp plugins/floodgate/key.pem plugins/Geyser-Spigot/
```

> ⚠️ **Never share `key.pem` or commit it to GitHub.** Anyone with it can bypass authentication on your server. (See the `.gitignore` tip below.)

### 5. Start the server again

```bash
./start.sh
```

---

## Part 5: Let Players Connect

### Same network (LAN)

Find your machine's local IP:

```bash
hostname -I
```

- **Java players:** connect to `<local-ip>:25565`
- **Bedrock players:** add a server with `<local-ip>` and port `19132`

### Over the internet — Option 1: Port forwarding

In your router, forward these to your server's local IP:

| Port | Protocol | For |
|------|----------|-----|
| 25565 | TCP | Java |
| 19132 | UDP | Bedrock |

Also open them in the server's firewall:

```bash
sudo ufw allow 25565/tcp
sudo ufw allow 19132/udp
```

Players then connect with your **public IP** (or a domain/DDNS pointed at it).

### Over the internet — Option 2: Tunneling (no port forwarding)

If you can't (or don't want to) open ports on your router, use a tunneling service like **playit.gg**. It gives you a public address that forwards traffic to your server. Create one tunnel for Java (TCP 25565) and one for Bedrock (UDP 19132).

https://playit.gg

---

## Option B: Using a Hosting Provider

If you don't want to run hardware at home, there are two common routes:

### Managed Minecraft hosting

Many providers give you a web panel where you pick "Paper" as the server type, set the version, and upload plugins. With these:

1. Select **Paper** as the server jar and pick the version.
2. Make sure the panel's **Java version is set to 25** (some panels don't switch automatically).
3. Upload Geyser-Spigot, Floodgate, and ViaVersion to the `plugins` folder (via the panel's file manager or SFTP).
4. Start → stop → check `plugins/Geyser-Spigot/config.yml` as in Part 4.
5. Ask the host which port to use for Bedrock — many give you a single port, in which case set `clone-remote-port: true` in the Geyser config so Bedrock uses the same port as Java.

### VPS (virtual private server)

A VPS is just a Linux machine in the cloud, so you follow **Parts 1–5 exactly as written**, except:

- Connect via SSH instead of sitting at the machine
- Open ports in the VPS provider's firewall/security group, not your home router
- Players connect with the VPS's public IP

---

## Updating the Server

1. Stop the server and **make a backup** (see below).
2. Download the new Paper jar into the server folder.
3. Update the jar name in `start.sh`.
4. Re-download Geyser and Floodgate with the same `wget -O` commands from Part 4 (they always pull the latest build), and grab the latest ViaVersion.
5. Start the server and check the console for errors.

> PaperMC warns that you **can't safely downgrade** a world after upgrading to a new version, so always back up first.

---

## Backups

Simplest version — stop the server, then zip the whole folder:

```bash
cd ~
tar -czvf mc-backup-$(date +%F).tar.gz papermc-server/
```

Keep copies in at least two places (e.g. another drive and cloud storage).

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `requires running the server with Java 25 or above` | Install Java 25 (Part 1) and check `java -version` |
| Server stops immediately on first launch | Accept the EULA (`eula=true` in `eula.txt`) |
| `Permission denied` running `./start.sh` | Run `chmod +x start.sh` |
| Server dies when SSH disconnects | Run it inside `screen` (Part 3) |
| Bedrock players can't connect | Check UDP 19132 is forwarded/tunneled and open in the firewall (it's **UDP**, not TCP) |
| Bedrock players asked to log in to Java | Make sure Floodgate is installed and Geyser's `auth-type` is `floodgate` |
| Geyser key error | Copy `key.pem` from `plugins/floodgate/` to `plugins/Geyser-Spigot/` |

---

## Keep Secrets Out of This Repo

If you ever push your server folder to GitHub, add a `.gitignore` like this:

```gitignore
*.pem
world*/
logs/
cache/
*.jar
```

---

## Useful Links

- PaperMC downloads: https://papermc.io/downloads/paper
- PaperMC docs: https://docs.papermc.io
- Java install guide: https://docs.papermc.io/misc/java-install
- Geyser downloads: https://geysermc.org/download
- Floodgate setup: https://geysermc.org/wiki/floodgate/setup
- ViaVersion: https://hangar.papermc.io/ViaVersion/ViaVersion
- playit.gg: https://playit.gg
