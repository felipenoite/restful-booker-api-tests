# 🐛 Análise de Bugs — Restful-Booker API

> Bugs identificados durante a execução dos testes. Cada item contém severidade, passos para reprodução, resultado atual vs esperado, impacto e sugestão de correção.

---

## Resumo

| ID | Severidade | Endpoint | Status |
|---|---|---|---|
| BUG-001 | 🔴 Alta | `POST /booking` | Aberto |
| BUG-002 | 🔴 Alta | `POST /booking` | Aberto |
| BUG-003 | 🟠 Média | `POST /booking` | Aberto |
| BUG-004 | 🟡 Baixa | `POST /auth` | Aberto |
| BUG-005 | 🔴 Alta | `POST /booking` | Aberto |

---

## BUG-001 — Ausência de validação de campos obrigatórios

| Campo | Detalhe |
|---|---|
| **ID** | BUG-001 |
| **Severidade** | 🔴 Alta |
| **Endpoint** | `POST /booking` |
| **Cenário relacionado** | POST-03, POST-04 |

**Descrição**

A API aceita criar reservas mesmo sem campos obrigatórios como `firstname`, `lastname` e `bookingdates`, retornando status `200` e persistindo dados incompletos.

**Passos para reproduzir**

1. Fazer `POST /booking` com o body abaixo:
```json
{
  "lastname": "Noite",
  "totalprice": 300,
  "depositpaid": true,
  "bookingdates": {
    "checkin": "2025-05-01",
    "checkout": "2025-05-05"
  }
}
```
2. Observar a resposta.

**Resultado atual**

```
HTTP 200 OK
{ "bookingid": 123, "booking": { "firstname": null, ... } }
```

**Resultado esperado**

```
HTTP 400 Bad Request
{ "error": "O campo 'firstname' é obrigatório." }
```

**Impacto**

Hóspedes cadastrados sem nome; dados inconsistentes no banco; possíveis falhas em integrações e relatórios que dependem desses campos.

**Sugestão de correção**

Implementar validação de schema no controller antes de persistir (ex: Joi no Node.js, Bean Validation no Java). Retornar `400 Bad Request` com mensagem indicando o campo ausente.

---

## BUG-002 — Sem validação de lógica de negócio nas datas

| Campo | Detalhe |
|---|---|
| **ID** | BUG-002 |
| **Severidade** | 🔴 Alta |
| **Endpoint** | `POST /booking` |
| **Cenário relacionado** | POST-06 |

**Descrição**

A API aceita reservas onde a data de `checkout` é anterior à data de `checkin`, o que é logicamente impossível num sistema de reservas de hotel.

**Passos para reproduzir**

1. Fazer `POST /booking` com o body abaixo:
```json
{
  "firstname": "Teste",
  "lastname": "Data Inválida",
  "totalprice": 200,
  "depositpaid": true,
  "bookingdates": {
    "checkin": "2025-12-31",
    "checkout": "2025-01-01"
  }
}
```
2. Observar a resposta.

**Resultado atual**

```
HTTP 200 OK — reserva criada com datas inválidas
```

**Resultado esperado**

```
HTTP 400 Bad Request
{ "error": "A data de checkout deve ser posterior ao checkin." }
```

**Impacto**

Reservas inválidas persistidas no banco; cálculo incorreto de diárias; inconsistência em relatórios financeiros e de ocupação.

**Sugestão de correção**

Adicionar regra de negócio na camada de serviço: validar que `checkout > checkin` antes de persistir. Retornar `400` com mensagem explicativa.

---

## BUG-003 — API aceita totalprice com valor negativo

| Campo | Detalhe |
|---|---|
| **ID** | BUG-003 |
| **Severidade** | 🟠 Média |
| **Endpoint** | `POST /booking` |
| **Cenário relacionado** | POST-05 |

**Descrição**

O campo `totalprice` aceita valores negativos sem qualquer validação, permitindo que reservas com preço `-999` sejam criadas com sucesso.

**Passos para reproduzir**

1. Fazer `POST /booking` com o body abaixo:
```json
{
  "firstname": "Teste",
  "lastname": "Negativo",
  "totalprice": -999,
  "depositpaid": true,
  "bookingdates": {
    "checkin": "2025-06-01",
    "checkout": "2025-06-05"
  }
}
```
2. Observar a resposta.

**Resultado atual**

```
HTTP 200 OK — reserva criada com totalprice: -999
```

**Resultado esperado**

```
HTTP 400 Bad Request
{ "error": "O campo 'totalprice' deve ser um número positivo." }
```

**Impacto**

Dados financeiros corrompidos; possível exploração em sistemas integrados de pagamento ou geração de relatórios incorretos.

