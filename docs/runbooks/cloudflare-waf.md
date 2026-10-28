# Cloudflare WAF a nyilvános szolgáltatások elé

Két szabály, mindkettő elérhető az ingyenes csomagban is.

## 1. Geo-szabály (Custom rule)
A médiaszolgáltatások csak magyar IP-ről érhetők el.
```
(http.host in {"jellyfin.example.com" "request.example.com" "photos.example.com"} and ip.src.country ne "HU")
```
Action: **Block**

A Nextcloud szándékosan nincs benne, mert megosztási linkeket külföldről is meg kell tudni nyitni.

## 2. Bejelentkezés-korlátozás (Rate limiting rule)
```
(http.host in {"jellyfin.example.com" "request.example.com" "photos.example.com"}
  and (http.request.uri.path contains "/auth/"
       or http.request.uri.path contains "/Users/AuthenticateByName"))
or (http.host eq "cloud.example.com"
  and (http.request.uri.path eq "/login" or http.request.uri.path eq "/index.php/login"))
```
- Characteristics: IP
- 5 kérés / 10 másodperc → Block 10 másodpercre

| Útvonal | Alkalmazás |
|---|---|
| `/auth/` | Immich, Jellyseerr |
| `/Users/AuthenticateByName` | Jellyfin |
| `/login`, `/index.php/login` | Nextcloud |

A Nextcloud útvonalai **pontos egyezéssel** szerepelnek. Az első változat `contains "/login"` volt, ami a Nextcloud asztali kliens `/login/v2/poll` lekérdezését is letiltotta (lásd [postmortemek](../postmortemek.md), 3. pont).

## Ellenőrzés
**Security → Events** alatt látszik, mit blokkolt a szabály. Ha egy TV-s vagy mobil app nem működik, először itt érdemes nézni.
