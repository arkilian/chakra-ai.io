# ChakraAgents.ai - Documentação Completa

> **IA Agêntica como Serviço**  
> Plataforma open-source para construir, testar e implementar fluxos de trabalho de agentes de IA

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Características Principais](#características-principais)
- [Arquiteturas Suportadas](#arquiteturas-suportadas)
- [Requisitos do Sistema](#requisitos-do-sistema)
- [Instalação](#instalação)
- [Configuração](#configuração)
- [Arquitetura do Sistema](#arquitetura-do-sistema)
- [Guias de Utilização](#guias-de-utilização)
- [Exemplos Práticos](#exemplos-práticos)
- [Referência da API](#referência-da-api)
- [Resolução de Problemas](#resolução-de-problemas)
- [Contribuir](#contribuir)
- [Licença](#licença)

## 🎯 Visão Geral

**ChakraAgents.ai** é uma plataforma completa para desenvolvimento de sistemas de IA agêntica, permitindo criar agentes autónomos capazes de tomar decisões, raciocinar e executar tarefas complexas. A plataforma integra-se com os principais fornecedores de LLM (Large Language Models) e suporta múltiplas arquiteturas de agentes.

### O Que é IA Agêntica?

IA agêntica refere-se a sistemas de inteligência artificial capazes de:
- **Autonomia**: Tomar decisões de forma independente
- **Raciocínio**: Analisar problemas e planear soluções
- **Aprendizagem**: Adaptar-se com base em experiências
- **Colaboração**: Trabalhar com outros agentes para objetivos comuns

## ✨ Características Principais

### 1. **Editor Visual de Fluxos de Trabalho**
Interface intuitiva para desenhar e orquestrar fluxos de trabalho de agentes sem necessidade de programação extensa.

### 2. **Capacidades RAG (Retrieval-Augmented Generation)**
Conecte os seus agentes a:
- Bases de dados de documentos privados
- Repositórios de conhecimento empresarial
- Sistemas de armazenamento vetorial
- Fontes de dados personalizadas

### 3. **Múltiplas Arquiteturas de Agentes**
Suporte nativo para diferentes padrões arquiteturais:
- **Supervisor**: Arquitetura hierárquica
- **Swarm**: Colaboração peer-to-peer
- **RAG**: Agentes aumentados com conhecimento

### 4. **Implementação de API**
Transforme fluxos de trabalho de agentes em endpoints API robustos e escaláveis.

### 5. **Monitorização e Análise**
Acompanhe:
- Desempenho dos agentes
- Utilização de recursos
- Métricas de sucesso
- Custos operacionais

### 6. **Integração com Principais LLMs**
Suporte para:
- OpenAI (GPT-4, GPT-3.5, etc.)
- Anthropic (Claude)
- Google Vertex AI
- Modelos personalizados

## 🏗️ Arquiteturas Suportadas

### Arquitetura Supervisor

**Descrição**: Sistema hierárquico onde um agente supervisor coordena agentes trabalhadores especializados.

**Vantagens**:
- ✅ Controlo centralizado e previsível
- ✅ Gestão de contexto e memória simplificada
- ✅ Encaminhamento claro de tarefas
- ✅ Ideal para processos sequenciais

**Desvantagens**:
- ❌ Possível gargalo no supervisor
- ❌ Menos flexibilidade adaptativa
- ❌ Ponto único de falha

**Casos de Uso**:
- Triagem de suporte ao cliente
- Processamento de documentos com múltiplas etapas
- Fluxos de trabalho regulamentados
- Orquestração de tarefas complexas

```
┌─────────────┐
│  Supervisor │
└─────┬───────┘
      │
      ├─────────┬─────────┬─────────┐
      │         │         │         │
   ┌──▼──┐   ┌─▼───┐  ┌──▼──┐   ┌──▼──┐
   │Agent│   │Agent│  │Agent│   │Agent│
   │  1  │   │  2  │  │  3  │   │  4  │
   └─────┘   └─────┘  └─────┘   └─────┘
```

### Arquitetura Swarm

**Descrição**: Múltiplos agentes colaboram como pares, com coordenação descentralizada.

**Vantagens**:
- ✅ Alta tolerância a falhas
- ✅ Escalabilidade natural
- ✅ Comportamento emergente e adaptativo
- ✅ Distribuição eficiente de carga

**Desvantagens**:
- ❌ Rastreamento complexo de decisões
- ❌ Gestão de estado mais elaborada
- ❌ Possível necessidade de afinação

**Casos de Uso**:
- Sistemas multi-domínio de perguntas e respostas
- Análise colaborativa de dados
- Resolução de problemas complexos
- Cenários com tarefas ambíguas

```
   ┌──────┐     ┌──────┐
   │Agent │◄───►│Agent │
   │  1   │     │  2   │
   └───▲──┘     └──▲───┘
       │   ╲   ╱   │
       │    ╳ ╳    │
       │   ╱   ╲   │
   ┌───▼──┐     ┌──▼───┐
   │Agent │◄───►│Agent │
   │  3   │     │  4   │
   └──────┘     └──────┘
```

### Arquitetura RAG (Retrieval-Augmented Generation)

**Descrição**: Agentes aumentados com capacidade de recuperação e síntese de informação de fontes de conhecimento privadas.

**Características**:
- 🔍 Pesquisa semântica em documentos
- 📚 Integração com bases de conhecimento
- 🎯 Respostas baseadas em fontes verificáveis
- 🔄 Atualização dinâmica de conhecimento

**Componentes**:
1. **Vector Store**: Armazena embeddings de documentos
2. **Retriever**: Recupera documentos relevantes
3. **LLM**: Gera respostas baseadas no contexto
4. **Synthesizer**: Combina informações de múltiplas fontes

**Casos de Uso**:
- Assistentes de documentação empresarial
- Sistemas de conformidade e compliance
- Suporte técnico baseado em manuais
- Análise de grandes volumes de documentos

```
┌────────────┐
│   Query    │
└─────┬──────┘
      │
      ▼
┌─────────────┐      ┌────────────────┐
│  Retriever  │◄────►│  Vector Store  │
└─────┬───────┘      └────────────────┘
      │
      ▼
┌─────────────┐      ┌────────────────┐
│     LLM     │◄────►│   Documents    │
└─────┬───────┘      └────────────────┘
      │
      ▼
┌─────────────┐
│  Response   │
└─────────────┘
```

## 💻 Requisitos do Sistema

### Requisitos Mínimos

- **Python**: 3.10 ou superior
- **Node.js**: 16.x ou superior
- **PostgreSQL**: 12.x ou superior (para produção)
- **Memória RAM**: Mínimo 4GB (recomendado 8GB)
- **Espaço em Disco**: Mínimo 2GB

### Dependências Opcionais

- **Docker**: Para implementação containerizada
- **Redis**: Para cache e gestão de filas
- **Nginx**: Para proxy reverso em produção

## 🚀 Instalação

### 1. Clonar o Repositório

```bash
git clone https://github.com/sudsk/ChakraAgents.ai.git
cd ChakraAgents.ai
```

### 2. Configuração do Backend

```bash
# Navegar para o diretório backend
cd backend

# Criar ambiente virtual Python
python -m venv venv

# Ativar ambiente virtual
# No Linux/Mac:
source venv/bin/activate
# No Windows:
# venv\Scripts\activate

# Instalar dependências
pip install -r requirements.txt

# Copiar ficheiro de configuração
cp .env.example .env

# Editar .env com as suas configurações
nano .env  # ou use o seu editor preferido

# Executar migrações da base de dados
alembic upgrade head

# Iniciar servidor backend
uvicorn app.main:app --reload
```

O backend estará disponível em: `http://localhost:8000`

### 3. Configuração do Frontend

```bash
# Abrir nova janela de terminal e navegar para o diretório frontend
cd frontend

# Instalar dependências
npm install

# Copiar ficheiro de configuração
cp .env.example .env

# Editar .env com as suas configurações
nano .env

# Iniciar servidor de desenvolvimento
npm start
```

O frontend estará disponível em: `http://localhost:3000`

### 4. Verificação da Instalação

Aceda a `http://localhost:3000` no seu navegador. Deverá ver a interface do ChakraAgents.ai.

## ⚙️ Configuração

### Variáveis de Ambiente - Backend

Crie um ficheiro `.env` no diretório `backend/` com as seguintes configurações:

```env
# Base de Dados
POSTGRES_URI=postgresql://utilizador:palavra-passe@localhost:5432/chakraagents
DATABASE_URL=${POSTGRES_URI}

# Chaves API dos LLMs
OPENAI_API_KEY=sk-your-openai-api-key-here
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key-here
GOOGLE_APPLICATION_CREDENTIALS=/caminho/para/credenciais.json

# Vector Store (opcional)
VECTOR_STORE_TYPE=chroma  # ou 'faiss', 'pinecone', 'weaviate'
VECTOR_STORE_CONNECTION=http://localhost:8001

# CORS
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:8000

# Segurança
SECRET_KEY=gere-uma-chave-secreta-forte-aqui
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Configurações de Log
LOG_LEVEL=INFO
LOG_FILE=logs/app.log

# Redis (opcional)
REDIS_URL=redis://localhost:6379/0
```

### Variáveis de Ambiente - Frontend

Crie um ficheiro `.env` no diretório `frontend/` com as seguintes configurações:

```env
# URL da API Backend
REACT_APP_API_URL=http://localhost:8000

# Funcionalidades
REACT_APP_FEATURE_RAG=true
REACT_APP_FEATURE_SWARM=true
REACT_APP_FEATURE_SUPERVISOR=true

# Analytics (opcional)
REACT_APP_GA_TRACKING_ID=UA-XXXXXXXXX-X

# Ambiente
REACT_APP_ENV=development
```

### Configuração da Base de Dados

#### PostgreSQL

```bash
# Criar base de dados
createdb chakraagents

# Ou via psql
psql -U postgres
CREATE DATABASE chakraagents;
CREATE USER chakra_user WITH PASSWORD 'sua_palavra_passe';
GRANT ALL PRIVILEGES ON DATABASE chakraagents TO chakra_user;
```

### Configuração de Vector Store

#### ChromaDB (Recomendado para Desenvolvimento)

```bash
pip install chromadb
```

```python
# No seu código Python
import chromadb

client = chromadb.Client()
collection = client.create_collection("documentos")
```

#### FAISS (Alternativa Local)

```bash
pip install faiss-cpu  # ou faiss-gpu para GPU
```

#### Pinecone (Cloud)

```bash
pip install pinecone-client
```

## 🏛️ Arquitetura do Sistema

### Componentes Principais

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend (React)                      │
│                  Interface de Utilizador                 │
└─────────────────────┬───────────────────────────────────┘
                      │
                      │ HTTP/REST
                      │
┌─────────────────────▼───────────────────────────────────┐
│                  API Gateway                             │
│          (Autenticação, Rate Limiting)                   │
└─────────────────────┬───────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
┌───────▼────┐ ┌──────▼──────┐ ┌───▼──────────┐
│  Backend   │ │   Vector    │ │   LLM        │
│  (FastAPI) │ │   Store     │ │   Providers  │
└───────┬────┘ └─────────────┘ └──────────────┘
        │
        │
┌───────▼────────┐
│   PostgreSQL   │
│   (Database)   │
└────────────────┘
```

### Stack Tecnológico

**Frontend**:
- React 18.x
- Chakra UI (componentes de interface)
- Redux Toolkit (gestão de estado)
- Axios (cliente HTTP)
- React Router (navegação)

**Backend**:
- FastAPI (framework web)
- LangChain (orquestração de LLM)
- LangGraph (fluxos de trabalho de agentes)
- SQLAlchemy (ORM)
- Alembic (migrações)
- Pydantic (validação de dados)

**Base de Dados**:
- PostgreSQL (dados relacionais)
- Redis (cache, filas)

**Vector Stores**:
- ChromaDB
- FAISS
- Pinecone
- Weaviate

## 📚 Guias de Utilização

### Criar o Seu Primeiro Agente

#### 1. Agente Simples

```python
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI

# Inicializar LLM
llm = OpenAI(temperature=0)

# Definir ferramentas
ferramentas = [
    Tool(
        name="Pesquisa",
        func=lambda x: f"Resultado para: {x}",
        description="Útil para pesquisar informação"
    )
]

# Criar agente
agente = initialize_agent(
    ferramentas,
    llm,
    agent="zero-shot-react-description",
    verbose=True
)

# Executar
resposta = agente.run("Qual é a capital de Portugal?")
print(resposta)
```

#### 2. Agente com RAG

```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

# Carregar documentos
from langchain.document_loaders import TextLoader
loader = TextLoader("documentos/info.txt")
documentos = loader.load()

# Criar vector store
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(documentos, embeddings)

# Criar chain RAG
qa = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=vectorstore.as_retriever()
)

# Fazer pergunta
resposta = qa.run("Qual é a informação sobre o produto X?")
print(resposta)
```

#### 3. Arquitetura Supervisor

```python
from langgraph.graph import Graph
from langchain.llms import OpenAI

# Definir agentes trabalhadores
class AgenteAnalise:
    def executar(self, entrada):
        return "Análise completa"

class AgenteExecucao:
    def executar(self, entrada):
        return "Execução completa"

# Criar supervisor
class Supervisor:
    def __init__(self):
        self.agentes = {
            "analise": AgenteAnalise(),
            "execucao": AgenteExecucao()
        }
    
    def delegar(self, tarefa):
        # Lógica de delegação
        if "analisar" in tarefa.lower():
            return self.agentes["analise"].executar(tarefa)
        else:
            return self.agentes["execucao"].executar(tarefa)

# Usar
supervisor = Supervisor()
resultado = supervisor.delegar("Analisar dados de vendas")
```

#### 4. Arquitetura Swarm

```python
class AgenteSwarm:
    def __init__(self, nome, especialidade):
        self.nome = nome
        self.especialidade = especialidade
    
    def processar(self, tarefa):
        # Processar tarefa
        return f"{self.nome} processou: {tarefa}"
    
    def colaborar(self, outros_agentes, tarefa):
        # Colaborar com outros agentes
        resultados = [self.processar(tarefa)]
        for agente in outros_agentes:
            resultados.append(agente.processar(tarefa))
        return self.sintetizar(resultados)
    
    def sintetizar(self, resultados):
        return f"Síntese de {len(resultados)} contribuições"

# Criar swarm
agentes = [
    AgenteSwarm("Agente1", "Análise Financeira"),
    AgenteSwarm("Agente2", "Análise Técnica"),
    AgenteSwarm("Agente3", "Análise de Mercado")
]

# Colaborar
resultado = agentes[0].colaborar(agentes[1:], "Avaliar investimento")
```

### Implementar API

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class Pergunta(BaseModel):
    texto: str
    contexto: str = None

@app.post("/api/agente/perguntar")
async def perguntar(pergunta: Pergunta):
    try:
        # Processar com agente
        resposta = agente.run(pergunta.texto)
        return {"resposta": resposta, "sucesso": True}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/api/agente/estado")
async def obter_estado():
    return {
        "ativo": True,
        "modelo": "gpt-4",
        "versao": "1.0.0"
    }
```

## 💡 Exemplos Práticos

### Exemplo 1: Assistente de Documentação

```python
from langchain.document_loaders import DirectoryLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chains import ConversationalRetrievalChain
from langchain.llms import OpenAI

# Carregar documentos
loader = DirectoryLoader("./docs", glob="**/*.md")
documentos = loader.load()

# Dividir em chunks
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
chunks = text_splitter.split_documents(documentos)

# Criar vectorstore
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings)

# Criar chain conversacional
qa_chain = ConversationalRetrievalChain.from_llm(
    OpenAI(temperature=0),
    vectorstore.as_retriever(),
    return_source_documents=True
)

# Usar
chat_history = []
pergunta = "Como instalar a aplicação?"
resultado = qa_chain({"question": pergunta, "chat_history": chat_history})
print(resultado["answer"])
```

### Exemplo 2: Sistema de Triagem de Suporte

```python
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI
from langchain.memory import ConversationBufferMemory

class SistemaTriagem:
    def __init__(self):
        self.categorias = {
            "tecnico": "Equipa Técnica",
            "faturacao": "Equipa Financeira",
            "comercial": "Equipa de Vendas"
        }
    
    def categorizar(self, pergunta):
        # Lógica de categorização
        llm = OpenAI(temperature=0)
        prompt = f"""
        Categorize a seguinte pergunta em uma destas categorias:
        - tecnico
        - faturacao
        - comercial
        
        Pergunta: {pergunta}
        Categoria:
        """
        categoria = llm(prompt).strip().lower()
        return self.categorias.get(categoria, "Equipa Geral")
    
    def criar_ticket(self, pergunta, categoria):
        return {
            "id": "TICKET-001",
            "pergunta": pergunta,
            "equipa": categoria,
            "prioridade": "normal",
            "estado": "aberto"
        }

# Usar
sistema = SistemaTriagem()
pergunta = "O meu servidor não está a responder"
categoria = sistema.categorizar(pergunta)
ticket = sistema.criar_ticket(pergunta, categoria)
print(f"Ticket criado: {ticket}")
```

### Exemplo 3: Análise Colaborativa Multi-Agente

```python
from typing import List, Dict

class AgenteAnalitico:
    def __init__(self, nome: str, especialidade: str):
        self.nome = nome
        self.especialidade = especialidade
        self.llm = OpenAI(temperature=0.7)
    
    def analisar(self, dados: str) -> Dict:
        prompt = f"""
        Como especialista em {self.especialidade}, analise os seguintes dados:
        
        {dados}
        
        Forneça insights específicos da sua área de especialidade.
        """
        analise = self.llm(prompt)
        return {
            "agente": self.nome,
            "especialidade": self.especialidade,
            "analise": analise
        }

class Sintetizador:
    def __init__(self):
        self.llm = OpenAI(temperature=0.3)
    
    def sintetizar(self, analises: List[Dict]) -> str:
        texto_analises = "\n\n".join([
            f"{a['especialidade']}: {a['analise']}"
            for a in analises
        ])
        
        prompt = f"""
        Sintetize as seguintes análises numa conclusão coerente:
        
        {texto_analises}
        
        Conclusão:
        """
        return self.llm(prompt)

# Criar sistema
agentes = [
    AgenteAnalitico("Ana", "Análise Financeira"),
    AgenteAnalitico("Bruno", "Análise de Mercado"),
    AgenteAnalitico("Carlos", "Análise Técnica")
]

# Analisar
dados = "Dados da empresa XYZ para Q4 2023..."
analises = [agente.analisar(dados) for agente in agentes]

# Sintetizar
sintetizador = Sintetizador()
conclusao = sintetizador.sintetizar(analises)
print(conclusao)
```

## 🔌 Referência da API

### Endpoints Principais

#### Gestão de Agentes

**GET** `/api/v1/agents`
- Listar todos os agentes
- Resposta: `{ "agents": [...], "total": 10 }`

**POST** `/api/v1/agents`
- Criar novo agente
- Body: `{ "name": "...", "type": "...", "config": {...} }`

**GET** `/api/v1/agents/{agent_id}`
- Obter detalhes de um agente

**PUT** `/api/v1/agents/{agent_id}`
- Atualizar agente

**DELETE** `/api/v1/agents/{agent_id}`
- Eliminar agente

#### Execução de Agentes

**POST** `/api/v1/agents/{agent_id}/execute`
- Executar agente
- Body: `{ "input": "...", "context": {...} }`
- Resposta: `{ "output": "...", "metadata": {...} }`

**POST** `/api/v1/agents/{agent_id}/stream`
- Executar agente com streaming
- Retorna: Server-Sent Events (SSE)

#### Gestão de Fluxos de Trabalho

**GET** `/api/v1/workflows`
- Listar fluxos de trabalho

**POST** `/api/v1/workflows`
- Criar novo fluxo de trabalho

**POST** `/api/v1/workflows/{workflow_id}/execute`
- Executar fluxo de trabalho

#### Vector Store / RAG

**POST** `/api/v1/documents/upload`
- Fazer upload de documentos
- Body: `multipart/form-data`

**GET** `/api/v1/documents`
- Listar documentos indexados

**POST** `/api/v1/documents/search`
- Pesquisar documentos
- Body: `{ "query": "...", "limit": 10 }`

**DELETE** `/api/v1/documents/{doc_id}`
- Eliminar documento

#### Monitorização

**GET** `/api/v1/metrics`
- Obter métricas do sistema

**GET** `/api/v1/metrics/agents/{agent_id}`
- Métricas de um agente específico

**GET** `/api/v1/health`
- Estado de saúde do sistema

### Autenticação

Todos os endpoints (exceto `/health`) requerem autenticação via JWT:

```bash
# Obter token
curl -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "user", "password": "pass"}'

# Usar token
curl -X GET http://localhost:8000/api/v1/agents \
  -H "Authorization: Bearer {seu_token}"
```

### Exemplos de Utilização

```python
import requests

# Configuração
BASE_URL = "http://localhost:8000/api/v1"
TOKEN = "seu_token_jwt"

headers = {
    "Authorization": f"Bearer {TOKEN}",
    "Content-Type": "application/json"
}

# Criar agente
novo_agente = {
    "name": "Assistente de Vendas",
    "type": "supervisor",
    "config": {
        "llm": "gpt-4",
        "temperature": 0.7
    }
}

response = requests.post(
    f"{BASE_URL}/agents",
    json=novo_agente,
    headers=headers
)
agent_id = response.json()["id"]

# Executar agente
pergunta = {
    "input": "Quais são os produtos mais vendidos?",
    "context": {"periodo": "ultimo_mes"}
}

response = requests.post(
    f"{BASE_URL}/agents/{agent_id}/execute",
    json=pergunta,
    headers=headers
)
resultado = response.json()
print(resultado["output"])
```

## 🔧 Resolução de Problemas

### Problemas Comuns

#### 1. Erro de Conexão à Base de Dados

**Problema**: `connection refused` ou `database does not exist`

**Solução**:
```bash
# Verificar se PostgreSQL está a correr
sudo systemctl status postgresql

# Criar base de dados se não existir
createdb chakraagents

# Verificar configuração no .env
# Formato: postgresql://utilizador:senha@host:porta/basededados
```

#### 2. Erro de API Key Inválida

**Problema**: `Invalid API key` ou `401 Unauthorized`

**Solução**:
- Verificar se a chave está correcta no ficheiro `.env`
- Confirmar que a chave tem permissões adequadas
- Verificar se não há espaços em branco na chave

```bash
# Verificar variável de ambiente
echo $OPENAI_API_KEY
```

#### 3. Erro de Memória Insuficiente

**Problema**: `Out of memory` durante processamento de documentos

**Solução**:
```python
# Reduzir tamanho dos chunks
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,  # Reduzido de 1000
    chunk_overlap=50
)

# Processar documentos em lotes
for i in range(0, len(documentos), 10):
    lote = documentos[i:i+10]
    processar_lote(lote)
```

#### 4. Lentidão na Execução

**Problema**: Agentes demoram muito tempo a responder

**Soluções**:
```python
# 1. Usar modelo mais rápido
llm = OpenAI(model="gpt-3.5-turbo")  # em vez de gpt-4

# 2. Reduzir temperatura para respostas mais diretas
llm = OpenAI(temperature=0)

# 3. Implementar cache
from langchain.cache import InMemoryCache
langchain.llm_cache = InMemoryCache()

# 4. Limitar tokens de resposta
llm = OpenAI(max_tokens=500)
```

#### 5. Vector Store Não Encontra Documentos

**Problema**: Pesquisas não retornam resultados relevantes

**Solução**:
```python
# 1. Ajustar número de resultados
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 10}  # aumentar de 4 para 10
)

# 2. Usar pesquisa por similaridade com threshold
docs = vectorstore.similarity_search_with_score(
    pergunta,
    k=5,
    score_threshold=0.7
)

# 3. Reindexar documentos
vectorstore = Chroma.from_documents(
    documentos,
    embeddings,
    persist_directory="./chroma_db"
)
vectorstore.persist()
```

### Logs e Debug

#### Ativar Logs Detalhados

```python
import logging

# Configurar logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Logs de LangChain
from langchain.globals import set_debug
set_debug(True)
```

#### Ver Logs do Sistema

```bash
# Logs do backend
tail -f logs/app.log

# Logs do Docker (se usar)
docker-compose logs -f backend
```

## 🤝 Contribuir

Contribuições são bem-vindas! Siga estas diretrizes:

### Processo de Contribuição

1. **Fork** o repositório
2. **Clone** o seu fork
3. **Crie** um branch para a sua funcionalidade
4. **Implemente** as alterações
5. **Teste** as suas alterações
6. **Commit** com mensagens descritivas
7. **Push** para o seu fork
8. **Crie** um Pull Request

### Diretrizes de Código

```python
# Seguir PEP 8 para Python
# Usar type hints
def processar_documento(doc: str, opcoes: Dict[str, Any]) -> Dict:
    """
    Processa um documento.
    
    Args:
        doc: O documento a processar
        opcoes: Opções de processamento
        
    Returns:
        Resultado do processamento
    """
    pass

# Documentar funções e classes
# Escrever testes unitários
def test_processar_documento():
    resultado = processar_documento("teste", {})
    assert resultado is not None
```

### Executar Testes

```bash
# Backend
cd backend
pytest tests/ -v

# Frontend
cd frontend
npm test

# Cobertura
pytest --cov=app tests/
```

## 📄 Licença

Este projeto está licenciado sob a Licença MIT. Consulte o ficheiro `LICENSE` para mais detalhes.

---

## 📞 Suporte e Comunidade

- **GitHub Issues**: [Reportar problemas](https://github.com/sudsk/ChakraAgents.ai/issues)
- **Documentação**: [Wiki do projeto](https://github.com/sudsk/ChakraAgents.ai/wiki)
- **Discussões**: [GitHub Discussions](https://github.com/sudsk/ChakraAgents.ai/discussions)

## 🙏 Agradecimentos

Este projeto foi construído com base em:
- [LangChain](https://github.com/langchain-ai/langchain)
- [LangGraph](https://github.com/langchain-ai/langgraph)
- [FastAPI](https://fastapi.tiangolo.com/)
- [React](https://react.dev/)
- [Chakra UI](https://chakra-ui.com/)

---

**Desenvolvido com ❤️ pela comunidade ChakraAgents.ai**
