# API Gateway - CLAUDE.md

## Stack Tecnológica
- **Framework:** FastAPI (Python 3.11+)
- **Reverse Proxy:** Roteamento inteligente de tráfego
- **Segurança:** Decodificação de JWT, validação de segurança
- **ORM:** SQLAlchemy (apenas para validação, sem schema próprio)
- **Formatação:** `black` e `isort`

## Responsabilidades
- Validador central de segurança
- Decodificação e verificação de tokens JWT
- Roteamento inteligente para serviços internos (auth-service, core-management-api)
- Injeção de headers com metadados do usuário (`X-User-Id`, `X-User-Role`)

## Comandos de Desenvolvimento

### Instalar dependências
```bash
pip install -r requirements.txt
```

### Rodar em modo de desenvolvimento
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Rodar testes
```bash
pytest -v
```

### Formatar código
```bash
black app/ tests/
isort app/ tests/
```

## Regras de Code Style
- Funções entre 4 e 20 linhas
- Arquivos com menos de 500 linhas
- Nomes específicos (evite sufixos genéricos)
- Tipagem estrita (proibido `any`)
- Early returns, máximo 2 níveis de indentação
- Exceções semânticas com valor ofensivo
- Preservar comentários existentes
- Docstrings em funções públicas