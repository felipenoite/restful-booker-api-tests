# 🏨 Restful-Booker — API Testing

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
| **Nível 2** | ✅ Completo | Testes de segurança, performance e automação via Newman |

---

## 🛠️ Ferramentas Utilizadas

| Ferramenta | Finalidade |
|---|---|
| Postman | Criação, organização e execução dos testes |
| Newman | Execução da collection via linha de comando |
| newman-reporter-htmlextra | Geração de relatório HTML visual |
| Node.js (≥ 18.x) | Runtime necessário para o Newman |

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
# Verificar versão do Node.js (precisa ser ≥ 18)
node --version

# Instalar o Newman e o reporter localmente no projeto
npm install newman
npm install newman-reporter-htmlextra
```

### Executar via Postman (interface gráfica)

1. Abra o Postman
2. Clique em **Import** e selecione o arquivo `restful-booker-collection.json`
3. A collection será importada com todas as variáveis já configuradas
4. Clique em **Run collection** para executar todos os testes em sequência

### Executar via Newman (linha de comando)

> ⚠️ **Importante:** Use `npx newman` em vez de `newman` diretamente para garantir que o reporter local seja encontrado. No Windows (PowerShell), coloque os reporters entre aspas.

```bash
# Criar a pasta de evidências antes de rodar (se ainda não existir)
mkdir evidencias

# Gerar apenas o relatório HTML
npx newman run restful-booker-collection.json --delay-request 500 --reporters "htmlextra" --reporter-htmlextra-export evidencias/report.html

# Gerar relatório HTML e ver resultado no terminal ao mesmo tempo
npx newman run restful-booker-collection.json --delay-request 500 --reporters "cli,htmlextra" --reporter-htmlextra-export evidencias/report.html
```

### Visualizar o relatório

Após a execução, abra o arquivo gerado no navegador:

```
evidencias/report.html
```

O relatório mostra cada request com status, tempo de resposta, assertions que passaram e falharam, e os logs do console — incluindo avisos de bugs registrados durante a execução.

---

## 🔐 Variáveis de Ambiente

As variáveis são gerenciadas diretamente na collection, sem necessidade de arquivo externo.

| Variável | Valor Inicial | Preenchimento |
|---|---|---|
| `base_url` | `https://restful-booker.herokuapp.com` | Fixo |
| `token` | *(vazio)* | Automático após AUTH-01 |
| `booking_id` | *(vazio)* | Automático após POST-01 |

> ⚠️ A ordem padrão da collection já garante que `{{token}}` e `{{booking_id}}` sejam preenchidos antes dos testes que dependem deles.

---

## 📄 Documentação Complementar

| Arquivo | Conteúdo |
|---|---|
| [docs/cenarios-de-teste.md](docs/cenarios-de-teste.md) | Detalhamento de todos os cenários testados |
| [docs/analise-de-bugs.md](docs/analise-de-bugs.md) | Bugs encontrados com severidade, impacto e sugestão |
| [docs/resultados.md](docs/resultados.md) | Resultados de execução e resumo geral |

---

## ⚡ Ordem de Execução Recomendada

```
01 - Health Check
02 - Autenticação          → gera {{token}}
05 - Criar Reserva         → gera {{booking_id}}
03 - Listar Reservas
04 - Buscar por ID
06 - Atualizar (PUT)
07 - Atualizar (PATCH)
08 - Deletar
09 - Segurança
10 - Performance
```

---

## 📋 Premissas Assumidas

- A API é pública e não requer configuração local
- Credenciais padrão `admin` / `password123` conforme documentação oficial
- A API pode apresentar instabilidade por ser hospedada em ambiente gratuito — retries são normais
- Testes de performance foram realizados por request individual via Postman/Newman, com foco em tempo de resposta unitário
- Cenários negativos que expõem bugs reais da API são registrados como `console.warn` no script e documentados em `docs/analise-de-bugs.md`

---

*Projeto desenvolvido para fins de avaliação técnica — QA Engineer*
