# API Gateway - Plano de Execução Detalhado

## Visão Geral

O **API Gateway** é a camada central de segurança e roteamento da plataforma SafeHire AI. Atua como proxy reverso, validador de JWT e distribuidor inteligente de tráfego para os serviços internos.

### Propósito
- Validar tokens JWT de todas as requisições
- Decodificar metadados do usuário (id, role)
- Rotear tráfego para serviços internos apropriados
- Injetar headers `X-User-Id` e `X-User-Role` nos serviços downstream
- Aplicar rate limiting e segurança
- Expor apenas endpoints necessários externamente

### Stack Tecnológica
- **Framework**: FastAPI (assíncrono)
- **Linguagem**: Python 3.11+
- **Autenticação**: JWT validation
- **Proxy**: httpx para requisições upstream
- **Cache**: Valkey para rate limiting
- **Formatting**: black, isort

---

## Arquitetura do Serviço

```
api-gateway/
├── app/
│   ├── __init__.py
│   ├── main.py              # Entry point FastAPI
│   ├── config.py            # Configurações
│   ├── middleware/          # Middleware customizados
│   │   ├── __init__.py
│   │   ├── auth.py          # JWT validation middleware
│   │   ├── rate_limit.py    # Rate limiting
│   │   └── security.py      # Security headers
│   ├── routers/             # Rotas externas
│   │   ├── __init__.py
│   │   ├── auth.py          # Rotas para auth-service
│   │   ├── vagas.py         # Rotas para core-management-api
│   │   └── admin.py         # Rotas admin
│   ├── services/            # Serviços de roteamento
│   │   ├── __init__.py
│   │   ├── jwt_service.py   # JWT operations
│   │   ├── proxy_service.py # Proxy para serviços upstream
│   │   └── rate_limit_service.py  # Rate limiting logic
│   └── utils/               # Utilitários
│       ├── __init__.py
│       └── headers.py       # Header manipulation
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   │   ├── __init__.py
│   │   ├── test_jwt_service.py
│   │   ├── test_proxy_service.py
│   │   └── test_rate_limit.py
│   ├── integration/
│   │   ├── __init__.py
│   │   ├── test_auth_routing.py
│   │   └── test_core_routing.py
│   └── fakes/
│       ├── __init__.py
│       ├── fake_httpx.py
│       └── fake_valkey.py
├── Dockerfile
├── requirements.txt
├── .env.example
├── pyproject.toml
├── README.md
└── CLAUDE.md
```

---

## Roadmap de Implementação

### Fase 1: Configuração Base (Dia 1)
- [ ] Criar estrutura de pastas do projeto
- [ ] Criar `pyproject.toml` com configs
- [ ] Criar `requirements.txt` com dependências
- [ ] Criar `Dockerfile`
- [ ] Criar `.env.example`
- [ ] Criar `conftest.py` com fixtures

### Fase 2: Configuração e Utils (Dia 1-2)
- [ ] Implementar `config.py` com Pydantic Settings
- [ ] Implementar `utils/headers.py`:
  - `build_upstream_headers()`
  - `extract_auth_token()`
  - `inject_user_headers()`

### Fase 3: JWT Service (Dia 2)
- [ ] Implementar `services/jwt_service.py`:
  - `decode_jwt_token()`
  - `verify_token_signature()`
  - `extract_user_metadata()`
  - `validate_token_expiration()`

### Fase 4: Proxy Service (Dia 2-3)
- [ ] Implementar `services/proxy_service.py`:
  - `forward_request_to_auth()`
  - `forward_request_to_core()`
  - `forward_request_with_auth()`
  - `handle_upstream_response()`
  - `handle_upstream_error()`

### Fase 5: Rate Limiting Service (Dia 3)
- [ ] Implementar `services/rate_limit_service.py`:
  - `check_rate_limit()`
  - `increment_request_count()`
  - `reset_rate_limit()`
  - `get_remaining_requests()`

### Fase 6: Middleware Layer (Dia 3-4)
- [ ] Implementar `middleware/auth.py`:
  - `AuthMiddleware` para validação JWT
  - `require_auth()` decorator
  - `require_role()` decorator
- [ ] Implementar `middleware/rate_limit.py`:
  - `RateLimitMiddleware`
- [ ] Implementar `middleware/security.py`:
  - Security headers injection
  - CORS configuration

