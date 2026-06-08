# Sistema de Finanças Pessoais — Documentação

Documentação central do projeto. Uma plataforma de finanças pessoais, focada no nível de detalhe e customização que apps comuns não entregam — a planilha evoluída. Cada pessoa é dona dos seus dados e compartilha lançamentos específicos com quem quiser, item a item.

---

## Sobre o projeto

| | |
|---|---|
| **Tipo** | Sistema de finanças pessoais |
| **Usuários** | Independentes — cada um dono dos seus dados; compartilhamento item a item via vínculo |
| **Plataformas** | Mobile primeiro (Fase 1) · Web depois (Fase 2) |
| **Backend** | Django REST Framework + PostgreSQL |
| **Mobile** | React Native + Expo + TypeScript |
| **Web** | React + TypeScript + Tailwind + shadcn/ui + React Bits |
| **Infra** | Oracle Cloud (Always Free) |

---

## Índice da documentação

### [01 · Produto](01-produto/visao.md)
Documento de visão — o problema que o sistema resolve, os 8 módulos, decisões de produto e o que o sistema **não** é. Comece por aqui.

### [02 · Requisitos](02-requisitos/requisitos.md)
Requisitos funcionais (RF), regras de negócio (RN) e requisitos não funcionais (RNF) numerados. Inclui o módulo de scanner de cupom fiscal. É a base do backend.

### [03 · Arquitetura](03-arquitetura/modelo-de-dados.md)
Modelo de dados completo — entidades, campos tipados, constraints e relacionamentos. Traduz direto para os models do Django e tabelas do PostgreSQL.

### [04 · Design](04-design/fluxo-de-telas.md)
Fluxo de telas — mapa de navegação, descrição de cada tela e como o usuário transita entre elas.

### [05 · Protótipos](05-prototipos/mvp-scanner-cupom.md)
MVP mobile isolado para validar a leitura de cupom fiscal (QR Code da NFC-e + OCR de fallback) antes de integrar ao sistema principal.

---

## Os 8 módulos do sistema

1. **Renda / Receitas** — entradas, salário, recorrência
2. **Gastos do dia a dia** — variáveis, com scanner de cupom para compras de mercado
3. **Gastos fixos** — Tipo A (valor fixo) e Tipo B (valor estimado)
4. **Cartão de crédito** — faturas, fixos no cartão, parcelamentos, variáveis
5. **Dívidas e parcelamentos** — visão consolidada de longo prazo
6. **Metas de economia** — objetivos com progresso
7. **Investimentos** — controle de aportes (Fase 1)
8. **Patrimônio líquido** — ativos − passivos, com histórico

---

## Roadmap de desenvolvimento

- [x] Documento de visão
- [x] Requisitos
- [x] Modelo de dados
- [x] Fluxo de telas
- [x] MVP scanner de cupom (documentado)
- [ ] Validação do MVP scanner
- [ ] Setup do backend Django + models
- [ ] Autenticação JWT
- [ ] Endpoints dos módulos core
- [ ] App mobile (Fase 1)
- [ ] App web (Fase 2)

---

## Convenção de versionamento dos documentos

Cada documento traz no topo seu status e versão (ex: `v0.4`). Mudanças relevantes são registradas na seção **Registro de Decisões** ao final de cada documento. Os documentos referenciam uns aos outros pela versão, então ao atualizar um, verifique se os dependentes precisam acompanhar.

| Documento | Versão atual |
|---|---|
| Visão | v0.6 |
| Requisitos | v0.5 |
| Modelo de dados | v0.2 |
| Fluxo de telas | v0.2 |

---

## Repositórios do projeto

Este é o repositório de **documentação**. Os demais:

- `backend` — API Django REST
- `mobile` — App React Native (Fase 1)
- `frontend` — App web React (Fase 2)

---

*Documentação viva. Mantida em paralelo ao desenvolvimento.*