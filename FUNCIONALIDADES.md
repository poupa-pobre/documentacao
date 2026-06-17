# Poupa Pobre — Inventário de Funcionalidades

> Lista de tudo que **já existe** no app (backend + mobile) e o que **ainda falta**.
> Atualizado em **13/06/2026**. Fonte: código real + `backend/CLAUDE.md` + `mobile/CLAUDE.md`.

**O que é:** finanças pessoais — "a planilha evoluída". Categorização granular e
fatura de cartão realista que os apps de prateleira não dão. Cada pessoa é um
**usuário independente**, dono dos próprios dados (privado por padrão). O **mês** é
a unidade central: você abre o app e vê o status do mês ("o que já saiu" × "o que
ainda vem").

**Stack:** Backend Django 6 + DRF + PostgreSQL (Docker). Mobile React Native +
Expo SDK 54 + TypeScript + NativeWind. Entrega **mobile first**; web depois.

**Estado geral:** Backend Fases 0–4 completas + boa parte da Fase 5. **225 testes
passando.** Mobile: todas as telas principais construídas, `tsc` limpo.

Legenda: ✅ pronto · 🧪 pronto, falta testar no celular · 🔲 não feito

---

## Requisitos obrigatórios da disciplina

O app precisa usar **3 tecnologias**: banco de dados, mídia e GPS. Todas atendidas:

| Requisito | Como é atendido | Status |
|---|---|---|
| **Banco de dados** | PostgreSQL no backend (Django ORM, ~15 apps) + SecureStore no app | ✅ |
| **Mídia (câmera/vídeo/áudio)** | Câmera ao vivo no scanner de cupom (`expo-camera` + QR + OCR ML Kit) + foto do comprovante (`expo-image-picker`) | ✅ |
| **GPS** | Local da compra: busca por endereço + localização atual (`expo-location`) + mapa do Google (`react-native-maps`); gastos plotados no mapa | ✅ |

---

## 0 · Base / Autenticação

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Cadastro de conta (email + senha) | ✅ `POST /auth/registro/` | ✅ tela Cadastro (já loga) |
| Login JWT | ✅ `POST /auth/login/` | ✅ tela Login |
| Sessão persistente (restaura ao abrir) | ✅ `GET /auth/me/` | ✅ `AuthProvider`, tokens no SecureStore |
| Refresh automático do token em 401 | ✅ `/auth/refresh/` | ✅ Axios faz 1 refresh, senão derruba sessão |
| Logout (com blacklist) | ✅ `/auth/logout/` | ✅ em Ajustes |
| Recuperação de senha por email | ✅ | 🔲 sem tela ainda |
| **Vínculo entre 2 usuários** (compartilhar) | ✅ convite→aceite/recusa, par único | 🔲 sem tela ainda |

---

## 1 · Receitas (entradas)

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Cadastrar receita | ✅ `POST /receitas/` | ✅ via FAB "Lançar" |
| Status prevista × recebida | ✅ property derivada de `data_real` | ✅ |
| Marcar como recebida | ✅ `POST /receitas/{id}/receber/` | ✅ (lançamento já marca recebido hoje) |
| Receita recorrente (pré-cria a do mês seguinte) | ✅ idempotente | parcial (cria recebida hoje) |
| Cobertura do salário (saldo × faturas) | ✅ devolvido no `receber` | ✅ refletido no Dashboard |
| Receita compartilhada (rateio dono/vínculo) | ✅ RN-021 | 🔲 depende de telas de vínculo |
| Lista de receitas do mês com editar/excluir | ✅ API tem | 🔲 sem tela dedicada |

---

## 2 · Gastos variáveis + Scanner de cupom

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Cadastrar gasto | ✅ `POST /gastos/` | ✅ via FAB "Lançar" |
| Formas: pix / débito / crédito / dinheiro | ✅ | ✅ (crédito exige cartão) |
| `mes_referencia` derivado (competência do cartão no crédito) | ✅ | ✅ |
| Filtros (mês/categoria/cartão/forma) | ✅ | ✅ usados no Dashboard/Relatórios |
| Gasto compartilhado (rateio) | ✅ RN-021 | 🔲 depende de vínculo |
| **Local da compra (GPS)** — busca + localização atual + mapa | ✅ `latitude`/`longitude`/`local_nome` | ✅ `LocalPicker` (busca, GPS, mapa Google) |
| **Compra detalhada** (itens: nome/qtd/un/valor) | ✅ `CompraDetalhada`+`ItemCompra` | ✅ tela `compra.tsx` (manual-first) |
| **Scanner — QR da NFC-e** (busca itens na SEFAZ) | ✅ raspagem server-side | 🧪 lê QR; precisa device físico |
| **Scanner — OCR (fallback)** | ✅ parser heurístico de cupom | 🧪 OCR on-device ML Kit; precisa device físico |
| Comprovante de maquininha (só valor → gasto simples) | ✅ `forma_sugerida` | ✅ roteia pro modo simples |
| Foto do comprovante anexada (`comprovante`) | ✅ ImageField (Pillow) | ✅ `expo-image-picker` |

> **Scanner:** lógica e build prontos. Só dá pra calibrar OCR/QR com **cupons reais
> num celular físico** (o emulador usa câmera sintética).

---

## 3 · Gastos fixos (Tipo A fixo / Tipo B estimado)

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Cadastrar template fixo (A ou B) | ✅ `POST /gastos-fixos/` | ✅ `NovoFixoSheet` (FAB) |
| Gerar instância mensal | ✅ job `gerar_gastos_fixos` + na criação | ✅ aparece no mês corrente |
| Dar check / pagar (B exige valor real) | ✅ `POST /gastos-fixos-mensais/{id}/pagar/` | ✅ tela `fixos.tsx` (B abre `ValorRealSheet`) |
| Marcar atrasos (vencido → atrasado) | ✅ job `marcar_atrasos` | ✅ status refletido |
| Soft delete + reativar | ✅ | ✅ |

---

## 4 · Cartões e Faturas

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Cadastrar cartão (fechamento + vencimento) | ✅ `POST /cartoes/` RN-040 | ✅ `NovoCartaoSheet` |
| Soft delete (inativar) + reativar | ✅ | parcial |
| Fatura mensal (`UNIQUE(cartao, mês)`) | ✅ | ✅ |
| Composição da fatura (fixos + parcelas + variáveis) | ✅ `composicao()` com subtotais | ✅ `FaturaSheet` |
| Total da fatura derivado ao vivo | ✅ `SerializerMethodField` | ✅ |
| Limite usado × disponível | ✅ | ✅ |
| Pagar fatura | ✅ `POST /faturas/{id}/pagar/` | ✅ |
| Cor do cartão | — (cosmético) | ✅ derivada do id no app |

---

## 5 · Dívidas e Parcelamentos

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Cadastrar dívida/parcelamento | ✅ `POST /dividas/` gera parcelas RN-050 | ✅ `NovaDividaSheet` (FAB) |
| Parcelas ligadas à fatura (no cartão) | ✅ | ✅ invalida fatura/dashboard |
| Projeção de quitação (pago/restante/mês) | ✅ RN-051 | ✅ no `DividaCard` |
| Pagar parcela | ✅ `POST /parcelas/{id}/pagar/` | ✅ "Pagar parcela N" |
| Excluir dívida | ✅ DELETE | ✅ long-press |

---

## 6 · Metas de economia

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Cadastrar meta (nome/emoji/cor/alvo/prazo) | ✅ `POST /metas/` | ✅ `NovaMetaSheet` |
| Registrar aporte (incrementa valor) | ✅ `POST /aportes-meta/` RF-061 | ✅ `AporteSheet` |
| Progresso/ritmo (no_ritmo, aporte necessário) | ✅ RN-060 derivado | ✅ anel de progresso |
| Excluir meta | ✅ DELETE | ✅ long-press |
| "Aporte em meta" pelo FAB | — | 🔲 "em breve" (existe na aba Metas) |

---

## 7 · Investimentos (Fase 1: só aportes, sem rendimento)

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Cadastrar aporte | ✅ `POST /investimentos/` | ✅ `NovoInvestimentoSheet` |
| Consolidado (total, por tipo, por mês) | ✅ `GET /investimentos/consolidado/` RF-071 | ✅ hero + distribuição por tipo |
| Excluir aporte | ✅ | ✅ long-press |
| Rendimento / cotação | 🔲 fora do escopo da Fase 1 | 🔲 |

---

## 8 · Patrimônio (net worth)

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Cálculo ao vivo (ativos − passivos) | ✅ `calcular_patrimonio` RF-080 | ✅ tela `patrimonio.tsx` |
| Cadastrar/excluir bens | ✅ `POST/DELETE /bens/` | ✅ `NovoBemSheet` + long-press |
| Histórico mensal (snapshot) | ✅ job `gerar_snapshot_patrimonio` | parcial (mostra atual) |
| Gráfico de evolução | ✅ dados no histórico | 🔲 sem gráfico de série |

---

## Visão do mês / Dashboard

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Dashboard do mês (cards + seções) | ✅ `GET /dashboard/?mes=` | ✅ tela inicial |
| Cards: receitas previsto×recebido, fixos X/Y, faturas, saldo, economia | ✅ | ✅ |
| Fixos pendentes, faturas por cartão, últimos 5 lançamentos | ✅ | ✅ |
| Status do mês (fechado/aberto) | ✅ | ✅ |
| **Mapa de gastos do mês** (gastos com local plotados) | ✅ lista `/gastos/?mes_referencia=` | ✅ tela `mapa.tsx` (Ajustes → Mapa de gastos) |

---

## Categorias

| Funcionalidade | Backend | Mobile |
|---|---|---|
| 12 categorias predefinidas (seed no cadastro) | ✅ signal + `seed_categorias` | ✅ |
| Criar/editar categoria custom (teto 10) | ✅ | ✅ `CategoriaSheet` |
| Subcategorias e Tags | ✅ `/subcategorias` `/tags` | parcial |
| Soft delete + restaurar | ✅ `POST /restaurar/` | ✅ seção "Excluídas" |
| Excluir com lançamentos → reatribuir | ✅ 400 pede `reatribuir_para` | ✅ `ReatribuirCategoriaSheet` |

---

## Relatórios

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Gastos por categoria (este mês × anterior) | ✅ `GET /relatorios/gastos-por-categoria/?mes=` RF-100 | ✅ donut + lista |
| **Exportar em PDF** (RF-101) | 🔲 **não feito** | 🔲 |
| Filtros avançados de relatório (RF-102) | 🔲 só filtra por mês | 🔲 |

---

## Importação automática de Pix (por notificação Android)

| Funcionalidade | Backend | Mobile |
|---|---|---|
| Ler notificação do banco e parsear Pix (recebido/enviado) | ✅ `pix.py` + `MovimentacaoDetectada` | 🧪 `PixWatcher` + módulo nativo |
| Lista branca de bancos | ✅ mapa `BANCOS` | 🧪 `bancos.ts` (11 bancos) |
| Caixa de revisão (confirmar → Receita/Gasto, ou ignorar) | ✅ `/movimentacoes-detectadas/` | 🧪 tela `revisao.tsx` |
| Dedupe de notificação repetida | ✅ | — |
| Permissão "acesso a notificações" | — | 🧪 banner chama settings do Android |

> **Pronto no código e no build (Android).** Falta o **teste de fogo num celular
> físico** (passo a passo em `mobile/TESTE-PIX.md`). Limitações conhecidas:
> só Android, só no dev build, e **não captura com o app totalmente fechado**
> (refino futuro). **Risco nº1:** o package real do banco (ex.: Nubank) pode não
> bater com a lista branca.

---

## O que ainda falta (resumo — Fase 5, periféricos)

| Item | Requisito | Status |
|---|---|---|
| **Notificações/alertas** (email + tentativa Google Calendar) | RF-090/091 | 🔲 app `notificacoes` nem existe |
| **Importação de arquivo OFX/CSV** (sugestão de categoria, dedupe) | RF-110/111 | 🔲 `importacao` só tem o Pix |
| **Export PDF dos relatórios** | RF-101/102 | 🔲 não feito |
| Testar Pix por notificação no celular | — | 🧪 |
| Calibrar scanner de cupom com cupons reais | — | 🧪 device físico |
| Telas de **vínculo/compartilhamento** no mobile | RF-002 | 🔲 backend pronto, falta UI |
| Listas de Receitas/Gastos do mês com editar/excluir | — | 🔲 API pronta, falta UI |
| Recuperação de senha no mobile | RF-001 | 🔲 backend pronto, falta UI |

---

## Como rodar (referência rápida)

```bash
# Backend (terminal 1)
cd backend && docker compose up            # Postgres + Django em :8000

# Mobile (terminal 2)
cd mobile && npm install && npm start      # Expo Go (tudo menos scanner/Pix)

# Scanner de cupom + Pix por notificação → exigem dev build Android:
cd mobile && npx expo prebuild --clean && npx expo run:android
```

Detalhes completos em `SETUP.md` (raiz), `backend/CLAUDE.md` e `mobile/CLAUDE.md`.
Usuário de teste: `teste@poupa.com` / `teste1234`.
