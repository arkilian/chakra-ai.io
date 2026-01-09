# Perguntas Frequentes (FAQ)

Respostas às perguntas mais comuns sobre ChakraAgents.ai.

## Índice

- [Geral](#geral)
- [Instalação](#instalação)
- [Configuração](#configuração)
- [Utilização](#utilização)
- [Resolução de Problemas](#resolução-de-problemas)
- [Licenciamento e Custos](#licenciamento-e-custos)

## Geral

### O que é ChakraAgents.ai?

ChakraAgents.ai é uma plataforma open-source para criar, testar e implementar sistemas de IA agêntica. Permite construir agentes inteligentes que podem raciocinar, tomar decisões e colaborar de forma autónoma.

### Qual a diferença entre ChakraAgents.ai e outras frameworks como LangChain?

ChakraAgents.ai é construído sobre LangChain, mas adiciona:
- Interface visual para design de fluxos de trabalho
- Múltiplas arquiteturas pré-configuradas (Supervisor, Swarm, RAG)
- API REST para deployment imediato
- Sistema de monitorização integrado
- Gestão de documentos RAG simplificada

### ChakraAgents.ai é gratuito?

Sim, ChakraAgents.ai é open-source sob licença MIT. No entanto, terá custos associados:
- APIs dos LLMs (OpenAI, Anthropic, etc.)
- Infraestrutura (servidores, bases de dados)
- Serviços cloud (se usar)

### Que fornecedores de LLM são suportados?

Atualmente suporta:
- OpenAI (GPT-3.5, GPT-4, GPT-4-Turbo)
- Anthropic (Claude, Claude-2)
- Google (Vertex AI, PaLM)
- Modelos locais via Ollama
- Qualquer modelo compatível com OpenAI API

### Posso usar modelos locais sem custos de API?

Sim! Use Ollama ou modelos Hugging Face:

```python
from langchain.llms import Ollama

llm = Ollama(model="llama2")
agente = criar_agente(llm=llm)
```

## Instalação

### Quais são os requisitos mínimos?

- **Hardware**: CPU 2 cores, 4GB RAM, 10GB disco
- **Software**: Python 3.10+, Node.js 16+, PostgreSQL 12+
- **Sistema Operativo**: Linux, macOS, ou Windows com WSL2

### A instalação via Docker é mais fácil?

Sim, Docker simplifica muito:

```bash
git clone https://github.com/sudsk/ChakraAgents.ai.git
cd ChakraAgents.ai
docker-compose up -d
```

Tudo fica configurado automaticamente.

### Posso instalar sem Docker?

Sim, siga o [Guia de Instalação Manual](./guias/INSTALACAO.md#instalação-manual).

### Quanto tempo demora a instalação?

- Via Docker: 10-15 minutos
- Manual: 30-45 minutos (primeira vez)

### Como atualizar para versão mais recente?

```bash
# Docker
docker-compose pull
docker-compose up -d

# Manual
git pull origin main
pip install -r requirements.txt --upgrade
npm install
alembic upgrade head
```

## Configuração

### Onde coloco minhas chaves API?

No ficheiro `.env`:

```env
OPENAI_API_KEY=sk-sua-chave-aqui
ANTHROPIC_API_KEY=sk-ant-sua-chave-aqui
```

Nunca commit este ficheiro ao git!

### Como configurar múltiplos ambientes?

Crie ficheiros `.env` separados:

```bash
.env.development
.env.staging
.env.production
```

E carregue conforme ambiente:

```python
from dotenv import load_dotenv
import os

ambiente = os.getenv('AMBIENTE', 'development')
load_dotenv(f'.env.{ambiente}')
```

### Posso usar base de dados diferente de PostgreSQL?

Sim, suporta qualquer BD compatível com SQLAlchemy:
- PostgreSQL (recomendado)
- MySQL/MariaDB
- SQLite (apenas desenvolvimento)

### Como configurar SSL/HTTPS?

Use Nginx ou Traefik como reverse proxy:

```nginx
server {
    listen 443 ssl;
    server_name seu-dominio.com;
    
    ssl_certificate /caminho/cert.pem;
    ssl_certificate_key /caminho/key.pem;
    
    location / {
        proxy_pass http://localhost:8000;
    }
}
```

## Utilização

### Como criar meu primeiro agente?

Siga o [Tutorial Primeiro Agente](./tutoriais/PRIMEIRO_AGENTE.md).

Resumo rápido:
1. Aceda à interface web
2. Clique "Novo Agente"
3. Configure nome e tipo
4. Adicione ferramentas
5. Teste

### Qual arquitetura devo escolher?

Depende do caso de uso:
- **Supervisor**: Fluxos sequenciais bem definidos
- **Swarm**: Problemas que requerem múltiplas perspectivas
- **RAG**: Respostas baseadas em documentos

Veja [Guia de Arquiteturas](./guias/ARQUITETURAS.md) para detalhes.

### Como adicionar documentos para RAG?

Via API:

```bash
curl -X POST http://localhost:8000/api/v1/documents/upload \
  -H "Authorization: Bearer {token}" \
  -F "files=@documento.pdf"
```

Ou programaticamente:

```python
sistema_rag = SistemaRAG("./documentos")
sistema_rag.adicionar_documentos(["novo conteúdo"])
```

### Como implementar agente em produção?

1. Criar agente e testar
2. Usar endpoint `/agents/{id}/execute`
3. Implementar como API REST
4. Configurar monitorização

Veja [Guia de Deployment](./guias/BOAS_PRATICAS.md#deployment).

### Como manter contexto entre perguntas?

Use memória conversacional:

```python
from langchain.memory import ConversationBufferMemory

memoria = ConversationBufferMemory()
agente = initialize_agent(
    ferramentas,
    llm,
    memory=memoria
)
```

### Posso integrar com sistemas existentes?

Sim! ChakraAgents.ai fornece:
- API REST completa
- Webhooks para eventos
- SDKs para Python e JavaScript
- Documentação OpenAPI

## Resolução de Problemas

### "Connection refused" ao iniciar

**Problema**: Serviço não iniciou corretamente

**Soluções**:
1. Verifique se porta está disponível: `lsof -i :8000`
2. Veja logs: `docker-compose logs backend`
3. Verifique variáveis de ambiente

### "Invalid API key"

**Problema**: Chave OpenAI/Anthropic inválida

**Soluções**:
1. Verifique chave no `.env`
2. Confirme que chave está ativa
3. Teste chave diretamente:

```python
import openai
openai.api_key = "sua-chave"
openai.Model.list()
```

### Agente responde muito devagar

**Causas comuns**:
- Modelo muito grande (GPT-4)
- Muitos documentos RAG
- Contexto muito longo

**Soluções**:
1. Use modelo mais rápido (GPT-3.5)
2. Limite documentos RAG (top-k menor)
3. Reduza histórico de conversação
4. Implemente cache

### Respostas inconsistentes

**Problema**: Agente dá respostas diferentes

**Soluções**:
1. Reduza temperatura: `temperature=0`
2. Seja mais específico no prompt
3. Use exemplos (few-shot)
4. Adicione validações

### "Out of memory"

**Problema**: Memória insuficiente

**Soluções**:
1. Reduza tamanho de chunks RAG
2. Processe em lotes menores
3. Aumente RAM do servidor
4. Use paginação

### Docker não inicia

**Problema**: Erro ao iniciar containers

**Soluções**:
1. Verifique Docker está a correr: `docker ps`
2. Reconstrua imagens: `docker-compose build --no-cache`
3. Limpe volumes antigos: `docker-compose down -v`
4. Veja logs detalhados: `docker-compose logs -f`

### Base de dados não conecta

**Problema**: Erro de conexão PostgreSQL

**Soluções**:
1. Verifique se PostgreSQL está a correr
2. Confirme credenciais no `.env`
3. Teste conexão:

```bash
psql -U user -d chakraagents -h localhost
```

### "Module not found"

**Problema**: Dependência Python faltando

**Solução**:

```bash
pip install -r requirements.txt --force-reinstall
```

### Frontend não carrega

**Problemas comuns**:

```bash
# Limpar cache
rm -rf node_modules package-lock.json
npm cache clean --force
npm install

# Verificar porta
lsof -i :3000

# Verificar env
cat frontend/.env
```

## Licenciamento e Custos

### ChakraAgents.ai tem custos?

A plataforma é gratuita (open-source), mas há custos:

**Custos de APIs LLM**:
- OpenAI: $0.002/1K tokens (GPT-3.5) a $0.06/1K (GPT-4)
- Anthropic: Similar ao OpenAI
- Modelos locais: Grátis (mas custos de hardware)

**Custos de Infraestrutura**:
- Servidor: €5-50/mês (VPS básico)
- Base de dados: Incluído ou €5-20/mês
- Armazenamento: €0.01-0.10/GB/mês

**Estimativa mensal (pequeno projeto)**:
- APIs LLM: €10-50
- Servidor: €10-20
- Total: €20-70/mês

### Como reduzir custos?

1. **Use modelos mais baratos**:
   - GPT-3.5 em vez de GPT-4
   - Modelos locais quando possível

2. **Implemente cache**:
   - Cachear perguntas frequentes
   - Usar Redis

3. **Otimize prompts**:
   - Prompts mais curtos
   - Menos exemplos few-shot

4. **Limite contexto**:
   - Reduzir histórico conversacional
   - Top-k menor em RAG

5. **Monitorize uso**:
   - Alertas de custos
   - Limites por utilizador

### Posso usar comercialmente?

Sim! Licença MIT permite uso comercial. No entanto:
- Respeite termos de uso dos LLMs
- Implemente medidas de segurança
- Considere liability/garantias

### Preciso de suporte enterprise?

Para suporte dedicado, considere:
- Consultoria especializada
- SLAs garantidos
- Desenvolvimento personalizado
- Formação de equipas

Contacte: enterprise@chakraagents.ai

## Contribuir e Comunidade

### Como contribuir?

1. Fork o repositório
2. Crie branch para funcionalidade
3. Implemente com testes
4. Submeta Pull Request

Veja [Guia de Contribuição](../CONTRIBUTING.md).

### Onde obter ajuda?

- 💬 [Discord](https://discord.gg/chakraagents)
- 📧 Email: support@chakraagents.ai
- 🐛 [GitHub Issues](https://github.com/sudsk/ChakraAgents.ai/issues)
- 📚 [Documentação](../README.md)

### Como reportar bug?

Abra issue no GitHub com:
- Descrição do problema
- Passos para reproduzir
- Comportamento esperado vs atual
- Versão e ambiente
- Logs relevantes

### Posso sugerir funcionalidades?

Sim! Abra discussão no GitHub:
- Descreva funcionalidade
- Caso de uso
- Benefícios esperados
- Implementação sugerida (opcional)

## Questões Técnicas Avançadas

### Como escalar horizontalmente?

Use múltiplas instâncias com load balancer:

```yaml
# docker-compose.scale.yml
services:
  backend:
    deploy:
      replicas: 3
  
  nginx:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
```

### Como implementar autenticação custom?

Estenda sistema de autenticação:

```python
from fastapi_users import FastAPIUsers

fastapi_users = FastAPIUsers(
    get_user_manager,
    [auth_backend],
)

app.include_router(
    fastapi_users.get_auth_router(auth_backend),
    prefix="/auth",
)
```

### Como adicionar fornecedor LLM custom?

Implemente interface LangChain:

```python
from langchain.llms.base import LLM

class MeuLLMCustom(LLM):
    def _call(self, prompt: str, stop: Optional[List[str]] = None) -> str:
        # Sua lógica aqui
        return resposta
    
    @property
    def _llm_type(self) -> str:
        return "custom"
```

### Suporta múltiplas línguas?

Sim! LLMs modernos são multilíngues. Configure:

```python
prompt_template = """
Responda sempre em {lingua}.

Pergunta: {pergunta}
"""
```

### Como implementar fine-tuning?

ChakraAgents.ai usa LLMs via API. Para fine-tuning:

1. Colete dados de treino
2. Use plataforma do fornecedor (OpenAI, etc.)
3. Configure agente para usar modelo fine-tuned

```python
llm = OpenAI(model="ft:gpt-3.5-turbo:seu-modelo")
```

---

**Não encontrou sua pergunta?**

- Consulte [Documentação Completa](../README.md)
- Pergunte no [Discord](https://discord.gg/chakraagents)
- Abra [Issue no GitHub](https://github.com/sudsk/ChakraAgents.ai/issues)
