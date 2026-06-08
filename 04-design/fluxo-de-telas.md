# Documento de Fluxo de Telas — Sistema de Finanças Pessoais

> **Status:** Em construção · Versão 0.2  
> **Última atualização:** Maio de 2026  
> **Referência:** Requisitos v0.5 · Visão v0.6

---

## Convenções

- **[T]** — Tela completa (página própria)
- **[M]** — Modal / painel sobreposto
- **[C]** — Componente dentro de uma tela
- `→` — navegação por clique
- `⟳` — atualização automática sem troca de tela

---

## 1. Estrutura de Navegação

```
Login [T]
│
└── Dashboard — Visão do Mês [T]  ← tela inicial após login
        │
        ├── Cartões de Crédito [T]
        │       └── Detalhe da Fatura [T]
        │
        ├── Gastos Fixos [T]
        │
        ├── Lançamentos — Dia a Dia [T]
        │
        ├── Dívidas e Parcelamentos [T]
        │
        ├── Metas de Economia [T]
        │
        ├── Investimentos [T]
        │
        ├── Patrimônio Líquido [T]
        │
        ├── Relatórios [T]
        │
        └── Configurações [T]
                ├── Perfil e usuários
                ├── Categorias
                ├── Cartões
                ├── Notificações
                └── Importação de dados
```

A navegação principal fica em uma **sidebar lateral fixa** (desktop) ou **menu inferior** (mobile — Fase 2).

---

## 2. Fluxo: Autenticação

### [T] Login
**Campos:** E-mail · Senha  
**Ações:**
- `Entrar` → valida credenciais → Dashboard
- `Esqueci minha senha` → [M] modal de recuperação por e-mail
- Sessão persistente: se token válido, redireciona direto para Dashboard sem exibir login

---

## 3. Fluxo: Dashboard — Visão do Mês

### [T] Dashboard
Tela central do sistema. Exibe o resumo financeiro do mês selecionado.

**Cabeçalho:**
- Mês/ano atual com setas de navegação (← mês anterior · mês seguinte →)
- Indicador de status do mês: Aberto / Fechado

**Cards de resumo [C]:**
| Card | Conteúdo |
|---|---|
| Receitas | Total previsto × total recebido no mês |
| Gastos Fixos | Total dos fixos × quantos já foram pagos (X/Y) |
| Cartões | Soma das faturas abertas dos cartões ativos |
| Saldo Disponível | Receitas recebidas − (fixos pagos + gastos variáveis + faturas pagas) |
| Economia do Mês | Receitas recebidas − total de gastos realizados |

**Seção: Gastos Fixos pendentes [C]**
- Lista dos compromissos fixos não pagos com vencimento mais próximo
- Cada item tem botão de check rápido inline
- Link `Ver todos` → [T] Gastos Fixos

**Seção: Faturas dos Cartões [C]**
- Um card por cartão com: nome, total da fatura, limite disponível, status
- Clique no card → [T] Detalhe da Fatura

**Seção: Últimos lançamentos [C]**
- 5 lançamentos mais recentes do dia a dia
- Link `Ver todos` → [T] Lançamentos

**Ação global:**
- Botão `+ Lançar` (fixo, sempre visível) → [M] Modal de novo lançamento

---

## 4. Fluxo: Lançamento Rápido

### [M] Modal — Novo Lançamento
Acessado pelo botão `+ Lançar` em qualquer tela.

**Passo 1 — Tipo:**
- Gasto do dia a dia
- Receita
- Gasto fixo (novo)
- Parcelamento (novo)
- Aporte em meta
- Aporte em investimento

Cada tipo exibe seu formulário específico (campos conforme RF correspondente).

**Comportamento:**
- Formulário inline no modal, sem troca de página
- Salvar → fecha modal → ⟳ atualiza os cards do Dashboard
- Cancelar → fecha modal sem salvar

---

## 5. Fluxo: Gastos Fixos

### [T] Gastos Fixos
Exibe todos os gastos fixos do mês selecionado agrupados por tipo.

**Grupo A — Compromissos Fixos:**
- Lista com: nome · categoria · vencimento · valor · forma de pagamento · status
- Status visual: 🟡 Pendente · ✅ Pago · 🔴 Atrasado
- Check inline → [M] Confirmar pagamento (para Tipo B: informa valor real)
- Clique no item → [M] Editar gasto fixo

**Grupo B — Recorrentes Estimados:**
- Mesma estrutura, mas ao dar check abre campo para informar valor real do mês

**Ações:**
- `+ Novo gasto fixo` → [M] Formulário de cadastro (escolhe Tipo A ou B)
- Filtros: Tipo · Categoria · Status · Forma de pagamento

---

## 6. Fluxo: Cartões de Crédito

### [T] Cartões
Painel com todos os cartões cadastrados.

**Por cartão:**
- Nome · Limite total · Limite disponível · Limite utilizado (barra de progresso)
- Valor da fatura atual · Status da fatura · Vencimento
- Clique → [T] Detalhe da Fatura

**Ações:**
- `+ Adicionar cartão` → [M] Formulário de cadastro de cartão

---

### [T] Detalhe da Fatura
Exibe a fatura completa de um cartão em um mês específico.

