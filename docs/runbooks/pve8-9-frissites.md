# Proxmox VE 8 → 9 helyben frissítés

Egy node-os Proxmox frissítése Debian 12-ről (bookworm) Debian 13-ra (trixie). Előfeltétel: a `pve8to9 --full` ne adjon FAIL-t, és legyen friss mentés minden konténerről.

## Előkészítés

A `pve8to9 --full` által talált hibák, amiket előtte javítani kellett:
- egy harmadik féltől származó apt repó (Tailscale) még régi Debian verzióra mutatott,
- a `/etc/sysctl.conf` egyedi beállításait át kellett tenni a `/etc/sysctl.d/` alá,
- hiányzott az `intel-microcode` (ehhez a `non-free-firmware` repó kell),
- az enterprise repó előfizetés nélkül hibát adott, ezért eltávolítottam.

## 0. Kapcsolódás

- **Ne a webes Shellből**, mert frissítés közben a webes felület is újraindul.
- Helyi hálózatról, SSH-val vagy az iDRAC konzollal, `tmux`-ban, hogy egy megszakadt kapcsolat ne szakítsa meg a frissítést:
```
apt install -y tmux
tmux        # újracsatlakozás: tmux attach
```

## 1. Konténerek leállítása
```
for ct in $(pct list | awk 'NR>1 && $2=="running"{print $1}'); do pct shutdown $ct; done
pct list
```

## 2. Repók átállítása trixie-re
```
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list /etc/apt/sources.list.d/*.list
grep -rh '^deb' /etc/apt/sources.list /etc/apt/sources.list.d/
apt update
```

## 3. Frissítés
```
apt dist-upgrade
```
| Kérdés | Válasz |
|---|---|
| `/etc/issue` | N (marad a régi) |
| `/etc/lvm/lvm.conf` | Y (az új verzió kell) |
| `/etc/default/grub` | N (marad a régi) |
| Egyéb konfigfájl | N |
| Szolgáltatások újraindítása | Yes |

## 4. Ellenőrzés, újraindítás
```
pve8to9 --full | grep -E '^(FAIL|WARN)|TOTAL'
reboot
```
Ha itt FAIL van, nem szabad újraindítani.

## 5. Utána
```
pveversion
pct list
```
Utólag kiderült, hogy NTP kliens nem volt telepítve, ezért feltettem a `chrony`-t.

## Visszalépési terv

Ha a gép nem jön fel, az iDRAC konzolon a GRUB menüben (Advanced options) a régi kernellel elindul. Végső esetben a mentésből a konténerek egy újratelepített hostra visszaállíthatók.
