# 🏹 Laboratorio: Compromiso Total de "Lian\_Yu" (TryHackMe - ID [Room ID])

Este documento detalla el análisis de vulnerabilidades y el proceso de Penetration Test (Pentest) realizado sobre la máquina objetivo `10.10.46.117`. El ejercicio culminó con éxito, logrando el **acceso root** y la obtención de las flags de usuario y administrador.

---

## 1. 📌 Resumen Ejecutivo

El objetivo fue comprometido debido a una secuencia de pistas temáticas (vigilante) y fallos críticos en la gestión de esteganografía que llevaron a la escalada de privilegios SUID.

| Estado del Objetivo | Tipo de Compromiso | Vulnerabilidad Crítica | Impacto |
| :--- | :--- | :--- | :--- |
| **COMPROMETIDO** | Control Total (Root) | Mala Configuración de **SUID (`pkexec`)** | **CRÍTICO** |

**Banderas Obtenidas:**

* **User Flag:** `THM{P3O P7E_K33P_53CR3T5_C0MPUT3R5_D0N'T}`
* **Root Flag:** `THM{MY_WORD_IS_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_C0MPL3T3D_OR_I'LL_BE_D34D}`

---

## 2. 🛡️ Metodología de Ataque (PTES/OSSTMM)

El ataque se ejecutó siguiendo una metodología estructurada, donde cada pista temática (Arrow/Deathstroke) se convirtió en un componente crítico para el avance.

### Fase A: Reconocimiento y Obtención de Acceso

| Paso | Tarea Clave | Herramienta | Resultado y Pista |
| :--- | :--- | :--- | :--- |
| **1.** | Escaneo de Servicios | Nmap | Puertos **21 (FTP)**, **22 (SSH)** y **80 (HTTP)** abiertos. |
| **2.** | Enumeración Web | Gobuster | Directorios `/island/` y `/island/2100m/` encontrados. |
| **3.** | Descubrimiento Credenciales | curl / Base58 | Token **`RTy8yhB0dscX`** decodificado a **`!#th3h00d`**. |
| **4.** | Acceso Inicial | FTP | Acceso como usuario **`vigilante`**:`!#th3h00d`. |
| **5.** | Pista de Esteganografía | StegSeek | Archivo `aa.jpg` contenía un ZIP con contraseña **`password`**. |
| **6.** | Obtención de Credencial | `unzip` / `cat shado` | Credencial para el usuario **`slade`**:`M3tahuman`. |

### Fase B: Post-Explotación y Escalada de Privilegios

| Paso | Tarea Clave | Herramienta | Resultado y Vector |
| :--- | :--- | :--- | :--- |
| **7.** | Acceso de Usuario | ssh | Acceso como usuario **`slade`** y obtención de `user.txt`. |
| **8.** | Detección de Vector | `find / -perm -u=s` | Identificación del binario crítico: **/usr/bin/pkexec** con el *bit* SUID. |
| **9.** | Referencia de Exploit | GTFOBins | Confirmación del método de explotación para `pkexec` SUID. |
| **10.** | Escalada de Privilegios | `sudo pkexec` | Obtención de una **shell de root (#)** tras autenticación. |
| **11.** | Impacto Final | `cat root.txt` | Lectura de la *root flag*, confirmando el compromiso total. |

---

## 3. 🚨 Análisis de Vulnerabilidades Encontradas

### 3.1. Vulnerabilidad Crítica: Binario SUID Inseguro (`pkexec`)

**Vector:** Escalada de Privilegios.

**Descripción:** El binario `/usr/bin/pkexec` (PolicyKit) estaba configurado con el *bit* SUID. El binario permitió la ejecución de una *shell* con privilegios de `root` después de una simple autenticación con la contraseña del usuario `slade`, lo que indica una configuración de permisos insegura.

```bash

## 5. 💻 Apéndice B: Registro Detallado de Comandos

| \# | Fase | Propósito | Comando Ejecutado |
| :--- | :--- | :--- | :--- |
| **1** | Reconocimiento | Escaneo de servicios. | `nmap -sC -sV 10.10.46.117` |
| **2** | Enumeración Web | Descubrir ruta. | `gobuster dir -u http://10.10.46.117/ -w [wordlist]...` |
| **3** | Enumeración Web | Ver contenido y token. | `curl http://10.10.46.117/island/` |
| **4** | Descubrimiento | Decodificar Base58 y usar FTP. | `ftp vigilante@10.10.46.117` |
| **5** | Post-Explotación | Descargar archivos críticos. | `ftp> get .other_user`, `get aa.jpg` |
| **6** | Post-Explotación | Forzar esteganografía. | `stegseek aa.jpg` |
| **7** | Post-Explotación | Obtener credencial. | `unzip aa.jpg.out` y `cat shado` |
| **8** | Control Usuario | Conexión y `user.txt`. | `ssh slade@10.10.46.117` seguido de `cat user.txt` |
| **9** | Escalada | Buscar binarios SUID. | `find / -perm -u=s -type f 2>/dev/null` |
| **10** | Control Total | Ejecutar el Exploit. | `sudo pkexec /bin/sh` |
| **11** | Control Total | Obtención de `root.txt`. | `cat /root/root.txt` |
