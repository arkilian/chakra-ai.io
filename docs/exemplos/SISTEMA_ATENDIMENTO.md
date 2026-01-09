# Exemplo: Sistema Completo de Atendimento ao Cliente

Este exemplo demonstra como construir um sistema completo de atendimento ao cliente usando ChakraAgents.ai com arquitetura Supervisor + RAG.

## 📋 Visão Geral

Vamos criar um sistema que:
- Recebe perguntas de clientes
- Classifica e encaminha para agente especializado
- Usa RAG para consultar base de conhecimento
- Gera respostas personalizadas
- Regista interações para análise

## 🏗️ Arquitetura

```
Cliente
   ↓
API Gateway
   ↓
Supervisor
   ├─→ Agente Triagem
   ├─→ Agente Técnico (+ RAG)
   ├─→ Agente Comercial (+ RAG)
   └─→ Agente Financeiro (+ RAG)
   ↓
Resposta + Métricas
```

## 📁 Estrutura do Projeto

```
atendimento-cliente/
├── agentes/
│   ├── __init__.py
│   ├── supervisor.py
│   ├── triagem.py
│   ├── tecnico.py
│   ├── comercial.py
│   └── financeiro.py
├── rag/
│   ├── __init__.py
│   ├── sistema_rag.py
│   └── documentos/
│       ├── manuais_tecnicos/
│       ├── catalogo_produtos/
│       └── politicas_financeiras/
├── api/
│   ├── __init__.py
│   ├── main.py
│   └── rotas.py
├── utils/
│   ├── __init__.py
│   ├── logger.py
│   └── metricas.py
├── tests/
│   └── ...
├── .env.example
├── requirements.txt
└── README.md
```

## 🔧 Implementação

### 1. Configuração (requirements.txt)

```txt
langchain>=0.1.0
openai>=1.0.0
chromadb>=0.4.0
fastapi>=0.104.0
uvicorn>=0.24.0
python-dotenv>=1.0.0
pydantic>=2.0.0
sqlalchemy>=2.0.0
alembic>=1.12.0
redis>=5.0.0
prometheus-client>=0.19.0
```

### 2. Sistema RAG (rag/sistema_rag.py)

```python
from langchain.document_loaders import DirectoryLoader, TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI
from typing import Dict, List
import os

class SistemaRAGEspecializado:
    """Sistema RAG para área específica de atendimento"""
    
    def __init__(
        self,
        area: str,
        diretorio_documentos: str,
        persist_directory: str = None
    ):
        self.area = area
        self.diretorio_documentos = diretorio_documentos
        self.persist_directory = persist_directory or f"./chroma_{area}"
        
        # Inicializar componentes
        self.embeddings = OpenAIEmbeddings()
        self.llm = OpenAI(temperature=0)
        
        # Carregar ou criar vectorstore
        if os.path.exists(self.persist_directory):
            self.vectorstore = Chroma(
                persist_directory=self.persist_directory,
                embedding_function=self.embeddings
            )
            print(f"✅ Vector store de {area} carregado")
        else:
            self._criar_vectorstore()
    
    def _criar_vectorstore(self):
        """Criar vector store a partir de documentos"""
        print(f"📚 Carregando documentos de {self.area}...")
        
        # Carregar documentos
        loader = DirectoryLoader(
            self.diretorio_documentos,
            glob="**/*.{txt,md}",
            loader_cls=TextLoader
        )
        documentos = loader.load()
        
        print(f"   {len(documentos)} documentos carregados")
        
        # Dividir em chunks
        text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200
        )
        chunks = text_splitter.split_documents(documentos)
        
        print(f"   {len(chunks)} chunks criados")
        
        # Criar vectorstore
        self.vectorstore = Chroma.from_documents(
            chunks,
            self.embeddings,
            persist_directory=self.persist_directory
        )
        
        self.vectorstore.persist()
        print(f"✅ Vector store de {self.area} criado")
    
    def consultar(self, pergunta: str, k: int = 3) -> Dict:
        """Consultar base de conhecimento"""
        
        # Criar retrieval QA chain
        qa_chain = RetrievalQA.from_chain_type(
            llm=self.llm,
            chain_type="stuff",
            retriever=self.vectorstore.as_retriever(
                search_kwargs={"k": k}
            ),
            return_source_documents=True
        )
        
        # Executar consulta
        resultado = qa_chain({"query": pergunta})
        
        return {
            "resposta": resultado["result"],
            "fontes": [
                {
                    "conteudo": doc.page_content[:200],
                    "metadata": doc.metadata
                }
                for doc in resultado["source_documents"]
            ],
            "area": self.area
        }
```

