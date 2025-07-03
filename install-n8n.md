### PROMT

´´´
Preciso de orientações quanto a instalação de um sistema ubuntu.
Irei preparar um servidor para n8n, preciso inicialmente saber qual é a melhor forma para fazer o particionamento para um disco de 1024gb.

´´´
### Requisitos típicos do servidor n8n:
- Sistema de arquivos confiável.
- Espaço para arquivos temporários e logs.
- Isolamento do sistema, dados e logs.
- Possível uso de banco de dados externo (PostgreSQL, SQLite, etc).
- Docker (caso use containers).

### Sugestão de particionamento (Manual)
Vamos considerar LVM (opcional) e ext4 (ou xfs) como sistema de arquivos.

- Partição	Tamanho Sugerido	Montagem	Finalidade
- /boot	1 GB	/boot	Arquivos de boot do sistema
- swap	2 a 4 GB	swap	Memória de troca (depende da RAM)
- / (root)	30 GB	/	Sistema principal (binários, libs, etc)
- /var	20 a 50 GB	/var	Logs, cache, temporários (Docker usa aqui)
- /home	20 GB (opcional)	/home	Usuários, se necessário
- /opt	10 GB (opcional)	/opt	Softwares de terceiros
- /data	Todo restante	/data	Dados do n8n, banco de dados, etc

### Detalhes:

- /var: Docker e muitos serviços gravam dados temporários aqui. Deixe bastante espaço se for usar containers.
- /data: Monte onde você armazenará fluxos do n8n, backups, arquivos e banco de dados (caso local).
- swap: Não exagere — 2GB a 4GB é suficiente se você tiver bastante RAM (8GB+).
- LVM: Recomendado se quiser flexibilidade futura para redimensionar partições.

## Após instalação

Instale dependências:

´´´
sudo apt update && sudo apt install docker.io docker-compose curl -y
´´´

Instale n8n via Docker:

´´´
docker run -it --rm \
    -p 5678:5678 \
    -v /data/n8n:/home/node/.n8n \
    n8nio/n8n
´´´
Você pode ajustar /data/n8n para o caminho do seu disco real.


## Recomendação para instalar o n8n

- Diretório de dados persistentes
Use a partição /data para armazenar:

- Os dados do n8n
- O banco de dados, se for local (SQLite ou PostgreSQL)
- Backups

´´´
/data/n8n         ← dados da aplicação
/data/db          ← banco de dados (opcional)
´´´

Crie os diretórios com:

´´´
sudo mkdir -p /data/n8n /data/db
sudo chown -R $USER:$USER /data/n8n /data/db
´´´


## 🐳 Instalação com Docker + Docker Compose

1. Instalar Docker e Docker Compose

´´´
sudo apt update && sudo apt install -y docker.io docker-compose
sudo systemctl enable docker --now
´´´

2. Criar docker-compose.yml
Crie em /data/n8n/docker-compose.yml:

´´´
version: '3'

services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    volumes:
      - /data/n8n:/home/node/.n8n
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=senha_forte_aqui
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - TZ=America/Sao_Paulo
    restart: always
´´´

Altere senha_forte_aqui para uma senha real e segura.


3. Adicione o usuário ao grupo do docker

´´´
sudo usermod -aG docker $USER
´´´

Depois relogue ou reinicie a sessão para aplicar as permissões ao grupo.

Você pode verificar se funcionou com:

´´´
groups
´´´

Devei aparecer algo como:

´´´
pedrosilva : pedrosilva sudo docker
´´´

4. Subir o container

´´´
cd /data/n8n
docker-compose up -d
´´´

## Acessar o n8n
Acesse no navegador:

´´´
http://<ip-do-servidor>:5678
´´´

Use o login/senha definidos no docker-compose.yml.



´´´
´´´


´´´
´´´


´´´
´´´

