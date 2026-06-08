# Modelo de Dados — Sistema de Finanças Pessoais

> **Status:** Em construção · Versão 0.2  
> **Última atualização:** Maio de 2026  
> **Referência:** Requisitos v0.5  
> **Destino:** Models Django + tabelas PostgreSQL

---

## Convenções

- **PK** — chave primária · **FK** — chave estrangeira
- Tipos seguem PostgreSQL; entre parênteses o equivalente no Django
- `DECIMAL(12,2)` para todo valor monetário (até ~10 bilhões com 2 casas)
- Toda tabela tem `id` (BIGSERIAL/UUID) como PK e timestamps `created_at` / `updated_at` implícitos
- `mes_referencia` é sempre `DATE` no primeiro dia do mês (ex: 2026-05-01) — facilita filtros e agrupamentos
- Enums implementados como `VARCHAR` com `choices` no Django

---

## 1. Núcleo: Usuário e Vínculo

Cada pessoa é um **usuário independente**, dono dos seus próprios dados, **privados por padrão**. Não existe conta familiar compartilhada: o compartilhamento acontece **item a item** entre usuários conectados por um **vínculo** (ver abaixo).

### Usuario
É o usuário raiz do sistema (implementado como **custom user model** do Django — definir antes da primeira migração).

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| nome | VARCHAR(100) | NOT NULL | — |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Login |
| senha_hash | VARCHAR(255) | NOT NULL | bcrypt/argon2 |

Toda entidade de dados pertence a **um** usuário via `usuario_id` (o **dono**). É por esse campo que tudo é filtrado (scoping por usuário).

### Vinculo
Conexão entre **dois usuários** que querem compartilhar lançamentos. Criada por convite e aceite.

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_solicitante_id | BIGINT | FK → Usuario, NOT NULL | Quem enviou o convite |
| usuario_destinatario_id | BIGINT | FK → Usuario, NOT NULL | Quem recebeu |
| status | VARCHAR(10) | DEFAULT 'pendente' | pendente / aceito / recusado |
| accepted_at | TIMESTAMP | NULL | Quando foi aceito |

**Constraint:** UNIQUE (usuario_solicitante_id, usuario_destinatario_id) — sem convites duplicados entre o mesmo par.

**Relacionamento:** `Usuario N:M Usuario` através de `Vinculo`. Um usuário pode ter vínculos com mais de uma pessoa.

### Compartilhamento de lançamentos
Os lançamentos compartilháveis (**Receita, Gasto, GastoFixo, Divida**) trazem, quando compartilhados, três campos comuns:

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| compartilhado | BOOLEAN | DEFAULT false | Se o item é dividido com um vínculo |
| vinculo_id | BIGINT | FK → Vinculo, NULL | Vínculo com quem se compartilha (obrigatório se compartilhado) |
| valor_dono | DECIMAL(12,2) | NULL | Porção de quem lançou |
| valor_vinculado | DECIMAL(12,2) | NULL | Porção da outra pessoa |

**Regra (aplicação):** se `compartilhado = true`, então `valor_dono + valor_vinculado = valor` e `vinculo_id` é obrigatório. O item continua pertencendo ao **dono** (`usuario_id`); o usuário vinculado o enxerga como "compartilhado comigo" através do vínculo, sem duplicação.

---

## 2. Categorização

Categorias, subcategorias e tags são **por usuário** (cada um tem as suas).