### 3. Agente de Triagem (agentes/triagem.py)

```python
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate

class AgenteTriagem:
    """Classifica perguntas de clientes por categoria"""
    
    CATEGORIAS = {
        "tecnico": "Problemas técnicos, bugs, configuração",
        "comercial": "Informações sobre produtos, preços, vendas",
        "financeiro": "Faturação, pagamentos, reembolsos",
        "geral": "Outras perguntas"
    }
    
    def __init__(self):
        self.llm = OpenAI(temperature=0)
        self.prompt = PromptTemplate(
            template="""
            Classifique a seguinte pergunta de cliente numa destas categorias:
            
            Categorias disponíveis:
            {categorias}
            
            Pergunta do cliente:
            {pergunta}
            
            Responda APENAS com o nome da categoria (tecnico, comercial, financeiro, ou geral).
            """,
            input_variables=["categorias", "pergunta"]
        )
    
    def classificar(self, pergunta: str) -> Dict:
        """Classificar pergunta"""
        
        # Formatar categorias
        categorias_texto = "\n".join([
            f"- {cat}: {desc}"
            for cat, desc in self.CATEGORIAS.items()
        ])
        
        # Executar classificação
        prompt_formatado = self.prompt.format(
            categorias=categorias_texto,
            pergunta=pergunta
        )
        
        categoria = self.llm(prompt_formatado).strip().lower()
        
        # Validar categoria
        if categoria not in self.CATEGORIAS:
            categoria = "geral"
        
        return {
            "categoria": categoria,
            "descricao": self.CATEGORIAS[categoria],
            "confianca": 0.9 if categoria != "geral" else 0.5
        }
```

### 4. Agentes Especializados (agentes/tecnico.py)

```python
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI
from langchain.memory import ConversationBufferMemory
from typing import Dict
import os

class AgenteTecnico:
    """Agente especializado em suporte técnico"""
    
    def __init__(self, sistema_rag):
        self.sistema_rag = sistema_rag
        self.llm = OpenAI(temperature=0.7)
        
        # Criar ferramentas
        self.ferramentas = [
            Tool(
                name="Consultar_Documentacao",
                func=self._consultar_docs,
                description="""
                Use para consultar manuais técnicos e documentação.
                Input: pergunta sobre problema técnico
                """
            ),
            Tool(
                name="Verificar_Status_Sistema",
                func=self._verificar_status,
                description="""
                Use para verificar status de sistemas e serviços.
                Input: nome do sistema a verificar
                """
            )
        ]
        
        # Criar agente
        self.memoria = ConversationBufferMemory(
            memory_key="chat_history",
            return_messages=True
        )
        
        self.agente = initialize_agent(
            self.ferramentas,
            self.llm,
            agent="conversational-react-description",
            memory=self.memoria,
            verbose=True
        )
    
    def _consultar_docs(self, pergunta: str) -> str:
        """Consultar documentação técnica"""
        resultado = self.sistema_rag.consultar(pergunta)
        return resultado["resposta"]
    
    def _verificar_status(self, sistema: str) -> str:
        """Verificar status de sistema (simulado)"""
        # Em produção, consultaria API real
        sistemas_status = {
            "api": "Operacional",
            "base de dados": "Operacional",
            "servidor": "Operacional",
            "email": "Manutenção programada"
        }
        
        sistema_lower = sistema.lower()
        status = sistemas_status.get(sistema_lower, "Desconhecido")
        
        return f"Status do {sistema}: {status}"
    
    def processar(self, pergunta: str, contexto: Dict = None) -> Dict:
        """Processar pergunta técnica"""
        
        # Adicionar contexto ao prompt se disponível
        if contexto:
            pergunta_completa = f"""
            Contexto do cliente: {contexto}
            
            Pergunta: {pergunta}
            """
        else:
            pergunta_completa = pergunta
        
        # Executar agente
        resposta = self.agente.run(pergunta_completa)
        
        return {
            "resposta": resposta,
            "agente": "tecnico",
            "ferramentas_usadas": self._extrair_ferramentas_usadas()
        }
    
    def _extrair_ferramentas_usadas(self) -> List[str]:
        """Extrair ferramentas que foram usadas"""
        # Implementação simplificada
        return ["Consultar_Documentacao"]
```

