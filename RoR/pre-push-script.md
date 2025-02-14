# Script pre-push para Projeto Ruby on Rails
Este arquivo descreve o funcionamento do script `pre-push` utilizado em nosso projeto Ruby on Rails. O script é executado automaticamente sempre que um `git push` é realizado, e seu objetivo é garantir que o código esteja livre de erros e siga as boas práticas antes de ser enviado para o repositório remoto.

## Outputs
#### Success
![image](https://github.com/user-attachments/assets/30e49872-9cdb-40ba-8913-bed9d69642ce)
#### Lint failure fixed automatically
![image](https://github.com/user-attachments/assets/42989ee9-92dd-4a19-9aef-a91dfa9decda)
#### Test Fails
![image](https://github.com/user-attachments/assets/d31a8940-a45b-40d2-a93c-80009f5efbf8)


## Estrutura do Script

O script está localizado no diretório `scripts/git-hooks/` e possui o seguinte conteúdo:

```bash
#!/bin/sh
LOG_FILE="pre-push_output.log"
LOG_FILE_AFTER="pre-push_output.after.log"
LOG_FILE_BEFORE="pre-push_output.before.log"
LOG_FILE_TEST="pre-push_output.test.log"

IGNORE_WARNING="ruby: warning: Ruby was built without YJIT support"

DATE_TIME=$(date "+%d-%m-%Y %H:%M:%S")

echo "[$DATE_TIME] Pre-push iniciado" > "$LOG_FILE"
echo "🔍 Rodando testes e lint antes do push..." >> $LOG_FILE

# Rodar Rubocop (Lint) com auto-correção e registrar os erros corrigidos
echo "\n🔧 Executando Rubocop com autocorreção..." >> $LOG_FILE
bin/rubocop -f github --autocorrect 2>&1 | grep -v "$IGNORE_WARNING" | sed 's/::error/📌/g' >> $LOG_FILE_BEFORE

bin/rubocop -f github 2>&1 | grep -v "$IGNORE_WARNING" | sed 's/::error/📌/g' >> $LOG_FILE_AFTER

# Comparar antes e depois para exibir erros corrigidos, removendo "::error"
DIFF=$(diff "$LOG_FILE_BEFORE" "$LOG_FILE_AFTER" | sed 's/< //g')
# echo "$DIFF"

if [ -n "$DIFF" ]; then
  echo "🚀 Erros corrigidos automaticamente pelo Rubocop:" >> $LOG_FILE
  echo "$DIFF" >> $LOG_FILE
else
  echo "✅ Nenhum erro corrigido automaticamente.\n" >> $LOG_FILE
fi  

# Capturando se ainda há erros no Rubocop, alterando formatação para evitar detecção como erro
AFTER_FAILS=$(grep -c "file=" "$LOG_FILE_AFTER" 2>/dev/null)
AFTER_FAILS=${AFTER_FAILS:-0}  # Se estiver vazio, define como 0
AFTER_FAILS=$(echo "$AFTER_FAILS" | awk '{print $1}')
# echo "$AFTER_FAILS"
if [ "$AFTER_FAILS" -gt 0 ]; then
  echo "❌ Ainda existem erros no Rubocop. Corrija-os antes de commitar" | tee -a "$LOG_FILE"
  cat $LOG_FILE_AFTER | tee -a $LOG_FILE
  # Remover arquivos temporários
  rm -f $LOG_FILE_BEFORE $LOG_FILE_AFTER
  exit 1
else
  echo "✅ Nenhum erro no Rubocop.\n" >> $LOG_FILE
fi
# Remover arquivos temporários
rm -f $LOG_FILE_BEFORE $LOG_FILE_AFTER

# Rodar Testes
echo "🔧 Subindo o container e Executando os testes..." >> $LOG_FILE
docker compose up -d > /dev/null 2>&1 &
bin/rails db:test:prepare test test:system 2>&1 | grep -v "$IGNORE_WARNING" > $LOG_FILE_TEST

TEST_FAILS=$(grep -c -E "Failure:|Error:" "$LOG_FILE_TEST" 2>/dev/null)
TEST_FAILS=${TEST_FAILS:-0}  # Se estiver vazio, define como 0
TEST_FAILS=$(echo "$TEST_FAILS" | awk '{print $1}')
# echo "$TEST_FAILS"
# Capturando se houve falha nos testes
if [ "$TEST_FAILS" -gt 0 ]; then
  echo "❌ Falha nos testes. Corrija os erros antes de commitar!" | tee -a "$LOG_FILE"
  cat $LOG_FILE_TEST | tee -a $LOG_FILE
  # Remover arquivos temporários
  rm -f $LOG_FILE_TEST
  exit 1
else
  echo "✅ Nenhum erro nos Testes.\n" >> $LOG_FILE
fi
# Remover arquivos temporários
rm -f $LOG_FILE_TEST

# Se tudo estiver certo
echo "✅ Tudo certo! Commit permitido." | tee -a "$LOG_FILE"
exit 0
```

### Descrição das Etapas

1. **Execução do Rubocop**: O script inicia executando o Rubocop, que é uma ferramenta de lint para Ruby. O Rubocop é rodado com a opção `--autocorrect`, corrigindo automaticamente qualquer erro de formatação e registrando as mudanças.

2. **Comparação de Erros Corrigidos**: Após a execução do Rubocop com correção automática, o script compara os arquivos de log gerados antes e depois da execução para identificar erros corrigidos automaticamente.

3. **Verificação de Erros no Rubocop**: Se o Rubocop encontrar erros que não puderam ser corrigidos automaticamente, o script exibe esses erros e interrompe o processo de push, solicitando que o usuário corrija os problemas.

4. **Execução dos Testes**: Após a verificação do Rubocop, o script sobe o container Docker necessário e executa os testes do projeto (incluindo testes de sistema).

5. **Verificação de Falhas nos Testes**: Se algum teste falhar, o push é interrompido, e o script exibe as falhas para correção.

6. **Finalização**: Se não houver erros no Rubocop ou nos testes, o push é permitido, e o script conclui com uma mensagem de sucesso.

## Instalação e Configuração

### 1. Caminho do Script

O script `pre-push` deve ser colocado no diretório `scripts/git-hooks/` dentro do seu projeto Ruby on Rails. O caminho completo seria:

```
scripts/git-hooks/pre-push
```

### 2. Permissão de Execução

Após adicionar o script ao repositório, é necessário conceder permissões de execução ao arquivo para que ele possa ser executado como um script shell.

Para isso, execute o seguinte comando no terminal:

```bash
chmod +x scripts/git-hooks/pre-push
```

### 3. Criação do Link Simbólico

O Git busca os hooks na pasta `.git/hooks/` dentro do repositório. Para garantir que o Git utilize o script `pre-push`, é necessário criar um link simbólico do arquivo `pre-push` no diretório `.git/hooks/`.

Execute o seguinte comando para criar o link simbólico:

```bash
ln -s ../../scripts/git-hooks/pre-push .git/hooks/pre-push
```

Isso garante que o Git utilize o script sempre que um `git push` for realizado.

### 4. Arquivos Temporários

O script gera alguns arquivos de log temporários durante sua execução:

- `pre-push_output.log`
- `pre-push_output.after.log`
- `pre-push_output.before.log`
- `pre-push_output.test.log`

Esses arquivos são removidos automaticamente ao final da execução do script para evitar acúmulo de dados desnecessários.

## Considerações Finais

Este script `pre-push` ajuda a garantir que o código enviado para o repositório esteja livre de problemas de formatação e de falhas nos testes, automatizando a verificação de qualidade antes de cada `git push`.
