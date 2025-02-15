# 🧵 Automação de Exportação de Commits por Data

Este guia descreve o processo de configuração de um script que gera um arquivo contendo os commits feitos pelo usuário em uma determinada data e oferece a opção de abri-lo no VS Code.

---

## 📌 1. Instalar Dependências Necessárias

Antes de iniciar, instale as ferramentas necessárias:

```sh
sudo apt update && sudo apt install -y zenity translate-shell
```

---

## 📌 2. Criar o Script de Exportação de Commits

Crie um arquivo chamado `git_commits_by_date.sh` e adicione o seguinte conteúdo:

```sh
#!/bin/bash

# Definir a data alvo (último dia útil, se não fornecida)
if [ -z "$1" ]; then
  LAST_BUSINESS_DAY=$(date -d "yesterday" +%Y-%m-%d)
  if [[ $(date -d "$LAST_BUSINESS_DAY" +%u) -eq 6 ]]; then
    LAST_BUSINESS_DAY=$(date -d "2 days ago" +%Y-%m-%d)
  elif [[ $(date -d "$LAST_BUSINESS_DAY" +%u) -eq 7 ]]; then
    LAST_BUSINESS_DAY=$(date -d "3 days ago" +%Y-%m-%d)
  fi
  SLEEPTIME=180 # 3 minutos 
  DATE="$LAST_BUSINESS_DAY"
else
  SLEEPTIME=5 # 5 segundos 
  DATE="$1"
fi

FILE="$HOME/git_project/commits-$DATE.txt"
cd "$HOME/git_project"

echo "Original" > "$FILE"
git log --no-merges --author="$(git config user.name)" --since="$DATE 00:00" --until="$DATE 23:59" --oneline --pretty=format:"%s" | tac | \
  sed -e "s/feat:/\n- feat:/" | \
  sed -e "s/refactor:/\n- refactor:/" | \
  sed -e "s/chore:/\n- chore:/" | \
  sed -e "s/style:/\n- style:/" | \
  sed -e "s/test:/\n- test:/" | \
  sed "/^$/d" >> $FILE

echo -e "\n--------\nTranslated" >> $FILE
git log --no-merges --author="$(git config user.name)" --since="$DATE 00:00" --until="$DATE 23:59" --oneline --pretty=format:"%s" | tac | \
  sed -e "s/feat:/\n- Novo:/" | \
  sed -e "s/refactor:/\n- Refatoração\/Melhoria:/" | \
  sed -e "s/chore:/\n- Config\/CORE:/" | \
  sed -e "s/style:/\n- Estilos:/" | \
  sed -e "s/test:/\n- Testes:/" | \
  sed "/^$/d" | \
  trans -b -s en -t pt >> $FILE

sleep $SLEEPTIME
echo "Commits exportados para $FILE"

# Exibir notificação
notify-send "Commits exportados" "O arquivo $FILE foi gerado com sucesso."

# Perguntar ao usuário se deseja abrir o arquivo no VS Code
zenity --info  --title="O arquivo de commits para o dia $DATE foi gerado." --text="Deseja abri-lo no VS Code?" --width=320
if [ $? -eq 0 ]; then
  /usr/bin/code "$FILE"
fi
```

Dê permissão de execução ao script:

```sh
chmod +x ~/git_commits_by_date.sh
```

---

## 📌 3. Criar um Alias para Execução Manual

Para rodar o script manualmente, edite o arquivo `~/.zshrc` (se usar ZSH) ou `~/.bashrc` (se usar Bash):

```sh
nano ~/.zshrc
```

Adicione:

```sh
alias gitCommitsByDate="bash ~/git_commits_by_date.sh"
```

Carregue as alterações:

```sh
source ~/.zshrc  # Ou source ~/.bashrc
```

Agora você pode rodar o script manualmente com:

```sh
gitCommitsByDate  # Usa a data do último dia útil
gitCommitsByDate 2025-02-13  # Para uma data específica
```

---

## 📌 4. Configurar Execução Automática Após Login

Você pode configurar o script para rodar automaticamente após o login do usuário de algumas maneiras:

---

### 🔹 **Opção 1: Usar systemd no login do usuário** (Recomendado)
O `systemd` permite rodar o script logo após o usuário fazer login, garantindo que o ambiente gráfico já esteja carregado.

#### **1️⃣ Verificar o Display Atual**
Antes de criar o serviço do Systemd, execute:
```bash
echo $DISPLAY
```
Anote o valor retornado, pois ele será necessário no arquivo do systemd.

#### **2️⃣ Criar o Serviço do Systemd do usuário**
Crie o arquivo de serviço:
```bash
sudo nano /etc/systemd/user/git_commits_by_date.service
```
Adicione o seguinte conteúdo (substitua `:1` pelo valor do `DISPLAY`):
```ini
[Unit]
Description=Executar script para commits
After=graphical.target

[Service]
Type=oneshot
User=$USER
Group=$USER
Environment=DISPLAY=:1
Environment=XDG_RUNTIME_DIR=/run/user/1000
ExecStart=/bin/bash /home/$USER/git_commits_by_date.sh

[Install]
WantedBy=default.target
```

#### **3️⃣ Habilite e inicie o serviço**
```sh
systemctl --user daemon-reexec
systemctl --user enable git_commits_by_date.service
systemctl --user start git_commits_by_date.service
```
> ⚠️ O `--user` faz com que o serviço rode apenas no login do usuário, sem precisar de `sudo`.

---

### 🔹 **Opção 2: Adicionar no `.profile` ou `.bashrc`**
Se preferir algo mais simples, adicione ao final do arquivo `~/.profile` ou `~/.bashrc`:
```sh
bash /home/$USER/gitCommitsByDate &
```
Isso executará o script sempre que o usuário fizer login no terminal. Porém, pode não funcionar bem para ambientes gráficos.

---

### 🔹 **Opção 3: Adicionar no "Startup Applications" (GUI)**
Se estiver usando GNOME, KDE ou outra interface gráfica, pode configurar na GUI:
1. Abra **"Aplicativos de Inicialização"** (Startup Applications).
2. Clique em **Adicionar**.
3. Preencha:
   - **Nome:** Git Commits Export
   - **Comando:** `/bin/bash /home/$USER/gitCommitsByDate`
   - **Comentário:** Exporta commits ao logar no sistema.
4. Salve e reinicie o sistema para testar.

---

### 🚀 Teste e escolha o melhor método para você!