### 5. Agente Supervisor (agentes/supervisor.py)

```python
from typing import Dict
from datetime import datetime
from .triagem import AgenteTriagem
from .tecnico import AgenteTecnico
from .comercial import AgenteComercial
from .financeiro import AgenteFinanceiro
from rag.sistema_rag import SistemaRAGEspecializado

class SupervisorAtendimento:
    """Coordena atendimento ao cliente"""
    
    def __init__(self):
        # Inicializar sistemas RAG
        print("🔧 Inicializando sistemas RAG...")
        self.rag_tecnico = SistemaRAGEspecializado(
            "tecnico",
            "./rag/documentos/manuais_tecnicos"
        )
        self.rag_comercial = SistemaRAGEspecializado(
            "comercial",
            "./rag/documentos/catalogo_produtos"
        )
        self.rag_financeiro = SistemaRAGEspecializado(
            "financeiro",
            "./rag/documentos/politicas_financeiras"
        )
        
        # Inicializar agentes
        print("🤖 Inicializando agentes...")
        self.agente_triagem = AgenteTriagem()
        self.agentes = {
            "tecnico": AgenteTecnico(self.rag_tecnico),
            "comercial": AgenteComercial(self.rag_comercial),
            "financeiro": AgenteFinanceiro(self.rag_financeiro)
        }
        
        print("✅ Supervisor inicializado")
    
    def processar_pergunta(
        self,
        pergunta: str,
        contexto_cliente: Dict = None
    ) -> Dict:
        """Processar pergunta de cliente"""
        
        # 1. Classificar pergunta
        print(f"\n📥 Pergunta recebida: {pergunta[:50]}...")
        classificacao = self.agente_triagem.classificar(pergunta)
        categoria = classificacao["categoria"]
        
        print(f"🏷️  Categoria: {categoria}")
        
        # 2. Delegar ao agente especializado
        if categoria in self.agentes:
            agente = self.agentes[categoria]
            resultado = agente.processar(pergunta, contexto_cliente)
        else:
            # Categoria geral - resposta simples
            resultado = {
                "resposta": "Obrigado pela sua pergunta. Vou encaminhá-la para nossa equipa de suporte geral.",
                "agente": "geral"
            }
        
        # 3. Adicionar metadados
        resultado["classificacao"] = classificacao
        resultado["timestamp"] = datetime.now().isoformat()
        
        print(f"✅ Resposta gerada por agente {resultado['agente']}")
        
        return resultado
```

### 6. API REST (api/main.py)

```python
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel
from typing import Optional, Dict
import os
from dotenv import load_dotenv

from agentes.supervisor import SupervisorAtendimento
from utils.logger import configurar_logger
from utils.metricas import registar_metrica

# Carregar variáveis de ambiente
load_dotenv()

# Configurar logger
logger = configurar_logger("api-atendimento")

# Criar app
app = FastAPI(
    title="API Atendimento ao Cliente",
    description="Sistema de atendimento com IA",
    version="1.0.0"
)

# Inicializar supervisor (singleton)
supervisor = None

@app.on_event("startup")
async def startup():
    global supervisor
    logger.info("🚀 Iniciando API...")
    supervisor = SupervisorAtendimento()
    logger.info("✅ API pronta")

# Modelos Pydantic
class Pergunta(BaseModel):
    texto: str
    cliente_id: Optional[str] = None
    session_id: Optional[str] = None
    contexto: Optional[Dict] = None

class Resposta(BaseModel):
    resposta: str
    categoria: str
    agente: str
    confianca: float
    timestamp: str
    fontes: Optional[list] = None

# Endpoints
@app.post("/api/v1/perguntar", response_model=Resposta)
async def perguntar(pergunta: Pergunta):
    """Processar pergunta de cliente"""
    try:
        logger.info(f"📥 Pergunta de {pergunta.cliente_id or 'anónimo'}")
        
        # Processar
        resultado = supervisor.processar_pergunta(
            pergunta.texto,
            pergunta.contexto
        )
        
        # Registar métrica
        registar_metrica(
            "pergunta_processada",
            {
                "categoria": resultado["classificacao"]["categoria"],
                "agente": resultado["agente"]
            }
        )
        
        # Retornar resposta
        return Resposta(
            resposta=resultado["resposta"],
            categoria=resultado["classificacao"]["categoria"],
            agente=resultado["agente"],
            confianca=resultado["classificacao"]["confianca"],
            timestamp=resultado["timestamp"],
            fontes=resultado.get("fontes")
        )
        
    except Exception as e:
        logger.error(f"❌ Erro: {str(e)}")
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/api/v1/health")
async def health():
    """Verificar saúde do sistema"""
    return {
        "status": "healthy",
        "supervisor": supervisor is not None
    }

@app.get("/api/v1/metricas")
async def metricas():
    """Obter métricas do sistema"""
    # Implementar coleta de métricas
    return {
        "total_perguntas": 0,
        "por_categoria": {},
        "tempo_medio_resposta": 0.0
    }

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        app,
        host="0.0.0.0",
        port=8000,
        log_level="info"
    )
```