### Fase 7: External Routes (Dia 4-5)
- [ ] Implementar `routers/auth.py`:
  - `POST /api/auth/register`
  - `POST /api/auth/login`
  - `POST /api/auth/refresh`
  - `POST /api/auth/verify`
- [ ] Implementar `routers/vagas.py`:
  - `GET /api/vagas`
  - `GET /api/vagas/{id}`
  - `POST /api/vagas/{id}/aplicar` (auth)
- [ ] Implementar `routers/admin.py`:
  - `GET /api/admin/dashboard` (auth + recrutador)
  - `POST /api/admin/vagas` (auth + recrutador)
  - `GET /api/admin/candidatos/{id}` (auth + recrutador)

### Fase 8: Entry Point (Dia 5)
- [ ] Implementar `app/main.py`:
  - Inicializar FastAPI
  - Configurar CORS
  - Registrar middleware
  - Registrar routers
  - Health check
- [ ] Configurar logging estruturado JSON

### Fase 9: Testes (Dia 5-6)
- [ ] Testes unitários de JWT service
- [ ] Testes unitários de proxy service
- [ ] Testes unitários de rate limit
- [ ] Testes de integração de roteamento
- [ ] Testes de middleware
- [ ] Testes de segurança
- [ ] Testes de rate limiting

### Fase 10: Documentação (Dia 6-7)
- [ ] OpenAPI/Swagger docs
- [ ] README.md
- [ ] Diagramas de arquitetura

---

## TodoList Detalhada

### Configuração
- [x] Criar estrutura de pastas
- [ ] Criar `pyproject.toml`:
  ```toml
  [tool.black]
  line-length = 88
  target-version = ['py311']

  [tool.isort]
  profile = "black"

  [tool.pytest.ini_options]
  testpaths = ["tests"]

  [tool.mypy]
  strict = true
  ```
- [ ] Criar `requirements.txt`:
  ```
  fastapi>=0.104.0
  uvicorn[standard]>=0.24.0
  pydantic>=2.0.0
  pydantic-settings>=2.0.0
  httpx>=0.25.0
  python-jose[cryptography]>=3.3.0
  redis>=5.0.0
  pytest>=7.4.0
  pytest-asyncio>=0.21.0
  pytest-cov>=4.1.0
  black>=23.10.0
  isort>=5.12.0
  mypy>=1.6.0
  ```
- [ ] Criar `Dockerfile`:
  ```dockerfile
  FROM python:3.11-slim

  WORKDIR /app

  COPY requirements.txt .
  RUN pip install --no-cache-dir -r requirements.txt

  COPY . .

  EXPOSE 8000

  CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
  ```
- [ ] Criar `.env.example`:
  ```env
  # Auth Service
  AUTH_SERVICE_URL=http://auth-service:8000

  # Core Management API
  CORE_SERVICE_URL=http://core-management-api:8000

  # JWT
  JWT_SECRET_KEY=your-secret-key-here
  JWT_ALGORITHM=HS256

  # Rate Limiting (Valkey)
  VALKEY_URL=redis://valkey:6379/0
  RATE_LIMIT_REQUESTS=100
  RATE_LIMIT_WINDOW=60

  # CORS
  ALLOWED_ORIGINS=http://localhost:3000,http://frontend-app:3000

  # App
  APP_NAME=API Gateway
  APP_VERSION=1.0.0
  DEBUG=False
  ```

### Configuração
- [ ] `app/config.py`:
  ```python
  from pydantic_settings import BaseSettings

  class Settings(BaseSettings):
    auth_service_url: str
    core_service_url: str
    jwt_secret_key: str
    jwt_algorithm: str = "HS256"
    valkey_url: str
    rate_limit_requests: int = 100
    rate_limit_window: int = 60
    allowed_origins: str
    debug: bool = False

    class Config:
      env_file = ".env"

  settings = Settings()
  ```

### Utils Headers
- [ ] `app/utils/headers.py`:
  ```python
  from fastapi import Request
  from typing import Dict

  USER_ID_HEADER = "X-User-Id"
  USER_ROLE_HEADER = "X-User-Role"

  def extract_auth_token(request: Request) -> str | None:
    """Extrai token JWT do header Authorization."""
    auth_header = request.headers.get("authorization", "")
    if auth_header.startswith("Bearer "):
      return auth_header[7:]
    return None

  def inject_user_headers(
    base_headers: Dict[str, str],
    user_id: int,
    user_role: str
  ) -> Dict[str, str]:
    """Injeta headers de usuário nos headers upstream."""
    headers = base_headers.copy()
    headers[USER_ID_HEADER] = str(user_id)
    headers[USER_ROLE_HEADER] = user_role
    return headers
  ```

