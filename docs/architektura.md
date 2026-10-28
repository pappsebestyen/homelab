# Architektúra

## Hardver

| Elem | Részlet |
|---|---|
| Szerver | Dell PowerEdge R510 (2U, 8 db 3.5"-es hely), használtan vásárolva |
| CPU | 2× Intel Xeon L5630 (4 mag / 8 szál, 2.4 GHz) |
| RAM | 80 GB DDR3 ECC Registered |
| RAID | Dell PERC H700 |
| Távoli kezelés | iDRAC 6 Enterprise (nincs hálózatra kötve) |
| Táp | 2× 750 W redundáns |

A vásárláskor az egyik RAM modul és a hirdetésben szereplő 3 TB-os SAS disk hibásnak bizonyult, ezeket kivettem, a médiának saját 2 TB-os diskem került a helyére.

## Virtualizáció

- **Proxmox VE 9** (Debian 13 alapon), egyetlen node.
- A PVE 9 telepítője ezen a hardveren nem futott le, ezért a rendszer PVE 7-ről indult, és helyben frissült 8-ra, majd 9-re. A 8 → 9 lépései: [runbooks/pve8-9-frissites.md](runbooks/pve8-9-frissites.md).
- Minden szolgáltatás **LXC konténerben** fut. A Docker-alapú szolgáltatások unprivileged konténerben, `nesting=1` és `keyctl=1` beállítással futnak.

## Tárolás

| Disk | Kialakítás | Mire |
|---|---|---|
| 250 GB | LVM (`pve`), thin pool | Proxmox rendszer + a konténerek rootfs-e |
| 2× 500 GB | PERC RAID1 → LVM thin | fotók (Immich), Nextcloud adatok |
| 2 TB | LVM thin | médiakönyvtár |
| 320 GB | LVM thin, jelenleg üres | tartalék |
| 500 GB | külön disk, a RAID vezérlőn kívül, ext4 | mentések |

A nagy adatkönyvtárak bind mounttal kerülnek be a konténerekbe, így a konténerek kicsik maradnak és gyorsan menthetők.

## Hálózat

- Egy LAN alhálózat, a szolgáltatások fix IP-vel.
- **Külső elérés:** Cloudflare Tunnel egy külön konténerből. A tunnel közvetlenül a szolgáltatásokra mutat, reverse proxy nincs közben (egy korábban telepített, de nem használt Nginx Proxy Managert megszüntettem).
- **Távoli adminisztráció:** Tailscale a hoston, illetve Cloudflare Access.
- A routeren nincs port forward, és az UPnP ki van kapcsolva.

## Docker Compose stackek

A Nextcloud stack: `nextcloud:production-apache`, külön cron konténer, `postgres:16-alpine`, `redis:7-alpine`, `collabora/code` (Nextcloud Office). A jelszavak `.env` fájlban vannak (600-as jogosultság), a compose fájl csak hivatkozik rájuk. A reverse proxy mögötti működéshez beállítva: trusted proxies, `overwriteprotocol https`, háttérfeladatok cronnal.
