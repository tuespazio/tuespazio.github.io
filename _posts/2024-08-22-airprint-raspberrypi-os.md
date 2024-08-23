---
layout: post
title: Airprint RaspberryPi OS
date: 2024-08-22 22:41 -0600
categories: [Linux, RaspberryPi]
tags: [airprint,linux,cups]     # TAG names should always be lowercase

---
Para habilitar AirPrint en CUPS (Common UNIX Printing System), sigue estos pasos:

1. Asegúrate de que CUPS esté instalado y funcionando en tu sistema.

2. Instala Avahi, que es necesario para el descubrimiento de servicios:
   ```
   sudo apt-get install avahi-daemon
   ```

3. Instala los paquetes necesarios para AirPrint:
   ```
   sudo apt-get install cups-filters
   ```

4. Edita el archivo de configuración de CUPS:
   ```
   sudo nano /etc/cups/cupsd.conf
   ```

5. En el archivo, busca la sección "Browsing" y asegúrate de que tenga estas líneas:
   ```
   Browsing On
   BrowseLocalProtocols dnssd
   ```

6. En la misma sección, agrega o modifica estas líneas:
   ```
   ServerAlias *
   ```

7. Guarda los cambios y cierra el editor.

8. Reinicia el servicio CUPS:
   ```
   sudo systemctl restart cups
   ```

9. Asegúrate de que tu impresora esté configurada correctamente en CUPS.

10. Verifica que tu dispositivo iOS y tu servidor CUPS estén en la misma red.

Después de seguir estos pasos, tus impresoras configuradas en CUPS deberían ser detectables por dispositivos iOS a través de AirPrint.

