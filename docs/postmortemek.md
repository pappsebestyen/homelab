# Postmortemek

Rövid leírások olyan hibákról, amikből tanultam. Formátum: mi történt, mi volt az oka, mit csináltam, mit tanultam.

## 1. A tunnel konténer nem indult el újraindítás után

- **Mi történt:** a Cloudflare Tunnel konténert Debian 11-ről 12-re frissítettem, és utána a tunnel nem indult el magától, így minden külső szolgáltatás elérhetetlen lett.
- **Ok:** a `systemd-networkd-wait-online` szolgáltatás a konténerben a végtelenségig várt egy hálózati állapotra, ami LXC-ben sosem jön meg, és blokkolta a `cloudflared` indulását.
- **Megoldás:** a szolgáltatás maszkolása (`systemctl mask systemd-networkd-wait-online`), és `nesting=1` a konténeren.
- **Tanulság:** LXC-ben nem minden systemd egység működik úgy, mint egy VM-ben. Frissítés után mindig ellenőrizni kell az újraindulást, nem csak a futást.

## 2. A unprivileged visszaállítás törölte a konténert

- **Mi történt:** egy privilegizált konténert a mentéséből unprivileged módban akartam visszaállítani. A visszaállítás hibával leállt, és a Proxmox a meglévő konténert is törölte.
- **Ok:** a mentésben eszközfájlok voltak (pl. `/dev/random`, Docker belső fájlok), amiket unprivileged módban nem lehet létrehozni (`mknod`).
- **Megoldás:** a mentés ép volt, így a `--ignore-unpack-errors 1` kapcsolóval újra visszaállítottam, és működött.
- **Tanulság:** ezért kell mindig friss mentés egy kockázatos lépés előtt. A `--force` visszaállítás a régi konténert akkor is eltávolítja, ha az új nem jön létre.

## 3. A rate limit kitiltotta a saját asztali appomat

- **Mi történt:** a bejelentkezési rate limit bevezetése után a Nextcloud asztali kliens nem tudott belépni.
- **Ok:** a szabály minden olyan útvonalat figyelt, ami tartalmazza a `/login` szöveget. Az asztali app bejelentkezéskor másodpercenként lekérdezi a `/login/v2/poll` címet, így azonnal elérte a limitet.
- **Megoldás:** a szabályt pontos egyezésre szűkítettem (`/login` és `/index.php/login`). A Cloudflare Security Events naplójából derült ki, mit blokkolt.
- **Tanulság:** egy biztonsági szabály túl tág feltétele ugyanúgy hiba, mint a hiányzó szabály. Bevezetés után mindegyik klienssel tesztelni kell.

## 4. Kétszer látszott ugyanaz a tárhely

- **Mi történt:** a Proxmox két tárolót mutatott ugyanazzal a foglaltsággal, ami gyanús volt.
- **Ok:** az egyik storage bejegyzés a `storage.cfg`-ben rossz LVM volume groupra mutatott, így ugyanazt a thin poolt látta két néven.
- **Megoldás:** a hibás bejegyzés törlése, az érintett konténer diskjének átírása a helyes tárolóra.
- **Tanulság:** ha két szám gyanúsan egyezik, érdemes utánanézni. Egy rossz storage-bejegyzés dupla foglaláshoz vagy adatvesztéshez is vezethetne.

## 5. Major frissítés után eltérő verziók

- **Mi történt:** a Jellyfin frissítése után a webes felület és a szerver verziója eltért.
- **Ok:** a sima `apt upgrade` visszatartja azokat a csomagokat, amelyek új függőséget hoznak, így a szerver régi maradt, a web felület frissült.
- **Megoldás:** `apt full-upgrade`, előtte mentés.
- **Tanulság:** a csomagkezelő kimenetét („kept back”) el kell olvasni, nem csak lefuttatni a parancsot.
