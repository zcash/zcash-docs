# Instrucciones para Respaldar una Billetera

## Visión de conjunto

Realizar una copia de seguridad de sus claves privadas de Zcash es la mejor manera de ser proactivo para evitar la pérdida de acceso a sus ZEC.

Problemas provocados por bugs en el código, un error del usuario, fallos del dispositivo, etc. pueden conducir a la pérdida de acceso a su billetera (y como resultado, a las claves privadas de sus direcciones, necesarias para realizar gastos desde ellas).

Sin importar la causa potencial del daño o la pérdida de una billetera, recomendamos encarecidamente que todos los usuarios realicen copias de seguridad regularmente. Cada vez que sea generada una nueva dirección en la billetera, recomendamos realizar una nueva copia de seguridad para que todas las claves privadas de las direcciones de su billetera estén seguras.

Tenga en cuenta que una copia de seguridad es un duplicado de los datos necesarios para utilizar ZEC, por lo que la ubicación de su(s) copia(s) de seguridad es también una consideración importante. No debe almacenar copias de seguridad de sus datos donde serían tanto o más susceptibles a una pérdida o robo.

## Instrucciones para respaldar su billetera y/o sus claves privadas

Estas instrucciones son específicas para el cliente oficial Zcash para Linux. Para realizar copias de seguridad con billeteras de terceros, consulte las guías de usuario o los canales de soporte proporcionados para dichos servicios.

Hay varias maneras de asegurarse de tener al menos un respaldo de las claves privadas necesarias para gastar sus ZEC y ver sus ZEC blindados.

Para todos los métodos, deberá establecer un directorio de exportación en su archivo de configuración (`zcash.conf` ubicado en el directorio de datos, que es `~/.zcash/` a menos que haya sido sobreescrito con el valor `datadir=`):

`exportdir=/dirección/del/directorio/de/exportación/elegido`

Puede elegir cualquier directorio dentro del directorio principal como ubicación para los archivos exportados y para la copia de seguridad. Si el directorio no existe, se creará.

Tenga en cuenta que zcashd deberá ser detenido y reiniciado para que las ediciones en el archivo de configuración tengan efecto.

### Utilizar `backupwallet`

Para crear una copia de seguridad de su billetera, utilice:

`zcash-cli backupwallet <nombredelrespaldo>`

La copia de seguridad será una copia exacta del estado actual de su archivo `wallet.dat` almacenado en el directorio de exportación especificado en el archivo de configuración. También aparecerá la ruta del archivo.

Si genera una nueva dirección Zcash, no se reflejará en el archivo de la copia de seguridad.

Si su archivo `wallet.dat` original resulta inaccesible por cualquier motivo, puede usar su respaldo copiándolo en su directorio de datos y cambiando el nombre de la copia a `wallet.dat`.

### Utilizar `z_exportwallet` y `z_importwallet`

Si prefiere exportar sus claves privadas en un formato legible, puede utilizar:

`zcash-cli z_exportwallet <nombredelrespaldo>`

Esto generará un archivo en el directorio de exportación incluyendo todas las claves privadas transparentes y blindadas junto con sus direcciones públicas asociadas. La ruta del archivo aparecerá en la línea de comandos.

Para importar a una billetera las claves que fueron exportadas previamente a un archivo, utilice:

`zcash-cli z_importwallet </dirección/del/directorio/de/exportación/nombredelrespaldo>`

### Utilizar `z_exportkey`, `z_importkey`, `dumpprivkey` y `importprivkey`

Si prefiere exportar la clave privada de una sola dirección blindada, puede utilizar:

`zcash-cli z_exportkey <dirección-z>`

Esto mostrará la clave privada y no creará un nuevo archivo.

Para exportar la clave privada de una sola dirección transparente, puede utilizar el comando heredado de Bitcoin:

`zcash-cli dumpprivkey <dirección-t>`

Esto mostrará la clave privada y no creará un nuevo archivo.

Para importar la clave privada de una dirección blindada, utilice:

`zcash-cli z_importkey <clave-privada-z>`

Esto agregará la clave a su billetera y volverá a escanearla en busca de transacciones asociadas, si no formaban ya parte de la billetera.

El proceso de re-escaneo puede tomar algunos minutos para una nueva clave privada. Para omitirlo, utilice:

`zcash-cli z_importkey <clave-privada-z> no`

Para obtener otras instrucciones sobre el ajuste detallado del re-escaneo de la billetera, consulte la documentación de ayuda del comando:

`zcash-cli help z_importkey`

Para importar la clave privada de una dirección transparente, utilice:

`zcash-cli importprivkey <clave-privada-t>`

Esto tiene la misma función que `z_importkey` pero funciona con direcciones transparentes.

Consulte la documentación de ayuda del comando para obtener instrucciones sobre el ajuste detallado del re-escaneo de la billetera:

`zcash-cli help importprivkey`

### Utilizar `dumpwallet`

Este comando heredado de Bitcoin está obsoleto. Exportará claves privadas de una manera similar a `z_exportwallet` pero sólo para direcciones transparentes.
