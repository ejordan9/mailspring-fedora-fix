# mailspring-fedora-fix
Fix for Mailspring losing Gmail sessions on Fedora Linux//Solución para la pérdida de sesiones de Gmail en Mailspring sobre Fedora Linux
Solución para la pérdida de sesiones de Gmail en Mailspring sobre Fedora Linux
Este problema, ocurre con mayor frecuencia cuando se instala Mailspring desde la tienda de aplicaciones, Flatpak o Snap, debido a que estas versiones funcionan en un entorno aislado (sandbox) que impide el acceso adecuado al gestor de credenciales del sistema (`gnome-keyring`). También puede verse afectado por una dependencia rota en el paquete `.rpm` oficial.


Si usas Mailspring en Fedora y, tras reiniciar tu sistema, mailspring siempre te pide las credenciales de tus cuentas de Gmail o correos y te pide volver a conectar cada vez, no estás solo. Este es un error molesto que ocurre por una combinación de factores entre:

a) El manejo de credenciales con gnome-keyring
b) La versión de Mailspring instalada desde la tienda (Flatpak o Snap)
c) Y una dependencia obsoleta (lsb-core-noarch) en el RPM oficial

Comparto el paso a paso probado y funcional para resolverlo definitivamente.

Causa raíz
El problema tiene dos componentes:

1. Mailspring no logra guardar el token OAuth en el keyring debido a que:
 - El gnome-keyring-daemon no está activo
 - O no se le permite acceso por el sandbox (Snap/Flatpak)

2. La versión .rpm oficial de Mailspring requiere una dependencia rota (lsb-core-noarch) que Fedora ya no proporciona, lo cual impide instalarlo desde el sitio oficial si no lo forzas.


Solución completa paso a paso
1. Desinstala completamente Mailspring y limpia tu configuración

killall mailspring

rm -rf ~/.config/Mailspring ~/.mailspring ~/.cache/Mailspring ~/.local/share/Mailspring

Desinstala Mailspring según cómo lo hayas instalado.

2. Verifica que gnome-keyring-daemon esté funcionando
systemctl --user status gnome-keyring-daemon.service

Si está inactivo, actívalo:
systemctl --user enable --now gnome-keyring-daemon.service

Asegúrate también de que el servicio org.freedesktop.secrets esté controlado por el keyring:
busctl --user status org.freedesktop.secrets

Debe indicar: Exe=/usr/bin/gnome-keyring-daemon

3. Descarga el .rpm oficial desde el sitio de Mailspring
🔗 Descargar desde: https://www.getmailspring.com/download

4. Instálalo ignorando la dependencia rota
sudo rpm -i --nodeps ./mailspring-1.15.1-0.1.x86_64.rpm

Nota: Puedes usar rpmrebuild para quitar la dependencia de forma más limpia si prefieres.

5. Conecta tus cuentas en Mailspring y prueba persistencia
Ejecuta:
mailspring
Conecta tus cuentas de Gmail. Luego reinicia el sistema.

✅ Al volver a iniciar, Mailspring ya no te pedirá reconectar. La sesión se guarda correctamente usando gnome-keyring.

✅ Conclusión
Si estás en Fedora y Mailspring te pide reconectar las cuentas de Gmail tras cada reinicio, la causa puede ser un problemas entre dependencias obsoletas, mal manejo del keyring o instalación vía Snap/Flatpak. Siguiendo esta guía paso a paso, al menos yo logre resolverlo de forma definitiva.
