
# rag-fastapi

RAG FastAPI — um exemplo simples de aplicação Retrieval-Augmented Generation (RAG) construída com FastAPI.

Este projeto permite fazer upload de documentos PDF, transformar o conteúdo em chunks, gerar embeddings via OpenAI, armazenar os vetores em um banco MongoDB com suporte a busca vetorial e responder perguntas utilizando apenas o contexto recuperado dos chunks.

## Funcionalidades

- Upload de PDFs (limite de 15 páginas) via endpoint multipart.
- Chunking simples do texto extraído do PDF.
- Geração de embeddings usando a API OpenAI.
- Armazenamento de chunks e embeddings em MongoDB.
- Busca vetorial por similaridade (pipeline com $vectorSearch).
- Endpoint de pergunta (`/api/v1/rag/ask`) que retorna resposta gerada pela LLM com referências às fontes (IDs dos chunks).

## Estrutura principal

- `app/main.py` — instancia a API FastAPI e inclui as rotas.
- `app/api/v1/rag.py` — endpoints de upload e de perguntas (RAG).
- `app/rag/chunk.py` — lógica de chunking simples.
- `app/rag/store.py` — funções de inserção de documentos, embeddings e busca vetorial.
- `app/llm/openai_client.py` — integração com OpenAI para embeddings e chat completion.
- `app/db/mongo.py` — cliente MongoDB assíncrono (motor).
- `app/schemas/rag.py` — modelos pydantic das requests/responses.

## Requisitos

- Python 3.10+
- MongoDB com suporte a busca vetorial (ou indexação/vetor equivalente configurado)
- Conta e chave da OpenAI
- Dependências listadas em `app/requirements.txt`

## Variáveis de ambiente

Coloque as variáveis em um arquivo `.env` na raiz do projeto ou no diretório onde executar a app:

- `OPENAI_API_KEY` — chave da API OpenAI (obrigatório)
- `OPENAI_EMBEDDING_MODEL` — modelo de embeddings (opcional, padrão: `text-embedding-3-large`)
- `OPENAI_CHAT_MODEL` — modelo de chat (opcional, padrão: `gpt-4o-mini`)
- `MONGODB_URI` — URI de conexão com MongoDB (opcional, padrão: `mongodb://localhost:27017`)
- `MONGODB_DB` — nome do banco de dados MongoDB (opcional, padrão: `curso_api`)

## Como executar (desenvolvimento)

1. Criar e preencher um arquivo `.env` com pelo menos `OPENAI_API_KEY` e `MONGODB_URI` (se necessário).
2. Instalar dependências:

```powershell
cd app
pip install -r requirements.txt
```

3. Executar a aplicação com uvicorn:

```powershell
cd ..
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Após iniciar, a documentação automática estará disponível em `http://localhost:8000/docs`.

## Endpoints principais

- POST `/api/v1/rag/documents` — upload de PDF
	- Multipart form: `file` (PDF) e `title` (string)
	- Resposta: `{ "doc_id": "<id>", "chunks": <n> }`

- POST `/api/v1/rag/ask` — pergunta sobre um documento
	- JSON body: `{ "doc_id": "<id>", "question": "...", "k": 5 }`
	- Resposta: `{ "answer": "...", "sources": ["chunk_id=<id>", ...] }`

Exemplo (curl):

```bash
curl -F "file=@/caminho/para/documento.pdf" -F "title=MeuDoc" http://localhost:8000/api/v1/rag/documents

curl -X POST http://localhost:8000/api/v1/rag/ask -H "Content-Type: application/json" -d '{"doc_id":"<id>","question":"Qual é o objetivo do documento?","k":3}'
```

## Observações

- O projeto é um exemplo e não contém autenticação por padrão. Para produção, adicionar autenticação, validação adicional, limites de taxa e tratamentos de segurança.
- Ajuste os parâmetros de chunking, modelo e temperatura conforme suas necessidades.

## Próximos passos sugeridos

- Adicionar exemplos automatizados nos testes.
- Adicionar badges (build, coverage) e instruções de deploy.
- Implementar autenticação e controle de acesso aos documentos.

---

Licença: ver `LICENSE`.

