# 📱 Script `bin/localnetwork` para Ruby on Rails
O script `bin/localnetwork` tem como função exibir o endereço IP local da máquina que pode ser acessado pela rede local. Ele também garante que o processo continue rodando para evitar que o Foreman o finalize.

### Conteúdo do `bin/localnetwork`:
```sh
#!/usr/bin/env sh

# Obtém o IP local que começa com 192.168
IP=$(hostname -I | tr ' ' '\n' | grep '^192\.168' | head -n 1)

# Exibe o IP na rede local
if [ -n "$IP" ]; then
  echo "Servidor disponível na sua rede local em: http://$IP:3000"
else
  echo "Nenhum IP local encontrado na faixa 192.168.x.x"
fi

# Mantém o processo rodando para que o Foreman não o mate
while true; do sleep 3600; done
```

### Explicação do funcionamento:
1. O script tenta obter o IP local da máquina dentro da faixa `192.168.x.x`.
2. Se um IP é encontrado, ele exibe uma mensagem indicando o endereço para acesso.
3. Se não houver um IP na faixa `192.168.x.x`, uma mensagem de erro é exibida.
4. Um loop infinito (`while true; do sleep 3600; done`) impede que o script seja finalizado, garantindo que o Foreman o mantenha ativo.

### Utilização no `Procfile.dev`
Para utilizar o script `localnetwork`, ele deve ser incluído no `Procfile.dev` da seguinte maneira:
```plaintext
localnetwork: bin/wait-for-it localhost:3000 -- bin/localnetwork
```
Isso garante que o script só será executado após o servidor web estar disponível na porta 3000.

