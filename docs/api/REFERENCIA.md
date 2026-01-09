# Referência Completa da API

Documentação detalhada de todos os endpoints da API REST do ChakraAgents.ai.

## Base URL

```
Desenvolvimento: http://localhost:8000/api/v1
Produção: https://api.chakraagents.ai/v1
```

## Autenticação

Todos os endpoints (exceto `/health` e `/auth/*`) requerem autenticação via JWT Bearer token.

### Obter Token

**POST** `/auth/login`

Autenticar e obter token de acesso.

**Request Body:**
```json
{
  "username": "string",
  "password": "string"
}
```

**Response:** `200 OK`
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

**Exemplo:**
```bash
curl -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "senha"}'
```

### Usar Token

Incluir token no header Authorization:

```bash
curl -X GET http://localhost:8000/api/v1/agents \
  -H "Authorization: Bearer {seu_token}"
```

## Endpoints de Agentes

### Listar Agentes

**GET** `/agents`

Lista todos os agentes disponíveis.

**Query Parameters:**
- `page` (opcional): Número da página (padrão: 1)
- `per_page` (opcional): Items por página (padrão: 20, máx: 100)
- `type` (opcional): Filtrar por tipo (`simple`, `supervisor`, `swarm`, `rag`)
- `status` (opcional): Filtrar por estado (`active`, `inactive`)

**Response:** `200 OK`
```json
{
  "agents": [
    {
      "id": "uuid",
      "name": "Assistente de Vendas",
      "description": "Agente para apoio a vendas",
      "type": "simple",
      "status": "active",
      "created_at": "2024-01-01T10:00:00Z",
      "updated_at": "2024-01-01T10:00:00Z",
      "config": {
        "llm": "gpt-3.5-turbo",
        "temperature": 0.7
      }
    }
  ],
  "total": 10,
  "page": 1,
  "per_page": 20
}
```

**Exemplo:**
```bash
curl -X GET "http://localhost:8000/api/v1/agents?type=simple&page=1" \
  -H "Authorization: Bearer {token}"
```

### Criar Agente

**POST** `/agents`

Cria um novo agente.

**Request Body:**
```json
{
  "name": "string (obrigatório)",
  "description": "string (opcional)",
  "type": "simple | supervisor | swarm | rag",
  "config": {
    "llm": "gpt-3.5-turbo | gpt-4 | claude-2",
    "temperature": 0.0-1.0,
    "max_tokens": 1-4000,
    "tools": ["tool_name"],
    "system_prompt": "string (opcional)"
  }
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "name": "Novo Agente",
  "type": "simple",
  "status": "active",
  "created_at": "2024-01-01T10:00:00Z",
  "config": {...}
}
```

**Exemplo:**
```bash
curl -X POST http://localhost:8000/api/v1/agents \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Assistente Técnico",
    "type": "simple",
    "config": {
      "llm": "gpt-4",
      "temperature": 0.3
    }
  }'
```

### Obter Agente

**GET** `/agents/{agent_id}`

Obtém detalhes de um agente específico.

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "name": "Assistente de Vendas",
  "description": "...",
  "type": "simple",
  "status": "active",
  "config": {...},
  "statistics": {
    "total_executions": 150,
    "success_rate": 0.95,
    "avg_response_time": 2.5
  },
  "created_at": "2024-01-01T10:00:00Z",
  "updated_at": "2024-01-01T10:00:00Z"
}
```

### Atualizar Agente

**PUT** `/agents/{agent_id}`

Atualiza configuração de um agente.

**Request Body:**
```json
{
  "name": "string (opcional)",
  "description": "string (opcional)",
  "status": "active | inactive (opcional)",
  "config": {
    "temperature": 0.7,
    "max_tokens": 500
  }
}
```

**Response:** `200 OK`
```json
{
  "id": "uuid",
  "name": "Assistente Atualizado",
  ...
}
```

### Eliminar Agente

**DELETE** `/agents/{agent_id}`

Elimina um agente permanentemente.

**Response:** `204 No Content`

## Execução de Agentes

### Executar Agente

**POST** `/agents/{agent_id}/execute`

Executa um agente com input específico.

**Request Body:**
```json
{
  "input": "string (obrigatório)",
  "context": {
    "session_id": "uuid (opcional)",
    "user_id": "string (opcional)",
    "metadata": {}
  },
  "options": {
    "stream": false,
    "return_intermediate_steps": false,
    "max_iterations": 10
  }
}
```

**Response:** `200 OK`
```json
{
  "execution_id": "uuid",
  "agent_id": "uuid",
  "output": "Resposta do agente...",
  "status": "completed",
  "execution_time": 2.5,
  "tokens_used": {
    "prompt": 150,
    "completion": 200,
    "total": 350
  },
  "intermediate_steps": [],
  "timestamp": "2024-01-01T10:00:00Z"
}
```

**Exemplo:**
```bash
curl -X POST http://localhost:8000/api/v1/agents/{agent_id}/execute \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "input": "Qual é o preço do produto X?",
    "context": {
      "session_id": "session-123"
    }
  }'
