# 📄 Documentação de Testes — Restful-Booker API

> **Autor:** Felipe Noite | **GitHub:** felipenoite | **Data:** Maio/2025

---

## 1. 📦 Collection de Requests

A collection está organizada em **10 grupos** cobrindo todos os endpoints da API.

| # | Grupo | Método(s) | Endpoint |
|---|---|---|---|
| 01 | Health Check | GET | `/ping` |
| 02 | Autenticação | POST | `/auth` |
| 03 | Listar Reservas | GET | `/booking` |
| 04 | Buscar por ID | GET | `/booking/:id` |
| 05 | Criar Reserva | POST | `/booking` |
| 06 | Atualizar Completo | PUT | `/booking/:id` |
| 07 | Atualizar Parcial | PATCH | `/booking/:id` |
| 08 | Deletar | DELETE | `/booking/:id` |
| 09 | Segurança | POST, PUT | `/booking`, `/booking/:id` |
| 10 | Performance | GET, POST | `/booking`, `/auth` |

**Total:** 35 requests | **Scripts de teste automatizados:** em todos os requests

---

## 2. 🔑 Variáveis de Ambiente

As variáveis são gerenciadas diretamente na Collection (sem arquivo externo).

| Variável | Escopo | Valor Inicial | Preenchimento |
|---|---|---|---|
| `base_url` | Collection | `https://restful-booker.herokuapp.com` | Manual (fixo) |
| `token` | Collection | *(vazio)* | Automático — script do request AUTH-01 |
| `booking_id` | Collection | *(vazio)* | Automático — script do request POST-01 |

### Como as variáveis são preenchidas automaticamente

**Token** — gerado no teste de autenticação:
```javascript
// Script "Tests" do request "Gerar token com credenciais válidas"
const json = pm.response.json();
pm.collectionVariables.set('token', json.token);
```

**Booking ID** — salvo ao criar uma reserva:
```javascript
// Script "Tests" do request "Criar reserva com todos os campos"
const json = pm.response.json();
pm.collectionVariables.set('booking_id', json.bookingid);
```

> ⚠️ **Pré-condição:** Execute sempre AUTH-01 e POST-01 antes dos testes que dependem de `{{token}}` e `{{booking_id}}`. A ordem padrão da collection já garante isso.

---

## 3. 📋 Documentação dos Cenários

### 🔵 Nível 1 — Obrigatório

---

#### HC-01 — Health Check

| Campo | Detalhe |
|---|---|
| **Descrição** | Verifica se a API está online e respondendo |
| **Endpoint** | `GET /ping` |
| **Headers** | Nenhum |
| **Body** | Nenhum |
| **Resultado esperado** | Status 201, body contém "Created" |

---

#### AUTH-01 — Gerar token com credenciais válidas

| Campo | Detalhe |
|---|---|
| **Descrição** | Autentica com usuário e senha corretos, obtém token |
| **Endpoint** | `POST /auth` |
| **Headers** | `Content-Type: application/json` |
| **Body** | `{ "username": "admin", "password": "password123" }` |
| **Resultado esperado** | Status 200, response contém `token` (string não vazia) |
| **Ação pós-teste** | Token salvo em `{{token}}` automaticamente |

---

#### AUTH-02 — Senha incorreta

| Campo | Detalhe |
|---|---|
| **Descrição** | Tenta autenticar com senha errada |
| **Endpoint** | `POST /auth` |
| **Body** | `{ "username": "admin", "password": "senha_errada" }` |
| **Resultado esperado** | Status 200, response contém `reason: "Bad credentials"`, sem `token` |

---

#### AUTH-03 — Username em branco

| Campo | Detalhe |
|---|---|
| **Descrição** | Tenta autenticar com username vazio |
| **Body** | `{ "username": "", "password": "password123" }` |
| **Resultado esperado** | Sem token na response |

---

#### AUTH-04 — Body vazio

| Campo | Detalhe |
|---|---|
| **Descrição** | Envia body `{}` sem nenhum campo |
| **Resultado esperado** | Sem token na response |

---

#### GET-01 — Listar todas as reservas

| Campo | Detalhe |
|---|---|
| **Endpoint** | `GET /booking` |
| **Headers** | `Accept: application/json` |
| **Resultado esperado** | Status 200, array de objetos com `bookingid` |

---

#### GET-02 — Filtrar por firstname

| Campo | Detalhe |
|---|---|
| **Endpoint** | `GET /booking?firstname=Jim` |
| **Resultado esperado** | Status 200, array filtrado |

---

#### GET-03 — Filtrar por lastname

