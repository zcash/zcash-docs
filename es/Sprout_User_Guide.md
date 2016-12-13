# Guía Zcash 1.0 "Brote"

¡Bienvenidos! Esta guía está destinada a que puedas utilizar la red oficial de Zcash. Zcash tiene actualmente algunas limitaciones: sólo soporta oficialmente Linux, requiere 64 bits, y en algunas situaciones requiere mucha memoria y consumo de CPU para crear transacciones.

Por favor, háznoslo saber si te encuentras con algún problema. Tenemos previsto hacer que funcione con un uso menos intensivo de memoria y CPU y que soporte más arquitecturas de computadoras y sistemas operativos en el futuro.

## ¿Actualizar?

Si estabas participando en nuestros testnets alfa/beta/rc, asegúrate de que tu `~/.zcash/zcash.conf` no contenga `testnet=1` o `addnode=betatestnet.z.cash`. Si estás utilizando una distribución basada en Debian, puedes seguir las [instrucciones para Debian] (https://github.com/zcash/zcash/wiki/Debian-binary-packages) para instalar zcash en tu sistema. De lo contrario, puedes actualizar tu instantánea local de nuestro código:

```
git fetch origin
git checkout v1.0.1
./zcutil/fetch-params.sh
./zcutil/build.sh -j$(nproc)
```

También asegúrate de que tu directorio ``~/.zcash`` sólo contiene ``zcash.conf`` para empezar.

## Una nota rápida sobre terminología

Zcash admite dos tipos diferentes de direcciones. Una _z-addr_, o 'dirección z' (que comienza con una **z**) es una dirección que utiliza pruebas de conocimiento-cero y otros cifrados para proteger la privacidad del usuario. También hay _t-addrs_, o 'direcciones t' (que comienzan con una **t**) que son similares a las direcciones de Bitcoin.

## Requisitos

Actualmente, necesitarás:

* Linux (preferentemente una distribución basada en Debian)
* 64-bits
* 4 GB de memoria disponible

Las interfaces son un cliente de línea de comandos (`zcash-cli`)  y una interfaz de Llamada de Procedimiento Remoto (RPC, por sus siglas en inglés), que se documenta aquí:

https://github.com/zcash/zcash/blob/v1.0.1/doc/payment-api.md

## Seguridad

Antes de instalar, actualizar o ejecutar zcash, asegúrate de haber chequeado cualquier problema de seguridad. Visita nuestra página de seguridad:

https://z.cash/support/security.html

## Comenzar

### Sistemas operativos basados en Debian

Sigue las instrucciones aquí: https://github.com/zcash/zcash/wiki/Debian-binary-packages

### Compílalo tu mismo

En sistemas basados en Ubuntu/Debian:

```bash
$ sudo apt-get install \
build-essential pkg-config libc6-dev m4 g++-multilib \
autoconf libtool ncurses-dev unzip git python \
zlib1g-dev wget bsdmainutils automake
```

En sistemas basados en Fedora:

```bash
$ sudo dnf install \
git pkgconfig automake autoconf ncurses-devel python wget \
gtest-devel gcc gcc-c++ libtool patch
```

Busca nuestro repositorio con git y ejecuta `fetch-params.sh`  de esta manera:

```bash
$ git clone https://github.com/zcash/zcash.git
$ cd zcash/
$ git checkout v1.0.1
$ ./zcutil/fetch-params.sh
```

Esto buscará nuestras claves de comprobación y verificación para 'Brote' (las últimas creadas en la [Ceremonia de Generación de Parámetros](https://github.com/zcash/mpc)), y las colocará en `~/.zcash-params/`. Estas claves tienen un tamaño de un poco menos de 911 MB, por lo que puede tomar algún tiempo descargarlas.

El mensaje impreso por ``git checkout`` sobre un "detached head" es normal y no indica un problema.

#### Compilación

Asegúrate de que has instalado correctamente todas las dependencias de paquetes de sistema como fue descrito anteriormente. A continuación, ejecuta la compilación, por ejemplo:

```bash
$ ./zcutil/build.sh -j$(nproc)
```

Esto debería compilar nuestras dependencias y crear `zcashd`. (Nota: si no tienes `nproc`, sustituye el número de tus procesadores.)

#### Testeo

Las pruebas tardan un tiempo en correr y pueden requerir hasta 8 GB de RAM. Si prefieres empezar de inmediato, puedes pasar a la siguiente sección. Si deseas ejecutar las pruebas para asegurarte de que Zcash está funcionando, ejecuta:

```bash
$ ./qa/zcash/full-test-suite.sh
```

También puedes ejecutar las pruebas RPC, que tardan mucho más:

```bash
$ ./qa/pull-tester/rpc-tests.sh
```

Las pruebas necesitan mucha memoria para ejecutarse correctamente. Un error de falta de memoria usualmente causará un resultado FAIL o ERROR con "std::bad_alloc" en alguna parte del informe.

## Configuración

Crea el directorio `~/.zcash` y agrega un archivo de configuración en `~/.zcash/zcash.conf` utilizando los siguientes comandos:

```bash
mkdir -p ~/.zcash
echo "addnode=mainnet.z.cash" >~/.zcash/zcash.conf
echo "rpcuser=username" >>~/.zcash/zcash.conf
echo "rpcpassword=`head -c 32 /dev/urandom | base64`" >>~/.zcash/zcash.conf
```

Ten en cuenta que esto sobrescribirá cualquier configuración `zcash.conf` que puedas haber agregado de testnet. Puedes conservar un `zcash.conf` de testnet, pero asegúrate de que los parámetros `testnet=1` y `addnode=betatestnet.z.cash` sean eliminados; utiliza en su lugar `addnode=mainnet.z.cash`. Te recomendamos firmemente que utilices una contraseña aleatoria para evitar [posibles problemas de seguridad con acceso a la interfaz RPC](https://github.com/zcash/zcash/blob/master/doc/security-warnings.md#rpc-interface).

### Habilitar el minado de CPU:

Si quieres habilitar el minado de CPU, ejecuta estos comandos:

```bash
$ echo 'gen=1' >> ~/.zcash/zcash.conf
$ echo "genproclimit=$(nproc)" >> ~/.zcash/zcash.conf
```

El minero por defecto no es eficiente, pero ha sido bien revisado. Para utilizar un solver mucho más eficiente pero no revisado, puedes ejecutar este comando:

```bash
$ echo 'equihashsolver=tromp' >> ~/.zcash/zcash.conf
```

Nota, probablemente quieras leer la [[Guía de Minado]] para obtener más detalles sobre el minado.

## Ejecutar Zcash:

Ahora, ejecuta zcashd!

```bash
$ ./src/zcashd
```

Para ejecutarlo en segundo plano (sin la pantalla de métricas de nodo que se muestra normalmente) usa ``./src/zcashd --daemon``.

Deberías poder utilizar el RPC después de que termine de cargar. Aquí está una manera rápida de probar:

```bash
$ ./src/zcash-cli getinfo
```

**NOTA**: Si estás familiarizado con la interfaz RPC de bitcoind, puedes utilizar muchas de esas llamadas para enviar ZEC entre direcciones `t-addr`. No admitimos la función 'Cuentas' (que también ha sido despreciada en``bitcoind``) —sólo el string vacío ``""`` puede ser utilizado como nombre de cuenta.

**NOTA**: El nodo principal de la red en mainnet.z.cash también es accesible a través del servicio oculto Tor en zcmaintvsivr7pcn.onion.

Para ver los pares a los que estás conectado:

```bash
$ ./src/zcash-cli getpeerinfo
```

## Utilizar Zcash

En primer lugar, deseas obtener dinero Zcash. ¡Puedes comprarlo de un cambio, de otros usuarios, o vender mercancías y servicios por ZEC! Cómo exactamente obtener Zcash (con seguridad) escapa al alcance de este documento, pero deberías tener cuidado. ¡Evita las estafas!

### Generar un t-addr

Empecemos por generar un t-addr.

```bash
$ ./src/zcash-cli getnewaddress
tb4oHp2v54vfmdgQ3v3SNuQga8JKHTNi2a1
```

### Recibir Zcash con un z-addr

Ahora generemos un z-addr.

```bash
$ ./src/zcash-cli z_getnewaddress
ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG
```

Esto crea una dirección privada y almacena su clave en tu archivo de billetera local. ¡Entrega esta dirección al remitente!

Un z-addr es bastante grande, por lo que es fácil cometer errores. Vamos a ponerlo en una variable de entorno para evitar errores:

```bash
$ ZADDR='ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG'
```

Para obtener una lista de todas las direcciones de tu billetera para las que tienes una clave de gasto, ejecuta este comando:

```bash
$ ./src/zcash-cli z_listaddresses
```

Deberías ver algo como:
```json
[
"zta6qngiR3U7HxYopyTWkaDLwYBd83D5MT7Jb9gpgTzPLMZytzRbtdPP1Syv4RvRgHeoZrJWSask3DyfwXG9DGPMWMvX7aC",
"ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG"
]
```

¡Genial! Ahora envía tu z-addr al remitente. Deberías eventualmente ver su transacción al chequear:

```bash
$ ./src/zcash-cli z_listreceivedbyaddress "$ZADDR"
```
```json
[
{
"txid" : "af1665b317abe538148114a45322f28151925501c081949cc7a5207ef21cb750",
"amount" : 1.23,
"memo" : "48656c6c6f20ceb2210000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
}
]
```

### Enviar dinero con tu z-addr

Si alguien te pasa su z-addr...

```bash
$ FRIEND='ztcDe8krwEt1ozWmGZhBDWrcUfmK3Ue5D5z1f6u2EZLLCjQq7mBRkaAPb45FUH4Tca91rF4R1vf983ukR71kHyXeED4quGV'
```

Puedes enviar 0,8 ZEC haciendo...

```bash
$ ./src/zcash-cli z_sendmany "$ZADDR" "[{\"amount\": 0.8, \"address\": \"$FRIEND\"}]"
```

Después de esperar alrededor de un minuto, puedes comprobar si la operación ha terminado y ha producido un resultado:
```bash
$ ./src/zcash-cli z_getoperationresult
```
```json
[
{
"id" : "opid-4eafcaf3-b028-40e0-9c29-137da5612f63",
"status" : "success",
"creation_time" : 1473439760,
"result" : {
"txid" : "3b85cab48629713cc0caae99a49557d7b906c52a4ade97b944f57b81d9b0852d"
},
"execution_secs" : 51.64785629
}
]
```


## Problemas de Seguridad Conocidos

Cada versión contiene un documento `./doc/security-warnings.md` describiendo los problemas de seguridad que sabemos que afectan a esa versión. Puedes encontrar la versión más reciente de este documento aquí:

https://github.com/zcash/zcash/blob/master/doc/security-warnings.md

Consulta también nuestra página de seguridad para ver las notificaciones recientes y otros contenidos:

https://z.cash/es/support/security.html

https://z.cash/es/support/security.html
