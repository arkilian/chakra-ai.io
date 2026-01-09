# Tutorial: Criar o Seu Primeiro Agente

Neste tutorial, irá aprender a criar o seu primeiro agente de IA usando ChakraAgents.ai. Vamos construir um assistente simples que responde a perguntas sobre produtos.

## 📋 O Que Irá Aprender

- Criar e configurar um agente básico
- Definir ferramentas e capacidades
- Testar o agente localmente
- Implementar o agente como API
- Monitorizar o desempenho

## ⏱️ Tempo Estimado

30-45 minutos

## 📚 Pré-requisitos

- ChakraAgents.ai instalado e a correr
- Chave API do OpenAI configurada
- Conhecimentos básicos de Python (opcional)

## Passo 1: Aceder à Interface

1. Abra o navegador e aceda a `http://localhost:3000`
2. Faça login com as credenciais:
   - Utilizador: `admin`
   - Senha: `admin` (altere após primeiro login)

## Passo 2: Criar Novo Agente

### Via Interface Web

1. Clique no botão **"Novo Agente"** no canto superior direito
2. Preencha o formulário:
   - **Nome**: `Assistente de Produtos`
   - **Descrição**: `Agente que responde a perguntas sobre produtos`
   - **Tipo**: Selecione `Simples`
   - **Modelo LLM**: Selecione `gpt-3.5-turbo`

3. Clique em **"Criar"**

### Via Código Python

```python
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI
from langchain.memory import ConversationBufferMemory

# Configurar LLM
llm = OpenAI(
    model_name="gpt-3.5-turbo",
    temperature=0.7,
    openai_api_key="sua-chave-aqui"
)

# Criar memória para contexto
memoria = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

# Definir ferramentas (vamos adicionar no próximo passo)
ferramentas = []

# Criar agente
agente = initialize_agent(
    ferramentas,
    llm,
    agent="conversational-react-description",
    memory=memoria,
    verbose=True
)
```

## Passo 3: Adicionar Ferramentas ao Agente

As ferramentas permitem que o agente execute ações específicas.

### Ferramenta 1: Consultar Catálogo de Produtos

```python
def consultar_catalogo(query: str) -> str:
    """
    Simula consulta a um catálogo de produtos.
    Em produção, isto conectaria a uma base de dados real.
    """
    catalogo = {
        "laptop": {
            "nome": "Laptop Pro 15",
            "preco": 1299.99,
            "stock": 15,
            "descricao": "Laptop profissional com processador i7, 16GB RAM, 512GB SSD"
        },
        "rato": {
            "nome": "Rato Wireless Premium",
            "preco": 29.99,
            "stock": 50,
            "descricao": "Rato wireless ergonómico com 6 botões programáveis"
        },
        "teclado": {
            "nome": "Teclado Mecânico RGB",
            "preco": 89.99,
            "stock": 30,
            "descricao": "Teclado mecânico com iluminação RGB e switches azuis"
        }
    }
    
    query_lower = query.lower()
    
    # Procurar produto
    for key, produto in catalogo.items():
        if key in query_lower or produto["nome"].lower() in query_lower:
            return f"""
Produto: {produto['nome']}
Preço: €{produto['preco']}
Stock: {produto['stock']} unidades
Descrição: {produto['descricao']}
            """
    
    return "Produto não encontrado no catálogo."

# Criar ferramenta
ferramenta_catalogo = Tool(
    name="Consultar_Catalogo",
    func=consultar_catalogo,
    description="""
    Útil para consultar informações sobre produtos.
    Input deve ser o nome ou tipo de produto.
    Exemplo: 'laptop', 'rato', 'teclado'
    """
)
```

### Ferramenta 2: Verificar Disponibilidade