### JWT Service
- [ ] `app/services/jwt_service.py`:
  ```python
  from jose import JWTError, jwt
  from app.config import settings

  def decode_jwt_token(token: str) -> dict:
    """Decodifica JWT token e retorna payload."""
    try:
      payload = jwt.decode(
        token,
        settings.jwt_secret_key,
        algorithms=[settings.jwt_algorithm]
      )
      return payload
    except JWTError as e:
      raise InvalidTokenError(f"Token inválido: {e}")

  def extract_user_metadata(payload: dict) -> tuple[int, str]:
    """Extrai user_id e role do payload JWT."""
    user_id = payload.get("sub")
    role = payload.get("role")
    if not user_id or not role:
      raise InvalidTokenError("Token sem metadados necessários")
    return int(user_id), role

  def validate_token_expiration(payload: dict) -> None:
    """Valida se token não expirou."""
    exp = payload.get("exp")
    if exp and exp < datetime.utcnow().timestamp():
      raise TokenExpiredError("Token expirado")
  ```

### Proxy Service
- [ ] `app/services/proxy_service.py`:
  ```python
  import httpx
  from typing import Dict, Any
  from fastapi import Request

  class ProxyService:
    def __init__(self, timeout: float = 30.0):
      self.timeout = timeout

    async def forward_request(
      self,
      method: str,
      url: str,
      headers: Dict[str, str],
      body: bytes | None = None,
      query_params: Dict[str, Any] | None = None
    ) -> httpx.Response:
      """Encaminha requisição para serviço upstream."""
      async with httpx.AsyncClient(timeout=self.timeout) as client:
        response = await client.request(
          method=method,
          url=url,
          headers=headers,
          content=body,
          params=query_params
        )
        return response

    async def forward_to_auth(
      self,
      request: Request,
      path: str
    ) -> httpx.Response:
      """Encaminha para auth-service."""
      ...
  ```

### Rate Limit Service
- [ ] `app/services/rate_limit_service.py`:
  ```python
  import redis.asyncio as redis
  from app.config import settings

  class RateLimitService:
    def __init__(self):
      self.redis = redis.from_url(settings.valkey_url, decode_responses=True)

    async def check_rate_limit(self, identifier: str) -> bool:
      """Verifica se identificador atingiu limite."""
      key = f"ratelimit:{identifier}"
      count = await self.redis.incr(key)

      if count == 1:
        await self.redis.expire(key, settings.rate_limit_window)

      return count <= settings.rate_limit_requests

    async def get_remaining_requests(self, identifier: str) -> int:
      """Retorna requisições restantes."""
      key = f"ratelimit:{identifier}"
      count = await self.redis.get(key) or 0
      return max(0, settings.rate_limit_requests - int(count))
  ```

### Middleware Auth
- [ ] `app/middleware/auth.py`:
  ```python
  from fastapi import Request, HTTPException, status
  from starlette.middleware.base import BaseHTTPMiddleware
  from app.services.jwt_service import decode_jwt_token, extract_user_metadata
  from app.utils.headers import inject_user_headers, USER_ID_HEADER, USER_ROLE_HEADER

  class AuthMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
      # Rotas públicas não precisam de auth
      if request.url.path in ["/api/auth/login", "/api/auth/register", "/health"]:
        return await call_next(request)

      token = extract_auth_token(request)
      if not token:
        raise HTTPException(
          status_code=status.HTTP_401_UNAUTHORIZED,
          detail="Token não fornecido"
        )

      payload = decode_jwt_token(token)
      user_id, user_role = extract_user_metadata(payload)

      # Injeta metadados no state
      request.state.user_id = user_id
      request.state.user_role = user_role

      return await call_next(request)
  ```

### External Routes
- [ ] `app/routers/auth.py`:
  ```python
  from fastapi import APIRouter, Request, Response
  from app.services.proxy_service import ProxyService
  from app.config import settings

  router = APIRouter(prefix="/api/auth", tags=["Auth"])
  proxy = ProxyService()

  @router.post("/login")
  async def login(request: Request, response: Response):
    """Proxy para login no auth-service."""
    upstream_response = await proxy.forward_to_auth(request, "/auth/login")
    return Response(
      content=upstream_response.content,
      status_code=upstream_response.status_code,
      headers=dict(upstream_response.headers)
    )

  @router.post("/register")
  async def register(request: Request, response: Response):
    """Proxy para registro no auth-service."""
    ...
  ```

