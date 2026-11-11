# Notas de trabajo · UD1 · Adquirir un equipo comprometido

- **Analista:** Paula Bernal García
- **Caso:** 20261111-001 · **Fecha:** 11/11/2026 · **Horario:** 16:30 – 19:30
- **Escenario:** ejercicio de laboratorio. Máquina virtual `FORENSIC_10` (PC-ADMIN, Windows) en VirtualBox sobre un anfitrión Windows 10. Simula el cuarto de informática de Almandra Diseño.
- **Solicitante:** Bahía Sistemas.
- **Objetivo:** adquirir las evidencias del equipo de administración (sospecha de compromiso por conexiones extrañas) y de un pendrive sin etiquetar hallado junto al teclado, preservando la cadena de custodia.
---

## 1. Cronología

| Hora | Qué hice |
| --- | --- |
| 16:30 | Inicio de la intervención. |
| 16:40 | Hora del sistema: 4:40 PM, igual que la hora real (desfase 0 min). Captura 1: estado inicial. Escribí mi marca `[AFI-PaulaBernal-20261103]` en PC-ADMIN (Bloc de notas sin guardar y `marca-nombreapellido.txt` en el escritorio) y copié ese fichero al pendrive hallado. Primera acción sobre la evidencia. |
| 16:45 | Pendrive conectado **sin** bloqueo de escritura, solo para copiar la marca. |
| 17:05 | Hash de la RAM (E01). |
| 17:06 | Búsqueda de mi marca desde Windows 10 (Sysinternals `strings` / `Select-String`). Captura 2: marca encontrada. |
| 17:10 | Hashes de los datos volátiles (E02). |
| 17:25 | Hashes de los ficheros de la VM (E05). |
| 17:32 | Captura 3: imagen del disco virtual. |
| 17:40 | Hash de la imagen del disco (E04). |
| 18:08 | Captura 4: comprobación del bloqueo USB en estado PROTEGIDO. |
| 18:20 | Hash de la imagen del pendrive (E06). |
| 19:30 | Fin. Apagado normal de PC-ADMIN (ver sección 6). |

---

## 2. Procesos paso a paso

Orden seguido: de más a menos volátil (RAM, datos volátiles, red, triaje) y después en frío (disco, configuración de la VM y pendrive).

### 2.0 Paso previo: marca

- **Qué hice:** escribí la marca `[AFI-PaulaBernal-20261103]` en el Bloc de notas (sin guardar) y en `marca-nombreapellido.txt` del escritorio, y copié el fichero al pendrive hallado (16:40–16:45).
- **Para qué:** poder demostrar después que la marca aparece en la RAM, en el triaje y en la imagen.
- **Captura:** `capturas/Estado Inicial 2026-11-11 164103.jpg`.

### 2.1 E01 · Memoria RAM

