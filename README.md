# media-stack
Production Docker media server | NVIDIA 4K transcoding | IT portfolio
# 🐳 Production Media Server Stack (4K NVIDIA Transcoding)
**Self-hosted streaming platform serving 3+ TB media across 5 devices via Tailscale VPN**

![Jellyfin Dashboard](screenshot-jellyfin.png) ![docker stats](screenshot-docker-stats.png)

## What It Does
- **Jellyfin**: 4K transcoding with NVIDIA GPU acceleration (handles 10+ streams)
- **Jellyseerr**: Request management interface (like Overseerr for Jellyfin)
- **Audiobookshelf**: Audiobook/podcast organizer 
- **Portainer**: Docker GUI management dashboard
- **3 media volumes** across 12TB storage with proper Linux permissions

## Tech Stack & Skills Demonstrated
```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    runtime: nvidia                    # ← GPU acceleration
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,utility,video,graphics
      - PUID=1000                       # ← Linux permissions mastery
      - PGID=1000
    ports: ["8096:8096"]
    volumes:
      - "/mnt/d/docker/jellyfin/config:/config"
      - "/mnt/d/docker/jellyfin/cache:/cache"
      - /mnt/e:/data/media1            # 3x multi-TB media mounts
      - /mnt/f:/data/media2
      - /mnt/g:/data/media3
    devices: ["/dev/dri:/dev/dri"]     # ← Hardware passthrough
    restart: unless-stopped

  jellyseerr:
    image: fallenbagel/jellyseerr:latest
    ports: ["5055:5055"]
    volumes: ["/mnt/d/docker/jellyseerr/config:/app/config"]
    dns: ["1.1.1.1", "8.8.8.8"]       # ← Production DNS
    restart: unless-stopped

  audiobookshelf:
    image: ghcr.io/advplyr/audiobookshelf:latest
    ports: ["13378:80"]
    volumes:
      - /mnt/d/docker/audiobookshelf/config:/config
      - "/mnt/f/Audio Books:/audiobooks"
    restart: unless-stopped

  portainer:
    image: portainer/portainer-ce:latest
    ports:
      - "127.0.0.1:9000:9000"         # ← Security: localhost binding
      - "127.0.0.1:9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock  # Docker-in-Docker
    restart: unless-stopped
