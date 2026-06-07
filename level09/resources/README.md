# 𝕃𝕖𝕧𝕖𝕝 𝟘𝟡

## 🎯 Objetivo
O objetivo deste nível é recuperar a senha original descriptografando um arquivo de texto que foi ofuscado por um algoritmo de substituição customizado em um binário.

## 🔍 Análise da Vulnerabilidade

* **Tipo:** *Custom/Weak Cryptography* (Criptografia Fraca/Customizada) e *Reverse Engineering* (Engenharia Reversa).
* **Arquivo Alvo:** `level09` (Executável) e `token` (Arquivo ofuscado).
* **Comportamento:** O executável atua como um encriptador que aplica uma cifra de substituição simples. Ao analisar o comportamento dinâmico e o código descompilado, descobrimos que o algoritmo iterava sobre a string inserida e somava o índice atual (`i`) da posição do caractere ao seu valor na tabela ASCII.

    Exemplo: A string `aaaa` (índices 0, 1, 2, 3) se transformava em `abcd`, pois:
    * `a` (índice 0): valor ASCII + 0 = `a`
    * `a` (índice 1): valor ASCII + 1 = `b`
    * `a` (índice 2): valor ASCII + 2 = `c`

    Para resolver, bastava reverter a matemática: em vez de somar o índice, nosso script precisaria subtrair a posição do caractere para revelar o texto original do arquivo `token`.

## 💻 Passos para Exploração (Exploit)

1.  **Reconhecimento e Teste Dinâmico:**
    Identificamos os arquivos e testamos o binário com entradas padronizadas para observar a saída.
    ```bash
    ./level09 aaaa
    # Saída: abcd
    ```

2.  **Engenharia Reversa:**
    Transferimos os arquivos para o ambiente local via `scp` e utilizamos um descompilador para confirmar a lógica em C. A linha crítica do algoritmo original era:
    ```c
    putchar((int)*(char *)(local_120 + *(int *)(param_2 + 4)) + local_120);
    ```

3.  **Preparação do Exploit (Criação do Decryptor):**
    Criamos um script em Python (`decrypt.py`) no nosso ambiente local para abrir o arquivo `token` original baixado, ler os bytes e aplicar a operação inversa (subtração do índice com módulo 256).

4.  **Execução do Exploit:**
    Rodamos o script contra o arquivo `token`.
    ```bash
    python3 decrypt.py token_level09
    # Saída: Decrypted result: f3iji1ju5yuevaus41q1afiuq
    ```

## 📜 Scripts Utilizados
* `resources/decrypt.py`: Script em Python construído após a engenharia reversa. Ele lê o arquivo ofuscado (`token`) e aplica a operação matemática inversa (subtração do índice com módulo 256) para revelar a flag original em texto claro.

## 🚩 Solução / Flag
A lógica de criptografia foi revertida com sucesso, transformando os bytes ilegíveis do arquivo `token` de volta na flag original para acessar a conta `flag09`.
```bash
su flag09
# Inserir a senha decifrada: `f3iji1ju5yuevaus41q1afiuq`
getflag
```
Senha para o próximo nível: `f3iji1ju5yuevaus41q1afiuq``

## 🛡️ Prevenção (Como corrigir)
1. **Evitar "Roll your own crypto"**: Nunca se deve criar algoritmos de criptografia próprios ou baseados em cifras de substituição simples para proteger dados sensíveis. A segurança não deve depender do segredo do algoritmo (Security through obscurity).

2. **Utilizar Padrões da Indústria**: Para proteger dados em repouso, deve-se usar bibliotecas criptográficas consolidadas e testadas matematicamente (como AES e outras.), junto com um gerenciamento de chaves seguro.
