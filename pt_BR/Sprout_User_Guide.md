# Guia Zcash 1.0 "Sprout"

Bem vindo! Este guia destina-se a colocá-lo em execução na rede oficial da Zcash. A Zcash atualmente tem algumas limitações: ele só suporta oficialmente o Linux, requer 64 bits e em algumas situações requer muita memória e consumo de CPU para criar transações.

Por favor, deixe-nos saber se você executar em snags. Planejamos melhorar a eficiência de memória/CPU menos intenso e adicionar suporte de mais arquiteturas e sistemas operacionais no futuro.

## Atualizando?

Se você estava rodando com nossos testnets alpha/beta/rc, certifique-se de que seu ~/.zcash/zcash.conf` não contém `testnet=1` ou `addnode=betatestnet.z.cash`. Se você estiver usando uma distribuição baseada em Debian, siga as [instruções do Debian](https://github.com/zcash/zcash/wiki/Debian-binary-packages) para instalar a zcash em seu sistema. Caso contrário, você pode atualizar seu snapshot local do nosso código:

```
git fetch origin
git checkout v1.0.1
./zcutil/fetch-params.sh
./zcutil/build.sh -j$(nproc)
```

Certifique-se também de que seu diretório ``~/.zcash`` contém somente ``zcash.conf`` para começar.

## Uma nota rápida sobre a terminologia

Zcash suporta dois tipos diferentes de endereços, um _z-addr_ (que começa com um **z**) que é um endereço que usa provas de conhecimento-nulo e outra criptografia para proteger a privacidade do usuário. Existem também os _t-addrs_ (que começam com um **t**) que são semelhantes aos endereços do Bitcoin.

## Requisitos

Atualmente, você precisará:

* Linux (mais fácil com uma distribuição baseada em Debian)
* 64-bit
* 4GB de memória livre

As interfaces são um cliente de linha de comando (`zcash-cli`) e uma interface Remote Procedure Call (RPC), que está documentada aqui:

https://github.com/zcash/zcash/blob/v1.0.1/doc/payment-api.md

## Segurança

Antes de instalar, atualizar ou executar a zcash, certifique-se de verificar se há problemas de segurança. Consulte a nossa página Segurança:

https://z.cash/support/security.html

## Vamos começar

### Sistemas operacionais baseados em Debian

Siga as instruções aqui: https://github.com/zcash/zcash/wiki/Debian-binary-packages

### Compile você mesmo

Em sistemas baseados em Ubuntu/Debian:

```bash
$ sudo apt-get install \
      build-essential pkg-config libc6-dev m4 g++-multilib \
      autoconf libtool ncurses-dev unzip git python \
      zlib1g-dev wget bsdmainutils automake
```

Em sistemas baseados em Fedora:

```bash
$ sudo dnf install \
      git pkgconfig automake autoconf ncurses-devel python wget \
      gtest-devel gcc gcc-c++ libtool patch