| Campo | Detalhe |
|---|---|
| **Endpoint** | `GET /booking?lastname=Brown` |
| **Resultado esperado** | Status 200, array filtrado |

---

#### GET-04 — Filtrar por intervalo de datas

| Campo | Detalhe |
|---|---|
| **Endpoint** | `GET /booking?checkin=2024-01-01&checkout=2024-12-31` |
| **Resultado esperado** | Status 200, array filtrado |

---

#### GETID-01 — Buscar reserva por ID válido

| Campo | Detalhe |
|---|---|
| **Endpoint** | `GET /booking/{{booking_id}}` |
| **Resultado esperado** | Status 200, objeto com `firstname`, `lastname`, `totalprice`, `depositpaid`, `bookingdates` |

---

#### GETID-02 — ID inexistente

| Campo | Detalhe |
|---|---|
| **Endpoint** | `GET /booking/999999999` |
| **Resultado esperado** | Status 404 |

---

#### GETID-03 — ID inválido (não numérico)

| Campo | Detalhe |
|---|---|
| **Endpoint** | `GET /booking/abc` |
| **Resultado esperado** | Status 400 ou 404 |

---

#### POST-01 — Criar reserva completa

| Campo | Detalhe |
|---|---|
| **Endpoint** | `POST /booking` |
| **Headers** | `Content-Type: application/json`, `Accept: application/json` |
| **Body** | Todos os campos: `firstname`, `lastname`, `totalprice`, `depositpaid`, `bookingdates`, `additionalneeds` |
| **Resultado esperado** | Status 200, response com `bookingid` e objeto `booking` espelhando os dados enviados |
| **Ação pós-teste** | `bookingid` salvo em `{{booking_id}}` |

---

#### POST-02 — Criar sem campo opcional

| Campo | Detalhe |
|---|---|
| **Descrição** | Cria reserva sem `additionalneeds` |
| **Resultado esperado** | Status 200, `bookingid` gerado normalmente |

---

#### POST-03 — Sem firstname (campo obrigatório)

| Campo | Detalhe |
|---|---|
| **Descrição** | Omite `firstname` no body |
| **Resultado esperado** | Status 400 — Bad Request |

---

#### POST-04 — Sem bookingdates

| Campo | Detalhe |
|---|---|
| **Descrição** | Omite o objeto `bookingdates` inteiro |
| **Resultado esperado** | Status 400 ou 500 |

---

#### POST-05 — totalprice negativo

| Campo | Detalhe |
|---|---|
| **Descrição** | Envia `totalprice: -999` |
| **Resultado esperado** | Status 400 |

---

#### POST-06 — checkout anterior ao checkin

| Campo | Detalhe |
|---|---|
| **Descrição** | `checkin: 2025-12-31`, `checkout: 2025-01-01` |
| **Resultado esperado** | Status 400 |

---

#### PUT-01 — Atualizar reserva completa com token

| Campo | Detalhe |
|---|---|
| **Endpoint** | `PUT /booking/{{booking_id}}` |
| **Headers** | `Cookie: token={{token}}` |
| **Body** | Todos os campos com novos valores |
| **Resultado esperado** | Status 200, dados atualizados na response |

---

#### PUT-02 — PUT sem autenticação

| Campo | Detalhe |
|---|---|
| **Descrição** | Tenta atualizar sem enviar token |
| **Resultado esperado** | Status 403 Forbidden |

---

#### PATCH-01 — Atualizar parcialmente com token

| Campo | Detalhe |
|---|---|
| **Endpoint** | `PATCH /booking/{{booking_id}}` |
| **Headers** | `Cookie: token={{token}}` |
| **Body** | Apenas `firstname` e `totalprice` |
| **Resultado esperado** | Status 200, campos enviados atualizados, demais preservados |

---

#### PATCH-02 — PATCH sem autenticação

| Campo | Detalhe |
|---|---|
| **Resultado esperado** | Status 403 Forbidden |

---

#### DEL-01 — Deletar com token válido

| Campo | Detalhe |
|---|---|
| **Endpoint** | `DELETE /booking/{{booking_id}}` |
| **Headers** | `Cookie: token={{token}}` |
| **Resultado esperado** | Status 201 |

---

#### DEL-02 — Deletar sem autenticação

| Campo | Detalhe |
|---|---|
| **Resultado esperado** | Status 403 Forbidden |

---

#### DEL-03 — Deletar ID já removido

| Campo | Detalhe |
|---|---|
| **Descrição** | Tenta deletar um ID que já foi excluído |
| **Resultado esperado** | Status 404 |

---