**Cabeçalho:**
- Nome do cartão · Mês de referência · Vencimento · Status
- Total da fatura · Limite disponível

**Seção 1 — Gastos Fixos no Cartão:**
- Lista de cobranças recorrentes esperadas com checkbox de confirmação
- Cada item: nome · valor · ✅ cobrado / ⬜ aguardando

**Seção 2 — Parcelamentos:**
- Tabela: descrição · parcela atual/total · valor da parcela · valor total da compra

**Seção 3 — Gastos Variáveis:**
- Lista de gastos do dia a dia lançados neste cartão neste mês
- Agrupados por categoria com subtotal

**Rodapé:**
- Subtotal de cada seção
- **Total geral**
- Botão `Marcar como Paga` → [M] Confirmação com data e valor

---

## 7. Fluxo: Lançamentos — Dia a Dia

### [T] Lançamentos
Lista de todos os gastos variáveis do mês.

**Visualizações:**
- Lista cronológica (padrão)
- Agrupada por categoria

**Por lançamento:**
- Data · Descrição · Categoria · Valor · Forma de pagamento
- Ícone de compartilhado + com quem (se aplicável)
- Clique → [M] Editar lançamento

**Filtros:**
- Categoria · Forma de pagamento · Origem (próprios / compartilhados) · Período · Tag

**Ações:**
- `+ Novo lançamento` → [M] Formulário
- `Importar arquivo` → [T] Fluxo de importação (ver seção 11)

---

## 8. Fluxo: Dívidas e Parcelamentos

### [T] Dívidas e Parcelamentos
Visão consolidada de todos os compromissos de longo prazo.

**Cards de resumo:**
- Total em aberto · Total já pago · Próxima parcela (data e valor)

**Lista de dívidas/parcelamentos:**
- Nome · Tipo · Progresso (barra: X de Y parcelas) · Próxima parcela · Quitação prevista
- Clique → [M] Detalhe com todas as parcelas e histórico

**Ações:**
- `+ Novo parcelamento` → [M] Formulário

---

## 9. Fluxo: Metas de Economia

### [T] Metas
Cards visuais de cada meta com indicador de progresso.

**Por meta:**
- Nome · Valor alvo · Valor atual · Progresso (%) · Data alvo
- Indicador: no prazo / atrasada (baseado no ritmo de aportes)
- Clique → [M] Detalhe com histórico de aportes

**Ações:**
- `+ Nova meta` → [M] Formulário
- `+ Registrar aporte` (dentro do detalhe) → [M] Formulário de aporte

---

## 10. Fluxo: Investimentos

### [T] Investimentos
Visão dos aportes realizados e totais acumulados.

**Cards de resumo:**
- Total geral aportado · Total por tipo (renda fixa, ações, etc.)

**Lista de aportes:**
- Data · Tipo · Instituição · Descrição · Valor aportado
- Clique → [M] Editar aporte

**Ações:**
- `+ Registrar aporte` → [M] Formulário

---

## 11. Fluxo: Importação de Arquivo

### [T] Importação
Fluxo em 4 passos sequenciais (wizard):

**Passo 1 — Upload:**
- Arraste ou selecione arquivo OFX ou CSV
- Sistema valida formato e exibe prévia do número de transações encontradas

**Passo 2 — Revisão:**
- Tabela com todas as transações importadas
- Para cada linha: data · descrição · valor · categoria sugerida (editável) · forma de pagamento (editável)
- Transações duplicadas sinalizadas com ⚠️ (usuário decide manter ou descartar)

**Passo 3 — Confirmação:**
- Resumo: X transações a importar · Y duplicatas descartadas
- Botão `Confirmar importação`

**Passo 4 — Resultado:**
- Confirmação de sucesso com total importado
- Link para ver os lançamentos importados

---

## 12. Fluxo: Relatórios

### [T] Relatórios
Hub de todos os relatórios disponíveis.

**Seleção de relatório:**
- Cards com nome e descrição de cada relatório disponível
- Clique → abre o relatório com filtros padrão (mês atual)

**Filtros globais dos relatórios:**
- Período (mês/ano ou intervalo personalizado)
- Origem (meus lançamentos / compartilhados comigo / ambos)
- Categoria

**Por relatório:**
- Visualização em tela com gráficos e tabelas
- Botão `Exportar PDF` → gera e baixa o PDF

---

## 13. Fluxo: Configurações

### [T] Configurações
Painel com subseções:

**Perfil e vínculos:**
- Dados do usuário · Alterar senha
- Vínculos: enviar convite por e-mail · convites recebidos (aceitar/recusar) · vínculos ativos (desfazer)

**Categorias:**
- Lista das 12 categorias pré-definidas (renomeáveis)
- Lista das categorias customizadas (criar · renomear · excluir)
- Contador: X de 10 categorias customizadas utilizadas

**Cartões:**
- Lista de cartões cadastrados com opção de editar / ativar / inativar
- `+ Adicionar cartão`

**Notificações:**
- Toggle por tipo de alerta (ativo/inativo)
- Campo de antecedência configurável (dias) por tipo
- Canal: e-mail (sempre ativo) · Google Calendar (conectar/desconectar)

**Importação:**
- Histórico de importações realizadas (data · arquivo · quantidade de transações)

---

*Este documento é vivo. Referência: Requisitos v0.5*