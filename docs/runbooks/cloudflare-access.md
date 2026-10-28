# Cloudflare Access az admin felületek elé

Cél: az admin felületek (Proxmox, Sonarr, Radarr, Prowlarr, letöltőkliens, fájlkezelő, kezdőlap) ne legyenek elérhetők nyilvánosan, csak egy engedélyezett felhasználónak. A felhasználói szolgáltatások (Jellyfin, Jellyseerr, Immich, Nextcloud) nem kerülnek Access mögé, mert vendégek és mobil appok is használják őket; ezeket a [WAF szabályok](cloudflare-waf.md) védik.

## 1. Bejelentkezési mód
**Zero Trust → Settings → Authentication → Login methods**: One-time PIN bekapcsolva (e-mailben érkező kód).

## 2. Szabály
**Access → Policies → Add a policy**
- Name: `Csak admin`
- Action: `Allow`
- Session duration: `1 week`
- Include → `Emails` → az engedélyezett e-mail cím

## 3. Alkalmazás
**Access → Applications → Add an application → Self-hosted**
- Name: `Homelab admin`
- Public hostnames: `proxmox.example.com`, `sonarr.example.com`, `radarr.example.com`, … (minden admin aldomain)
- Policy: `Csak admin`

## 4. Próba
Inkognitó ablakban az admin aldomainre a Cloudflare bejelentkező oldalának kell megjelennie, és egy másik e-mail címmel nem szabad beengednie.

## Ismert korlát
A mobil appok (pl. *arr kliensek) nem tudnak átmenni az e-mail kódos Accessen. Megoldás lehet egy Access service token vagy Tailscale az appnak.