```python
def verificar_disponibilidade(produto: str) -> str:
    """
    Verifica se um produto está disponível para envio imediato.
    """
    disponibilidade = {
        "laptop": "Disponível - Envio em 24h",
        "rato": "Disponível - Envio em 24h",
        "teclado": "Pré-venda - Envio em 5-7 dias"
    }
    
    produto_lower = produto.lower()
    
    for key, status in disponibilidade.items():
        if key in produto_lower:
            return f"Status de disponibilidade: {status}"
    
    return "Produto não encontrado."

# Criar ferramenta
ferramenta_disponibilidade = Tool(
    name="Verificar_Disponibilidade",
    func=verificar_disponibilidade,
    description="""
    Verifica a disponibilidade e tempo de envio de produtos.
    Input deve ser o nome do produto.
    """
)
```

### Adicionar Ferramentas ao Agente

```python
# Atualizar lista de ferramentas
ferramentas = [
    ferramenta_catalogo,
    ferramenta_disponibilidade
]

# Recriar agente com ferramentas
agente = initialize_agent(
    ferramentas,
    llm,
    agent="conversational-react-description",
    memory=memoria,
    verbose=True
)
```

## Passo 4: Configurar Prompt do Sistema

Definir personalidade e comportamento do agente:

```python
from langchain.prompts import PromptTemplate

template_sistema = """
Você é um assistente de vendas prestável e profissional.

Suas responsabilidades:
- Responder a perguntas sobre produtos de forma clara e concisa
- Fornecer informações precisas sobre preços e disponibilidade
- Ser cortês e útil em todas as interações
- Sugerir produtos relevantes quando apropriado

Diretrizes:
- Use sempre as ferramentas disponíveis para obter informações atualizadas
- Se não souber algo, admita e ofereça ajuda alternativa
- Seja proativo em oferecer informações adicionais úteis
- Mantenha respostas concisas mas completas

Ferramentas disponíveis: {tools}
Histórico de conversa: {chat_history}

Pergunta do cliente: {input}
{agent_scratchpad}
"""

prompt = PromptTemplate(
    template=template_sistema,
    input_variables=["tools", "chat_history", "input", "agent_scratchpad"]
)
```

## Passo 5: Testar o Agente

### Teste 1: Consulta Simples

```python
# Testar pergunta simples
resposta = agente.run("Olá, quanto custa o laptop?")
print(resposta)

# Resposta esperada:
# "O Laptop Pro 15 custa €1299.99. Está disponível para envio em 24h..."
```

### Teste 2: Múltiplas Perguntas

```python
# Pergunta com contexto
resposta = agente.run("Tens ratos wireless?")
print(resposta)

# Seguimento
resposta = agente.run("E quanto custa?")
print(resposta)

# O agente deve lembrar-se do contexto anterior
```

### Teste 3: Verificação de Disponibilidade

```python
resposta = agente.run("O teclado mecânico está disponível para envio imediato?")
print(resposta)

# Resposta esperada:
# "O Teclado Mecânico RGB está em pré-venda com envio previsto em 5-7 dias..."
```

## Passo 6: Melhorar com RAG (Opcional)

Adicionar documentação de produtos para respostas mais completas.

### 6.1 Preparar Documentos

```python
from langchain.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma

# Criar documentos de exemplo
documentos_texto = """
# Laptop Pro 15

## Especificações Técnicas
- Processador: Intel Core i7-12700H
- RAM: 16GB DDR5
- Armazenamento: 512GB NVMe SSD
- Ecrã: 15.6" FHD IPS
- Placa Gráfica: NVIDIA RTX 3050
- Bateria: Até 8 horas

## Características
- Teclado retroiluminado
- Leitor de impressões digitais
- Webcam 1080p
- WiFi 6E
- Bluetooth 5.2

## Garantia
2 anos de garantia do fabricante

---

# Rato Wireless Premium

## Especificações
- Conexão: 2.4GHz + Bluetooth 5.0
- DPI: 800-4000 ajustável
- Botões: 6 programáveis
- Bateria: Até 70 horas
- Design ergonómico

## Compatibilidade
Windows, macOS, Linux

---

# Teclado Mecânico RGB

## Características
- Switches: Cherry MX Blue
- Iluminação: RGB individual por tecla
- Layout: Português (QWERTY)
- Conexão: USB-C destacável
- Apoio para pulsos incluído

## Software
Software de configuração para macros e iluminação
"""

# Salvar em ficheiro
with open("produtos_docs.txt", "w", encoding="utf-8") as f:
    f.write(documentos_texto)
```

