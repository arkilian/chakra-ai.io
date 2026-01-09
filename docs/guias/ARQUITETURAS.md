# Guia de Arquiteturas

Este guia fornece uma análise aprofundada das diferentes arquiteturas de agentes suportadas pelo ChakraAgents.ai.

## Índice

- [Visão Geral](#visão-geral)
- [Arquitetura Supervisor](#arquitetura-supervisor)
- [Arquitetura Swarm](#arquitetura-swarm)
- [Arquitetura RAG](#arquitetura-rag)
- [Arquiteturas Híbridas](#arquiteturas-híbridas)
- [Escolher a Arquitetura Certa](#escolher-a-arquitetura-certa)
- [Padrões de Design](#padrões-de-design)

## Visão Geral

ChakraAgents.ai suporta três arquiteturas principais, cada uma com características únicas adequadas para diferentes cenários.

### Comparação Rápida

| Característica | Supervisor | Swarm | RAG |
|---|---|---|---|
| **Controlo** | Centralizado | Distribuído | Variável |
| **Coordenação** | Hierárquica | Peer-to-peer | Baseada em conhecimento |
| **Complexidade** | Média | Alta | Média-Alta |
| **Escalabilidade** | Limitada | Excelente | Boa |
| **Debugging** | Fácil | Desafiante | Médio |
| **Latência** | Média | Baixa | Variável |
| **Casos de Uso** | Fluxos definidos | Tarefas emergentes | Consulta de conhecimento |

## Arquitetura Supervisor

### Conceito

Um agente supervisor central coordena e delega tarefas a agentes trabalhadores especializados. O supervisor é responsável por:
- Receber e analisar requisições
- Decidir qual agente trabalhador deve processar
- Coordenar a comunicação entre agentes
- Agregar e retornar resultados

### Diagrama de Fluxo

```
Cliente
   ↓
Pergunta: "Analisar vendas e criar relatório"
   ↓
┌─────────────────────┐
│  Agente Supervisor  │
│                     │
│ 1. Analisar tarefa  │
│ 2. Decompor         │
│ 3. Delegar          │
│ 4. Agregar          │
└─────────┬───────────┘
          │
    ┌─────┼─────┬─────────────┐
    ↓     ↓     ↓             ↓
┌────────┐ ┌──────────┐ ┌─────────┐ ┌─────────┐
│Analista│ │Extração  │ │Relatório│ │Validador│
│  Dados │ │  Dados   │ │  Gerar  │ │         │
└────────┘ └──────────┘ └─────────┘ └─────────┘
    │         │             │           │
    └─────────┴─────────────┴───────────┘
                    ↓
            Resultado Final
```

### Implementação

#### 1. Definir Agente Supervisor

```python
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate

class AgenteSupervisor:
    def __init__(self):
        self.llm = OpenAI(temperature=0)
        self.agentes_trabalhadores = {}
        self.historico = []
    
    def registar_trabalhador(self, nome: str, agente):
        """Registar um agente trabalhador"""
        self.agentes_trabalhadores[nome] = agente
    
    def analisar_tarefa(self, tarefa: str) -> dict:
        """Analisar e decompor tarefa"""
        prompt = f"""
        Analise a seguinte tarefa e determine:
        1. Que agentes são necessários
        2. Em que ordem devem trabalhar
        3. Que informação cada um precisa
        
        Tarefa: {tarefa}
        
        Agentes disponíveis: {', '.join(self.agentes_trabalhadores.keys())}
        
        Retorne um plano em formato JSON.
        """
        
        plano = self.llm(prompt)
        return self._processar_plano(plano)
    
    def executar(self, tarefa: str) -> str:
        """Executar tarefa delegando a trabalhadores"""
        # Analisar tarefa
        plano = self.analisar_tarefa(tarefa)
        
        resultados = {}
        
        # Executar cada passo do plano
        for passo in plano['passos']:
            agente_nome = passo['agente']
            entrada = passo['entrada']
            
            # Substituir referências a resultados anteriores
            for key, valor in resultados.items():
                entrada = entrada.replace(f"${key}", valor)
            
            # Executar agente trabalhador
            agente = self.agentes_trabalhadores[agente_nome]
            resultado = agente.executar(entrada)
            resultados[passo['id']] = resultado
        
        # Agregar resultados
        return self.agregar_resultados(resultados, tarefa)
    
    def agregar_resultados(self, resultados: dict, tarefa_original: str) -> str:
        """Agregar resultados de múltiplos agentes"""
        prompt = f"""
        Tarefa original: {tarefa_original}
        
        Resultados dos agentes:
        {self._formatar_resultados(resultados)}
        
        Crie uma resposta final coerente e completa.
        """
        
        return self.llm(prompt)
```

#### 2. Definir Agentes Trabalhadores

```python
class AgenteTrabalhador:
    def __init__(self, nome: str, especialidade: str, ferramentas: list):
        self.nome = nome
        self.especialidade = especialidade
        self.llm = OpenAI(temperature=0.7)
        self.ferramentas = ferramentas
        
        # Criar agente com ferramentas específicas
        self.agente = initialize_agent(
            ferramentas,
            self.llm,
            agent="zero-shot-react-description",
            verbose=True
        )
    
    def executar(self, tarefa: str) -> str:
        """Executar tarefa específica"""
        prompt = f"""
        Como especialista em {self.especialidade}, execute a seguinte tarefa:
        
        {tarefa}
        
        Use as ferramentas disponíveis conforme necessário.
        """
        
        return self.agente.run(prompt)

# Exemplo: Criar trabalhadores especializados
def criar_ferramenta_analise():
    def analisar_dados(dados: str) -> str:
        # Simulação de análise
        return f"Análise completa: {len(dados)} registos processados"
    
    return Tool(
        name="Analisar_Dados",
        func=analisar_dados,
        description="Analisa conjuntos de dados"
    )

# Criar agentes trabalhadores
agente_analista = AgenteTrabalhador(
    nome="Analista",
    especialidade="Análise de Dados",
    ferramentas=[criar_ferramenta_analise()]
)

agente_relatorios = AgenteTrabalhador(
    nome="Gerador_Relatorios",
    especialidade="Geração de Relatórios",
    ferramentas=[]
)

# Criar supervisor
supervisor = AgenteSupervisor()
supervisor.registar_trabalhador("Analista", agente_analista)
supervisor.registar_trabalhador("Gerador_Relatorios", agente_relatorios)

# Usar
resultado = supervisor.executar(
    "Analise os dados de vendas do último mês e crie um relatório"
)
```

### Vantagens

✅ **Controlo Previsível**: Fluxo de execução claro e rastreável
✅ **Debugging Fácil**: Simples identificar onde ocorrem problemas
✅ **Gestão de Contexto**: Supervisor mantém estado global
✅ **Especialização**: Cada trabalhador foca numa tarefa específica

### Desvantagens

❌ **Gargalo**: Supervisor pode ser ponto de contenção
❌ **Flexibilidade Limitada**: Difícil adaptar a tarefas não previstas
❌ **Escalabilidade**: Limitada pela capacidade do supervisor

### Casos de Uso Ideais

- 📋 Processamento de documentos com múltiplas etapas
- 🎫 Sistemas de triagem e encaminhamento
- 📊 Pipelines de análise de dados
- 🔄 Fluxos de trabalho com dependências claras

## Arquitetura Swarm

### Conceito

Múltiplos agentes peer colaboram diretamente sem hierarquia central. Características:
- Comunicação peer-to-peer
- Decisões emergentes
- Auto-organização
- Alta resiliência

### Diagrama de Fluxo

```
         Tarefa Complexa
                │
    ┌───────────┼───────────┐
    ↓           ↓           ↓
┌────────┐  ┌────────┐  ┌────────┐
│Agente 1│◄─┤Agente 2│─►│Agente 3│
│Finance │  │Market  │  │Tech    │
└───┬────┘  └───┬────┘  └───┬────┘
    │           │           │
    └───────►Colaboração◄───┘
              ↓
    ┌─────────────────┐
    │   Consenso/      │
    │   Síntese        │
    └─────────────────┘
              ↓
         Resultado
```

### Implementação

#### 1. Agente Swarm Base

```python
from typing import List, Dict, Any
import asyncio

class AgenteSwarm:
    def __init__(
        self,
        nome: str,
        especialidade: str,
        llm,
        ferramentas: List[Tool] = None
    ):
        self.nome = nome
        self.especialidade = especialidade
        self.llm = llm
        self.ferramentas = ferramentas or []
        self.peers = []
        self.mensagens_recebidas = []
    
    def conectar_peer(self, peer: 'AgenteSwarm'):
        """Conectar a outro agente do swarm"""
        if peer not in self.peers:
            self.peers.append(peer)
            peer.peers.append(self)
    
    async def processar_tarefa(self, tarefa: str) -> Dict[str, Any]:
        """Processar tarefa do ponto de vista da especialidade"""
        prompt = f"""
        Como especialista em {self.especialidade}, analise:
        
        {tarefa}
        
        Forneça insights específicos da sua área.
        """
        
        resultado = await self._executar_llm(prompt)
        
        return {
            "agente": self.nome,
            "especialidade": self.especialidade,
            "analise": resultado,
            "confianca": self._calcular_confianca(resultado)
        }
    
    async def solicitar_opiniao_peers(self, tarefa: str) -> List[Dict]:
        """Solicitar opiniões de peers"""
        tarefas = [
            peer.processar_tarefa(tarefa)
            for peer in self.peers
        ]
        return await asyncio.gather(*tarefas)
    
    async def colaborar(self, tarefa: str) -> str:
        """Colaborar com peers para resolver tarefa"""
        # Processar localmente
        meu_resultado = await self.processar_tarefa(tarefa)
        
        # Obter opiniões de peers
        opinioes_peers = await self.solicitar_opiniao_peers(tarefa)
        
        # Sintetizar resultados
        todas_analises = [meu_resultado] + opinioes_peers
        sintese = await self._sintetizar(todas_analises, tarefa)
        
        return sintese
    
    async def _sintetizar(
        self,
        analises: List[Dict],
        tarefa_original: str
    ) -> str:
        """Sintetizar múltiplas análises"""
        prompt = f"""
        Tarefa original: {tarefa_original}
        
        Análises de múltiplos especialistas:
        
        {self._formatar_analises(analises)}
        
        Sintetize estas perspectivas numa conclusão coerente,
        considerando:
        1. Áreas de consenso
        2. Perspectivas conflituantes
        3. Insights únicos de cada especialidade
        4. Nível de confiança de cada análise
        
        Forneça uma recomendação final equilibrada.
        """
        
        return await self._executar_llm(prompt)
    
    async def _executar_llm(self, prompt: str) -> str:
        """Executar LLM de forma assíncrona"""
        # Simulação - em produção usar async LLM
        return self.llm(prompt)
    
    def _calcular_confianca(self, resultado: str) -> float:
        """Calcular nível de confiança no resultado"""
        # Lógica simplificada - pode ser mais sofisticada
        if "não tenho certeza" in resultado.lower():
            return 0.5
        if "definitivamente" in resultado.lower():
            return 0.9
        return 0.7
    
    def _formatar_analises(self, analises: List[Dict]) -> str:
        texto = []
        for analise in analises:
            texto.append(f"""
            Agente: {analise['agente']} ({analise['especialidade']})
            Confiança: {analise['confianca']*100:.0f}%
            Análise: {analise['analise']}
            """)
        return "\n".join(texto)
```

#### 2. Orquestração do Swarm

```python
class OrquestradorSwarm:
    def __init__(self):
        self.agentes = []
    
    def adicionar_agente(self, agente: AgenteSwarm):
        """Adicionar agente ao swarm"""
        # Conectar novo agente a todos os existentes
        for agente_existente in self.agentes:
            agente.conectar_peer(agente_existente)
        
        self.agentes.append(agente)
    
    async def resolver_tarefa(
        self,
        tarefa: str,
        metodo: str = "colaborativo"
    ) -> str:
        """
        Resolver tarefa usando swarm
        
        Métodos:
        - colaborativo: Todos os agentes colaboram
        - votacao: Agentes votam na melhor solução
        - especialista_primario: Um agente lidera, outros apoiam
        """
        
        if metodo == "colaborativo":
            return await self._metodo_colaborativo(tarefa)
        elif metodo == "votacao":
            return await self._metodo_votacao(tarefa)
        elif metodo == "especialista_primario":
            return await self._metodo_especialista_primario(tarefa)
    
    async def _metodo_colaborativo(self, tarefa: str) -> str:
        """Todos colaboram igualmente"""
        # Escolher agente coordenador (pode ser rotativo)
        coordenador = self.agentes[0]
        return await coordenador.colaborar(tarefa)
    
    async def _metodo_votacao(self, tarefa: str) -> str:
        """Cada agente propõe solução, melhor é escolhida"""
        # Obter propostas de todos
        propostas = await asyncio.gather(*[
            agente.processar_tarefa(tarefa)
            for agente in self.agentes
        ])
        
        # Votar na melhor
        votos = await self._realizar_votacao(propostas, tarefa)
        
        # Retornar vencedora
        melhor_proposta = max(votos, key=lambda x: x['votos'])
        return melhor_proposta['analise']
    
    async def _realizar_votacao(
        self,
        propostas: List[Dict],
        tarefa: str
    ) -> List[Dict]:
        """Cada agente vota nas propostas"""
        for proposta in propostas:
            proposta['votos'] = 0
        
        for agente in self.agentes:
            voto = await self._votar(agente, propostas, tarefa)
            propostas[voto]['votos'] += 1
        
        return propostas
    
    async def _votar(
        self,
        agente: AgenteSwarm,
        propostas: List[Dict],
        tarefa: str
    ) -> int:
        """Agente vota na melhor proposta"""
        prompt = f"""
        Tarefa: {tarefa}
        
        Propostas:
        {self._formatar_propostas(propostas)}
        
        Qual proposta melhor responde à tarefa?
        Responda apenas com o número (0, 1, 2, etc.)
        """
        
        resposta = await agente._executar_llm(prompt)
        try:
            return int(resposta.strip())
        except:
            return 0
    
    def _formatar_propostas(self, propostas: List[Dict]) -> str:
        return "\n\n".join([
            f"{i}. {p['analise']}"
            for i, p in enumerate(propostas)
        ])
```

#### 3. Exemplo de Uso

```python
# Criar agentes especializados
agente_financas = AgenteSwarm(
    nome="Analista_Financeiro",
    especialidade="Análise Financeira e Investimentos",
    llm=OpenAI(temperature=0.7)
)

agente_mercado = AgenteSwarm(
    nome="Analista_Mercado",
    especialidade="Análise de Mercado e Tendências",
    llm=OpenAI(temperature=0.7)
)

agente_tecnico = AgenteSwarm(
    nome="Analista_Tecnico",
    especialidade="Análise Técnica e Gráficos",
    llm=OpenAI(temperature=0.7)
)

# Criar orquestrador
orquestrador = OrquestradorSwarm()
orquestrador.adicionar_agente(agente_financas)
orquestrador.adicionar_agente(agente_mercado)
orquestrador.adicionar_agente(agente_tecnico)

# Resolver tarefa
import asyncio

tarefa = """
Analisar se devemos investir na empresa XYZ.
Considerar aspectos financeiros, de mercado e técnicos.
"""

resultado = asyncio.run(
    orquestrador.resolver_tarefa(tarefa, metodo="colaborativo")
)

print(resultado)
```

### Vantagens

✅ **Alta Resiliência**: Falha de um agente não paralisa sistema
✅ **Escalabilidade**: Fácil adicionar novos agentes
✅ **Diversidade**: Múltiplas perspectivas sobre problemas
✅ **Flexibilidade**: Adapta-se a tarefas não previstas

### Desvantagens

❌ **Complexidade**: Debugging é mais desafiante
❌ **Coordenação**: Pode haver conflitos entre agentes
❌ **Overhead**: Comunicação entre agentes adiciona latência
❌ **Imprevisibilidade**: Comportamento pode ser emergente

### Casos de Uso Ideais

- 🔬 Análise multi-perspectiva
- 🤔 Problemas sem solução óbvia
- 🎯 Tarefas que requerem consenso
- 🌐 Sistemas distribuídos

## Arquitetura RAG

### Conceito

Retrieval-Augmented Generation combina LLMs com acesso a bases de conhecimento externas.

### Componentes

```
Pergunta
   ↓
┌─────────────┐
│  Embedding  │ ← Converter pergunta em vetor
└──────┬──────┘
       ↓
┌─────────────┐
│Vector Store │ ← Pesquisar documentos similares
│  (Chroma)   │
└──────┬──────┘
       ↓
┌─────────────┐
│  Retriever  │ ← Top-K documentos mais relevantes
└──────┬──────┘
       ↓
┌─────────────┐
│     LLM     │ ← Gerar resposta com contexto
│  + Prompt   │
└──────┬──────┘
       ↓
   Resposta
```

### Implementação Completa

```python
from langchain.document_loaders import DirectoryLoader, TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chains import RetrievalQA
from langchain.prompts import PromptTemplate

class SistemaRAG:
    def __init__(
        self,
        diretorio_documentos: str,
        persist_directory: str = "./chroma_db"
    ):
        self.embeddings = OpenAIEmbeddings()
        self.persist_directory = persist_directory
        self.vectorstore = None
        self.qa_chain = None
        
        # Carregar e processar documentos
        self._carregar_documentos(diretorio_documentos)
    
    def _carregar_documentos(self, diretorio: str):
        """Carregar e processar documentos"""
        # Carregar todos os documentos
        loader = DirectoryLoader(
            diretorio,
            glob="**/*.md",
            loader_cls=TextLoader
        )
        documentos = loader.load()
        
        print(f"Carregados {len(documentos)} documentos")
        
        # Dividir em chunks
        text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200,
            length_function=len
        )
        chunks = text_splitter.split_documents(documentos)
        
        print(f"Criados {len(chunks)} chunks")
        
        # Criar vector store
        self.vectorstore = Chroma.from_documents(
            chunks,
            self.embeddings,
            persist_directory=self.persist_directory
        )
        
        self.vectorstore.persist()
        print("Vector store criado e persistido")
    
    def criar_qa_chain(
        self,
        chain_type: str = "stuff",
        return_source_documents: bool = True
    ):
        """Criar chain de pergunta-resposta"""
        # Prompt personalizado
        template = """
        Use o seguinte contexto para responder à pergunta.
        Se não souber a resposta, diga que não sabe.
        Não invente informação.
        
        Contexto: {context}
        
        Pergunta: {question}
        
        Resposta detalhada:
        """
        
        prompt = PromptTemplate(
            template=template,
            input_variables=["context", "question"]
        )
        
        # Criar retriever com configurações
        retriever = self.vectorstore.as_retriever(
            search_type="similarity",
            search_kwargs={"k": 5}  # Top 5 documentos
        )
        
        # Criar chain
        self.qa_chain = RetrievalQA.from_chain_type(
            llm=OpenAI(temperature=0),
            chain_type=chain_type,
            retriever=retriever,
            return_source_documents=return_source_documents,
            chain_type_kwargs={"prompt": prompt}
        )
    
    def perguntar(self, pergunta: str) -> Dict[str, Any]:
        """Fazer pergunta ao sistema RAG"""
        if not self.qa_chain:
            self.criar_qa_chain()
        
        resultado = self.qa_chain({"query": pergunta})
        
        return {
            "pergunta": pergunta,
            "resposta": resultado["result"],
            "fontes": [
                {
                    "conteudo": doc.page_content,
                    "metadata": doc.metadata
                }
                for doc in resultado.get("source_documents", [])
            ]
        }
    
    def pesquisar_documentos(
        self,
        query: str,
        k: int = 5
    ) -> List[Dict]:
        """Pesquisar documentos similares"""
        docs_scores = self.vectorstore.similarity_search_with_score(
            query,
            k=k
        )
        
        return [
            {
                "conteudo": doc.page_content,
                "metadata": doc.metadata,
                "score": score
            }
            for doc, score in docs_scores
        ]
    
    def adicionar_documentos(self, novos_documentos: List[str]):
        """Adicionar novos documentos ao vector store"""
        # Criar chunks
        text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200
        )
        
        # Processar novos documentos
        from langchain.schema import Document
        docs = [Document(page_content=texto) for texto in novos_documentos]
        chunks = text_splitter.split_documents(docs)
        
        # Adicionar ao vector store
        self.vectorstore.add_documents(chunks)
        self.vectorstore.persist()
        
        print(f"Adicionados {len(chunks)} novos chunks")

# Uso
sistema_rag = SistemaRAG("./documentos")
sistema_rag.criar_qa_chain()

resultado = sistema_rag.perguntar(
    "Como configurar autenticação OAuth?"
)

print(f"Resposta: {resultado['resposta']}")
print(f"\nFontes consultadas: {len(resultado['fontes'])}")
```

### Vantagens

✅ **Respostas Verificáveis**: Baseadas em documentos reais
✅ **Conhecimento Atualizado**: Fácil adicionar novos documentos
✅ **Redução de Alucinações**: LLM usa informação factual
✅ **Transparência**: Mostra fontes usadas

### Desvantagens

❌ **Dependência de Qualidade**: Documentos ruins = respostas ruins
❌ **Custo de Processamento**: Embeddings têm custo
❌ **Latência**: Pesquisa adiciona tempo
❌ **Gestão de Dados**: Requer manutenção do vector store

### Casos de Uso Ideais

- 📚 Assistentes de documentação
- 🏢 Bases de conhecimento empresarial
- ⚖️ Compliance e regulamentos
- 🔍 Pesquisa semântica em documentos

## Arquiteturas Híbridas

### Supervisor + RAG

Combinar supervisor com capacidades RAG para cada trabalhador.

```python
class AgenteTrabalhadorRAG(AgenteTrabalhador):
    def __init__(self, nome, especialidade, ferramentas, vectorstore):
        super().__init__(nome, especialidade, ferramentas)
        self.vectorstore = vectorstore
        self.qa_chain = RetrievalQA.from_chain_type(
            llm=self.llm,
            retriever=vectorstore.as_retriever()
        )
    
    def executar_com_conhecimento(self, tarefa: str) -> str:
        # Primeiro consultar base de conhecimento
        contexto = self.qa_chain.run(tarefa)
        
        # Depois executar tarefa com contexto
        prompt = f"""
        Contexto da base de conhecimento:
        {contexto}
        
        Tarefa: {tarefa}
        """
        
        return self.agente.run(prompt)
```

### Swarm + RAG

Agentes swarm com acesso a conhecimento partilhado.

```python
class AgenteSwarmRAG(AgenteSwarm):
    def __init__(self, nome, especialidade, llm, vectorstore):
        super().__init__(nome, especialidade, llm)
        self.vectorstore = vectorstore
    
    async def processar_tarefa_com_conhecimento(
        self,
        tarefa: str
    ) -> Dict[str, Any]:
        # Consultar conhecimento relevante
        docs = self.vectorstore.similarity_search(tarefa, k=3)
        contexto = "\n".join([doc.page_content for doc in docs])
        
        # Processar com contexto
        prompt = f"""
        Conhecimento relevante:
        {contexto}
        
        Como especialista em {self.especialidade}, analise:
        {tarefa}
        """
        
        resultado = await self._executar_llm(prompt)
        
        return {
            "agente": self.nome,
            "especialidade": self.especialidade,
            "analise": resultado,
            "fontes": [doc.metadata for doc in docs]
        }
```

## Escolher a Arquitetura Certa

### Matriz de Decisão

| Cenário | Arquitetura Recomendada |
|---|---|
| Fluxo definido, passos sequenciais | Supervisor |
| Problema requer múltiplas perspectivas | Swarm |
| Respostas devem basear-se em documentos | RAG |
| Pipeline de processamento complexo | Supervisor + RAG |
| Análise colaborativa com conhecimento | Swarm + RAG |
| Sistema de triagem e encaminhamento | Supervisor |
| Resolução criativa de problemas | Swarm |
| FAQ baseado em manuais | RAG |

### Perguntas-Chave

1. **A tarefa tem passos bem definidos?**
   - Sim → Supervisor
   - Não → Swarm

2. **Precisa de múltiplas perspectivas?**
   - Sim → Swarm
   - Não → Supervisor ou RAG

3. **Respostas devem basear-se em documentos?**
   - Sim → RAG (possivelmente híbrida)
   - Não → Supervisor ou Swarm

4. **O fluxo pode mudar dinamicamente?**
   - Sim → Swarm
   - Não → Supervisor

## Padrões de Design

### 1. Cadeia de Responsabilidade

```python
# Agentes processam sequencialmente até um conseguir
class CadeiaAgentes:
    def __init__(self):
        self.agentes = []
    
    def adicionar(self, agente):
        self.agentes.append(agente)
    
    def processar(self, tarefa):
        for agente in self.agentes:
            if agente.pode_processar(tarefa):
                return agente.executar(tarefa)
        return "Nenhum agente pode processar esta tarefa"
```

### 2. Pipeline

```python
# Output de um agente é input do próximo
class Pipeline:
    def __init__(self, agentes: List):
        self.agentes = agentes
    
    def executar(self, entrada):
        resultado = entrada
        for agente in self.agentes:
            resultado = agente.processar(resultado)
        return resultado
```

### 3. Fan-out/Fan-in

```python
# Distribuir tarefa a múltiplos agentes e agregar
async def fan_out_fan_in(tarefa, agentes):
    # Fan-out
    resultados = await asyncio.gather(*[
        agente.processar(tarefa)
        for agente in agentes
    ])
    
    # Fan-in
    return agregar_resultados(resultados)
```

## Conclusão

A escolha da arquitetura depende dos requisitos específicos:
- **Supervisor**: Controlo e previsibilidade
- **Swarm**: Flexibilidade e resiliência
- **RAG**: Conhecimento verificável
- **Híbridas**: Combinar vantagens

Experimente diferentes arquiteturas e itere baseado em resultados reais.

## Próximos Passos

- 💡 [Exemplos Práticos](../exemplos/)
- 🎓 [Tutorial Avançado](../tutoriais/AGENTES_AVANCADOS.md)
- 📖 [Boas Práticas](./BOAS_PRATICAS.md)
