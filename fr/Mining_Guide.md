# Guide de Minage de Zcash

Bienvenue ! Ce guide est destiné à vous faire miner Zcash, a.k.a "ZEC", sur le mainnet Zcash.
L'unité pour miner est la Sol/s (solution par seconde).

Si vous rencontrez des problèmes, n'hésitez pas à nous contacter. Cela demande beaucoup de travail pour rendre cela utilisable et vos commentaires nous aident à prioriser les problèmes les plus importants. Pour l'aide utilisateur, nous recommandons d'utiliser notre forum :

https://forum.z.cash/

## Préparation

Tout d'abord, vous devez configurer un noeud local Zcash. Suivez le [Guide Utilisateur  1.0](https://github.com/zcash/zcash-docs/blob/master/fr/Sprout_User_Guide.md) jusqu'à la section "Compilation", puis revenez sur ce guide. (Vous pouvez aussi faire la section "Tests" si vous le désirez !)

## Configuration

Configurez votre noeud comme dans [1.0-User-Guide#configuration](https://github.com/zcash/zcash-docs/blob/master/fr/Sprout_User_Guide.md#configuration), jusqu'à la section [Activer le Minage CPU](https://github.com/zcash/zcash-docs/blob/master/fr/Sprout_User_Guide.md#activer-le-minage-cpu-).

## Minage

Maintenant, commençons à miner !
```bash
$ ./src/zcashd
```

Pour le lancer en tâche de fond (sans l'écran avec les métriques du noeud qui est normalement affiché) :

```bash
$ ./src/zcashd -daemon
```

Vous devriez voir la sortie suivante dans le registre de déboggage (`~/.zcash/debug.log`) :

```bash
Zcash Miner started
```

Félicitations ! Vous minez sur le mainnet.

### Dépenser la Récompense de Minage

Les pièces sont minées dans une t-addr (adresse transparente), mais peuvent seulement être dépensées vers une z-addr (adresse protégée). Référez vous à notre [Guide Utilisateur  1.0](https://github.com/zcash/zcash-docs/blob/master/fr/Sprout_User_Guide.md) pour de plus amples instructions sur comment utiliser la commande `z_sendmany` pour envoyer des pièces de t-addr vers une z-addr. Vous devez avoir au moins 4GB de mémoire vive pour cette opération.

## Modifications

### Miner vers une seule adresse

Le miner interne `zcashd` utilise une nouvelle adresse transparente pour chaque blocs minés. Si vous désirez utiliser la même adresse à chaque nouveau bloc miné, allez à la ligne suivante dans les fichiers `src/miner.cpp` (dans la fonction `ProcessBlockFound()`) et `src/wallet/wallet.cpp` (dans la fonction `CommitTransaction()`) :

```cpp
reservekey.KeepKey();
```

Enlevez ou commentez la ligne dans les deux fichiers.

### Utiliser des transactions P2PKH

Le miner interne `zcashd`  hérité de Bitcoin utilise des transactions P2PK pour les transactions coinbase. La tendance actuelle dans la blockchain Bitcoin est d'utiliser P2PKH à la place ; nous considérons [changer le mineur interne pour utiliser P2PKH](https://github.com/zcash/zcash/issues/945), mais pas pour la version 1.0.

Si vous désirez utiliser P2PKH pour les transactions coinbase, chercher la ligne suivante dans `src/miner.cpp` (dans la fonction `CreateNewBlockWithKey()`) :

```cpp
CScript scriptPubKey = CScript() << ToByteVector(pubkey) << OP_CHECKSIG;
```

Changez la en :

```cpp
CScript scriptPubKey = CScript() << OP_DUP << OP_HASH160 << ToByteVector(pubkey.GetID()) << OP_EQUALVERIFY << OP_CHECKSIG;
```
