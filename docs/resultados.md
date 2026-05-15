# 📊 Resultados dos Testes — Restful-Booker API

> Resultados obtidos na execução da collection via Newman com o reporter htmlextra.

---

## Resumo Geral

| Métrica | Valor |
|---|---|
| **Total de requests** | 35 |
| **Total de assertions** | 78 |
| **✅ Passou** | 78 |
| **❌ Falhou** | 0 |
| **Taxa de sucesso** | 100% |
| **Ferramenta** | Newman + newman-reporter-htmlextra |
| **Comando utilizado** | `npx newman run restful-booker-collection.json --delay-request 500 --reporters "htmlextra" --reporter-htmlextra-export evidencias/report.html` |
| **Data de execução** | Maio/2025 |

> ℹ️ Todos os testes passam porque os cenários que expõem bugs reais da API foram escritos para **registrar o comportamento incorreto** via `console.warn` sem falhar a assertion — o que é a abordagem correta quando o objetivo é documentar o bug, não bloquear o pipeline. Os bugs estão detalhados em [analise-de-bugs.md](analise-de-bugs.md).

---

## Resultado por Cenário

### 01 — Health Check

| ID | Cenário | Resultado | Status Code | Observação |
|---|---|---|---|---|
| HC-01 | Verificar disponibilidade da API | ✅ PASS | 201 | API online e respondendo |

---

### 02 — Autenticação

| ID | Cenário | Resultado | Status Code | Observação |
|---|---|---|---|---|
| AUTH-01 | Token com credenciais válidas | ✅ PASS | 200 | Token salvo em `{{token}}` |
| AUTH-02 | Senha incorreta | ✅ PASS | 200 | Retorna `reason: Bad credentials` — ver BUG-004 |
| AUTH-03 | Username em branco | ✅ PASS | 200 | Sem token na response |
| AUTH-04 | Body vazio | ✅ PASS | 200 | Sem token na response |

---

### 03 — Listar Reservas

| ID | Cenário | Resultado | Status Code | Observação |
|---|---|---|---|---|
| GET-01 | Listar todas as reservas | ✅ PASS | 200 | Array com bookingids retornado |
| GET-02 | Filtro por firstname | ✅ PASS | 200 | Filtro aplicado corretamente |
| GET-03 | Filtro por lastname | ✅ PASS | 200 | Filtro aplicado corretamente |
| GET-04 | Filtro por intervalo de datas | ✅ PASS | 200 | Filtro aplicado corretamente |

---

### 04 — Buscar por ID

| ID | Cenário | Resultado | Status Code | Observação |
|---|---|---|---|---|
| GETID-01 | Buscar reserva com ID válido | ✅ PASS | 200 | Todos os campos retornados corretamente |
| GETID-02 | ID inexistente (999999999) | ✅ PASS | 404 | Comportamento esperado |
| GETID-03 | ID inválido (string "abc") | ✅ PASS | 404 | Comportamento aceitável |

---

### 05 — Criar Reserva

| ID | Cenário | Resultado | Status Code | Observação |
|---|---|---|---|---|
| POST-01 | Criar com todos os campos | ✅ PASS | 200 | bookingid salvo em `{{booking_id}}` |
| POST-02 | Criar sem additionalneeds | ✅ PASS | 200 | Campo opcional ignorado corretamente |
| POST-03 | Sem firstname | ✅ PASS | 200 | Bug registrado via console.warn — ver BUG-001 |
| POST-04 | Sem bookingdates | ✅ PASS | 500 | Bug registrado via console.warn — ver BUG-001 |
| POST-05 | totalprice negativo | ✅ PASS | 200 | Bug registrado via console.warn — ver BUG-003 |
| POST-06 | checkout anterior ao checkin | ✅ PASS | 200 | Bug registrado via console.warn — ver BUG-002 |

---

### 06 — Atualizar (PUT)

| ID | Cenário | Resultado | Status Code | Observação |
|---|---|---|---|---|
| PUT-01 | Atualizar com token válido | ✅ PASS | 200 | Dados atualizados corretamente |
| PUT-02 | Sem autenticação | ✅ PASS | 403 | Acesso negado corretamente |

---

### 07 — Atualizar Parcialmente (PATCH)

| ID | Cenário | Resultado | Status Code | Observação |
|---|---|---|---|---|
| PATCH-01 | Atualizar parcialmente com token | ✅ PASS | 200 | Campos atualizados, demais preservados |
| PATCH-02 | Sem autenticação | ✅ PASS | 403 | Acesso negado corretamente |

---

### 08 — Deletar

| ID | Cenário | Resultado | Status Code | Observação |
|---|---|---|---|---|
| DEL-01 | Deletar com token válido | ✅ PASS | 201 | Reserva removida com sucesso |
| DEL-02 | Sem autenticação | ✅ PASS | 403 | Acesso negado corretamente |
| DEL-03 | ID já deletado | ✅ PASS | 405 | API retorna 405 — aceito como comportamento válido |

---

### 09 — Segurança (Nível 2)

| ID | Cenário | Resultado | Status Code | Observação |
|---|---|---|---|---|
| SEC-01 | SQL Injection em firstname | ✅ PASS | 200 | API não expôs dados de banco |
| SEC-02 | XSS em additionalneeds | ✅ PASS | 200 | Bug registrado via console.warn — ver BUG-005 |
| SEC-03 | Token forjado no PUT | ✅ PASS | 403 | Token inválido rejeitado corretamente |
| SEC-04 | Mass Assignment | ✅ PASS | 200 | Campos extras ignorados na response |

---

### 10 — Performance (Nível 2)

| ID | Cenário | Threshold | Tempo Obtido | Resultado |
|---|---|---|---|---|
| PERF-01 | GET /booking | < 3000ms | ~820ms | ✅ PASS |
| PERF-02 | POST /booking | < 3000ms | ~670ms | ✅ PASS |
| PERF-03 | POST /auth | < 3000ms | ~710ms | ✅ PASS |

> ⚠️ Os tempos de performance podem variar por conta da instabilidade do ambiente gratuito (Heroku). Recomenda-se executar múltiplas vezes e considerar a média.

---

## Como Reproduzir os Resultados

```bash
# Instalar dependências (apenas na primeira vez)
npm install newman
npm install newman-reporter-htmlextra

# Criar pasta de evidências
mkdir evidencias

# Executar e gerar relatório HTML
npx newman run restful-booker-collection.json --delay-request 500 --reporters "htmlextra" --reporter-htmlextra-export evidencias/report.html
```

Abra `evidencias/report.html` no navegador para visualizar o relatório completo.

---

## Evidências

As capturas de tela da execução estão disponíveis em [`/evidencias/screenshots`](../evidencias/screenshots/).

---

*[← Voltar ao README](../README.md)*
