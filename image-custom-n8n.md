## Passo a Passo: Criando o Dockerfile


### 1. Crie uma pasta para o seu projeto

Se ainda não tiver:

```
mkdir C:\n8n-custom
cd C:\n8n-custom
```

### 2. Vai abrir uma janela, cole o conteúdo abaixo:

```
# Começa com uma imagem Debian + Node
FROM node:18-bullseye

# Instala dependências necessárias
USER root
RUN apt-get update && \
    apt-get install -y python3-pip ffmpeg curl && \
    pip3 install yt-dlp

# Cria diretório para n8n e define como diretório de trabalho
RUN mkdir -p /home/node/.n8n
WORKDIR /home/node

# Instala o n8n
RUN npm install n8n -g

# Cria o usuário "node" (caso não exista) e define permissões
RUN chown -R node:node /home/node

# Usa o usuário "node"
USER node

# Porta padrão
EXPOSE 5678

# Comando para rodar o n8n
CMD ["n8n"]

```

Salve como Dockerfile (sem extensão!). Se ele salvar como Dockerfile.txt, renomeie no Explorer ou no terminal.

### 3. Abra o terminal nessa pasta e construa a imagem

```
docker build -t n8n-custom:yt-dlp .
```

### 4. Rode o container com essa imagem

```
docker run -d `
  --name n8n `
  -p 5678:5678 `
  -v C:\n8n-data:/home/node/.n8n `
  n8n-custom:yt-dlp
```
   
Isso cria um volume persistente em C:\n8n-data para salvar as configurações do n8n.