```

### Executar com Streaming

**POST** `/agents/{agent_id}/stream`

Executa agente com resposta em streaming (Server-Sent Events).

**Request Body:** (igual ao execute)

**Response:** `200 OK` (text/event-stream)
```
data: {"type": "start", "execution_id": "uuid"}

data: {"type": "token", "content": "Olá"}

data: {"type": "token", "content": ", "}

data: {"type": "token", "content": "como"}

data: {"type": "complete", "full_response": "Olá, como posso ajudar?"}
```

**Exemplo (JavaScript):**
```javascript
const eventSource = new EventSource(
  'http://localhost:8000/api/v1/agents/{agent_id}/stream',
  {
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  }
);

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log(data);
};
```

### Histórico de Execuções

**GET** `/agents/{agent_id}/executions`

Lista execuções anteriores do agente.

**Query Parameters:**
- `page`: Número da página
- `per_page`: Items por página
- `status`: Filtrar por estado (`completed`, `failed`, `running`)
- `start_date`: Data início (ISO 8601)
- `end_date`: Data fim (ISO 8601)

**Response:** `200 OK`
```json
{
  "executions": [
    {
      "execution_id": "uuid",
      "input": "...",
      "output": "...",
      "status": "completed",
      "execution_time": 2.5,
      "timestamp": "2024-01-01T10:00:00Z"
    }
  ],
  "total": 50,
  "page": 1,
  "per_page": 20
}
```

## Gestão de Documentos (RAG)

### Upload de Documentos

**POST** `/documents/upload`

Faz upload de documentos para indexação.

**Request:** `multipart/form-data`
- `files`: Ficheiros (PDF, TXT, MD, etc.)
- `collection`: Nome da coleção (opcional)
- `metadata`: JSON com metadados (opcional)

**Response:** `201 Created`
```json
{
  "documents": [
    {
      "id": "uuid",
      "filename": "manual.pdf",
      "size": 102400,
      "collection": "docs",
      "status": "processing",
      "uploaded_at": "2024-01-01T10:00:00Z"
    }
  ],
  "total_uploaded": 1
}
```

**Exemplo:**
```bash
curl -X POST http://localhost:8000/api/v1/documents/upload \
  -H "Authorization: Bearer {token}" \
  -F "files=@manual.pdf" \
  -F "collection=manuais"
```

### Listar Documentos

**GET** `/documents`

Lista documentos indexados.

**Query Parameters:**
- `collection`: Filtrar por coleção
- `status`: Filtrar por estado (`indexed`, `processing`, `failed`)

**Response:** `200 OK`
```json
{
  "documents": [
    {
      "id": "uuid",
      "filename": "manual.pdf",
      "collection": "docs",
      "status": "indexed",
      "chunks": 50,
      "size": 102400,
      "uploaded_at": "2024-01-01T10:00:00Z"
    }
  ],
  "total": 10
}
```

### Pesquisar Documentos

**POST** `/documents/search`

Pesquisa semântica em documentos.

**Request Body:**
```json
{
  "query": "string (obrigatório)",
  "collection": "string (opcional)",
  "limit": 10,
  "min_score": 0.7
}
```

**Response:** `200 OK`
```json
{
  "results": [
    {
      "document_id": "uuid",
      "chunk_id": "uuid",
      "content": "Trecho relevante do documento...",
      "score": 0.95,
      "metadata": {
        "filename": "manual.pdf",
        "page": 5
      }
    }
  ],
  "total": 5,
  "query_time": 0.15
}
```

### Eliminar Documento

**DELETE** `/documents/{document_id}`

Remove documento e seus embeddings.

**Response:** `204 No Content`

## Fluxos de Trabalho

### Criar Workflow

**POST** `/workflows`

Cria um novo fluxo de trabalho.

**Request Body:**
```json
{
  "name": "string",
  "description": "string",
  "steps": [
    {
      "id": "step1",
      "agent_id": "uuid",
      "input_mapping": {
        "input": "${workflow.input}"
      },
      "next": "step2"
    },
    {
      "id": "step2",
      "agent_id": "uuid",
      "input_mapping": {
        "input": "${step1.output}"
      }
    }
  ]
}
```

**Response:** `201 Created`

### Executar Workflow

**POST** `/workflows/{workflow_id}/execute`

Executa um fluxo de trabalho.

**Request Body:**
```json
{
  "input": "string ou object",
  "context": {}
}
```

**Response:** `200 OK`
```json
{
  "workflow_execution_id": "uuid",
  "status": "completed",
  "steps_executed": [
    {
      "step_id": "step1",
      "status": "completed",
      "output": "..."
    }
  ],
  "final_output": "...",
  "execution_time": 5.2
}
```

## Monitorização

### Estado de Saúde

**GET** `/health`

Verifica estado do sistema (não requer autenticação).

**Response:** `200 OK`
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "uptime": 86400,
  "services": {
    "database": "healthy",
    "vector_store": "healthy",
    "llm_providers": {
      "openai": "healthy",
      "anthropic": "healthy"
    }
  }
}
```

