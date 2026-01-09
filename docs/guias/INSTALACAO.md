# Guia de Instalação Completo

Este guia fornece instruções detalhadas para instalar e configurar o ChakraAgents.ai em diferentes ambientes.

## Índice

- [Requisitos](#requisitos)
- [Instalação via Docker](#instalação-via-docker)
- [Instalação Manual](#instalação-manual)
- [Configuração](#configuração)
- [Verificação](#verificação)
- [Resolução de Problemas](#resolução-de-problemas)

## Requisitos

### Hardware Mínimo

- **CPU**: 2 cores (4 cores recomendado)
- **RAM**: 4GB (8GB recomendado)
- **Disco**: 10GB livres (SSD recomendado)
- **Internet**: Conexão estável para acesso aos LLMs

### Software

#### Requisitos Essenciais

- **Python**: 3.10, 3.11 ou 3.12
- **Node.js**: 16.x, 18.x ou 20.x
- **npm**: 8.x ou superior (incluído com Node.js)
- **PostgreSQL**: 12.x, 13.x, 14.x ou 15.x

#### Opcional (mas Recomendado)

- **Docker**: 20.x ou superior
- **Docker Compose**: 2.x ou superior
- **Git**: 2.x ou superior
- **Redis**: 6.x ou superior

### Sistemas Operativos Suportados

- ✅ Ubuntu 20.04 LTS / 22.04 LTS
- ✅ macOS 11+ (Big Sur ou superior)
- ✅ Windows 10/11 (com WSL2 recomendado)
- ✅ Debian 11+
- ✅ CentOS 8+ / RHEL 8+

## Instalação via Docker

A forma mais rápida de começar é usar Docker.

### 1. Instalar Docker

#### Ubuntu/Debian

```bash
# Atualizar pacotes
sudo apt update
sudo apt upgrade -y

# Instalar dependências
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Adicionar repositório Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Adicionar utilizador ao grupo docker
sudo usermod -aG docker $USER
newgrp docker
```

#### macOS

```bash
# Instalar Homebrew (se ainda não tiver)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Instalar Docker Desktop
brew install --cask docker

# Ou descarregar diretamente
# https://www.docker.com/products/docker-desktop
```

#### Windows

1. Descarregar Docker Desktop: https://www.docker.com/products/docker-desktop
2. Instalar WSL2: `wsl --install`
3. Executar instalador do Docker Desktop
4. Ativar integração com WSL2 nas definições

### 2. Clonar Repositório

```bash
# Via HTTPS
git clone https://github.com/sudsk/ChakraAgents.ai.git

# Ou via SSH
git clone git@github.com:sudsk/ChakraAgents.ai.git

# Entrar no diretório
cd ChakraAgents.ai
```

### 3. Configurar Variáveis de Ambiente

```bash
# Copiar ficheiros de exemplo
cp .env.example .env
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# Editar ficheiro .env principal
nano .env
```

Configurar no `.env`:

```env
# Chaves API
OPENAI_API_KEY=sk-your-openai-key-here
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key-here

# Base de Dados
POSTGRES_USER=chakra
POSTGRES_PASSWORD=senha_segura_aqui
POSTGRES_DB=chakraagents

# Segurança
SECRET_KEY=$(openssl rand -hex 32)
```

### 4. Iniciar Serviços

```bash
# Iniciar todos os serviços
docker-compose up -d

# Ver logs
docker-compose logs -f

# Verificar estado dos serviços
docker-compose ps
```

### 5. Aceder à Aplicação

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **Documentação API**: http://localhost:8000/docs

## Instalação Manual

Para mais controlo ou ambientes de desenvolvimento.

### 1. Instalar Python

#### Ubuntu/Debian

```bash
sudo apt update
sudo apt install -y python3.10 python3.10-venv python3-pip
```

#### macOS

```bash
brew install python@3.10
```

#### Windows

Descarregar de https://www.python.org/downloads/ e instalar.

### 2. Instalar Node.js

#### Ubuntu/Debian

```bash
# Instalar via NodeSource
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

#### macOS

```bash
brew install node@18
```

#### Windows

Descarregar de https://nodejs.org/ e instalar.

### 3. Instalar PostgreSQL

#### Ubuntu/Debian

```bash
sudo apt install -y postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

#### macOS

```bash
brew install postgresql@14
brew services start postgresql@14
```

#### Windows

Descarregar de https://www.postgresql.org/download/windows/

### 4. Configurar Base de Dados

```bash
# Aceder ao PostgreSQL
sudo -u postgres psql

# Executar comandos SQL
CREATE DATABASE chakraagents;
CREATE USER chakra_user WITH ENCRYPTED PASSWORD 'sua_senha';
GRANT ALL PRIVILEGES ON DATABASE chakraagents TO chakra_user;
\q
```

### 5. Clonar e Configurar Backend

```bash
# Clonar repositório
git clone https://github.com/sudsk/ChakraAgents.ai.git
cd ChakraAgents.ai/backend

# Criar ambiente virtual
python3 -m venv venv

# Ativar ambiente virtual
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate  # Windows

# Atualizar pip
pip install --upgrade pip

# Instalar dependências
pip install -r requirements.txt

# Copiar e configurar .env
cp .env.example .env
nano .env
```

Configurar `backend/.env`:

```env
DATABASE_URL=postgresql://chakra_user:sua_senha@localhost:5432/chakraagents
OPENAI_API_KEY=sk-your-key
SECRET_KEY=gerar-chave-forte
ALLOWED_ORIGINS=http://localhost:3000
```

```bash
# Executar migrações
alembic upgrade head

# Iniciar servidor
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### 6. Configurar Frontend

```bash
# Nova janela de terminal
cd ChakraAgents.ai/frontend

# Instalar dependências
npm install

# Copiar e configurar .env
cp .env.example .env
nano .env
```

Configurar `frontend/.env`:

```env
REACT_APP_API_URL=http://localhost:8000
REACT_APP_FEATURE_RAG=true
```

```bash
# Iniciar servidor de desenvolvimento
npm start
```

## Configuração Avançada

### Redis (Opcional - para Cache)

```bash
# Ubuntu/Debian
sudo apt install redis-server
sudo systemctl start redis
sudo systemctl enable redis

# macOS
brew install redis
brew services start redis
```

Adicionar ao `.env`:
```env
REDIS_URL=redis://localhost:6379/0
```

### Vector Store

#### ChromaDB (Recomendado para Desenvolvimento)

Já incluído nas dependências Python. Sem configuração adicional necessária.

#### FAISS

```bash
# Instalar FAISS
pip install faiss-cpu  # ou faiss-gpu para GPU
```

#### Pinecone (Cloud)

```bash
pip install pinecone-client
```

Adicionar ao `.env`:
```env
PINECONE_API_KEY=your-pinecone-key
PINECONE_ENVIRONMENT=us-west1-gcp
```

### Configuração de Produção

#### 1. Usar Gunicorn para Backend

```bash
pip install gunicorn

# Iniciar com Gunicorn
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

#### 2. Configurar Nginx

```nginx
# /etc/nginx/sites-available/chakraagents
server {
    listen 80;
    server_name seu-dominio.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /api {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
# Ativar site
sudo ln -s /etc/nginx/sites-available/chakraagents /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

#### 3. Configurar SSL com Let's Encrypt

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d seu-dominio.com
```

#### 4. Configurar Systemd Services

Backend (`/etc/systemd/system/chakra-backend.service`):

```ini
[Unit]
Description=ChakraAgents Backend
After=network.target postgresql.service

[Service]
Type=notify
User=chakra
WorkingDirectory=/opt/ChakraAgents.ai/backend
Environment="PATH=/opt/ChakraAgents.ai/backend/venv/bin"
ExecStart=/opt/ChakraAgents.ai/backend/venv/bin/gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start chakra-backend
sudo systemctl enable chakra-backend
```

## Verificação

### 1. Verificar Backend

```bash
# Testar endpoint de saúde
curl http://localhost:8000/api/v1/health

# Resposta esperada:
# {"status": "healthy", "version": "1.0.0"}
```

### 2. Verificar Frontend

Aceda a http://localhost:3000 no navegador. Deverá ver a página inicial do ChakraAgents.ai.

### 3. Verificar Base de Dados

```bash
psql -U chakra_user -d chakraagents -c "SELECT version();"
```

### 4. Testar Criação de Agente

```bash
# Obter token de autenticação
TOKEN=$(curl -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}' | jq -r '.access_token')

# Criar agente de teste
curl -X POST http://localhost:8000/api/v1/agents \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Agente de Teste",
    "type": "simple",
    "config": {"llm": "gpt-3.5-turbo"}
  }'
```

## Resolução de Problemas

### Problema: Porta já em uso

```bash
# Verificar o que está a usar a porta
sudo lsof -i :8000
sudo lsof -i :3000

# Terminar processo
kill -9 <PID>

# Ou usar porta diferente
uvicorn app.main:app --port 8001
npm start -- --port 3001
```

### Problema: Erro ao conectar à base de dados

```bash
# Verificar se PostgreSQL está a correr
sudo systemctl status postgresql

# Verificar logs
sudo tail -f /var/log/postgresql/postgresql-*.log

# Testar conexão
psql -U chakra_user -d chakraagents -h localhost
```

### Problema: Módulo Python não encontrado

```bash
# Verificar se ambiente virtual está ativo
which python

# Reinstalar dependências
pip install -r requirements.txt --force-reinstall
```

### Problema: npm install falha

```bash
# Limpar cache
npm cache clean --force
rm -rf node_modules package-lock.json

# Reinstalar
npm install
```

### Logs e Debugging

```bash
# Logs do Backend
tail -f logs/app.log

# Logs do Frontend
# Ver consola do navegador ou terminal onde npm start está a correr

# Logs do PostgreSQL
sudo tail -f /var/log/postgresql/postgresql-*.log

# Logs do Docker
docker-compose logs -f [nome_servico]
```

## Próximos Passos

Após instalação bem-sucedida:

1. 🎓 [Criar o Primeiro Agente](../tutoriais/PRIMEIRO_AGENTE.md)
2. 🏗️ [Explorar Arquiteturas](./ARQUITETURAS.md)
3. 💡 [Ver Exemplos Práticos](../exemplos/)
4. 🔧 [Configurar Integrações](./INTEGRACOES.md)

## Suporte

- 📚 [Documentação Completa](../README.md)
- 💬 [Comunidade Discord](https://discord.gg/chakraagents)
- 🐛 [Reportar Problemas](https://github.com/sudsk/ChakraAgents.ai/issues)
