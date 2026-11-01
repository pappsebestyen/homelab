# Vaultwarden: döntés és védelem

A Vaultwarden a Bitwarden jelszókezelő szerverének egy kicsi, nyílt forrású változata. A hivatalos Bitwarden appok (böngésző, Android) működnek vele, csak a saját szerver címét kell megadni nekik.

## Döntés: saját szerver vagy felhős Bitwarden?

| | Saját Vaultwarden | Felhős Bitwarden |
|---|---|---|
| Ár | ingyenes, a prémium funkciókkal együtt | az alap ingyenes, a prémium fizetős |
| Adat helye | nálam | a szolgáltatónál (titkosítva) |
| Ha a szerver meghal | az appok offline másolata olvasható, de új eszköz nem tud szinkronizálni | nem az én gondom |
| Karbantartás, mentés | az én dolgom | nincs |
| Kitettség | egy újabb nyilvános szolgáltatás | nincs |

A titkosítás a kliensen történik, a szerver csak titkosított adatot lát. A döntő kérdés ezért nem a titkosság volt, hanem a **rendelkezésre állás és a mentés**: a szerver egyetlen gép egy régi rendszerdiskkel, ezért a saját szervert csak gépen kívüli mentéssel együtt vállaltam.

## Telepítés

- Külön unprivileged LXC (1 mag, 512 MB RAM, 4 GB disk), Docker Compose, `vaultwarden/server`, SQLite.
- Regisztráció csak a saját fiókom létrehozásáig volt nyitva, utána `SIGNUPS_ALLOWED=false`.
- **Nincs `ADMIN_TOKEN`**, így az admin felület teljesen ki van kapcsolva.
- A fiókon TOTP kétlépcsős azonosítás. A mesterjelszó és a helyreállító kód papíron, offline.

## Külső elérés és védelem

A Bitwarden appok nem tudnak átmenni a Cloudflare Access e-mail kódos belépésén, ezért ez a szolgáltatás Access nélkül, de több réteggel megy:

| Réteg | Szabály |
|---|---|
| Geo-szabály | csak magyar IP-ről érhető el |
| Admin tiltás | `starts_with(http.request.uri.path, "/admin")` → Block, a kikapcsolt admin felület mellé második védelemnek |
| Rate limit | a bejelentkezési végpont (`/identity/connect/token`) is a `Login limit` szabályban |

## Mentés és figyelés

- Heti `vzdump` a konténerről, benne az adatbázissal.
- **Gépen kívül:** havonta titkosított, jelszóval védett JSON export egy felhőtárhelyre. A fájl jelszava nem a felhőben van. Ha az egész szerver elveszik, ebből egy új Vaultwardenbe vagy a felhős Bitwardenbe vissza lehet tölteni.
- Uptime Kuma figyeli a `/alive` végpontot, hiba esetén ntfy riasztás.

## Átköltözés

A böngésző beépített jelszókezelőjéből CSV exporttal, Bitwarden importtal. A titkosítatlan CSV-t az import után töröltem, a böngésző jelszómentését kikapcsoltam, Androidon a Bitwarden lett az automatikus kitöltő.