**Sugestão de correção**

Validar que `totalprice > 0` antes de persistir. Retornar `400` com mensagem indicando que o campo deve ser positivo.

---

## BUG-004 — Endpoint /auth retorna HTTP 200 para credenciais inválidas

| Campo | Detalhe |
|---|---|
| **ID** | BUG-004 |
| **Severidade** | 🟡 Baixa |
| **Endpoint** | `POST /auth` |
| **Cenário relacionado** | AUTH-02, AUTH-03, AUTH-04 |

**Descrição**

Quando as credenciais fornecidas são inválidas, a API retorna status `200 OK` com body `{ "reason": "Bad credentials" }` em vez do correto `401 Unauthorized`, violando o padrão REST definido pela RFC 7235.

**Passos para reproduzir**

1. Fazer `POST /auth` com o body abaixo:
```json
{
  "username": "admin",
  "password": "senha_errada"
}
```
2. Observar o status code da resposta.

**Resultado atual**

```
HTTP 200 OK
{ "reason": "Bad credentials" }
```

**Resultado esperado**

```
HTTP 401 Unauthorized
{ "reason": "Bad credentials" }
```

**Impacto**

Clientes HTTP que verificam apenas o status code para determinar sucesso ou falha não detectarão a falha de autenticação. Dificulta integração com ferramentas e middlewares que seguem o padrão REST.

**Sugestão de correção**

Retornar `401 Unauthorized` para falhas de autenticação, conforme RFC 7235. O body com a mensagem de erro pode ser mantido.

---

## BUG-005 — Campo additionalneeds armazena e reflete payload XSS sem sanitização

| Campo | Detalhe |
|---|---|
| **ID** | BUG-005 |
| **Severidade** | 🔴 Alta |
| **Endpoint** | `POST /booking` |
| **Cenário relacionado** | SEC-02 |

**Descrição**

A API aceita e persiste conteúdo JavaScript malicioso no campo `additionalneeds` sem qualquer sanitização, refletindo o payload exatamente como foi enviado na response. Isso caracteriza uma vulnerabilidade de **Stored XSS (Cross-Site Scripting Armazenado)** — o tipo mais crítico de XSS, pois o payload fica salvo no banco e pode ser executado sempre que os dados forem exibidos em uma interface web.

**Passos para reproduzir**

1. Fazer `POST /booking` com o body abaixo:
```json
{
  "firstname": "XSS",
  "lastname": "Test",
  "totalprice": 100,
  "depositpaid": false,
  "bookingdates": {
    "checkin": "2025-01-01",
    "checkout": "2025-01-02"
  },
  "additionalneeds": "<script>alert('XSS')</script>"
}
```
2. Observar a response — o payload `<script>` é refletido integralmente.
3. Fazer `GET /booking/{id}` com o ID gerado e observar que o payload continua armazenado.

**Resultado atual**

```
HTTP 200 OK
{
  "bookingid": 5732,
  "booking": {
    ...
    "additionalneeds": "<script>alert('XSS')</script>"
  }
}
```

**Resultado esperado**

O campo deveria ser sanitizado antes de persistir, retornando:
```json
{ "additionalneeds": "&lt;script&gt;alert('XSS')&lt;/script&gt;" }
```
Ou a requisição deveria ser rejeitada com `400 Bad Request`.

**Impacto**

Se os dados da API forem exibidos em qualquer interface web sem tratamento adicional no frontend, o script será executado no navegador do usuário. Isso pode levar a roubo de sessão, redirecionamento malicioso e comprometimento de dados de usuários.

**Sugestão de correção**

Sanitizar entradas de texto no backend antes de persistir (ex: bibliotecas como `DOMPurify`, `sanitize-html` ou encoding de caracteres especiais HTML). A responsabilidade de sanitização não deve ser delegada exclusivamente ao frontend.

---

## 💡 Sugestões Gerais de Melhoria

Além dos bugs encontrados, os seguintes pontos foram identificados como oportunidades de melhoria na API:

1. **Paginação no GET /booking** — A listagem retorna todos os registros sem paginação, o que pode causar problemas de performance em escala.
2. **Rate Limiting** — O endpoint `/auth` não possui limite de tentativas, ficando vulnerável a ataques de força bruta.
3. **Versionamento da API** — Não há versionamento (ex: `/v1/booking`), dificultando a evolução sem quebrar clientes existentes.
4. **Mensagens de erro padronizadas** — Os erros retornados não seguem um padrão consistente de estrutura JSON.
5. **Documentação de autenticação** — A documentação não especifica o tempo de expiração do token gerado.

---

*[← Voltar ao README](../README.md)*