### 7. Utilitários (utils/logger.py)

```python
import logging
import json
from datetime import datetime

def configurar_logger(nome: str) -> logging.Logger:
    """Configurar logger estruturado"""
    
    logger = logging.getLogger(nome)
    logger.setLevel(logging.INFO)
    
    # Handler para ficheiro
    handler = logging.FileHandler(f"logs/{nome}.log")
    
    # Formatter JSON
    class JSONFormatter(logging.Formatter):
        def format(self, record):
            log_data = {
                "timestamp": datetime.now().isoformat(),
                "level": record.levelname,
                "logger": record.name,
                "message": record.getMessage(),
            }
            
            if record.exc_info:
                log_data["exception"] = self.formatException(record.exc_info)
            
            return json.dumps(log_data)
    
    handler.setFormatter(JSONFormatter())
    logger.addHandler(handler)
    
    # Handler para consola
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(
        logging.Formatter('%(levelname)s: %(message)s')
    )
    logger.addHandler(console_handler)
    
    return logger
```

## 🚀 Como Executar

### 1. Preparar Ambiente

```bash
# Criar diretórios
mkdir -p rag/documentos/{manuais_tecnicos,catalogo_produtos,politicas_financeiras}
mkdir -p logs

# Instalar dependências
pip install -r requirements.txt

# Configurar .env
cp .env.example .env
# Editar .env com suas chaves
```

### 2. Adicionar Documentos

Coloque documentos nas pastas apropriadas:
- `rag/documentos/manuais_tecnicos/` - Manuais técnicos
- `rag/documentos/catalogo_produtos/` - Informações de produtos
- `rag/documentos/politicas_financeiras/` - Políticas financeiras

### 3. Iniciar API

```bash
python api/main.py
```

### 4. Testar

```bash
# Pergunta técnica
curl -X POST http://localhost:8000/api/v1/perguntar \
  -H "Content-Type: application/json" \
  -d '{
    "texto": "Como configurar a integração com API?",
    "cliente_id": "cliente-123"
  }'

# Pergunta comercial
curl -X POST http://localhost:8000/api/v1/perguntar \
  -H "Content-Type: application/json" \
  -d '{
    "texto": "Quanto custa o plano enterprise?",
    "cliente_id": "cliente-456"
  }'
```

## 📊 Próximos Passos

1. **Adicionar Autenticação**
   - Implementar JWT
   - Rate limiting por cliente

2. **Melhorar RAG**
   - Adicionar mais documentos
   - Fine-tuning de embeddings
   - Reranking de resultados

3. **Dashboard de Monitorização**
   - Grafana para visualização
   - Alertas automáticos
   - Análise de sentimento

4. **Expandir Agentes**
   - Mais categorias especializadas
   - Multi-língua
   - Escalamento horizontal

## 📚 Recursos Adicionais

- [Documentação Completa](../README.md)
- [Guia de Arquiteturas](../guias/ARQUITETURAS.md)
- [API Reference](../api/REFERENCIA.md)
- [Boas Práticas](../guias/BOAS_PRATICAS.md)