### 6.2 Criar Vector Store

```python
# Carregar documento
loader = TextLoader("produtos_docs.txt", encoding="utf-8")
documentos = loader.load()

# Dividir em chunks
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)
chunks = text_splitter.split_documents(documentos)

# Criar embeddings e vector store
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(
    chunks,
    embeddings,
    persist_directory="./produtos_db"
)

# Persistir para uso futuro
vectorstore.persist()
```

### 6.3 Adicionar RAG ao Agente

```python
from langchain.chains import RetrievalQA

# Criar retriever
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 3}  # Retornar top 3 documentos mais relevantes
)

# Criar ferramenta RAG
def consultar_documentacao(query: str) -> str:
    """Consulta documentação detalhada dos produtos"""
    qa = RetrievalQA.from_chain_type(
        llm=llm,
        chain_type="stuff",
        retriever=retriever
    )
    return qa.run(query)

ferramenta_docs = Tool(
    name="Consultar_Documentacao",
    func=consultar_documentacao,
    description="""
    Útil para obter informações técnicas detalhadas sobre produtos.
    Use quando o cliente perguntar sobre especificações, características técnicas,
    garantia, compatibilidade, etc.
    """
)

# Adicionar à lista de ferramentas
ferramentas.append(ferramenta_docs)

# Recriar agente
agente = initialize_agent(
    ferramentas,
    llm,
    agent="conversational-react-description",
    memory=memoria,
    verbose=True
)
```

## Passo 7: Criar Endpoint API

Implementar o agente como serviço REST API.

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional

app = FastAPI(title="Assistente de Produtos API")

class Pergunta(BaseModel):
    texto: str
    session_id: Optional[str] = None

class Resposta(BaseModel):
    resposta: str
    session_id: str
    sucesso: bool

# Armazenar sessões (em produção, usar Redis ou DB)
sessoes = {}

@app.post("/api/perguntar", response_model=Resposta)
async def perguntar(pergunta: Pergunta):
    try:
        # Obter ou criar sessão
        session_id = pergunta.session_id or str(uuid.uuid4())
        
        if session_id not in sessoes:
            # Criar novo agente para sessão
            memoria = ConversationBufferMemory(
                memory_key="chat_history",
                return_messages=True
            )
            sessoes[session_id] = initialize_agent(
                ferramentas,
                llm,
                agent="conversational-react-description",
                memory=memoria,
                verbose=True
            )
        
        # Processar pergunta
        agente = sessoes[session_id]
        resposta_texto = agente.run(pergunta.texto)
        
        return Resposta(
            resposta=resposta_texto,
            session_id=session_id,
            sucesso=True
        )
        
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.delete("/api/sessao/{session_id}")
async def limpar_sessao(session_id: str):
    if session_id in sessoes:
        del sessoes[session_id]
        return {"mensagem": "Sessão eliminada com sucesso"}
    raise HTTPException(status_code=404, detail="Sessão não encontrada")

@app.get("/api/estado")
async def obter_estado():
    return {
        "ativo": True,
        "sessoes_ativas": len(sessoes),
        "versao": "1.0.0"
    }

# Iniciar servidor
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8001)
```

## Passo 8: Testar API

```bash
# Testar endpoint
curl -X POST http://localhost:8001/api/perguntar \
  -H "Content-Type: application/json" \
  -d '{
    "texto": "Quais são as especificações do laptop?"
  }'

