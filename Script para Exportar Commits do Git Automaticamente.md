# Configurando Script para Exportar Commits do Git Automaticamente

## 1. Instalar Dependências Necessárias
O script depende das seguintes ferramentas:
- `git` (para obter o histórico de commits)
- `trans` (para traduzir os commits)
- `zenity` (para exibir notificações)
- `notify-send` (para enviar notificações ao sistema)

Instale as dependências com:
```bash
sudo apt update && sudo apt install git translate-shell zenity libnotify-bin -y
```

## 2. Criar o Script Bash
Crie o arquivo do script:
```bash
touch ~/git_commits_by_date.sh
chmod +x ~/git_commits_by_date.sh
```
Abra o arquivo e adicione o seguinte conteúdo:
```bash
#!/bin/bash

# Função para determinar o último dia útil
if [ -z "$1" ]; then
  LAST_BUSINESS_DAY=$(date -d "yesterday" +%Y-%m-%d)
  if [[ $(date -d "$LAST_BUSINESS_DAY" +%u) -eq 6 ]]; then
    LAST_BUSINESS_DAY=$(date -d "2 days ago" +%Y-%m-%d)
  elif [[ $(date -d "$LAST_BUSINESS_DAY" +%u) -eq 7 ]]; then
    LAST_BUSINESS_DAY=$(date -d "3 days ago" +%Y-%m-%d)
  fi
  DATE="$LAST_BUSINESS_DAY"
else
  DATE="$1"
fi

FILE="~/git_project/commits-$DATE.txt"
cd ~/git_project

echo "Original" > $FILE
git log --no-merges --author="$(git config user.name)" --since="$DATE 00:00" --until="$DATE 23:59" --oneline --pretty=format:"%s" | tac | sed "s/^/- /" >> $FILE

echo -e "\n--------\nTranslated" >> $FILE
git log --no-merges --author="$(git config user.name)" --since="$DATE 00:00" --until="$DATE 23:59" --oneline --pretty=format:"%s" | tac | \
sed -e "s/\bfeat\b/Novo/" -e "s/\brefactor\b/Refatoração\/Melhoria/" | \
trans -b -s en -t pt | sed "s/^/- /" >> $FILE

# Notificação
notify-send "Commits exportados" "O arquivo $FILE foi gerado com sucesso."
zenity --question --text="O arquivo de commits foi gerado.\nDeseja abri-lo no VS Code?" --width=300
if [ $? -eq 0 ]; then
  /usr/bin/code "$FILE"
fi

echo "Commits exportados para $FILE"
```

## 3. Verificar o Display Atual
Antes de criar o serviço do Systemd, execute:
```bash
echo $DISPLAY
```
Anote o valor retornado, pois ele será necessário no arquivo do systemd.

## 4. Criar o Serviço do Systemd
Crie o arquivo de serviço:
```bash
sudo nano /etc/systemd/system/git_commits_by_date.service
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

## 5. Criar Alias no ZSH
Abra o arquivo de configuração:
```bash
nano ~/.zshrc
```
Adicione:
```bash
alias gitCommitsByDate="bash ~/git_commits_by_date.sh"
```
Salve e atualize o shell:
```bash
source ~/.zshrc
```

## 6. Habilitar e Testar o Serviço
```bash
sudo systemctl daemon-reload
sudo systemctl enable git_commits_by_date.service
sudo systemctl start git_commits_by_date.service
```

## 7. Executar o Script Manualmente
O script pode ser executado manualmente a qualquer momento com:
```bash
gitCommitsByDate
```
Ou passando uma data específica:
```bash
gitCommitsByDate 2024-02-12
```
Isso gerará o arquivo de commits do dia especificado.

---
Agora seu sistema exportará automaticamente os commits do último dia útil e enviará uma notificação, permitindo abrir o arquivo no VS Code! 🚀

