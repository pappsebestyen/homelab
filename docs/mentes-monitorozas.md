# Mentés és monitorozás

## Mentés

- **Proxmox vzdump job:** minden vasárnap 01:00, az összes konténer, snapshot mód (leállás nélkül), zstd tömörítés, 4 heti mentés marad meg.
- A cél egy külön disk, ami nem a RAID vezérlő mögött van, így a vezérlő hibája sem viszi magával.
- Egy teljes mentés kb. 44 GB, a 4 hét kb. 176 GB a ~460 GB-ból.
- A nagy médiakönyvtár nincs mentve (újra beszerezhető). A fotóknak több független példánya van (telefon → felhő → Immich).
- Nagyobb változtatás (pl. major frissítés) előtt mindig kézi mentés készül, amiből már vissza is állítottam konténert.

- **Gépen kívüli másolat a jelszótárolóról:** havonta titkosított, jelszóval védett export egy felhőtárhelyre, a fájl jelszava papíron, külön helyen.

**Ami még hiányzik:** gépen kívüli másolat a többi konténerről (3-2-1 szabály) és rendszeres, dokumentált visszaállítási próba.

## Monitorozás és riasztás

| Mit | Mivel | Riasztás |
|---|---|---|
| Szolgáltatások elérhetősége | Uptime Kuma | ntfy push |
| Disk állapot | smartd (napi rövid, heti hosszú teszt, mind az 5 disk a RAID vezérlőn át) | ntfy push |
| Mentési job eredménye | Proxmox értesítés, webhook | ntfy push |
| Áttekintés | Homepage kezdőlap élő widgetekkel | – |

Az értesítések egy helyre futnak be: a Proxmox értesítési szabálya csak az ntfy-ra küld, és a rootnak szóló rendszerlevelek (smartd, cron) is átirányítva oda kerülnek. Lásd a [postmortemek](postmortemek.md) 8. pontját.