# Resposta
{
  "resposta": "O Laptop Pro 15 tem as seguintes especificações: ...",
  "session_id": "abc123",
  "sucesso": true
}

# Continuar conversa na mesma sessão
curl -X POST http://localhost:8001/api/perguntar \
  -H "Content-Type: application/json" \
  -d '{
    "texto": "E o preço?",
    "session_id": "abc123"
  }'
```

## Passo 9: Adicionar Monitorização

```python
from datetime import datetime
import json

class MonitorAgente:
    def __init__(self):
        self.metricas = {
            "total_perguntas": 0,
            "tempo_resposta_medio": 0,
            "erros": 0,
            "perguntas_por_categoria": {}
        }
    
    def registar_pergunta(self, pergunta: str, tempo_resposta: float, sucesso: bool):
        self.metricas["total_perguntas"] += 1
        
        # Atualizar tempo médio
        total = self.metricas["total_perguntas"]
        media_anterior = self.metricas["tempo_resposta_medio"]
        self.metricas["tempo_resposta_medio"] = (
            (media_anterior * (total - 1) + tempo_resposta) / total
        )
        
        if not sucesso:
            self.metricas["erros"] += 1
        
        # Categorizar pergunta (simplificado)
        if "preço" in pergunta.lower() or "custa" in pergunta.lower():
            categoria = "preco"
        elif "disponivel" in pergunta.lower() or "stock" in pergunta.lower():
            categoria = "disponibilidade"
        else:
            categoria = "geral"
        
        self.metricas["perguntas_por_categoria"][categoria] = \
            self.metricas["perguntas_por_categoria"].get(categoria, 0) + 1
    
    def obter_metricas(self):
        return self.metricas

# Integrar com API
monitor = MonitorAgente()

@app.post("/api/perguntar", response_model=Resposta)
async def perguntar(pergunta: Pergunta):
    inicio = datetime.now()
    sucesso = False
    
    try:
        # ... código anterior ...
        sucesso = True
        return resposta
        
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
        
    finally:
        tempo_resposta = (datetime.now() - inicio).total_seconds()
        monitor.registar_pergunta(pergunta.texto, tempo_resposta, sucesso)

@app.get("/api/metricas")
async def obter_metricas():
    return monitor.obter_metricas()
```

## 🎉 Parabéns!

Criou com sucesso o seu primeiro agente! O agente agora pode:

- ✅ Responder a perguntas sobre produtos
- ✅ Consultar catálogo e disponibilidade
- ✅ Aceder a documentação técnica via RAG
- ✅ Manter contexto entre perguntas
- ✅ Funcionar como API REST
- ✅ Registar métricas de utilização

## 📈 Próximos Passos

### Melhorias Sugeridas

1. **Adicionar Mais Ferramentas**
   - Integração com sistema de pagamentos
   - Verificação de encomendas
   - Suporte a múltiplas línguas

2. **Melhorar Personalidade**
   - Ajustar temperatura do modelo
   - Refinar prompts do sistema
   - Adicionar exemplos few-shot

3. **Implementar Cache**
   - Usar Redis para cache de respostas
   - Armazenar embeddings
   - Cache de sessões

4. **Adicionar Autenticação**
   - OAuth 2.0
   - JWT tokens
   - Rate limiting

5. **Deploy em Produção**
   - Containerizar com Docker
   - Implementar em cloud
   - Configurar CI/CD

## 📚 Recursos Adicionais

- [Arquiteturas Avançadas](../guias/ARQUITETURAS.md)
- [Exemplos de Código](../exemplos/)
- [Referência API Completa](../api/REFERENCIA.md)
- [Boas Práticas](../guias/BOAS_PRATICAS.md)

## 💬 Precisa de Ajuda?

- Discord: https://discord.gg/chakraagents
- GitHub Issues: https://github.com/sudsk/ChakraAgents.ai/issues
- Documentação: https://docs.chakraagents.ai
