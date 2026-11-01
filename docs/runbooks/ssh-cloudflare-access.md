# SSH a hostra kívülről, nyitott port nélkül

Cél: a Proxmox host SSH-ja otthonon kívülről is elérhető legyen, de a routeren ne kelljen portot nyitni, és jelszóval ne lehessen kívülről belépni.

## Felépítés

```
kliens (ssh + cloudflared) → Cloudflare Access (e-mail kód) → tunnel → tunnel konténer → host :22
```

A host tűzfalán nem kellett változtatni: a tunnel forgalma a tunnel konténer LAN címéről érkezik, amit a szabály már engedett.

## 1. Tunnel és Access

1. Zero Trust → Networks → Tunnels → a tunnel → Public hostname → Add: `ssh.example.com`, típus **SSH**, cél `ssh://<host belső címe>:22`.
2. Access → Applications → `Homelab admin` → a hostname hozzáadása, így ugyanaz a szabály védi, mint a webes felületet.

## 2. Csak kulcsos belépés a tunnel felől

Kulcs a kliensen (`ssh-keygen -t ed25519`, jelmondattal), a nyilvános kulcs a hoston az `authorized_keys`-be. **Előbb ki kell próbálni a kulcsos belépést**, csak utána szabad a jelszót tiltani.

`/etc/ssh/sshd_config.d/10-csak-kulcs.conf`:
```
PermitRootLogin prohibit-password
PasswordAuthentication no
KbdInteractiveAuthentication no

# helyi hálózatról jelszó is mehet, a tunnel konténer címe kivételével
Match Address !<tunnel konténer címe>,<LAN alhálózat>
    PermitRootLogin yes
    PasswordAuthentication yes
```
```
sshd -t && systemctl reload ssh
```
A régi SSH ablakot nyitva kell hagyni, és egy újban kipróbálni. Ellenőrzés címenként: `sshd -T -C addr=<ip>,user=root,host=x | grep -i password`.

Fontos részlet: a tunnel forgalma is LAN címről érkezik, ezért a `Match` szabályból a tunnel konténer címét **ki kell zárni**, különben kívülről is menne a jelszó.

## 3. Kliens

`cloudflared` telepítése (Windowson `winget install Cloudflare.cloudflared`), majd `~/.ssh/config`:
```
Host proxmox
  HostName ssh.example.com
  User root
  ProxyCommand cloudflared access ssh --hostname %h
```
`ssh proxmox`: első alkalommal böngészőben kéri az Access belépést. Grafikus kliensben (pl. Tabby) ugyanez a parancs „proxy command”-ként adható meg.

## Visszalépés

`rm /etc/ssh/sshd_config.d/10-csak-kulcs.conf && systemctl reload ssh`, helyben vagy a webes Shellből.