- [ ] `app/routers/vagas.py`:
  ```python
  from fastapi import APIRouter, Request, Response, Depends
  from app.services.proxy_service import ProxyService
  from app.services.jwt_service import extract_user_metadata
  from app.utils.headers import inject_user_headers

  router = APIRouter(prefix="/api/vagas", tags=["Vagas"])
  proxy = ProxyService()

  @router.get("")
  async def listar_vagas(request: Request):
    """Lista vagas públicas (sem autenticação)."""
    upstream_response = await proxy.forward_to_core(request, "/vagas")
    ...

  @router.post("/{vaga_id}/aplicar")
  async def aplicar_vaga(vaga_id: str, request: Request):
    """Aplica à vaga (requer autenticação)."""
    user_id = request.state.user_id
    user_role = request.state.user_role

    if user_role != "candidato":
      raise HTTPException(status_code=403, detail="Apenas candidatos podem aplicar")

    headers = inject_user_headers({}, user_id, user_role)
    upstream_response = await proxy.forward_to_core_with_auth(request, f"/vagas/{vaga_id}/aplicar", headers)
    ...
  ```

### Tests
- [ ] `tests/conftest.py`:
  ```python
  import pytest
  from httpx import AsyncClient, ASGITransport
  from app.main import app
  from unittest.mock import AsyncMock, patch

  @pytest.fixture
  async def client():
    """Cliente HTTP assíncrono para testes."""
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as ac:
      yield ac

  @pytest.fixture
  def mock_proxy_service():
    """Mock do proxy service."""
    with patch("app.services.proxy_service.ProxyService") as mock:
      instance = AsyncMock()
      mock.return_value = instance
      yield instance
  ```

- [ ] `tests/unit/test_jwt_service.py`:
  ```python
  import pytest
  from app.services.jwt_service import decode_jwt_token, extract_user_metadata

  def test_decode_valid_token():
    """Testa decodificação de token válido."""
    ...

  def test_decode_invalid_token_raises_error():
    """Testa erro ao decodificar token inválido."""
    ...
  ```

- [ ] `tests/integration/test_auth_routing.py`:
  ```python
  import pytest
  from httpx import AsyncClient

  @pytest.mark.asyncio
  async def test_login_proxy(client: AsyncClient, mock_proxy_service):
    """Testa proxy de login para auth-service."""
    ...
  ```

---

## Validação e Critérios de Aceitação

### Funcional
- [ ] Requisições públicas são roteadas sem auth
- [ ] Requisições privadas validam JWT antes de rotear
- [ ] Token inválido retorna 401
- [ ] Token expirado retorna 401
- [ ] Headers `X-User-Id` e `X-User-Role` são injetados
- [ ] Rate limiting bloqueia requisições excessivas
- [ ] CORS permite origins configurados
- [ ] Respostas upstream são retornadas corretamente

### Técnico
- [ ] Proxy adiciona < 50ms de latência
- [ ] Async operations não bloqueiam
- [ ] Conexões httpx são pooladas
- [ ] Valkey connection pool funciona
- [ ] Errors upstream são tratados

### Segurança
- [ ] Tokens nunca são logados
- [ ] Sensitive headers não são repassados
- [ ] Rate limiting por IP/Token
- [ ] Security headers inyectados
- [ ] CORS restringe origins

### Testes
- [ ] Coverage >= 80%
- [ ] Fakes para httpx e valkey
- [ ] Testes de middleware auth
- [ ] Testes de roteamento

### Código
- [ ] Funções 4-20 linhas
- [ ] Arquivos < 500 linhas
- [ ] Nomes únicos
- [ ] Tipagem estrita
- [ ] Early returns

---

## Comandos de Desenvolvimento

```bash
# Instalar dependências
pip install -r requirements.txt

# Rodar em desenvolvimento
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Formatar código
black app tests
isort app tests

# Verificar tipos
mypy app

# Rodar testes
pytest tests -v --cov=app --cov-report=html
```

---

## Próximos Passos

Após completar este serviço:

1. Testar integração com auth-service
2. Testar integração com core-management-api
3. Monitorar latência de proxy
4. Configurar alertas de rate limit

---

## Referências

- `PROJECT_CONTEXT.md` - Arquitetura geral
- `CLAUDE.md` - Regras de código
- `plano-geral-execucao.md` - Roadmap global