```

Procure nosso repositório com o git e execute `fetch-params.sh` assim:

```bash
$ git clone https://github.com/zcash/zcash.git
$ cd zcash/
$ git checkout v1.0.1
$ ./zcutil/fetch-params.sh
```

Isto irá buscar as nossas chaves de provas e verificação do Sprout (as últimas criadas na [Cerimônia de Geração de Parâmetros] (https://github.com/zcash/mpc)), e colocá-las em `~/.zcash-params/`. Essas chaves têm pouco menos de 911MB de tamanho, portanto, pode levar algum tempo para fazer o download delas.

A mensagem impressa pelo ``git checkout`` sobre uma "cabeça destacada" é normal e não indica um problema.

#### Build

Certifique-se de ter instalado com êxito todas as dependências do pacote do seu sistema, conforme descrito acima. Em seguida, execute a compilação, por exemplo:

```bash
$ ./zcutil/build.sh -j$(nproc)
```

Isso deve compilar nossas dependências e montar o `zcashd`. (Nota: se você não tiver  `nproc`, então substitua o número de seus processadores.)

#### Testando

Os testes levam um tempo para ser executado e podem exigir até 8GB de RAM. Se você preferir começar imediatamente, você pode pular para a próxima seção. Se você quiser executar os testes para certificar-se de que a Zcash está funcionando, execute:

```bash
$ ./qa/zcash/full-test-suite.sh
```

Você também pode executar os testes RPC, que levam muito mais tempo:

```bash
$ ./qa/pull-tester/rpc-tests.sh
```

Os testes precisam de muita memória para serem executados com êxito. Um erro de falta de memória geralmente causará um resultado FAIL ou ERROR com "std::bad_alloc" em algum momento na saída.

## Configurando

Crie o diretório `~/.zcash` e coloque um arquivo de configuração em `~/.zcash/zcash.conf` usando os seguintes comandos:

```bash
mkdir -p ~/.zcash
echo "addnode=mainnet.z.cash" >~/.zcash/zcash.conf
echo "rpcuser=username" >>~/.zcash/zcash.conf
echo "rpcpassword=`head -c 32 /dev/urandom | base64`" >>~/.zcash/zcash.conf
```

Note que isso irá substituir quaisquer configurações do `zcash.conf` que você possa ter adicionado do testnet. Você pode manter um `zcash.conf` de testnet, mas certifique-se de que as configurações `testnet=1` e `addnode=betatestnet.z.cash` sejam removidas; use `addnode=mainnet.z.cash` em vez disso. É altamente recomendável que você use uma senha aleatória para evitar [potenciais problemas de segurança com acesso à interface RPC] (https://github.com/zcash/zcash/blob/master/doc/security-warnings.md#rpc-interface).

### Ativando a mineração por CPU:

Se você quiser ativar a mineração por CPU, execute estes comandos:

```bash
$ echo 'gen=1' >> ~/.zcash/zcash.conf
$ echo "genproclimit=$(nproc)" >> ~/.zcash/zcash.conf
```

O minerador padrão não é eficiente, mas tem sido bem revisto. Para usar um minerador muito mais eficiente, mas não revisado, você pode executar este comando:

```bash
$ echo 'equihashsolver=tromp' >> ~/.zcash/zcash.conf
```

Note, você provavelmente quer ler o [[Mining-Guide]] para aprender mais detalhes de mineração.

## Rodando a Zcash:

Agora, executar o zcashd!

```bash
$ ./src/zcashd
```

Para executá-lo em background (sem a tela de métricas de node exibida normalmente), use ``./src/zcashd --daemon``.

Você deve ser capaz de usar o RPC depois que ele termina de carregar. Aqui está uma maneira rápida de testar:

```bash
$ ./src/zcash-cli getinfo
```

**NOTA**: Se você estiver familiarizado com a interface RPC do bitcoind, você pode usar muitas dessas chamadas para enviar a ZEC entre endereços `t-addr`. Não suportamos o recurso 'Contas' (que também foi depreciado em ``bitcoind``) - somente a string vazia ``""`` pode ser usada como um nome de conta.

**NOTA**: O node da rede principal em mainnet.z.cash também é acessível através do serviço de anonimato Tor em zcmaintvsivr7pcn.onion.

Para ver os peers que você está conectado:

```bash
$ ./src/zcash-cli getpeerinfo
```

## Usando a Zcash

Primeiro, você deseja obter Zcash. Você pode comprá-los de uma exchange, de outros usuários, ou vender bens e serviços por eles! Como obter exatamente uma Zcash (com segurança) não está no escopo para este documento, mas você deve ter cuidado. Evite golpes!

### Gerando um t-addr

Vamos gerar o primeiro t-addr.

```bash
$ ./src/zcash-cli getnewaddress
tb4oHp2v54vfmdgQ3v3SNuQga8JKHTNi2a1
```

### Recebendo Zcash com um z-addr

Agora vamos gerar um z-addr.

```bash
$ ./src/zcash-cli z_getnewaddress
ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG
```

Isso cria um endereço particular e armazena sua chave em seu arquivo de carteira local. Dê este endereço ao remetente!

Um z-addr é bastante grande, por isso é fácil cometer erros com eles. Vamos colocá-lo em uma variável de ambiente para evitar erros:

```bash
$ ZADDR='ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG'
```

Para obter uma lista de todos os endereços da sua carteira para a qual tem uma chave de gastos, execute este comando:

```bash
$ ./src/zcash-cli z_listaddresses
```

Você deve ver algo como:
```json
[
    "zta6qngiR3U7HxYopyTWkaDLwYBd83D5MT7Jb9gpgTzPLMZytzRbtdPP1Syv4RvRgHeoZrJWSask3DyfwXG9DGPMWMvX7aC",
    "ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG"
]
```

Ótimo! Agora, envie seu z-addr para o remetente. Você deve eventualmente ver sua transação ao verificar:

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

### Enviando moedas com o seu z-addr

Se alguém lhe der o seu z-addr...

```bash
$ FRIEND='ztcDe8krwEt1ozWmGZhBDWrcUfmK3Ue5D5z1f6u2EZLLCjQq7mBRkaAPb45FUH4Tca91rF4R1vf983ukR71kHyXeED4quGV'
```

Você pode enviar 0.8 ZEC fazendo...

```bash
$ ./src/zcash-cli z_sendmany "$ZADDR" "[{\"amount\": 0.8, \"address\": \"$FRIEND\"}]"
```

Depois de esperar cerca de um minuto, você pode verificar se a operação foi concluída, produzindo o seguinte resultado:
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


## Problemas de Segurança Conhecidos

Cada versão contém um documento `./doc/security-warnings.md` descrevendo problemas de segurança conhecidos por afetar essa versão. Você pode encontrar a versão mais recente deste documento aqui:

https://github.com/zcash/zcash/blob/master/doc/security-warnings.md

Consulte também nossa página de segurança para notificações recentes e outros recursos:

https://z.cash/support/security.html
