# Guia de Mineração Zcash

Bem vindo! Este guia destina-se a obter Zcash através de mineração, ou "ZEC", no mainnet Zcash. A unidade para mineração é Sol/s (Soluções por segundo).

Se você tiver problemas, entre em contato conosco. Há muito trabalho necessário para torná-lo utilizável e a sua entrada nos ajudará a aparar as bordas mais afiadas. Para obter ajuda de um usuário, recomendamos que utilize o nosso fórum:

https://forum.z.cash/

## Setup

Primeiro, você precisa configurar seu node local da Zcash. Siga o [Guia de Usuário 1.0] (Guia de Usuário 1.0) até o final da seção "Compilação", depois volte aqui. (Você também pode fazer a seção "Testando" se você quiser!)

## Configuração

Configure seu node conforme [[Guia de Usuário 1.0#configuração]], incluindo a seção [Ativando a Mineração por CPU] (https://github.com/zcash/zcash/wiki/1.0-User-Guide#enabling-cpu-mining).

## Minerando

Agora, que comece a mineração!
```bash
$ ./src/zcashd
```

Para executá-lo em background (sem a tela de métricas do node que é exibida normalmente):

```bash
$ ./src/zcashd -daemon
```

Você deve ver a seguinte saída no log de depuração (`~/.zcash/debug.log`):

```bash
Zcash Miner started
```

Parabéns! Você está agora minerando em mainnet.

### Recompensas de Mineração

As moedas são mineradas em um t-addr (endereço transparente), mas só pode ser gasto em um z-addr (endereço blindado). Consulte o nosso [Guia do Usuário 1.0 ] (https://github.com/zcash/zcash/wiki/1.0-User-Guide) para obter instruções sobre como usar o comando `z_sendmany` para enviar moedas de um t-addr para um z-addr. Você precisará de pelo menos 4GB de RAM para esta operação.

## Modificações

### Minere para um único endereço

O minerador interno `zcashd` usa um novo endereço transparente para cada bloco minerado. Se você quiser usar o mesmo endereço para cada bloco minado, encontre a seguinte linha em `src/miner.cpp` (na função `ProcessBlockFound()`) e `src/wallet/wallet.cpp` (na função `CommitTransaction()`):

```cpp
reservekey.KeepKey();
```

Remover ou comentar essa linha em ambos os locais.

### Use transações P2PKH

O minerador interno `zcashd` herdado do Bitcoin usa P2PK para transações de coinbase. A tendência no blockchain do Bitcoin foi de usar P2PKH em vez disso; Estamos considerando [mudar o minerador interno para usar P2PKH](https://github.com/zcash/zcash/issues/945), mas não para a versão 1.0.

Se você quiser usar P2PKH para suas transações de coinbase, encontre a seguinte linha em `src/miner.cpp` (na função `CreateNewBlockWithKey()`):

```cpp
CScript scriptPubKey = CScript() << ToByteVector(pubkey) << OP_CHECKSIG;
```

Altere isso para:

```cpp
CScript scriptPubKey = CScript() << OP_DUP << OP_HASH160 << ToByteVector(pubkey.GetID()) << OP_EQUALVERIFY << OP_CHECKSIG;
```
