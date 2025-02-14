# Gerenciamento de Processos com `bin/manage_pids`

O script `bin/manage_pids` é utilizado para gerenciar os processos definidos no `Procfile.dev`. Ele permite visualizar os processos em execução, encerrar processos individuais ou finalizar todos os processos de uma vez.

## Motivação

A criação desse script foi motivada pelo fato de que, em algumas situações, ao matar o processo `bin/dev`, algumas portas permaneciam abertas ou em uso, impedindo a reinicialização correta dos serviços. Com esse script, é possível identificar esses processos ainda ativos e, se necessário, forçar a liberação de todas as portas e processos pendentes.

## Conteúdo do Script
```sh
#!/usr/bin/env sh

# Lista de processos do Procfile.dev
PROCESSES="bin/redis bin/pgsql bin/mail bin/web bin/css bin/localnetwork"

echo "📌 Processos rodando pelo Procfile.dev:"
for process in $PROCESSES; do
  PIDS=$(pgrep -f "$process")
  if [ -n "$PIDS" ]; then
    echo "$process -> ✅ PIDs: $(echo $PIDS | sed 's/ /, /g')"
  else
    echo "$process -> ❌ Não está rodando"
  fi
done

echo ""
echo "⚡ Opções:"
echo "1 - Matar um processo pelo PID"
echo "2 - Matar todos os processos do Procfile.dev"
echo "3 - Sair sem fazer nada"
read -p "Escolha uma opção [1-3]: " OPTION

case $OPTION in
  1)
    read -p "Digite o PID do processo que deseja matar: " PID
    if sudo kill -9 "$PID" 2>/dev/null; then
        echo "✅ Processo $PID encerrado com sudo."
    else
      echo "❌ Ainda falhou! Verifique se o PID está correto ou se você tem permissão."
    fi
    ;;
  2)
    echo "🛑 Matando todos os processos..."
    for process in $PROCESSES; do
      PIDS=$(pgrep -f "$process")
      if [ -n "$PIDS" ]; then
        sudo kill -9 $PIDS 2>/dev/null
        sudo pkill -f rails
        sudo pkill -f puma

        echo "✅ $process encerrado com sudo."
      fi
    done
    ;;
  3)
    echo "🔵 Nenhuma ação realizada."
    ;;
  *)
    echo "❌ Opção inválida!"
    ;;
esac
```

## Uso do Script

Para executar o script, utilize o seguinte comando na linha de comando:
```sh
bin/manage_pids
```

Ao rodar o script, ele:
1. Lista os processos rodando pelo `Procfile.dev`, exibindo seus respectivos PIDs.
2. Apresenta um menu com três opções:
   - **[1]** Matar um processo específico pelo PID.
   - **[2]** Matar todos os processos do `Procfile.dev`.
   - **[3]** Sair sem fazer nenhuma alteração.

## Funcionalidade

### Listagem de Processos
O script verifica os processos em execução com base na lista:
```sh
PROCESSES="bin/redis bin/pgsql bin/mail bin/web bin/css bin/localnetwork"
```
Para cada processo, ele busca os PIDs correspondentes e os exibe.

### Encerramento de Processos
- **Opção 1:** Permite ao usuário inserir manualmente um PID para ser encerrado com `kill -9`.
- **Opção 2:** Finaliza todos os processos listados no `Procfile.dev`, incluindo os processos do Rails e Puma com `pkill -f rails` e `pkill -f puma`.
- **Opção 3:** Sai do script sem realizar nenhuma ação.

## Exemplo de Saída do Script
```
📌 Processos rodando pelo Procfile.dev:
bin/redis -> ✅ PIDs: 1234
bin/pgsql -> ✅ PIDs: 5678
bin/mail -> ❌ Não está rodando
bin/web -> ✅ PIDs: 91011
bin/css -> ✅ PIDs: 1213
bin/localnetwork -> ✅ PIDs: 1415

⚡ Opções:
1 - Matar um processo pelo PID
2 - Matar todos os processos do Procfile.dev
3 - Sair sem fazer nada
Escolha uma opção [1-3]:
```

O script é útil para gerenciar processos do ambiente de desenvolvimento de forma prática e eficiente.