- **Herramienta:** Winpmem 4.0, ejecutado desde el USB de respuesta (64 GB).
- **Resultado:** `memoria31126823.raw` (1024 MB), guardado en `salida\` del USB de respuesta.
- **Hash:** SHA-256 en `hashes/` (RAM, 17:05).
- **Comando exacto:** Usé la herramienta Quickhash
- **Por qué esta herramienta:** Porque me pareció la más cómoda con su interfaz se hace todo muy sencillo.

### 2.2 E02 · Datos volátiles

- **Herramienta:** script `respuesta-envivo.ps1` (PowerShell), desde el USB de respuesta.
- **Resultado:** 22 ficheros `.txt` (fecha y hora, sincronización, arranque, sesiones, conexiones TCP, puertos UDP, netstat, caché ARP y DNS, rutas, configuración de red, recursos compartidos, procesos, servicios, tareas programadas, inicio automático, usuarios y administradores, discos y volúmenes, BitLocker, USB conectados y portapapeles) y `hashes-sha256.csv` generado por el propio script. Carpeta `\windows antigua\salida`.
- **Hash:** cada fichero está en `hashes/` (17:10).
- **Comando exacto de ejecución:** Script proporcionado por el profesor para la adquisición de los datos volátiles.
- **Por qué un script:** Porque al no ser una interfaz gráfica no ocupa tanto espacio en memoria y así no sobrescribimos la memoria.

### 2.3 E03 · Tráfico de red

- **Herramienta:** VBoxManage (VirtualBox), ejecutado desde el anfitrión Windows 10. No se instaló nada en PC-ADMIN, así que la captura no altera el equipo original.
- **Horas:** inicio de la captura [17:00], fin [17:15]
- **Resultado:** red.pcap, SHA-256 calculado a las 17:06 (en hashes/).
- **Comando exacto de ejecución:** VBoxManage controlvm "FORENSIC_10" nictracefile1 /ruta/caso/red.pcap ·· VBoxManage controlvm "FORENSIC_10" nictrace1 on # y "off" al terminar
- **Por qué así:** capturar desde el anfitrión evita instalar herramientas en el equipo sospechoso y no deja huella en él.

### 2.4 Búsqueda de la marca (triaje)

- **Qué hice:** busqué la marca desde Windows 10 (Sysinternals `strings` / `Select-String`) y no desde un Linux, porque Windows 10 es el sistema del anfitrión (17:06).
- **Captura:** `capturas/Evidencia de marca 2026-11-11 170613.jpg`.
- **Comando exacto y fichero donde apareció (RAM, triaje, imagen):** **COMPLETAR**

### 2.5 E04 · Disco de PC-ADMIN

- **Origen:** `FORENSIC_10_disk0.vdi` (≈ 13.045.760 KB, formato VDI). No hay instantáneas.
- **Herramienta:** FTK Imager 8.3.0.27 (formato E01) y QuickHash 3.3.4 (SHA-256) para el hash de origen y de la carpeta.
- **Resultado:** `FORENSI_10_disk0_image.E01` a `.E04` y `FORENSI_10_disk0_image.E01.txt`.
- **Captura:** `capturas/Imagen del disco virtual2026-11-11 173204.jpg`.
- **Opciones elegidas en FTK Imager:** Formato de salida: E01, fragmentación: 1500 MB por fichero, por eso la imagen se reparte en FORENSI_10_disk0_image.E01 a .E04.
- **Por qué FTK Imager:** Porque es una herramienta muy versátil y completa.

### 2.6 E05 · Configuración y registros de la VM

- **Qué copié:** `FORENSIC_10.vbox`, `FORENSIC_10.vbox-prev`, `FORENSIC_10_disk0.vdi` y la carpeta `Logs\` (`VBox.log`, `.1`, `.2`, `.3` y `VBoxHardening.log`).
- **Herramienta:** copia de ficheros y QuickHash (SHA-256).
- **Hash:** `hashes/` (17:25).

### 2.7 E06 · Pendrive RAYVAL (16 GB)

- **Bloqueo de escritura:** por software, con `bloqueo-usb.ps1`, activado **antes** de conectar el pendrive. Estado PROTEGIDO comprobado.
- **Captura:** `capturas/Comprobación usb PROTEGIDO 2026-11-11 180819.jpg`.
- **Herramienta:** FTK Imager 8.3.0.27 (Physical Drive → E01, Verify). Conectado solo al equipo anfitrión.
- **Resultado:** `USB_image.E01` a `.E08` y `USB_image.E01.txt`.
- **Comando del bloqueo y cómo lo comprobé:** Usé el software de `bloqueo_usb.ps1`.

---

## 3. Verificación de las imágenes

- Las imágenes `.E01` se adquirieron con la opción *Verify* de FTK Imager; los resultados están en los `.E01.txt`.
- Las imágenes maestras no se modifican. Se trabajará sobre copias verificadas por hash.
- Todos los hashes SHA-256 están en `hashes.sha256` y en la carpeta `hashes/`. Las herramientas y sus hashes constan en `hashes-herramientas.txt`.

---

## 4. Decisiones y su motivo

- **Orden de adquisición:** de más a menos volátil, para no perder la RAM ni el estado de la red antes de tocar el disco.
- **Sin instantáneas en VirtualBox:** no se hicieron, para no alterar la VM original.
- **Bloqueo de escritura en el pendrive:** activado antes de conectarlo, para no modificar la evidencia durante la adquisición.

---

## 6. Apagado de la máquina

- **Cómo:** apagado normal, después de adquirir todo lo volátil.
- **Cuándo:** 19:30 del 11/11/2026.
- **Por qué apagado normal y no cortando la corriente:** Ya se habían adquirido la RAM, los datos volátiles y el tráfico de red, que son lo único que se pierde al apagar. A partir de ahí no había motivo para mantener el equipo encendido, pero cortar la corriente (en VirtualBox, “Apagar la máquina”) puede dejar NTFS en un estado inconsistente y complicar el análisis posterior.

---

## 6. Incidencias y errores

1. **16:40: acción sobre el equipo original.** Escribí la marca en PC-ADMIN y copié el fichero al pendrive. Fue la primera acción sobre la evidencia.
2. **Formato del disco.** El enunciado habla de un `.vmdk`, el disco real es `FORENSIC_10_disk0.vdi` (formato por defecto de VirtualBox al importar el `.ova`).
3. **16:45: pendrive conectado sin bloqueo** en el paso 0, para copiar la marca. Después, la adquisición se hizo con el bloqueo activado antes de conectar.
4. **17:06: búsqueda de la marca desde Windows 10** y no desde Linux, por ser el sistema del anfitrión.
