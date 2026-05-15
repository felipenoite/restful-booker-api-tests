# 📋 Cenários de Teste — Restful-Booker API

> Detalhamento completo de todos os cenários cobertos na collection.

---

## Sumário

- [Nível 1 — Obrigatório](#nível-1--obrigatório)
  - [Health Check](#health-check)
  - [Autenticação](#autenticação)
  - [Listar Reservas](#listar-reservas)
  - [Buscar Reserva por ID](#buscar-reserva-por-id)
  - [Criar Reserva](#criar-reserva)
  - [Atualizar Reserva (PUT)](#atualizar-reserva-put)
  - [Atualizar Parcialmente (PATCH)](#atualizar-parcialmente-patch)
  - [Deletar Reserva](#deletar-reserva)
- [Nível 2 — Diferencial](#nível-2--diferencial)
  - [Segurança](#segurança)
  - [Performance](#performance)

---

## Nível 1 — Obrigatório

### Health Check

#### HC-01 — Verificar disponibilidade da API

| Campo | Detalhe |
|---|---|
| **Método** | GET |
| **Endpoint** | `/ping` |
| **Headers** | Nenhum |
| **Body** | Nenhum |
| **Resultado esperado** | Status `201`, body contém `"Created"` |
| **Tipo** | ✅ Positivo |

---

### Autenticação

#### AUTH-01 — Gerar token com credenciais válidas

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/auth` |
| **Headers** | `Content-Type: application/json` |
| **Body** | `{ "username": "admin", "password": "password123" }` |
| **Resultado esperado** | Status `200`, response contém `token` (string não vazia) |
| **Ação pós-teste** | Token salvo automaticamente em `{{token}}` |
| **Tipo** | ✅ Positivo |

#### AUTH-02 — Senha incorreta

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/auth` |
| **Body** | `{ "username": "admin", "password": "senha_errada" }` |
| **Resultado esperado** | Status `200`, response contém `reason: "Bad credentials"`, sem campo `token` |
| **Tipo** | ❌ Negativo |

#### AUTH-03 — Username em branco

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/auth` |
| **Body** | `{ "username": "", "password": "password123" }` |
| **Resultado esperado** | Response sem campo `token` |
| **Tipo** | ❌ Negativo |

#### AUTH-04 — Body vazio

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/auth` |
| **Body** | `{}` |
| **Resultado esperado** | Response sem campo `token` |
| **Tipo** | ❌ Negativo |

---

### Listar Reservas

#### GET-01 — Listar todas as reservas

| Campo | Detalhe |
|---|---|
| **Método** | GET |
| **Endpoint** | `/booking` |
| **Headers** | `Accept: application/json` |
| **Resultado esperado** | Status `200`, array de objetos contendo `bookingid` |
| **Tipo** | ✅ Positivo |

#### GET-02 — Filtrar por firstname

| Campo | Detalhe |
|---|---|
| **Método** | GET |
| **Endpoint** | `/booking?firstname=Jim` |
| **Resultado esperado** | Status `200`, array filtrado |
| **Tipo** | ✅ Positivo |

#### GET-03 — Filtrar por lastname

| Campo | Detalhe |
|---|---|
| **Método** | GET |
| **Endpoint** | `/booking?lastname=Brown` |
| **Resultado esperado** | Status `200`, array filtrado |
| **Tipo** | ✅ Positivo |

#### GET-04 — Filtrar por intervalo de datas

| Campo | Detalhe |
|---|---|
| **Método** | GET |
| **Endpoint** | `/booking?checkin=2024-01-01&checkout=2024-12-31` |
| **Resultado esperado** | Status `200`, array filtrado |
| **Tipo** | ✅ Positivo |

---

### Buscar Reserva por ID

#### GETID-01 — Buscar reserva com ID válido

| Campo | Detalhe |
|---|---|
| **Método** | GET |
| **Endpoint** | `/booking/{{booking_id}}` |
| **Headers** | `Accept: application/json` |
| **Resultado esperado** | Status `200`, objeto com `firstname`, `lastname`, `totalprice`, `depositpaid`, `bookingdates` |
| **Tipo** | ✅ Positivo |

#### GETID-02 — ID inexistente

| Campo | Detalhe |
|---|---|
| **Método** | GET |
| **Endpoint** | `/booking/999999999` |
| **Resultado esperado** | Status `404` |
| **Tipo** | ❌ Negativo |

#### GETID-03 — ID inválido (string não numérica)

| Campo | Detalhe |
|---|---|
| **Método** | GET |
| **Endpoint** | `/booking/abc` |
| **Resultado esperado** | Status `400` ou `404` |
| **Tipo** | ❌ Negativo |

---

### Criar Reserva

#### POST-01 — Criar reserva com todos os campos

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/booking` |
| **Headers** | `Content-Type: application/json`, `Accept: application/json` |
| **Body** | `firstname`, `lastname`, `totalprice`, `depositpaid`, `bookingdates` (checkin/checkout), `additionalneeds` |
| **Resultado esperado** | Status `200`, response com `bookingid` gerado e objeto `booking` espelhando os dados enviados |
| **Ação pós-teste** | `bookingid` salvo automaticamente em `{{booking_id}}` |
| **Tipo** | ✅ Positivo |

#### POST-02 — Criar sem campo opcional (additionalneeds)

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/booking` |
| **Body** | Todos os campos obrigatórios, sem `additionalneeds` |
| **Resultado esperado** | Status `200`, `bookingid` gerado normalmente |
| **Tipo** | ✅ Positivo |

#### POST-03 — Criar sem firstname (campo obrigatório)

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/booking` |
| **Body** | Body sem o campo `firstname` |
| **Resultado esperado** | Status `400 Bad Request` |
| **Tipo** | ❌ Negativo |

#### POST-04 — Criar sem bookingdates

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/booking` |
| **Body** | Body sem o objeto `bookingdates` |
| **Resultado esperado** | Status `400` ou `500` |
| **Tipo** | ❌ Negativo |

#### POST-05 — totalprice com valor negativo

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/booking` |
| **Body** | `totalprice: -999` |
| **Resultado esperado** | Status `400 Bad Request` |
| **Tipo** | ❌ Negativo |

#### POST-06 — checkout anterior ao checkin

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/booking` |
| **Body** | `checkin: 2025-12-31`, `checkout: 2025-01-01` |
| **Resultado esperado** | Status `400 Bad Request` |
| **Tipo** | ❌ Negativo |

---

### Atualizar Reserva (PUT)

#### PUT-01 — Atualizar reserva completa com token

| Campo | Detalhe |
|---|---|
| **Método** | PUT |
| **Endpoint** | `/booking/{{booking_id}}` |
| **Headers** | `Cookie: token={{token}}`, `Content-Type: application/json` |
| **Body** | Todos os campos com novos valores |
| **Resultado esperado** | Status `200`, response com dados atualizados |
| **Tipo** | ✅ Positivo |

#### PUT-02 — Atualizar sem autenticação

| Campo | Detalhe |
|---|---|
| **Método** | PUT |
| **Endpoint** | `/booking/{{booking_id}}` |
| **Headers** | Sem token |
| **Resultado esperado** | Status `403 Forbidden` |
| **Tipo** | ❌ Negativo |

---

### Atualizar Parcialmente (PATCH)

#### PATCH-01 — Atualizar campos parciais com token

| Campo | Detalhe |
|---|---|
| **Método** | PATCH |
| **Endpoint** | `/booking/{{booking_id}}` |
| **Headers** | `Cookie: token={{token}}` |
| **Body** | Apenas `firstname` e `totalprice` |
| **Resultado esperado** | Status `200`, campos enviados atualizados, demais preservados |
| **Tipo** | ✅ Positivo |

#### PATCH-02 — PATCH sem autenticação

| Campo | Detalhe |
|---|---|
| **Método** | PATCH |
| **Endpoint** | `/booking/{{booking_id}}` |
| **Headers** | Sem token |
| **Resultado esperado** | Status `403 Forbidden` |
| **Tipo** | ❌ Negativo |

---

### Deletar Reserva

#### DEL-01 — Deletar com token válido

| Campo | Detalhe |
|---|---|
| **Método** | DELETE |
| **Endpoint** | `/booking/{{booking_id}}` |
| **Headers** | `Cookie: token={{token}}` |
| **Resultado esperado** | Status `201` |
| **Tipo** | ✅ Positivo |

#### DEL-02 — Deletar sem autenticação

| Campo | Detalhe |
|---|---|
| **Método** | DELETE |
| **Endpoint** | `/booking/{{booking_id}}` |
| **Headers** | Sem token |
| **Resultado esperado** | Status `403 Forbidden` |
| **Tipo** | ❌ Negativo |

#### DEL-03 — Deletar ID já removido

| Campo | Detalhe |
|---|---|
| **Método** | DELETE |
| **Endpoint** | `/booking/{{booking_id}}` |
| **Headers** | `Cookie: token={{token}}` |
| **Descrição** | Tenta deletar um ID que já foi excluído anteriormente |
| **Resultado esperado** | Status `404` |
| **Tipo** | ❌ Negativo |

---

## Nível 2 — Diferencial

### Segurança

#### SEC-01 — SQL Injection em firstname

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/booking` |
| **Payload** | `"firstname": "' OR '1'='1"` |
| **Resultado esperado** | Status diferente de `500`; response não expõe mensagens de SQL, exception ou stack trace |
| **Tipo** | 🔒 Segurança |

#### SEC-02 — XSS em additionalneeds

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/booking` |
| **Payload** | `"additionalneeds": "<script>alert('XSS')</script>"` |
| **Resultado esperado** | Script não refletido na response; status diferente de `500` |
| **Tipo** | 🔒 Segurança |

#### SEC-03 — Token forjado no PUT

| Campo | Detalhe |
|---|---|
| **Método** | PUT |
| **Endpoint** | `/booking/1` |
| **Headers** | `Cookie: token=tokeninvalido12345` |
| **Resultado esperado** | Status `403 Forbidden` |
| **Tipo** | 🔒 Segurança |

#### SEC-04 — Mass Assignment com campos extras

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/booking` |
| **Payload** | Body com campos extras: `admin: true`, `role: "superuser"`, `__proto__` |
| **Resultado esperado** | Campos extras ignorados e não refletidos na response; status diferente de `500` |
| **Tipo** | 🔒 Segurança |

---

### Performance

#### PERF-01 — Tempo de resposta GET /booking

| Campo | Detalhe |
|---|---|
| **Método** | GET |
| **Endpoint** | `/booking` |
| **Threshold** | `< 3000ms` (ideal `< 1500ms`) |
| **Resultado esperado** | Status `200` dentro do limite de tempo |
| **Tipo** | ⚡ Performance |

#### PERF-02 — Tempo de resposta POST /booking

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/booking` |
| **Threshold** | `< 3000ms` |
| **Resultado esperado** | Status `200` dentro do limite |
| **Tipo** | ⚡ Performance |

#### PERF-03 — Tempo de resposta POST /auth

| Campo | Detalhe |
|---|---|
| **Método** | POST |
| **Endpoint** | `/auth` |
| **Threshold** | `< 3000ms` |
| **Resultado esperado** | Status `200` dentro do limite |
| **Tipo** | ⚡ Performance |

---

*[← Voltar ao README](../README.md)*