### Categoria
12 pré-definidas (não excluíveis) + até 10 customizadas por usuário.

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| nome | VARCHAR(60) | NOT NULL | — |
| cor | VARCHAR(7) | NULL | Hex (#RRGGBB) |
| icone | VARCHAR(40) | NULL | Nome do ícone |
| predefinida | BOOLEAN | DEFAULT false | Se é uma das 12 fixas |
| ativa | BOOLEAN | DEFAULT true | — |

**Regra:** máximo de 10 categorias com `predefinida = false` por usuário (validação na aplicação).

### Subcategoria
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| categoria_id | BIGINT | FK → Categoria, NOT NULL | — |
| nome | VARCHAR(60) | NOT NULL | — |

**Relacionamentos:** `Usuario 1:N Categoria` · `Categoria 1:N Subcategoria`

### Tag
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| nome | VARCHAR(40) | NOT NULL | — |

---

## 3. Receitas

### Receita
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| descricao | VARCHAR(120) | NOT NULL | — |
| valor | DECIMAL(12,2) | NOT NULL | — |
| data_prevista | DATE | NOT NULL | — |
| data_real | DATE | NULL | Preenchida ao receber |
| tipo | VARCHAR(20) | NOT NULL | salario / freelance / aluguel / bonus / outro |
| recorrente | BOOLEAN | DEFAULT false | — |
| compartilhada | BOOLEAN | DEFAULT false | Ver §1 · Compartilhamento |
| vinculo_id | BIGINT | FK → Vinculo, NULL | Se compartilhada |
| valor_dono | DECIMAL(12,2) | NULL | Se compartilhada |
| valor_vinculado | DECIMAL(12,2) | NULL | Se compartilhada |
| mes_referencia | DATE | NOT NULL | — |

**Status** é derivado: `recebida` se `data_real` preenchida, senão `prevista`.

**Relacionamento:** `Usuario 1:N Receita`

---

## 4. Gastos do Dia a Dia + Scanner

### Gasto
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono (quem gastou) |
| descricao | VARCHAR(120) | NOT NULL | — |
| valor | DECIMAL(12,2) | NOT NULL | Total do gasto |
| data | DATE | NOT NULL | — |
| categoria_id | BIGINT | FK → Categoria, NOT NULL | — |
| subcategoria_id | BIGINT | FK → Subcategoria, NULL | — |
| forma_pagamento | VARCHAR(15) | NOT NULL | dinheiro / pix / debito / credito |
| cartao_id | BIGINT | FK → Cartao, NULL | Obrigatório se crédito |
| compartilhado | BOOLEAN | DEFAULT false | Ver §1 · Compartilhamento |
| vinculo_id | BIGINT | FK → Vinculo, NULL | Se compartilhado |
| valor_dono | DECIMAL(12,2) | NULL | Se compartilhado |
| valor_vinculado | DECIMAL(12,2) | NULL | Se compartilhado |
| observacao | TEXT | NULL | — |
| origem | VARCHAR(10) | DEFAULT 'manual' | manual / qr / ocr |
| mes_referencia | DATE | NOT NULL | — |

### GastoTag (tabela de junção N:M)
| Campo | Tipo | Constraint |
|---|---|---|
| gasto_id | BIGINT | FK → Gasto |
| tag_id | BIGINT | FK → Tag |

PK composta (gasto_id, tag_id).

### CompraDetalhada
Detalhamento opcional de um gasto (compras de supermercado via scanner).

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| gasto_id | BIGINT | FK → Gasto, UNIQUE, NOT NULL | Relação 1:1 |
| estabelecimento | VARCHAR(120) | NULL | Nome do mercado |
| origem | VARCHAR(10) | NOT NULL | qr / ocr / manual |
| url_nfce | TEXT | NULL | URL do QR Code lido |

### ItemCompra
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| compra_detalhada_id | BIGINT | FK → CompraDetalhada, NOT NULL | — |
| nome | VARCHAR(120) | NOT NULL | — |
| valor | DECIMAL(12,2) | NOT NULL | — |
| quantidade | DECIMAL(10,3) | NULL | Ex: 1.500 (kg) |
| categoria_id | BIGINT | FK → Categoria, NULL | Herda a do gasto se nulo |
| identificado | BOOLEAN | DEFAULT true | false = veio em branco do OCR |

**Relacionamentos:**  
`Usuario 1:N Gasto` · `Categoria 1:N Gasto` · `Cartao 1:N Gasto`  
`Gasto 1:1 CompraDetalhada` · `CompraDetalhada 1:N ItemCompra` · `Gasto N:M Tag`

---

## 5. Gastos Fixos

### GastoFixo
Cadastro do gasto fixo (template). Tipo A = valor fixo; Tipo B = valor estimado/variável.

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| descricao | VARCHAR(120) | NOT NULL | — |
| tipo | VARCHAR(1) | NOT NULL | A (fixo) / B (estimado) |
| valor | DECIMAL(12,2) | NULL | Obrigatório se tipo A |
| valor_estimado | DECIMAL(12,2) | NULL | Referência se tipo B |
| dia_vencimento | SMALLINT | NULL | 1–31 |
| categoria_id | BIGINT | FK → Categoria, NOT NULL | — |
| forma_pagamento | VARCHAR(15) | NULL | cartao / debito / pix / boleto |
| cartao_id | BIGINT | FK → Cartao, NULL | Se pago no cartão |
| compartilhado | BOOLEAN | DEFAULT false | Ver §1 · Compartilhamento |
| vinculo_id | BIGINT | FK → Vinculo, NULL | Se compartilhado |
| valor_dono | DECIMAL(12,2) | NULL | Se compartilhado |
| valor_vinculado | DECIMAL(12,2) | NULL | Se compartilhado |
| ativo | BOOLEAN | DEFAULT true | — |

### GastoFixoMensal
Instância mensal gerada automaticamente a partir do template.

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| gasto_fixo_id | BIGINT | FK → GastoFixo, NOT NULL | — |
| mes_referencia | DATE | NOT NULL | — |
| valor_real | DECIMAL(12,2) | NULL | Informado no check (tipo B) |
| status | VARCHAR(10) | DEFAULT 'pendente' | pendente / pago / atrasado |
| data_pagamento | DATE | NULL | — |
| checked_at | TIMESTAMP | NULL | Quando deu check |

**Constraint:** UNIQUE (gasto_fixo_id, mes_referencia) — evita duplicar no mesmo mês.

**Relacionamentos:** `Usuario 1:N GastoFixo` · `GastoFixo 1:N GastoFixoMensal`

---

## 6. Cartões e Faturas

### Cartao
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| nome | VARCHAR(60) | NOT NULL | Apelido |
| limite_total | DECIMAL(12,2) | NOT NULL | — |
| dia_fechamento | SMALLINT | NOT NULL | 1–31 |
| dia_vencimento | SMALLINT | NOT NULL | 1–31 |
| status | VARCHAR(10) | DEFAULT 'ativo' | ativo / inativo |

### Fatura
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| cartao_id | BIGINT | FK → Cartao, NOT NULL | — |
| mes_referencia | DATE | NOT NULL | — |
| total | DECIMAL(12,2) | DEFAULT 0 | Calculado |
| status | VARCHAR(10) | DEFAULT 'aberta' | aberta / fechada / paga |
| data_pagamento | DATE | NULL | — |
| valor_pago | DECIMAL(12,2) | NULL | — |

**Constraint:** UNIQUE (cartao_id, mes_referencia).

A composição da fatura (fixos + parcelas + variáveis) vem por relacionamento: gastos com `cartao_id`, parcelas com `fatura_id` e gastos fixos no cartão são agregados por `mes_referencia`.

**Relacionamentos:** `Usuario 1:N Cartao` · `Cartao 1:N Fatura`

---

## 7. Dívidas e Parcelamentos

### Divida
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| descricao | VARCHAR(120) | NOT NULL | — |
| tipo | VARCHAR(20) | NOT NULL | parcelamento_cartao / financiamento / emprestimo / informal |
| valor_total | DECIMAL(12,2) | NOT NULL | — |
| numero_parcelas | SMALLINT | NOT NULL | — |
| valor_parcela | DECIMAL(12,2) | NOT NULL | — |
| parcela_inicial | SMALLINT | DEFAULT 1 | A partir de qual nº começa |
| data_primeira_parcela | DATE | NOT NULL | — |
| juros | DECIMAL(6,3) | NULL | % se houver |
| cartao_id | BIGINT | FK → Cartao, NULL | Se parcelamento no cartão |
| compartilhado | BOOLEAN | DEFAULT false | Ver §1 · Compartilhamento |
| vinculo_id | BIGINT | FK → Vinculo, NULL | Se compartilhado |
| valor_dono | DECIMAL(12,2) | NULL | Se compartilhado |
| valor_vinculado | DECIMAL(12,2) | NULL | Se compartilhado |

### Parcela
Geradas automaticamente ao cadastrar a dívida.

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| divida_id | BIGINT | FK → Divida, NOT NULL | — |
| numero | SMALLINT | NOT NULL | Ex: 3 (de 12) |
| valor | DECIMAL(12,2) | NOT NULL | — |
| mes_referencia | DATE | NOT NULL | — |
| data_vencimento | DATE | NOT NULL | — |
| status | VARCHAR(10) | DEFAULT 'pendente' | pendente / paga |
| fatura_id | BIGINT | FK → Fatura, NULL | Se cai numa fatura de cartão |

**Relacionamentos:** `Usuario 1:N Divida` · `Divida 1:N Parcela` · `Fatura 1:N Parcela` (opcional)

---

## 8. Metas de Economia

### Meta
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| nome | VARCHAR(100) | NOT NULL | — |
| valor_alvo | DECIMAL(12,2) | NOT NULL | — |
| valor_atual | DECIMAL(12,2) | DEFAULT 0 | — |
| data_alvo | DATE | NULL | — |
| contribuicao_mensal_planejada | DECIMAL(12,2) | NULL | — |

### AporteMeta
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| meta_id | BIGINT | FK → Meta, NOT NULL | — |
| valor | DECIMAL(12,2) | NOT NULL | — |
| data | DATE | NOT NULL | — |
| observacao | VARCHAR(255) | NULL | — |

**Relacionamentos:** `Usuario 1:N Meta` · `Meta 1:N AporteMeta`

---

## 9. Investimentos

### Investimento
Cada registro é um aporte (Fase 1 — sem cálculo de rendimento).

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| tipo | VARCHAR(20) | NOT NULL | renda_fixa / acoes / fiis / cripto / tesouro / poupanca / outro |
| instituicao | VARCHAR(80) | NULL | — |
| descricao | VARCHAR(120) | NULL | — |
| valor_aportado | DECIMAL(12,2) | NOT NULL | — |
| data_aporte | DATE | NOT NULL | — |

**Relacionamento:** `Usuario 1:N Investimento`

---

## 10. Patrimônio Líquido

### Bem
Ativos de valor estimado, inseridos manualmente.

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| descricao | VARCHAR(120) | NOT NULL | — |
| tipo | VARCHAR(20) | NOT NULL | imovel / veiculo / outro |
| valor_estimado | DECIMAL(12,2) | NOT NULL | — |

### PatrimonioSnapshot
Foto do patrimônio ao fim de cada mês (para o gráfico de evolução).

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| mes_referencia | DATE | NOT NULL | — |
| total_ativos | DECIMAL(12,2) | NOT NULL | — |
| total_passivos | DECIMAL(12,2) | NOT NULL | — |
| patrimonio_liquido | DECIMAL(12,2) | NOT NULL | ativos − passivos |

**Constraint:** UNIQUE (usuario_id, mes_referencia).

**Relacionamentos:** `Usuario 1:N Bem` · `Usuario 1:N PatrimonioSnapshot`

---

## 11. Notificações e Importação

### ConfigNotificacao
| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| tipo_alerta | VARCHAR(30) | NOT NULL | fatura_vencendo / conta_vencendo / conta_atrasada / salario_recebido / meta_atingida / gasto_anomalo |
| ativo | BOOLEAN | DEFAULT true | — |
| antecedencia_dias | SMALLINT | NULL | Configurável quando aplicável |
| canal | VARCHAR(15) | DEFAULT 'email' | email / calendar |

### Importacao
Histórico de importações de arquivo (OFX/CSV).

| Campo | Tipo | Constraint | Descrição |
|---|---|---|---|
| id | BIGSERIAL | PK | — |
| usuario_id | BIGINT | FK → Usuario, NOT NULL | Dono |
| arquivo_nome | VARCHAR(255) | NOT NULL | — |
| formato | VARCHAR(5) | NOT NULL | ofx / csv |
| quantidade_transacoes | INTEGER | NOT NULL | — |
| data_importacao | TIMESTAMP | NOT NULL | — |

**Relacionamentos:** `Usuario 1:N ConfigNotificacao` · `Usuario 1:N Importacao`

---

## 12. Resumo dos Relacionamentos

```
Usuario (1)
 ├─< Categoria (N) ─< Subcategoria (N)
 ├─< Tag (N)
 ├─< Receita (N)
 ├─< Gasto (N) ─1:1─ CompraDetalhada ─< ItemCompra (N)
 │      └─N:M─ Tag (via GastoTag)
 ├─< GastoFixo (N) ─< GastoFixoMensal (N)
 ├─< Cartao (N) ─< Fatura (N) ─< Parcela (N, opcional)
 ├─< Divida (N) ─< Parcela (N)
 ├─< Meta (N) ─< AporteMeta (N)
 ├─< Investimento (N)
 ├─< Bem (N)
 ├─< PatrimonioSnapshot (N)
 ├─< ConfigNotificacao (N)
 └─< Importacao (N)

Compartilhamento (entre usuários):
 Usuario ──< Vinculo >── Usuario        (conexão por convite/aceite, N:M)
 Receita / Gasto / GastoFixo / Divida → Vinculo   (quando compartilhado)

Referências cruzadas:
 Gasto      → Categoria, Subcategoria, Cartao
 GastoFixo  → Categoria, Cartao
 Divida     → Cartao
 Parcela    → Fatura (opcional)
 ItemCompra → Categoria (opcional)
```

---

## 13. Notas de Implementação

- **Scoping por usuário:** toda query filtra por `usuario_id` (o dono). Não há conta compartilhada — o cruzamento entre pessoas acontece **somente** via `Vinculo` nos itens marcados como compartilhados.
- **Visibilidade do compartilhado:** um item compartilhado pertence ao dono; o usuário do outro lado do vínculo o enxerga (somente leitura) como "compartilhado comigo", junto com a sua porção (`valor_vinculado`). O item **não é duplicado**.
- **Soft delete:** considerar `ativo`/`deleted_at` nas entidades que o usuário pode "remover" mas que têm histórico vinculado (Categoria, Cartao, GastoFixo), para não quebrar lançamentos antigos.
- **Valores compartilhados:** a validação `valor_dono + valor_vinculado = valor` (e a obrigatoriedade de `vinculo_id`) é regra de aplicação, não de banco.
- **mes_referencia denormalizado:** está presente em várias tabelas de propósito — acelera os filtros mensais que são o coração do sistema. O valor é derivado da data do lançamento conforme o ciclo (para cartão, considera o dia de fechamento).
- **Geração automática:** GastoFixoMensal, Fatura e Parcela são criados por rotinas automáticas (job mensal ou na criação do template/dívida), não manualmente.
- **Cálculos derivados** (saldo, total da fatura, patrimônio, progresso de meta) não são armazenados como verdade absoluta exceto no PatrimonioSnapshot — o resto é calculado em tempo de consulta para evitar inconsistências.

---

*Este documento é vivo. Referência: Requisitos v0.5*
