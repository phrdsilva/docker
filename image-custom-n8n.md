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
    apt-get install -y --no-install-recommends \
    python3-pip \
    ffmpeg \
    curl \
    imagemagick \
    fonts-noto \
    fonts-noto-cjk \
    fonts-noto-extra \
    fonts-noto-unhinted \
    fonts-noto-ui-core \
    fonts-noto-ui-extra && \
    pip3 install yt-dlp && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Confirma a presença da fonte (apenas log para depuração, opcional)
RUN find /usr/share/fonts -iname "NotoSerif_Condensed-BlackItalic.ttf" || echo "Fonte não encontrada"

# Cria diretório para n8n e define como diretório de trabalho
RUN mkdir -p /home/node/.n8n
WORKDIR /home/node

# Instala o n8n
RUN npm install n8n -g

# Ajusta permissões
RUN chown -R node:node /home/node

# Usa o usuário "node"
USER node

# Porta padrão
EXPOSE 5678

# Comando para rodar o n8n
CMD ["n8n"]

```

Salve como Dockerfile (sem extensão!). Se ele salvar como Dockerfile.txt, renomeie no Explorer ou no terminal.

### Observações Importantes

A fonte NotoSerif_Condensed-BlackItalic.ttf está presente no pacote fonts-noto-extra, mas às vezes ela fica com um caminho diferente. Um exemplo comum:

```
/usr/share/fonts/opentype/noto/NotoSerif-CondensedBlackItalic.ttf
```

No comando do convert, use o caminho correto:

```
-font "/usr/share/fonts/opentype/noto/NotoSerif-CondensedBlackItalic.ttf"
```
Ou valide dentro do container com:

```
docker exec -it n8n bash
find /usr/share/fonts -iname "*CondensedBlackItalic*"
```

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
   
### 5. Isso cria um volume persistente em C:\n8n-data para salvar as configurações do n8n.

Depois disso, entre no container e teste:

```
docker exec -it n8n bash
yt-dlp --version
```
Se aparecer a versão do yt-dlp, está tudo funcionando!


# Docker-Compose

Usar o docker-compose é uma maneira excelente de simplificar a inicialização e manutenção do seu ambiente. Vamos construir isso passo a passo.

✅ O que é o docker-compose.yml?

É um arquivo onde você define os serviços do seu projeto (como o n8n), volumes, portas, e variáveis de ambiente — tudo em um só lugar. Depois, você só precisa de um único comando para subir tudo.

Estrutura do projeto
Vamos organizar assim:

```
C:\n8n-custom\
├── Dockerfile           ✅ (você já tem)
├── docker-compose.yml   ✅ (vamos criar agora)
```

### 1. Crie o docker-compose.yml

Dentro da pasta C:\n8n-custom, execute no PowerShell:

```
notepad docker-compose.yml
```

Cole o seguinte conteúdo:

```
services:
  n8n:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: n8n
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
    restart: always

volumes:
  n8n_data:

```

Esse arquivo faz o seguinte:

- Constrói a imagem usando seu Dockerfile
- Mapeia a porta 5678, que é a interface web do n8n
- Cria um volume persistente para guardar dados do n8n
- Reinicia automaticamente se o container for parado ou se o sistema reiniciar

### 2. Subir o container com Docker Compose

Agora, no terminal (PowerShell ou CMD), dentro da pasta C:\n8n-custom:

```
docker-compose up -d
```

Ele vai:

- Construir sua imagem personalizada
- Criar e iniciar o container
- Gerenciar tudo automaticamente

### 3. Verificar se está funcionando

Abra no navegador:

```
http://localhost:5678
```

Você deve ver o n8n rodando.
Para testar se o yt-dlp está instalado:

```
docker exec -it n8n bash
yt-dlp --version
```

### 4. Dicas extras

Para parar tudo:

```
docker-compose down
```

Para reiniciar:

```
docker-compose restart
```

Para ver os logs ao vivo:

```
docker-compose logs -f
```


Reconstrua e reinicie

```
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```
