# Biztonság

A cél: a szolgáltatások elérhetők legyenek otthonon kívülről, de a támadási felület a lehető legkisebb maradjon.

## Rétegek

1. **Nincs nyitott port.** A routeren nincs port forward, az UPnP ki van kapcsolva. Minden bejövő forgalom a Cloudflare Tunnelen érkezik, amit a szerver kifelé épít fel.
2. **Cloudflare WAF** a nyilvános szolgáltatások előtt:
   - geo-szabály: a médiaszolgáltatások csak magyar IP-ről érhetők el,
   - rate limit a bejelentkezési végpontokon (5 kérés / 10 s / IP).
   Lásd: [runbooks/cloudflare-waf.md](runbooks/cloudflare-waf.md)
3. **Cloudflare Access (Zero Trust)** az admin felületek előtt (Proxmox, *arr, fájlkezelő, kezdőlap): csak egy engedélyezett e-mail címmel, egyszer használatos kóddal lehet belépni, és a kérés el sem jut a szerverig. Lásd: [runbooks/cloudflare-access.md](runbooks/cloudflare-access.md)
4. **Proxmox tűzfal:** bejövő forgalom alapból tiltva, csak a LAN-ról és a Tailscale-ről engedett.
5. **Kétlépcsős belépés (TOTP)** a Proxmoxon és a Nextcloudon. Az alkalmazások (pl. Obsidian WebDAV szinkron) külön app jelszót kapnak.
6. **Legkisebb jogosultság:**
   - a privilegizált konténereket unprivileged módra állítottam át, és töröltem a teljes hozzáférést adó `lxc.*` sorokat,
   - a kezdőlap widgetjei egy csak olvasási jogú (`PVEAuditor`) Proxmox API tokent használnak,
   - a webes fájlkezelőben a parancsfuttatás ki van kapcsolva, és a gyökérkönyvtára csak a szükséges mappákat látja.
7. **Titkok kezelése:** jelszavak és tokenek `.env` fájlokban (600), sosem a konfigurációban vagy a repóban.

## Privilegizált → unprivileged átállítás

Menet: mentés (`vzdump`, stop mód) → `pct restore --unprivileged 1 --ignore-unpack-errors 1` → a régi `lxc.*` sorok törlése → a bind mountolt könyvtárak tulajdonosának átállítása (`chown 100000:100000`, a UID-eltolás miatt). A tanulságos első próbálkozás a [postmortemek](postmortemek.md) között van.