### Métricas do Sistema

**GET** `/metrics`

Obtém métricas gerais do sistema.

**Response:** `200 OK`
```json
{
  "agents": {
    "total": 25,
    "active": 20,
    "inactive": 5
  },
  "executions": {
    "total": 10000,
    "today": 150,
    "success_rate": 0.95,
    "avg_response_time": 2.5
  },
  "resources": {
    "documents_indexed": 500,
    "total_storage_mb": 1024
  },
  "costs": {
    "total_tokens_today": 50000,
    "estimated_cost_usd": 0.75
  }
}
```

### Métricas de Agente

**GET** `/metrics/agents/{agent_id}`

Métricas detalhadas de um agente.

**Query Parameters:**
- `start_date`: Data início
- `end_date`: Data fim
- `granularity`: `hour | day | week | month`

**Response:** `200 OK`
```json
{
  "agent_id": "uuid",
  "period": {
    "start": "2024-01-01T00:00:00Z",
    "end": "2024-01-31T23:59:59Z"
  },
  "metrics": {
    "total_executions": 500,
    "successful": 475,
    "failed": 25,
    "success_rate": 0.95,
    "avg_response_time": 2.5,
    "median_response_time": 2.0,
    "p95_response_time": 4.5,
    "total_tokens": 150000,
    "avg_tokens_per_execution": 300
  },
  "timeline": [
    {
      "timestamp": "2024-01-01T00:00:00Z",
      "executions": 20,
      "success_rate": 0.90
    }
  ]
}
```

## Gestão de Utilizadores

### Listar Utilizadores

**GET** `/users`

Lista utilizadores do sistema (apenas admin).

**Response:** `200 OK`
```json
{
  "users": [
    {
      "id": "uuid",
      "username": "user1",
      "email": "user@example.com",
      "role": "admin | user",
      "created_at": "2024-01-01T10:00:00Z",
      "last_login": "2024-01-15T14:30:00Z"
    }
  ]
}
```

### Criar Utilizador

**POST** `/users`

Cria novo utilizador (apenas admin).

**Request Body:**
```json
{
  "username": "string",
  "email": "string",
  "password": "string",
  "role": "admin | user"
}
```

**Response:** `201 Created`

### Atualizar Utilizador

**PUT** `/users/{user_id}`

Atualiza utilizador.

**Request Body:**
```json
{
  "email": "string (opcional)",
  "password": "string (opcional)",
  "role": "admin | user (opcional)"
}
```

**Response:** `200 OK`

## Webhooks

### Configurar Webhook

**POST** `/webhooks`

Configura webhook para eventos.

**Request Body:**
```json
{
  "url": "https://seu-servidor.com/webhook",
  "events": [
    "agent.execution.completed",
    "agent.execution.failed",
    "document.indexed"
  ],
  "secret": "string (opcional)"
}
```

**Response:** `201 Created`
```json
{
  "id": "uuid",
  "url": "...",
  "events": [...],
  "status": "active",
  "created_at": "2024-01-01T10:00:00Z"
}
```

### Formato de Eventos

Webhooks recebem POST com:

```json
{
  "event": "agent.execution.completed",
  "timestamp": "2024-01-01T10:00:00Z",
  "data": {
    "agent_id": "uuid",
    "execution_id": "uuid",
    "output": "...",
    ...
  }
}
```

## Códigos de Estado HTTP

