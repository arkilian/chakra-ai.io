# Boas Práticas

Guia de melhores práticas para desenvolvimento, implementação e manutenção de sistemas com ChakraAgents.ai.

## Índice

- [Design de Agentes](#design-de-agentes)
- [Prompts Eficazes](#prompts-eficazes)
- [Gestão de Contexto](#gestão-de-contexto)
- [Segurança](#segurança)
- [Desempenho](#desempenho)
- [Monitorização](#monitorização)
- [Testes](#testes)
- [Deployment](#deployment)

## Design de Agentes

### 1. Princípio da Responsabilidade Única

Cada agente deve ter uma responsabilidade bem definida.

✅ **Bom:**
```python
# Agente focado em uma tarefa específica
agente_analise_sentimento = criar_agente(
    nome="Analisador de Sentimento",
    tarefa="Analisar sentimento de textos de clientes"
)
```

❌ **Mau:**
```python
# Agente com múltiplas responsabilidades não relacionadas
agente_tudo = criar_agente(
    nome="Agente Universal",
    tarefas=[
        "analisar sentimento",
        "gerar relatórios",
        "enviar emails",
        "processar pagamentos"
    ]
)
```

### 2. Nomeação Clara

Use nomes descritivos que indiquem função do agente.

✅ **Bom:**
- `AgenteTriage_Suporte_Tecnico`
- `Analisador_Documentos_Legais`
- `Gerador_Relatorios_Vendas`

❌ **Mau:**
- `Agente1`
- `Bot`
- `Helper`

### 3. Separação de Preocupações

Separe lógica de negócio, acesso a dados e apresentação.

```python
# Camada de Dados
class RepositorioProdutos:
    def buscar_produto(self, id: str):
        # Acesso à base de dados
        pass

# Camada de Lógica
class ServicoAgente:
    def __init__(self, repositorio):
        self.repositorio = repositorio
    
    def processar_consulta(self, consulta: str):
        # Lógica do agente
        pass

# Camada de Apresentação
class APIAgente:
    def __init__(self, servico):
        self.servico = servico
    
    def endpoint_consultar(self, request):
        # Interface REST
        pass
```

## Prompts Eficazes

### 1. Seja Específico e Claro

✅ **Bom:**
```python
prompt = """
Você é um assistente especializado em análise financeira.

Tarefa: Analisar demonstrações financeiras e identificar:
1. Tendências de receita (crescimento/declínio)
2. Indicadores de rentabilidade (margem bruta, líquida)
3. Saúde financeira (liquidez, endividamento)

Formato de resposta:
- Resumo executivo (max 3 parágrafos)
- Principais indicadores com valores
- Recomendações acionáveis

Demonstração financeira:
{dados_financeiros}
"""
```

❌ **Mau:**
```python
prompt = """
Analise isto: {dados}
"""
```

### 2. Fornecer Contexto

Inclua informação relevante para contexto.

```python
template = """
Contexto:
- Empresa: {nome_empresa}
- Setor: {setor}
- Período analisado: {periodo}
- Comparação com: {benchmark}

Dados históricos:
{historico}

Tarefa:
{tarefa_especifica}
"""
```

### 3. Usar Exemplos (Few-Shot)

Forneça exemplos do output esperado.

```python
prompt = """
Classifique o sentimento de reviews de clientes.

Exemplos:
Input: "Produto excelente, recomendo!"
Output: POSITIVO (confiança: 0.95)

Input: "Não gostei, demorou muito"
Output: NEGATIVO (confiança: 0.85)

Input: "É ok, nada de especial"
Output: NEUTRO (confiança: 0.70)

Agora classifique:
Input: "{review}"
Output:
"""
```

### 4. Definir Formato de Saída

Especifique exatamente como quer a resposta.

```python
prompt = """
Responda SEMPRE no seguinte formato JSON:

{
  "resposta": "texto da resposta",
  "confianca": 0.0-1.0,
  "fontes": ["fonte1", "fonte2"],
  "tags": ["tag1", "tag2"]
}

Pergunta: {pergunta}
"""
```

## Gestão de Contexto

### 1. Limitação de Tokens

Monitorize e limite uso de tokens.

```python
from langchain.memory import ConversationBufferWindowMemory

# Manter apenas últimas N mensagens
memoria = ConversationBufferWindowMemory(
    k=5,  # Últimas 5 interações
    return_messages=True
)
```

### 2. Resumir Histórico Longo

```python
from langchain.memory import ConversationSummaryMemory

# Resumir histórico automaticamente
memoria = ConversationSummaryMemory(
    llm=llm,
    max_token_limit=1000
)
```

### 3. Contexto por Sessão

Manter contexto separado por sessão.

```python
class GestorSessoes:
    def __init__(self):
        self.sessoes = {}
    
    def obter_ou_criar(self, session_id: str):
        if session_id not in self.sessoes:
            self.sessoes[session_id] = {
                "memoria": ConversationBufferMemory(),
                "contexto": {},
                "criado_em": datetime.now()
            }
        return self.sessoes[session_id]
    
    def limpar_antigas(self, max_idade_horas: int = 24):
        agora = datetime.now()
        self.sessoes = {
            sid: sessao
            for sid, sessao in self.sessoes.items()
            if (agora - sessao["criado_em"]).hours < max_idade_horas
        }
```

## Segurança

### 1. Validação de Input

Sempre validar e sanitizar inputs.

```python
from pydantic import BaseModel, validator
import re

class PerguntaValidada(BaseModel):
    texto: str
    session_id: str = None
    
    @validator('texto')
    def validar_texto(cls, v):
        # Remover caracteres perigosos
        v = re.sub(r'[<>]', '', v)
        
        # Limitar tamanho
        if len(v) > 1000:
            raise ValueError('Texto muito longo')
        
        # Verificar vazio
        if not v.strip():
            raise ValueError('Texto vazio')
        
        return v
```

### 2. Gestão Segura de Chaves

Nunca hardcode chaves API.

✅ **Bom:**
```python
import os
from dotenv import load_dotenv

load_dotenv()

openai_key = os.getenv('OPENAI_API_KEY')
if not openai_key:
    raise ValueError('OPENAI_API_KEY não configurada')
```

❌ **Mau:**
```python
openai_key = "sk-1234567890abcdef"  # NUNCA FAÇA ISTO!
```

### 3. Rate Limiting

Implementar limites de requisições.

```python
from fastapi import HTTPException
from datetime import datetime, timedelta
from collections import defaultdict

class RateLimiter:
    def __init__(self, max_requests: int = 100, window_seconds: int = 60):
        self.max_requests = max_requests
        self.window = timedelta(seconds=window_seconds)
        self.requests = defaultdict(list)
    
    def check(self, user_id: str):
        agora = datetime.now()
        
        # Remover requisições antigas
        self.requests[user_id] = [
            t for t in self.requests[user_id]
            if agora - t < self.window
        ]
        
        # Verificar limite
        if len(self.requests[user_id]) >= self.max_requests:
            raise HTTPException(
                status_code=429,
                detail="Rate limit excedido"
            )
        
        # Registar requisição
        self.requests[user_id].append(agora)
```

### 4. Sanitização de Outputs

Remover informação sensível dos outputs.

```python
import re

def sanitizar_output(texto: str) -> str:
    # Remover emails
    texto = re.sub(
        r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
        '[EMAIL_REMOVIDO]',
        texto
    )
    
    # Remover números de telefone
    texto = re.sub(
        r'\b\d{9,}\b',
        '[TELEFONE_REMOVIDO]',
        texto
    )
    
    # Remover chaves API
    texto = re.sub(
        r'sk-[a-zA-Z0-9]{20,}',
        '[API_KEY_REMOVIDA]',
        texto
    )
    
    return texto
```

## Desempenho

### 1. Cache de Respostas

Cachear respostas para perguntas comuns.

```python
from functools import lru_cache
import hashlib

class CacheAgente:
    def __init__(self):
        self.cache = {}
    
    def obter_cache_key(self, input: str, config: dict) -> str:
        # Criar chave única
        data = f"{input}:{str(config)}"
        return hashlib.md5(data.encode()).hexdigest()
    
    def obter(self, input: str, config: dict):
        key = self.obter_cache_key(input, config)
        return self.cache.get(key)
    
    def guardar(self, input: str, config: dict, output: str):
        key = self.obter_cache_key(input, config)
        self.cache[key] = {
            "output": output,
            "timestamp": datetime.now()
        }
    
    def limpar_antigos(self, max_idade_minutos: int = 60):
        agora = datetime.now()
        self.cache = {
            k: v for k, v in self.cache.items()
            if (agora - v["timestamp"]).seconds < max_idade_minutos * 60
        }
```

### 2. Processamento Assíncrono

Usar async para operações I/O.

```python
import asyncio
from typing import List

async def processar_multiplos(agente, inputs: List[str]):
    # Processar em paralelo
    tarefas = [
        processar_async(agente, input)
        for input in inputs
    ]
    return await asyncio.gather(*tarefas)

async def processar_async(agente, input: str):
    # Processamento assíncrono
    return await agente.arun(input)
```

### 3. Batch Processing

Processar múltiplos items em lote.

```python
def processar_lote(agente, inputs: List[str], batch_size: int = 10):
    resultados = []
    
    for i in range(0, len(inputs), batch_size):
        lote = inputs[i:i + batch_size]
        
        # Processar lote
        batch_results = asyncio.run(
            processar_multiplos(agente, lote)
        )
        
        resultados.extend(batch_results)
    
    return resultados
```

### 4. Escolher Modelo Apropriado

Use modelo mais simples quando possível.

```python
def escolher_modelo(complexidade: str) -> str:
    modelos = {
        "simples": "gpt-3.5-turbo",      # Rápido e barato
        "media": "gpt-3.5-turbo-16k",    # Contexto maior
        "complexa": "gpt-4",             # Mais capaz
        "especializada": "gpt-4-turbo"   # Mais recente
    }
    return modelos.get(complexidade, "gpt-3.5-turbo")
```

## Monitorização

### 1. Logging Estruturado

```python
import logging
import json

class LoggerAgente:
    def __init__(self, nome: str):
        self.logger = logging.getLogger(nome)
        self.logger.setLevel(logging.INFO)
        
        handler = logging.FileHandler('agentes.log')
        handler.setFormatter(
            logging.Formatter('%(message)s')
        )
        self.logger.addHandler(handler)
    
    def log_execucao(
        self,
        agent_id: str,
        input: str,
        output: str,
        tempo: float,
        sucesso: bool
    ):
        log_data = {
            "timestamp": datetime.now().isoformat(),
            "agent_id": agent_id,
            "input_length": len(input),
            "output_length": len(output),
            "execution_time": tempo,
            "success": sucesso
        }
        self.logger.info(json.dumps(log_data))
```

### 2. Métricas Customizadas

```python
from prometheus_client import Counter, Histogram, Gauge

# Definir métricas
execucoes_total = Counter(
    'agente_execucoes_total',
    'Número total de execuções',
    ['agent_id', 'status']
)

tempo_execucao = Histogram(
    'agente_tempo_execucao_segundos',
    'Tempo de execução',
    ['agent_id']
)

sessoes_ativas = Gauge(
    'agente_sessoes_ativas',
    'Número de sessões ativas'
)

# Usar
execucoes_total.labels(agent_id='abc', status='success').inc()
tempo_execucao.labels(agent_id='abc').observe(2.5)
```

### 3. Alertas

```python
class SistemaAlertas:
    def __init__(self, webhook_url: str):
        self.webhook_url = webhook_url
        self.thresholds = {
            "taxa_erro": 0.05,  # 5%
            "tempo_resposta": 5.0,  # 5 segundos
        }
    
    def verificar_metricas(self, metricas: dict):
        alertas = []
        
        if metricas["taxa_erro"] > self.thresholds["taxa_erro"]:
            alertas.append({
                "nivel": "CRITICO",
                "mensagem": f"Taxa de erro alta: {metricas['taxa_erro']*100:.1f}%"
            })
        
        if metricas["tempo_medio"] > self.thresholds["tempo_resposta"]:
            alertas.append({
                "nivel": "AVISO",
                "mensagem": f"Tempo de resposta alto: {metricas['tempo_medio']:.2f}s"
            })
        
        if alertas:
            self.enviar_alertas(alertas)
    
    def enviar_alertas(self, alertas: List[dict]):
        # Enviar para Slack, email, etc.
        pass
```

## Testes

### 1. Testes Unitários

```python
import pytest
from unittest.mock import Mock

def test_agente_responde_corretamente():
    # Arrange
    agente = criar_agente_teste()
    input = "Qual é o preço do produto X?"
    
    # Act
    output = agente.executar(input)
    
    # Assert
    assert "preço" in output.lower()
    assert "€" in output or "euro" in output.lower()

def test_agente_trata_input_invalido():
    agente = criar_agente_teste()
    
    with pytest.raises(ValueError):
        agente.executar("")
```

### 2. Testes de Integração

```python
def test_pipeline_completo():
    # Setup
    agente_triage = criar_agente_triage()
    agente_processador = criar_agente_processador()
    
    # Executar pipeline
    input = "Tenho um problema técnico"
    categoria = agente_triage.executar(input)
    resposta = agente_processador.executar(input, categoria)
    
    # Verificar
    assert categoria == "tecnico"
    assert len(resposta) > 0
```

### 3. Testes de Carga

```python
import asyncio
from time import time

async def teste_carga(agente, num_requests: int = 100):
    inicio = time()
    
    # Criar requisições
    tarefas = [
        agente.arun(f"Pergunta {i}")
        for i in range(num_requests)
    ]
    
    # Executar
    resultados = await asyncio.gather(*tarefas, return_exceptions=True)
    
    fim = time()
    
    # Análise
    sucessos = sum(1 for r in resultados if not isinstance(r, Exception))
    falhas = num_requests - sucessos
    tempo_total = fim - inicio
    
    print(f"Requests: {num_requests}")
    print(f"Sucessos: {sucessos}")
    print(f"Falhas: {falhas}")
    print(f"Tempo total: {tempo_total:.2f}s")
    print(f"RPS: {num_requests/tempo_total:.2f}")
```

## Deployment

### 1. Dockerfile

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Instalar dependências do sistema
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Copiar e instalar dependências Python
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código
COPY . .

# Variáveis de ambiente
ENV PYTHONUNBUFFERED=1

# Expor porta
EXPOSE 8000

# Comando de start
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 2. Docker Compose

```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/chakra
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:8000
    depends_on:
      - backend

  db:
    image: postgres:14
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=chakra
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### 3. CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: |
          pip install -r requirements.txt
          pytest tests/

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to production
        run: |
          # Deploy script
          ./deploy.sh
```

### 4. Configuração de Produção

```python
# config/production.py
import os

class ProductionConfig:
    # Segurança
    SECRET_KEY = os.getenv('SECRET_KEY')
    DEBUG = False
    
    # Base de dados
    SQLALCHEMY_DATABASE_URI = os.getenv('DATABASE_URL')
    SQLALCHEMY_POOL_SIZE = 20
    SQLALCHEMY_MAX_OVERFLOW = 40
    
    # Redis
    REDIS_URL = os.getenv('REDIS_URL')
    
    # Rate limiting
    RATELIMIT_ENABLED = True
    RATELIMIT_DEFAULT = "100/minute"
    
    # Logging
    LOG_LEVEL = "INFO"
    LOG_FILE = "/var/log/chakra/app.log"
    
    # Monitorização
    PROMETHEUS_ENABLED = True
    SENTRY_DSN = os.getenv('SENTRY_DSN')
```

## Checklist de Deploy

Antes de fazer deploy em produção:

- [ ] Testes passam (unitários, integração, carga)
- [ ] Variáveis de ambiente configuradas
- [ ] Chaves API seguras (não hardcoded)
- [ ] Logging configurado
- [ ] Monitorização ativa
- [ ] Alertas configurados
- [ ] Backup da base de dados configurado
- [ ] Rate limiting ativo
- [ ] SSL/TLS configurado
- [ ] Firewall configurado
- [ ] Documentação atualizada
- [ ] Rollback plan definido

## Recursos Adicionais

- [Documentação Completa](../README.md)
- [Exemplos de Código](../exemplos/)
- [Referência API](../api/REFERENCIA.md)
- [Guia de Resolução de Problemas](./TROUBLESHOOTING.md)
