# Guía de Minado de Zcash

¡Bienvenidos! Esta guía está destinada a hacer que puedas minar Zcash, a.k.a. "ZEC", en el mainnet de Zcash. La unidad de medida para la minería es Sol/s (Soluciones por segundo).

Si encuentras algún problema, por favor háznoslo saber. Se necesita un montón de trabajo para hacer que esto sea utilizable y tus sugerencias nos ayudarán a dar antes prioridad a las incidencias potencialmente más dañinas. Para obtener ayuda de otros usuarios, te recomendamos usar nuestro foro:

https://forum.z.cash/

## Preparación

En primer lugar, debes configurar tu nodo Zcash local. Sigue los pasos de la [Guía del Usuario 1.0](1.0 User Guide) hasta el final de la sección "Compilación", luego vuelve aquí. (También puedes hacer la sección "Testear" si lo deseas!)

## Configuración

Configura tu nodo según [[1.0-User-Guide#configuration]], incluyendo la sección [Activación de la Minería de CPU](https://github.com/zcash/zcash-docs/blob/master/es/Sprout_User_Guide.md#enabling-cpu-mining).

## Minado

Ahora, ¡comienza a Minar!
```bash
$ ./src/zcashd
```

Para ejecutarlo en segundo plano (sin la pantalla de métricas de nodo que normalmente se muestra):

```bash
$ ./src/zcashd -daemon
```

Deberías ver la siguiente salida en el registro de depuración (`~/.zcash/debug.log`):

```bash
Zcash Miner started
```

¡Felicitaciones! Ahora estás minando en el mainnet.

### Gasto de las Recompensas de Minado

Las monedas se extraen en un t-addr, (una dirección transparente), pero sólo se pueden gastar en un z-addr (una dirección blindada). Consulta nuestra [Guía del usuario 1.0] (https://github.com/zcash/zcash/wiki/1.0-User-Guide) para obtener instrucciones sobre cómo utilizar el comando `z_sendmany` para enviar monedas desde un t-addr a un z-addr. Necesitarás al menos 4 GB de RAM para esta operación.

## Modificaciones

### Minar a una sola dirección

El minero interno `zcashd` utiliza una nueva dirección transparente para cada bloque minado. Si deseas en cambio utilizar la misma dirección para cada bloque minado, busca la siguiente línea en `src/miner.cpp` (en la función `ProcessBlockFound()`) y en `src/wallet/wallet.cpp` (en la función `CommitTransaction()`):

```cpp
reservekey.KeepKey();
```

Elimina o transforma en comentario esa línea en ambos lugares.

### Utilizar transacciones P2PKH

El minero interno `zcashd` heredado de Bitcoin utiliza P2PK para transacciones de coinbase. La tendencia en el blockchain de Bitcoin ha sido de usar en su lugar P2PKH; estamos considerando [cambiar el minero interno para usar P2PKH] (https://github.com/zcash/zcash/issues/945), pero no para la versión 1.0.

Si deseas utilizar P2PKH para tus transacciones de coinbase, busca la siguiente línea en `src/miner.cpp` (en la función `CreateNewBlockWithKey()`):

```cpp
CScript scriptPubKey = CScript() << ToByteVector(pubkey) << OP_CHECKSIG;
```

Cámbiala a:

```cpp
CScript scriptPubKey = CScript() << OP_DUP << OP_HASH160 << ToByteVector(pubkey.GetID()) << OP_EQUALVERIFY << OP_CHECKSIG;
```