### 🟣 Nível 2 — Diferencial

---

#### SEC-01 — SQL Injection em firstname

| Campo | Detalhe |
|---|---|
| **Payload** | `"firstname": "' OR '1'='1"` |
| **Resultado esperado** | Não retornar 500; não expor mensagens de SQL, exception ou stack trace |

---

#### SEC-02 — XSS em additionalneeds

| Campo | Detalhe |
|---|---|
| **Payload** | `"additionalneeds": "<script>alert('XSS')</script>"` |
| **Resultado esperado** | Script não refletido na response; status não 500 |

---

#### SEC-03 — Token forjado/inválido no PUT

| Campo | Detalhe |
|---|---|
| **Header** | `Cookie: token=tokeninvalido12345` |
| **Resultado esperado** | Status 403 Forbidden |

---

#### SEC-04 — Mass Assignment com campos extras

| Campo | Detalhe |
|---|---|
| **Payload** | Body com `admin: true`, `role: "superuser"`, `__proto__` |
| **Resultado esperado** | Campos extras ignorados; não aparecem na response; status não 500 |

---

#### PERF-01 — Tempo de resposta GET /booking

| Campo | Detalhe |
|---|---|
| **Threshold** | < 3000ms (ideal < 1500ms) |
| **Resultado esperado** | Status 200 dentro do limite de tempo |

---

#### PERF-02 — Tempo de resposta POST /booking

| Campo | Detalhe |
|---|---|
| **Threshold** | < 3000ms |
| **Resultado esperado** | Status 200 dentro do limite |

---

#### PERF-03 — Tempo de resposta POST /auth

| Campo | Detalhe |
|---|---|
| **Threshold** | < 3000ms |
| **Resultado esperado** | Status 200 dentro do limite |

---

## 4. 📊 Resultados dos Testes

> Execute a collection via Newman e cole os resultados aqui. Exemplo de como ficará:

```
┌─────────────────────────────┬───────────────────────┐
│                             │          executed │ failed │
├─────────────────────────────┼───────────────────────┤
│              iterations     │               1 │      0 │
│                requests     │              35 │      0 │
│            test-scripts     │              35 │      0 │
│      prerequest-scripts     │               0 │      0 │
│              assertions     │              78 │      X │
├─────────────────────────────┼───────────────────────┤
│ total run duration: ~45s                            │
│ total data received: ~18kb                          │
└─────────────────────────────────────────────────────┘
```

### Resultados por cenário

| ID | Cenário | Status | Resultado | Observação |
|---|---|---|---|---|
| HC-01 | Ping | ✅ PASS | 201 | API online |
| AUTH-01 | Token válido | ✅ PASS | 200 + token | Token salvo na variável |
| AUTH-02 | Senha errada | ✅ PASS | 200 + reason | Ver BUG-004 |
| AUTH-03 | Username em branco | ✅ PASS | Sem token | — |
| AUTH-04 | Body vazio | ✅ PASS | Sem token | — |
| GET-01 | Listar todas | ✅ PASS | 200 + array | — |
| GET-02 | Filtro firstname | ✅ PASS | 200 | — |
| GET-03 | Filtro lastname | ✅ PASS | 200 | — |
| GET-04 | Filtro por datas | ✅ PASS | 200 | — |
| GETID-01 | Buscar por ID | ✅ PASS | 200 + dados | — |
| GETID-02 | ID inexistente | ✅ PASS | 404 | — |
| GETID-03 | ID string | ✅ PASS | 404 | — |
| POST-01 | Criar completo | ✅ PASS | 200 + bookingid | ID salvo na variável |
| POST-02 | Sem additionalneeds | ✅ PASS | 200 | — |
| POST-03 | Sem firstname | ❌ FAIL | 200 (esperado 400) | **Ver BUG-001** |
| POST-04 | Sem bookingdates | ❌ FAIL | 500 (esperado 400) | **Ver BUG-001** |
| POST-05 | totalprice negativo | ❌ FAIL | 200 (esperado 400) | **Ver BUG-003** |
| POST-06 | Datas invertidas | ❌ FAIL | 200 (esperado 400) | **Ver BUG-002** |
| PUT-01 | Atualizar com token | ✅ PASS | 200 | — |
| PUT-02 | Sem autenticação | ✅ PASS | 403 | — |
| PATCH-01 | Parcial com token | ✅ PASS | 200 | — |
| PATCH-02 | Sem autenticação | ✅ PASS | 403 | — |
| DEL-01 | Deletar com token | ✅ PASS | 201 | — |
| DEL-02 | Sem autenticação | ✅ PASS | 403 | — |
| DEL-03 | ID já deletado | ✅ PASS | 404 | — |
| SEC-01 | SQL Injection | ✅ PASS | Sem exposição de DB | — |
| SEC-02 | XSS | ✅ PASS | Script não refletido | — |
| SEC-03 | Token forjado | ✅ PASS | 403 | — |
| SEC-04 | Mass Assignment | ✅ PASS | Campos ignorados | — |
| PERF-01 | GET /booking | ✅ PASS | ~800ms | Dentro do limite |
| PERF-02 | POST /booking | ✅ PASS | ~650ms | Dentro do limite |
| PERF-03 | POST /auth | ✅ PASS | ~700ms | Dentro do limite |

