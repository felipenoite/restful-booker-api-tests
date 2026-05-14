# 🏨 Restful-Booker API Testing

> Projeto de automação de testes de API para a aplicação **Restful-Booker**, desenvolvido como parte de processo seletivo de QA Engineer.

---

## 👤 Autor

| Campo | Info |
|---|---|
| Nome | Felipe Noite |
| GitHub | [felipenoite](https://github.com/felipenoite) |
| LinkedIn | [linkedin.com/in/felipenoite](https://linkedin.com/in/felipenoite) |
| Email | felipeasnoite@gmail.com |
| Certificação | CTFL (ISTQB) |

---

## 🎯 Sobre o Projeto

Cobertura completa de testes da API [Restful-Booker](https://restful-booker.herokuapp.com/apidoc/index.html), um sistema de reservas de hotel usado para prática de API Testing.

### Níveis de cobertura

| Nível | Status | Descrição |
|---|---|---|
| **Nível 1** | ✅ Completo | Autenticação, CRUD de reservas, validação de campos obrigatórios |
| **Nível 2** | ✅ Completo | Testes de segurança, performance, automação via scripts |

---

## 🛠️ Ferramentas Utilizadas

| Ferramenta | Versão | Finalidade |
|---|---|---|
| Postman | Latest | Execução e automação dos testes |
| Newman | Latest | Execução via linha de comando (CI/CD) |
| Node.js | ≥ 18.x | Runtime para Newman |

---

## 📁 Estrutura do Repositório

```
restful-booker-api-tests/
│
├── restful-booker-collection.json   # Collection principal com todos os testes
├── README.md                        # Esta documentação
├── docs/
│   ├── cenarios-de-teste.md         # Detalhamento dos cenários
│   ├── analise-de-bugs.md           # Bugs encontrados durante os testes
│   └── resultados.md                # Resultados de execução
└── evidencias/
    └── screenshots/                 # Capturas de tela das execuções
```

---

## 🚀 Como Executar

### Pré-requisitos

```bash
# Instalar Node.js (https://nodejs.org)
node --version   # ≥ 18.x

# Instalar Newman globalmente
npm install -g newman

# Instalar reporter HTML (opcional, para relatório visual)
npm install -g newman-reporter-htmlextra
```

### Opção 1 — Postman (interface gráfica)

1. Abra o Postman
2. Clique em **Import** → selecione `restful-booker-collection.json`
3. A collection será importada com todas as variáveis já configuradas
4. Clique em **Run collection** para executar todos os testes em sequência

### Opção 2 — Newman (linha de comando)

```bash
# Execução básica
newman run restful-booker-collection.json

# Execução com relatório HTML
newman run restful-booker-collection.json \
  --reporters htmlextra \
  --reporter-htmlextra-export ./evidencias/report.html

# Execução com delay entre requests (recomendado para API pública)
newman run restful-booker-collection.json --delay-request 500
```

---

## 🔐 Variáveis de Ambiente

As variáveis são gerenciadas **diretamente na collection** (sem necessidade de arquivo `.env` externo):

| Variável | Valor padrão | Descrição |
|---|---|---|
| `base_url` | `https://restful-booker.herokuapp.com` | URL base da API |
| `token` | *(preenchido automaticamente)* | Token gerado pelo teste de autenticação |
| `booking_id` | *(preenchido automaticamente)* | ID salvo após criação de reserva |

> ⚠️ **Importante:** Execute sempre os testes de **Autenticação** e **Criar Reserva** antes dos testes que dependem do `{{token}}` e `{{booking_id}}`. A ordem padrão da collection já garante isso.

---

## 📋 Cenários de Teste

### 01 — Health Check

| ID | Cenário | Tipo | Resultado Esperado |
|---|---|---|---|
| HC-01 | Verificar disponibilidade da API via /ping | Positivo | Status 201 |

### 02 — Autenticação (`POST /auth`)

| ID | Cenário | Tipo | Resultado Esperado |
|---|---|---|---|
| AUTH-01 | Gerar token com credenciais válidas | Positivo | Status 200 + token na response |
| AUTH-02 | Credenciais com senha incorreta | Negativo | Status 200 + `reason: Bad credentials` |
| AUTH-03 | Username em branco | Negativo | Sem token na response |
| AUTH-04 | Body vazio `{}` | Negativo | Sem token na response |

### 03 — Listar Reservas (`GET /booking`)

| ID | Cenário | Tipo | Resultado Esperado |
|---|---|---|---|
| GET-01 | Listar todas as reservas | Positivo | Status 200 + array de bookingids |
| GET-02 | Filtrar por `firstname` | Positivo | Status 200 + array filtrado |
| GET-03 | Filtrar por `lastname` | Positivo | Status 200 + array filtrado |
| GET-04 | Filtrar por `checkin` e `checkout` | Positivo | Status 200 + array filtrado |

### 04 — Buscar Reserva (`GET /booking/:id`)

| ID | Cenário | Tipo | Resultado Esperado |
|---|---|---|---|
| GETID-01 | Buscar reserva com ID válido | Positivo | Status 200 + dados completos |
| GETID-02 | Buscar com ID inexistente (999999999) | Negativo | Status 404 |
| GETID-03 | Buscar com ID inválido (string "abc") | Negativo | Status 400 ou 404 |

### 05 — Criar Reserva (`POST /booking`)

| ID | Cenário | Tipo | Resultado Esperado |
|---|---|---|---|
| POST-01 | Criar com todos os campos | Positivo | Status 200 + bookingid gerado |
| POST-02 | Criar sem campo opcional `additionalneeds` | Positivo | Status 200 + bookingid gerado |
| POST-03 | Criar sem `firstname` (campo obrigatório) | Negativo | Status 400 |
| POST-04 | Criar sem `bookingdates` | Negativo | Status 400 ou 500 |
| POST-05 | Criar com `totalprice` negativo | Negativo | Status 400 |
| POST-06 | Criar com `checkout` anterior ao `checkin` | Negativo | Status 400 |

### 06 — Atualização Completa (`PUT /booking/:id`)

| ID | Cenário | Tipo | Resultado Esperado |
|---|---|---|---|
| PUT-01 | Atualizar com token válido | Positivo | Status 200 + dados atualizados |
| PUT-02 | Atualizar sem autenticação | Negativo | Status 403 |

### 07 — Atualização Parcial (`PATCH /booking/:id`)

| ID | Cenário | Tipo | Resultado Esperado |
|---|---|---|---|
| PATCH-01 | Atualizar campos parciais com token | Positivo | Status 200 + campo atualizado |
| PATCH-02 | PATCH sem autenticação | Negativo | Status 403 |

### 08 — Deletar Reserva (`DELETE /booking/:id`)

| ID | Cenário | Tipo | Resultado Esperado |
|---|---|---|---|
| DEL-01 | Deletar com token válido | Positivo | Status 201 |
| DEL-02 | Deletar sem autenticação | Negativo | Status 403 |
| DEL-03 | Deletar ID já removido | Negativo | Status 404 |

### 09 — Segurança (Nível 2)

| ID | Cenário | Tipo | Resultado Esperado |
|---|---|---|---|
| SEC-01 | SQL Injection no campo `firstname` | Segurança | Não retornar 500 nem expor dados de DB |
| SEC-02 | XSS no campo `additionalneeds` | Segurança | Script não deve ser refletido |
| SEC-03 | Token inválido/forjado no PUT | Segurança | Status 403 |
| SEC-04 | Mass Assignment com campos extras (`admin`, `role`) | Segurança | Campos extras ignorados |

### 10 — Performance (Nível 2)

| ID | Cenário | Threshold | Resultado Esperado |
|---|---|---|---|
| PERF-01 | GET /booking | < 3000ms | Tempo dentro do limite |
| PERF-02 | POST /booking | < 3000ms | Tempo dentro do limite |
| PERF-03 | POST /auth | < 3000ms | Tempo dentro do limite |

---

## 🐛 Análise de Bugs Identificados

### BUG-001 — API aceita reserva sem campos obrigatórios

| Campo | Detalhe |
|---|---|
| **ID** | BUG-001 |
| **Severidade** | Alta |
| **Endpoint** | `POST /booking` |
| **Descrição** | A API aceita criar uma reserva sem os campos `firstname` e `lastname`, retornando status 200. |
| **Passos** | Enviar POST /booking sem `firstname` |
| **Resultado atual** | Status 200 — reserva criada |
| **Resultado esperado** | Status 400 — Bad Request com mensagem de validação |
| **Impacto** | Dados inconsistentes no sistema; hóspedes sem nome cadastrado |

---

### BUG-002 — Sem validação de lógica de negócio nas datas

| Campo | Detalhe |
|---|---|
| **ID** | BUG-002 |
| **Severidade** | Alta |
| **Endpoint** | `POST /booking` |
| **Descrição** | A API aceita `checkout` com data anterior ao `checkin`, o que é logicamente impossível. |
| **Passos** | Enviar POST com `checkin: 2025-12-31` e `checkout: 2025-01-01` |
| **Resultado atual** | Status 200 — reserva criada |
| **Resultado esperado** | Status 400 com mensagem explicando a regra de negócio |
| **Impacto** | Reservas inválidas no banco; inconsistência nos relatórios |

---

### BUG-003 — API aceita totalprice negativo

| Campo | Detalhe |
|---|---|
| **ID** | BUG-003 |
| **Severidade** | Média |
| **Endpoint** | `POST /booking` |
| **Descrição** | Valores negativos são aceitos no campo `totalprice`. |
| **Passos** | Enviar POST com `totalprice: -999` |
| **Resultado atual** | Status 200 — reserva criada com preço negativo |
| **Resultado esperado** | Status 400 — campo deve aceitar apenas valores positivos |
| **Impacto** | Pode causar problemas em relatórios financeiros e cálculos |

---

### BUG-004 — Auth retorna 200 para credenciais inválidas (sem uso de 401)

| Campo | Detalhe |
|---|---|
| **ID** | BUG-004 |
| **Severidade** | Baixa |
| **Endpoint** | `POST /auth` |
| **Descrição** | Ao enviar credenciais inválidas, a API retorna status 200 com `reason: Bad credentials` em vez de 401 Unauthorized. |
| **Resultado atual** | Status 200 + `{"reason": "Bad credentials"}` |
| **Resultado esperado** | Status 401 Unauthorized |
| **Impacto** | Viola padrões REST; dificulta tratamento de erro em clientes HTTP |

---

## 💡 Sugestões de Melhoria

1. **Validação de campos obrigatórios** — Implementar validação no backend com retorno 400 e mensagens descritivas
2. **Validação de lógica de negócio** — Verificar se `checkout > checkin` antes de persistir
3. **Status HTTP corretos** — Usar 401 para auth falha, 422 para validação de dados
4. **Rate Limiting** — Não há limite de requisições; endpoint `/auth` está vulnerável a brute force
5. **HTTPS obrigatório** — Verificar redirecionamento de HTTP para HTTPS
6. **Paginação** — `GET /booking` retorna todos os registros sem paginação, o que pode ser problemático em escala
7. **Versionamento da API** — Não há versionamento (`/v1/booking`), dificultando evolução sem quebrar clientes

---

## 📊 Premissas Assumidas

- A API é pública e não requer configuração de ambiente local
- Credenciais padrão: `admin` / `password123` (documentadas oficialmente)
- O token gerado via `/auth` é válido por sessão e deve ser renovado se expirar
- A API pode apresentar instabilidade por ser hospedada no Heroku (free tier) — retries são normais
- Testes de performance foram realizados individualmente via Postman (não via ferramenta de carga como k6 ou JMeter), com foco em tempo de resposta por requisição

---

## ⚡ Execução Recomendada

Para garantir a ordem correta de execução com dependência de variáveis:

```
01 - Health Check
  → 02 - Autenticação (gera {{token}})
    → 05 - Criar Reserva (gera {{booking_id}})
      → 04 - Buscar por ID
      → 06 - PUT (atualizar)
      → 07 - PATCH (atualizar parcialmente)
      → 08 - DELETE
03 - Listar Reservas (independente)
09 - Segurança (independente)
10 - Performance (independente)
```

---

*Projeto desenvolvido para fins de avaliação técnica — QA Engineer*