- `200 OK`: Sucesso
- `201 Created`: Recurso criado
- `204 No Content`: Sucesso sem conteúdo
- `400 Bad Request`: Pedido inválido
- `401 Unauthorized`: Autenticação necessária
- `403 Forbidden`: Sem permissões
- `404 Not Found`: Recurso não encontrado
- `422 Unprocessable Entity`: Validação falhou
- `429 Too Many Requests`: Rate limit excedido
- `500 Internal Server Error`: Erro do servidor
- `503 Service Unavailable`: Serviço indisponível

## Tratamento de Erros

Todos os erros seguem este formato:

```json
{
  "error": {
    "code": "AGENT_NOT_FOUND",
    "message": "Agente com ID xyz não encontrado",
    "details": {
      "agent_id": "xyz"
    },
    "timestamp": "2024-01-01T10:00:00Z"
  }
}
```

### Códigos de Erro Comuns

- `INVALID_TOKEN`: Token inválido ou expirado
- `AGENT_NOT_FOUND`: Agente não encontrado
- `EXECUTION_FAILED`: Falha na execução
- `INVALID_INPUT`: Input inválido
- `RATE_LIMIT_EXCEEDED`: Limite de requisições excedido
- `LLM_ERROR`: Erro do fornecedor LLM
- `VECTOR_STORE_ERROR`: Erro do vector store

## Rate Limiting

- **Limite padrão**: 100 requisições/minuto
- **Burst**: 20 requisições/segundo
- **Headers de resposta**:
  ```
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 95
  X-RateLimit-Reset: 1640995200
  ```

Quando excedido:
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Limite de requisições excedido",
    "retry_after": 60
  }
}
```

## Paginação

Endpoints que retornam listas suportam paginação:

**Parâmetros:**
- `page`: Número da página (começa em 1)
- `per_page`: Items por página (padrão: 20, máx: 100)

**Resposta inclui:**
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 150,
    "total_pages": 8,
    "has_next": true,
    "has_prev": false
  }
}
```

## Exemplos Completos

### Criar e Executar Agente RAG

```python
import requests

API_URL = "http://localhost:8000/api/v1"
TOKEN = "seu_token"

headers = {
    "Authorization": f"Bearer {TOKEN}",
    "Content-Type": "application/json"
}

# 1. Upload documentos
files = {"files": open("manual.pdf", "rb")}
response = requests.post(
    f"{API_URL}/documents/upload",
    headers={"Authorization": f"Bearer {TOKEN}"},
    files=files,
    data={"collection": "manuais"}
)
print(f"Documento: {response.json()}")

# 2. Criar agente RAG
agente = {
    "name": "Assistente de Manuais",
    "type": "rag",
    "config": {
        "llm": "gpt-4",
        "temperature": 0.3,
        "collection": "manuais"
    }
}
response = requests.post(
    f"{API_URL}/agents",
    headers=headers,
    json=agente
)
agent_id = response.json()["id"]
print(f"Agente criado: {agent_id}")

# 3. Executar agente
pergunta = {
    "input": "Como configurar o sistema?",
    "context": {"session_id": "session-123"}
}
response = requests.post(
    f"{API_URL}/agents/{agent_id}/execute",
    headers=headers,
    json=pergunta
)
resultado = response.json()
print(f"Resposta: {resultado['output']}")

# 4. Ver métricas
response = requests.get(
    f"{API_URL}/metrics/agents/{agent_id}",
    headers=headers
)
print(f"Métricas: {response.json()}")
```

## SDKs Oficiais

### Python
```bash
pip install chakra-agents-sdk
```

```python
from chakra_agents import ChakraClient

client = ChakraClient(api_key="seu_token")

# Criar agente
agente = client.agents.create(
    name="Assistente",
    type="simple",
    config={"llm": "gpt-4"}
)

# Executar
resultado = agente.execute("Olá!")
print(resultado.output)
```

### JavaScript/TypeScript
```bash
npm install @chakra-agents/sdk
```

```typescript
import { ChakraClient } from '@chakra-agents/sdk';

const client = new ChakraClient({ apiKey: 'seu_token' });

// Criar agente
const agente = await client.agents.create({
  name: 'Assistente',
  type: 'simple',
  config: { llm: 'gpt-4' }
});

// Executar
const resultado = await agente.execute('Olá!');
console.log(resultado.output);
```

## Suporte

- 📚 [Documentação Completa](../README.md)
- 💬 [Discord](https://discord.gg/chakraagents)
- 🐛 [Reportar Bugs](https://github.com/sudsk/ChakraAgents.ai/issues)
- 📧 Email: api-support@chakraagents.ai