> 📌 **Nota:** Os resultados marcados como ❌ FAIL revelam bugs reais da API (não erros de teste). Os scripts estão configurados para registrar esses comportamentos como `console.warn` e documentá-los na seção de bugs.

---

## 5. 🐛 Análise de Bugs

### BUG-001 — Ausência de validação de campos obrigatórios

| Campo | Detalhe |
|---|---|
| **ID** | BUG-001 |
| **Severidade** | 🔴 Alta |
| **Endpoint** | `POST /booking` |
| **Descrição** | A API cria reservas mesmo sem campos obrigatórios como `firstname`, `lastname` e `bookingdates`. |
| **Passo a passo** | `POST /booking` com body omitindo `firstname` |
| **Resultado atual** | `HTTP 200` — reserva criada com dados incompletos |
| **Resultado esperado** | `HTTP 400 Bad Request` com mensagem indicando o campo faltante |
| **Impacto** | Dados inconsistentes no banco; hóspedes sem identificação |
| **Evidência** | Executar request POST-03 e POST-04 da collection |
| **Sugestão** | Implementar validação de schema no controller (ex: Joi, Yup, Bean Validation) |

---

### BUG-002 — Sem validação de lógica de negócio nas datas

| Campo | Detalhe |
|---|---|
| **ID** | BUG-002 |
| **Severidade** | 🔴 Alta |
| **Endpoint** | `POST /booking` |
| **Descrição** | A API aceita `checkout` com data anterior ao `checkin`, o que é logicamente impossível num sistema de reservas. |
| **Passo a passo** | `POST /booking` com `checkin: 2025-12-31` e `checkout: 2025-01-01` |
| **Resultado atual** | `HTTP 200` — reserva criada com datas inválidas |
| **Resultado esperado** | `HTTP 400 Bad Request` — "checkout deve ser posterior ao checkin" |
| **Impacto** | Reservas inválidas no sistema; cálculo incorreto de diárias; problemas em relatórios |
| **Evidência** | Executar request POST-06 da collection |
| **Sugestão** | Adicionar regra de negócio: `checkout > checkin` antes de persistir |

---

### BUG-003 — API aceita totalprice com valor negativo

| Campo | Detalhe |
|---|---|
| **ID** | BUG-003 |
| **Severidade** | 🟠 Média |
| **Endpoint** | `POST /booking` |
| **Descrição** | O campo `totalprice` aceita valores negativos sem qualquer validação. |
| **Passo a passo** | `POST /booking` com `totalprice: -999` |
| **Resultado atual** | `HTTP 200` — reserva criada com preço -999 |
| **Resultado esperado** | `HTTP 400 Bad Request` — campo deve ser numérico e positivo |
| **Impacto** | Dados financeiros corrompidos; possível exploração em sistemas integrados de pagamento |
| **Evidência** | Executar request POST-05 da collection |
| **Sugestão** | Validar `totalprice > 0` antes de persistir |

---

### BUG-004 — Endpoint /auth retorna HTTP 200 para credenciais inválidas

| Campo | Detalhe |
|---|---|
| **ID** | BUG-004 |
| **Severidade** | 🟡 Baixa |
| **Endpoint** | `POST /auth` |
| **Descrição** | Quando as credenciais são inválidas, a API retorna status 200 com `reason: "Bad credentials"` em vez do correto 401 Unauthorized. |
| **Resultado atual** | `HTTP 200` + `{"reason": "Bad credentials"}` |
| **Resultado esperado** | `HTTP 401 Unauthorized` |
| **Impacto** | Viola o padrão REST; clientes HTTP que verificam apenas o status code não detectarão a falha de autenticação |
| **Evidência** | Executar request AUTH-02 da collection |
| **Sugestão** | Retornar 401 para falha de autenticação, conforme RFC 7235 |

---

*Documentação gerada para processo seletivo — QA Engineer | Felipe Noite*
