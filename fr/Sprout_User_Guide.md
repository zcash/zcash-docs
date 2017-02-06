# Guide Zcash 1.0 "Pousse"

Bienvenue! Ce guide est destiné à vous permettre d'utiliser le réseau ZCash officiel. Actuellement, Zcash a encore quelques limitations: seulement Linux est officiellement supporté, requiert 64-bit, et dans certaines situations requiert une utilisation importante de la mémoire vive et du CPU pour créer des transactions.

N'hésitez pas à nous contacter si vous rencontrez un problème. Nous planifions de diminuer l'utilisation du CPU et de la mémoire ainsi que le support d'autres architectures et systèmes d'exploitation dans le future.

## Mise à jour

Si vous participez aux alpha/beta/rc testnets, vérifier que votre `~/.zcash/zcash.conf` ne contient pas `testnet=1` ou `addnode=betatestnet.z.cash`. Si vous êtes sur un distribution basé sur Debian, vous pouvez suivre les [instructions Debian](https://github.com/zcash/zcash/wiki/Debian-binary-packages) pour installer zcash sur votre système.
Dans le cas contraire, vous pouvez mettre à jour votre snapshot avec ce code:

```
git fetch origin
git checkout v1.0.3
./zcutil/fetch-params.sh
./zcutil/build.sh -j$(nproc)
```

Vérifier également que votre dossier ``~/.zcash``  contient seulement ``zcash.conf`` pour commencer.


## Bref note sur la terminologie

Zcash supporte deux type d'adresses, une _z-addr_ (qui commence avec un **z**) est un type d'adresse qui utilise les preuves à divulgation nulle de connaissance et d'autres chiffrements pour protéger la vie privée de l'utilisateur. Il y aussi les t-addr (qui commence avec un **t**) qui sont similaires aux adresses Bitcoin.

## Configuration Requise

Actuellement, il vous faudra:

* Linux (de préférence une distribution basée sur Debian)
* 64-bit
* 4GB de mémoire vive disponible.

Les interface sont un client en ligne de commande (`zcash-cli`) et une interface d'appel de procédure à distance (en anglais RPC), qui est documentée ici:

https://github.com/zcash/zcash/blob/v1.0.3/doc/payment-api.md

## Sécurité

Avant d'installer, de mettre à jour ou de mettre en marche zcash, assurez-vous d'avoir vérifier les potentielles problèmes de sécurité. Veuillez lire notre page sur la sécurité:

https://z.cash/support/security.html

## Démarrez

### Système d'exploitation basé sur Debian

Veuillez suivre les instructions ici: https://github.com/zcash/zcash/wiki/Debian-binary-packages

### Compiler vous même

Sur un système basé sur Ubuntu/Debian:

```bash
$ sudo apt-get install \
      build-essential pkg-config libc6-dev m4 g++-multilib \
      autoconf libtool ncurses-dev unzip git python \
      zlib1g-dev wget bsdmainutils automake
```

Sur un système basé sur Fedora:

```bash
$ sudo dnf install \
      git pkgconfig automake autoconf ncurses-devel python wget \
      gtest-devel gcc gcc-c++ libtool patch
```

Récupérer notre répertoire avec git et exécuter `fetch-params.sh` comme ceci:

```bash
$ git clone https://github.com/zcash/zcash.git
$ cd zcash/
$ git checkout v1.0.3
$ ./zcutil/fetch-params.sh
```

Cela va récupérer les clés de vérification et de preuve  (les clés finales créées lors de la [Cérémonie de Génération des Paramètres](https://github.com/zcash/mpc)), et les places dans `~/.zcash-params/`. Ces clés occupent un peu moins de 911MB d'espace disque, ainsi le téléchargement peu prendre du temps.

Le message affiché par ``git checkout`` sur une "detached head" est normal et n'indique pas un problème.

#### Compilation

Assurez-vous d'avoir correctement installé les dépendances listé ci dessus. Puis, lancer la compilation:

```bash
$ ./zcutil/build.sh -j$(nproc)
```
Cela devrait compiler les dépendances et `zcashd`. (Note: si vous n'avez pas de `nproc`, remplacez ce numéro par votre nombre de processeur.)

#### Tests

Les test prennent du temps à s'exécuter et peuvent requérir plus de 8GB de mémoire vive. Si vous préférez démarrer directement, vous pouvez directement aller à la section suivante. Si vous voulez lancer les tests pour vous assurer que ZCash fonctionne, exécutez:

```bash
$ ./qa/zcash/full-test-suite.sh
```

Vous pouvez aussi exécuter les tests RPC, qui prennent plus de temps:

```bash
$ ./qa/pull-tester/rpc-tests.sh
```

Ces tests nécessitent une quantité importante de mémoire vive pour s'exécuter correctement. Une erreur de mémoire insuffisante (out-of-memory error en anglais) résulte en général en un FAIL ou en une ERROR avec "std::bad_alloc" quelque part dans la sortie.

## Configuration

Créer le dossier `~/.zcash` et placer une fichier de configuration à `~/.zcash/zcash.conf` avec les commandes suivantes:

```bash
mkdir -p ~/.zcash
echo "addnode=mainnet.z.cash" >~/.zcash/zcash.conf
echo "rpcuser=username" >>~/.zcash/zcash.conf
echo "rpcpassword=`head -c 32 /dev/urandom | base64`" >>~/.zcash/zcash.conf
```

Notez qu'il est possible que cela écrase vos réglages  `zcash.conf` précédentes que vous auriez pi ajouter du testnet. Vous pouvez garder un `zcash.conf` du testnet, mais vérifier que les réglages `testnet=1` et  `addnode=betatestnet.z.cash` sont été supprimés; et remplacer par `addnode=mainnet.z.cash` à la place. Nous vous recommandons fortement d'utiliser une mot de passe aléatoire afin d'éviter des [problèmes de sécurité potentiel avec l'interface RPC](https://github.com/zcash/zcash/blob/master/doc/security-warnings.md#rpc-interface).

### Activer le minage CPU:

Si vous voulez activer le minage avec le CPU, exécutez ces commandes:

```bash
$ echo 'gen=1' >> ~/.zcash/zcash.conf
$ echo "genproclimit=$(nproc)" >> ~/.zcash/zcash.conf
```

Le mineur par défaut n'est pas efficace, mais a été bien passé en revue. Pour utiliser un mineur bien plus efficace mais non passé en revue, vous pouvez exécuter cette commande:

```bash
$ echo 'equihashsolver=tromp' >> ~/.zcash/zcash.conf
```

Note: vous pouvez lire le [Guide de Minage](https://github.com/zcash/zcash-docs/blob/master/fr/Mining_Guide.md) pour plus d'informations sur le minage.

## Exécuter Zcash:

Maintenant, lancer zcashd!

```bash
$ ./src/zcashd
```

Pour la lancer en tache de fond (sans l'écran avec les différentes métriques du noeud qui est normalement affiché) exécuter ``./src/zcashd --daemon``.

Vous devriez pouvoir utiliser l'interface RPC une fois que le chargement est terminé. Une façon rapide de tester:

```bash
$ ./src/zcash-cli getinfo
```

**NOTE**: Si vous êtes familier avec l'interface RPC de bitcoind, vous pouvez utiliser la plupart de ces commandes pour envoyer des ZEC entre les `t-addr` adresses. Nous ne supportons pas la fonctionnalité de 'Comptes' (qui est maintenant obsolète dans ``bitcoind``) — seulement une chaine de caractères vide ``""`` peut être utiliser comme nom de comptes.

**NOTE**: Le noeud principal du réseau accessible à mainnet.z.cash est aussi accessible via les services cachés de Tor à zcmaintvsivr7pcn.onion.

Pour voir à quels pairs vous êtes connectées:

```bash
$ ./src/zcash-cli getpeerinfo
```

## Utiliser Zcash

Tout d'abord, il vous faut obtenir des Zcash. Vous pouvez les acheter sur une plateforme d'échange, à d'autres utilisateurs, ou vendre des produits et services pour en récupérer! La meilleur façon d'obtenir des Zcash (de manière sure) n'est pas documenter ici, mais faites attention. Evitez les arnaques!

### Générez une t-addr

Tout d'abord, générons une t-addr.

```bash
$ ./src/zcash-cli getnewaddress
tb4oHp2v54vfmdgQ3v3SNuQga8JKHTNi2a1
```

### Recevoir des Zcash avec une z-addr

Maintenant, générons une z-addr.

```bash
$ ./src/zcash-cli z_getnewaddress
ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG
```

Cela va créer une adresse privée et stocker les clés dans votre fichier portefeuille local. Donnez cette adresse à votre expéditeur!

Les z-addr sont plutôt longue, une erreur est vite arrivé. Mettons la dans une variable d'environnement pour éviter les erreurs:

```bash
$ ZADDR='ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG'
```

Pour obtenir une liste de toutes les adresses de votre portefeuille dont vous avec les clés, exécutez cette commande:

```bash
$ ./src/zcash-cli z_listaddresses
```

Vous devriez voir quelque chose comme:
```json
[
    "zta6qngiR3U7HxYopyTWkaDLwYBd83D5MT7Jb9gpgTzPLMZytzRbtdPP1Syv4RvRgHeoZrJWSask3DyfwXG9DGPMWMvX7aC",
    "ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG"
]
```

Super! Maintenant envoyer votre z-addr à votre expéditeur. Vous devriez voir à un moment la transaction en exécutant:

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

### Envoyer des pièces avec votre z-addr

Si quelqu'un vous donne sa z-addr...

```bash
$ FRIEND='ztcDe8krwEt1ozWmGZhBDWrcUfmK3Ue5D5z1f6u2EZLLCjQq7mBRkaAPb45FUH4Tca91rF4R1vf983ukR71kHyXeED4quGV'
```

Vous pouvez lui envoyer 0.8 ZEC en faisant...

```bash
$ ./src/zcash-cli z_sendmany "$ZADDR" "[{\"amount\": 0.8, \"address\": \"$FRIEND\"}]"
```

Après avoir attendu environ une minute, vous pouvez vérifier si l'opération est finie et a produit un résultat:
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


## Problèmes de Sécurité Connus

Chaque version contient un document `./doc/security-warnings.md` décrivant les problèmes de sécurités connues qui touchent cette version. Vous pouvez trouver la version la plus récente de ce document ici:

https://github.com/zcash/zcash/blob/master/doc/security-warnings.md

Merci de nous contacter
Pour de plus amples informations sur les notifications récentes et d'autres ressources, veuillez consulter notre page sur la sécurité:

https://z.cash/support/security.html
