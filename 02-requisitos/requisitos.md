# Documento de Requisitos — Sistema de Finanças Pessoais

> **Status:** Em construção · Versão 0.5  
> **Última atualização:** Maio de 2026  
> **Referência:** Documento de Visão v0.5  
> **Leitores:** Autor · Desenvolvedor · Agente de IA  
> **Infraestrutura:** Oracle Cloud (uso pessoal)

---

## Convenções deste documento

- **RF** — Requisito Funcional
- **RN** — Regra de Negócio
- **RNF** — Requisito Não Funcional
- **[DEVE]** — obrigatório para o sistema funcionar
- **[DEVERIA]** — importante, mas não bloqueia o MVP
- **[PODE]** — desejável, implementado quando possível

---

## 1. Requisitos Gerais do Sistema

### RF-001 · Autenticação
O sistema **[DEVE]** exigir autenticação para acessar qualquer funcionalidade.
- Login por e-mail e senha
- Sessão persistente com token (JWT ou similar)
- Recuperação de senha via e-mail

### RF-002 · Usuários independentes e vínculo de compartilhamento
O sistema **[DEVE]** tratar cada pessoa como um **usuário independente**, dono dos seus próprios dados, **privados por padrão**. Não existe conta familiar compartilhada.
- Dois usuários podem se **conectar por um vínculo**: um envia o convite por e-mail, o outro aceita
- Cada um continua dono e responsável pelos seus lançamentos; nada é visível ao outro por padrão
- O compartilhamento é **item a item**: um lançamento pode ser marcado como compartilhado com um vínculo (ver RN-021)
- Não há hierarquia entre os usuários conectados

### RN-002 · Vínculo entre usuários
Um vínculo só passa a valer quando o convite é **aceito** pelo destinatário. Status: **pendente → aceito / recusado**.
- Compartilhar um lançamento exige um vínculo **aceito**
- Ao desfazer um vínculo, os itens já compartilhados deixam de ser visíveis ao outro usuário, mas permanecem com o dono

### RF-003 · Período base: mês
O sistema **[DEVE]** organizar todos os dados por mês/ano como unidade principal.
- A tela inicial exibe sempre o mês atual
- O usuário pode navegar entre meses anteriores e futuros
- Dados de meses anteriores são somente leitura após fechamento (ver RN-001)

### RN-001 · Fechamento de mês
Um mês é considerado **fechado** quando o usuário explicitamente o fecha ou quando já se passaram 15 dias do início do mês seguinte.  
Meses fechados não permitem edição de lançamentos, apenas consulta e exportação.

---

## 2. Módulo: Renda / Receitas

### RF-010 · Cadastro de receita
O sistema **[DEVE]** permitir registrar uma receita com os seguintes campos:

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| Descrição | Texto | Sim | Ex: "Salário maio" |
| Valor | Decimal | Sim | Valor recebido |
| Data prevista | Data | Sim | Quando deveria entrar |
| Data real | Data | Não | Quando entrou de fato |
| Tipo | Enum | Sim | Salário / Freelance / Aluguel / Bônus / Outro |
| Recorrente | Boolean | Sim | Se repete todo mês |
| Compartilhada | Boolean | Sim | Padrão: false |
| Vínculo | Referência | Só se compartilhada | Com quem se compartilha |
| Valor do dono / do vinculado | Decimal (×2) | Só se compartilhada | Soma deve ser igual ao valor total |

### RF-011 · Status da receita
Cada receita possui um status:
- **Prevista** — data real não preenchida
- **Recebida** — data real preenchida

### RN-010 · Gatilho ao marcar receita como recebida
Quando uma receita do tipo **Salário** é marcada como **Recebida**, o sistema deve:
1. Calcular o saldo disponível do usuário no mês
2. Verificar se a fatura(s) do(s) cartão(ões) do mês está coberta pelo saldo
3. Exibir um resumo: "Salário recebido. Saldo disponível: R$ X. Fatura(s): R$ Y. [Coberta / Atenção: falta R$ Z]"

### RN-011 · Receitas recorrentes
Se uma receita é marcada como recorrente, o sistema deve **pré-criar automaticamente** a mesma receita para o mês seguinte com status **Prevista**, mantendo todos os campos exceto a data real.

---

## 3. Módulo: Gastos do Dia a Dia

### RF-020 · Cadastro de gasto variável
O sistema **[DEVE]** permitir registrar um gasto com os seguintes campos:

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| Descrição | Texto | Sim | Ex: "Almoço no trabalho" |
| Valor | Decimal | Sim | — |
| Data | Data | Sim | Data do gasto |
| Categoria | Referência | Sim | Categoria criada pelo usuário |
| Subcategoria | Referência | Não | Subcategoria criada pelo usuário |
| Forma de pagamento | Enum | Sim | Dinheiro / Pix / Débito / Crédito |
| Cartão | Referência | Só se crédito | Qual cartão foi usado |
| Compartilhado | Boolean | Sim | Padrão: false |
| Vínculo | Referência | Só se compartilhado | Com quem se compartilha |
| Valor do dono / do vinculado | Decimal (×2) | Só se compartilhado | Soma deve ser igual ao valor total |
| Tags | Lista de texto | Não | Marcadores livres |
| Observação | Texto longo | Não | Campo livre |

### RF-021 · Categorias e subcategorias customizáveis

#### Categorias pré-definidas
O sistema inicia com as seguintes categorias já cadastradas e não excluíveis (podem ser renomeadas):

| # | Categoria | Exemplos de uso |
|---|---|---|
| 1 | Moradia | Aluguel, condomínio, IPTU, reformas |
| 2 | Alimentação | Mercado, restaurantes, delivery |
| 3 | Transporte | Combustível, Uber, manutenção, pedágio |
| 4 | Saúde | Plano de saúde, médico, farmácia, exames |
| 5 | Educação | Escola, faculdade, cursos, livros |
| 6 | Lazer | Cinema, viagens, hobbies, festas |
| 7 | Vestuário | Roupas, calçados, acessórios |
| 8 | Assinaturas | Streaming, apps, revistas |
| 9 | Comunicação | Celular, internet fixa |
| 10 | Pets | Ração, veterinário, banho e tosa |
| 11 | Beleza | Cabelo, estética, cuidados pessoais |
| 12 | Impostos e taxas | IR, IPVA, tarifas bancárias |

#### Categorias customizadas
O usuário pode criar **até 10 categorias adicionais** além das pré-definidas.

#### Regras gerais
- Subcategorias podem ser criadas livremente dentro de qualquer categoria (sem limite)
- Cada categoria pode ter cor e ícone definidos pelo usuário
- Categorias pré-definidas não podem ser excluídas, apenas renomeadas
- Categorias customizadas podem ser criadas, renomeadas e excluídas
- Ao excluir uma categoria customizada com lançamentos vinculados, o sistema solicita que o usuário reatribua os lançamentos antes de confirmar a exclusão
- Categorias são **pessoais**: cada usuário tem o seu próprio conjunto (12 pré-definidas + até 10 customizadas)

### RN-020 · Gasto no crédito
Quando a forma de pagamento for **Crédito**, o campo **Cartão** torna-se obrigatório.  
O gasto é vinculado à fatura aberta do cartão selecionado no mês correspondente à data do gasto.

### RN-021 · Gasto compartilhado
Quando um gasto é marcado como compartilhado:
- É obrigatório escolher um **vínculo aceito** (com quem se compartilha) — ver RN-002
- O sistema exibe dois campos de valor: a porção do dono e a porção do vinculado
- A soma dos dois valores **deve ser igual** ao valor total do gasto
- O sistema valida e bloqueia o salvamento se a soma não bater
- O gasto continua pertencendo ao dono; o vinculado o enxerga (somente leitura) com a sua porção

---

## 3A. Módulo: Compras de Supermercado (Scanner de Cupom)

Extensão do módulo de Gastos do Dia a Dia. Permite registrar uma compra com **detalhamento item a item**, sem digitar tudo manualmente. É uma feature primariamente **mobile** (depende da câmera).

### RF-022 · Registro de compra detalhada
O sistema **[DEVE]** permitir registrar uma compra composta por múltiplos itens:

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| Estabelecimento | Texto | Não | Nome do mercado (pode vir do cupom) |
| Data | Data | Sim | Data da compra |
| Itens | Lista | Sim | Cada item com nome e valor |
| Categoria geral | Referência | Sim | Padrão: Alimentação |
| Forma de pagamento | Enum | Sim | Dinheiro / Pix / Débito / Crédito |
| Cartão | Referência | Só se crédito | — |
| Compartilhado | Boolean | Sim | Padrão: false |

Cada **item** da compra contém:

| Campo | Tipo | Obrigatório |
|---|---|---|
| Nome | Texto | Sim |
| Valor | Decimal | Sim |
| Quantidade | Decimal | Não |
| Categoria do item | Referência | Não (herda a geral) |

### RF-023 · Captura via QR Code da NFC-e
O sistema **[DEVE]** ler o QR Code da Nota Fiscal de Consumidor Eletrônica (NFC-e) e extrair automaticamente os itens da compra.

- A URL contida no QR Code aponta para o portal da SEFAZ do estado emissor
- O sistema consulta os dados e popula a lista de itens automaticamente
- O usuário revisa antes de confirmar

### RF-024 · Captura via OCR (fallback)
Quando não houver QR Code ou a leitura falhar, o sistema **[DEVERIA]** permitir fotografar o cupom físico e extrair os itens via OCR.

- Itens identificados com confiança são preenchidos automaticamente
- Itens não identificados aparecem em branco para preenchimento manual
- O usuário pode adicionar, editar ou remover itens livremente

### RF-025 · Revisão antes de salvar
Antes de confirmar, o sistema **[DEVE]** exibir a lista completa de itens para revisão, permitindo editar nome/valor, remover itens e adicionar novos. O total é recalculado em tempo real.

### RN-022 · Origem da leitura
O sistema registra a origem de cada compra detalhada: **QR Code**, **OCR** ou **manual**. Isso permite avaliar a confiabilidade dos dados e melhorar o parser ao longo do tempo.

### RN-023 · Vínculo com Gastos do Dia a Dia
Uma compra detalhada é salva como **um lançamento** no módulo de Gastos do Dia a Dia, com o valor total. Os itens individuais ficam armazenados como detalhamento do lançamento, acessíveis ao abrir o gasto.

### RN-024 · Dependência de estado (SEFAZ)
A consulta via QR Code depende do portal da SEFAZ do estado emissor da nota. O domínio e o formato da página podem variar e mudar com o tempo (ex.: o RN migrou de `set.rn.gov.br` para `nfce.sefaz.rn.gov.br`). O sistema **[DEVERIA]** abstrair essa consulta através de um serviço intermediário (própria camada de backend ou API de terceiros como Nuvem Fiscal) para isolar essas mudanças.

> **Status:** Feature validada via MVP mobile separado antes da integração ao sistema principal.

---

## 4. Módulo: Gastos Fixos

### RF-030 · Tipo A — Compromissos Fixos
Gastos de valor fixo que se repetem mensalmente:

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| Descrição | Texto | Sim | Ex: "Aluguel" |
| Valor | Decimal | Sim | Valor fixo mensal |
| Dia de vencimento | Inteiro (1–31) | Sim | — |
| Forma de pagamento | Enum | Sim | Cartão / Débito / Pix / Boleto |
| Cartão | Referência | Só se cartão | Qual cartão |
| Categoria | Referência | Sim | — |
| Compartilhado | Boolean | Sim | Padrão: false |
| Vínculo | Referência | Só se compartilhado | Com quem se compartilha |
| Valor do dono / do vinculado | Decimal (×2) | Só se compartilhado | Soma = valor total |
| Ativo | Boolean | Sim | Padrão: true |

**Status mensal de cada compromisso:**
- **Pendente** — ainda não pago
- **Pago** — usuário deu check
- **Atrasado** — data de vencimento passou sem check

### RF-031 · Tipo B — Recorrentes Estimados
Gastos que se repetem mas com valor variável:

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| Descrição | Texto | Sim | Ex: "Conta de luz" |
| Valor estimado | Decimal | Não | Referência histórica |
| Dia de vencimento | Inteiro (1–31) | Não | — |
| Categoria | Referência | Sim | — |
| Compartilhado | Boolean | Sim | Padrão: false |

A cada mês, o usuário informa o **valor real** daquele mês no momento do pagamento.

### RN-030 · Geração automática mensal de gastos fixos
Todo mês, o sistema deve **pré-criar automaticamente** todos os gastos fixos ativos (Tipo A e B) com status **Pendente**, baseando-se nos cadastros ativos.

### RN-031 · Check de pagamento
Ao dar check em um gasto fixo:
- O sistema registra a data e hora do check
- O status muda para **Pago**
- Para Tipo B, o usuário informa o valor real antes de confirmar o check

### RN-032 · Alerta de atraso
Se a data de vencimento de um Compromisso Fixo (Tipo A) passar sem check, o sistema muda o status para **Atrasado** e dispara notificação (ver módulo de Notificações).

---

## 5. Módulo: Cartão de Crédito

### RF-040 · Cadastro de cartão
O sistema **[DEVE]** permitir cadastrar múltiplos cartões:

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| Nome / apelido | Texto | Sim | Ex: "Nubank", "Inter" |
| Limite total | Decimal | Sim | — |
| Dia de fechamento | Inteiro (1–31) | Sim | Dia em que a fatura fecha |
| Dia de vencimento | Inteiro (1–31) | Sim | Dia em que a fatura vence |
| Status | Enum | Sim | Ativo / Inativo |

### RF-041 · Fatura mensal
Para cada cartão e cada mês, o sistema gera automaticamente uma **fatura** composta por:

1. **Gastos fixos no cartão** — compromissos fixos (Tipo A) cuja forma de pagamento seja aquele cartão
2. **Parcelamentos** — parcelas do mês de compras parceladas vinculadas ao cartão
3. **Gastos variáveis** — gastos do dia a dia lançados naquele cartão naquele mês

A fatura exibe:
- Subtotal de cada categoria acima
- Total geral da fatura
- Limite utilizado × disponível
- Status: **Aberta / Fechada / Paga**

### RF-042 · Check dos fixos na fatura
Cada gasto fixo recorrente cobrado no cartão deve aparecer na fatura com um **indicador de check** — confirmando se a cobrança apareceu na fatura do mês ou não.

### RF-043 · Parcelamentos na fatura
Cada parcela exibe:
- Descrição da compra
- Valor da parcela
- Número da parcela atual e total (ex: 3/12)
- Valor total da compra original

### RN-040 · Ciclo da fatura
O sistema calcula automaticamente a **competência de cada gasto** com base no dia de fechamento do cartão:
- Gasto realizado **antes ou no dia de fechamento** → entra na fatura do mês atual
- Gasto realizado **após o dia de fechamento** → entra na fatura do mês seguinte

### RN-041 · Cobertura da fatura
Quando o usuário marca um salário como recebido, o sistema compara o **saldo disponível** com o **total das faturas abertas** de todos os cartões e exibe o resultado (ver RN-010).

### RN-042 · Pagamento da fatura
O usuário pode marcar a fatura como **Paga**, informando:
- Data do pagamento
- Valor pago (padrão: valor total da fatura)

---

## 6. Módulo: Dívidas e Parcelamentos

### RF-050 · Cadastro de parcelamento / dívida

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| Descrição | Texto | Sim | Ex: "iPhone 15 - 12x" |
| Tipo | Enum | Sim | Parcelamento cartão / Financiamento / Empréstimo / Dívida informal |
| Valor total | Decimal | Sim | — |
| Número de parcelas | Inteiro | Sim | — |
| Valor da parcela | Decimal | Sim | Calculado ou manual |
| Parcela inicial | Inteiro | Sim | A partir de qual número começa (padrão: 1) |
| Data da primeira parcela | Data | Sim | — |
| Juros (%) | Decimal | Não | Se houver |
| Cartão | Referência | Só se parcelamento cartão | — |
| Compartilhado | Boolean | Sim | Padrão: false |
| Vínculo | Referência | Só se compartilhado | Com quem se compartilha |
| Valor do dono / do vinculado | Decimal (×2) | Só se compartilhado | Soma = valor total |

### RN-050 · Geração das parcelas
Ao cadastrar um parcelamento, o sistema **gera automaticamente** todas as parcelas com suas respectivas datas e valores, vinculando cada uma ao mês correspondente.

### RN-051 · Projeção de quitação
O sistema calcula e exibe:
- Mês e ano estimados de quitação
- Valor total já pago
- Valor total restante

---

## 7. Módulo: Metas de Economia

### RF-060 · Cadastro de meta

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| Nome | Texto | Sim | Ex: "Reserva de emergência" |
| Valor alvo | Decimal | Sim | — |
| Valor atual | Decimal | Sim | Padrão: 0 |
| Data alvo | Data | Não | — |
| Contribuição mensal planejada | Decimal | Não | — |

### RF-061 · Registro de contribuição
O usuário pode registrar aportes à meta informando:
- Valor
- Data
- Observação opcional

O sistema atualiza o **valor atual** da meta a cada aporte.

### RN-060 · Progresso da meta
O sistema calcula e exibe:
- Percentual concluído (valor atual / valor alvo × 100)
- Quanto falta
- Se há data alvo: se o ritmo atual de aportes é suficiente para atingi-la no prazo

---

## 8. Módulo: Investimentos

### RF-070 · Cadastro de aporte (Fase 1)

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| Tipo | Enum | Sim | Renda fixa / Ações / FIIs / Cripto / Tesouro / Poupança / Outro |
| Instituição | Texto | Não | Ex: "XP", "Nubank" |
| Descrição | Texto | Não | Ex: "CDB 110% CDI" |
| Valor aportado | Decimal | Sim | — |
| Data do aporte | Data | Sim | — |

### RF-071 · Visão consolidada de investimentos
O sistema exibe:
- Total aportado por tipo
- Total geral aportado
- Histórico de aportes por mês

> Cálculo de rendimento automático reservado para Fase 2.

---

## 9. Módulo: Patrimônio Líquido

### RF-080 · Cálculo automático
O patrimônio líquido é calculado automaticamente pela fórmula:

```
Patrimônio Líquido = Ativos − Passivos
```

**Ativos (alimentados automaticamente):**
- Saldo disponível em conta (receitas recebidas − gastos pagos no mês)
- Total de aportes em investimentos

**Ativos (alimentados manualmente):**
- Bens: imóveis, veículos, outros (usuário informa valor estimado)

**Passivos (alimentados automaticamente):**
- Total de dívidas e parcelamentos em aberto
- Faturas de cartão não pagas

### RF-081 · Histórico do patrimônio
O sistema registra o patrimônio líquido ao final de cada mês, permitindo visualizar a evolução ao longo do tempo em gráfico de linha.

---

## 10. Módulo: Notificações e Alertas

### RF-090 · Canais de notificação
- **E-mail:** obrigatório na Fase 1
- **Google Calendar:** integração tentada na Fase 1 (criação de eventos para vencimentos)
- **WhatsApp:** Fase 2

### RF-091 · Tipos de alerta e configuração
Cada tipo de alerta pode ser **ativado ou desativado** pelo usuário. A antecedência é configurável individualmente.

| Tipo | Gatilho | Antecedência configurável |
|---|---|---|
| Fatura do cartão vencendo | Dia de vencimento da fatura | Sim |
| Conta fixa a pagar | Dia de vencimento do compromisso | Sim |
| Conta fixa em atraso | Vencimento sem check | Não (imediato) |
| Salário recebido | Check de receita recebida | Não (imediato) |
| Meta atingida | Valor atual ≥ valor alvo | Não (imediato) |
| Gasto fora do padrão | Gasto acima da média histórica da categoria | Sim (% de variação) |

---

## 11. Módulo: Relatórios

### RF-100 · Relatórios disponíveis

| Relatório | Descrição |
|---|---|
| Resumo mensal | Receitas, gastos fixos, gastos variáveis, saldo e economia do mês |
| Comparativo mês a mês | Evolução de cada categoria ao longo do tempo |
| Gastos por categoria | Distribuição percentual dos gastos (gráfico) |
| Evolução do patrimônio | Linha do tempo do patrimônio líquido mês a mês |
| Relatório de poupança | Aportes em metas e investimentos: planejado × realizado |
| Projeção de parcelamentos | Comprometimento futuro por mês com parcelamentos e dívidas |

### RF-101 · Exportação em PDF
Todos os relatórios **[DEVEM]** poder ser exportados em PDF com:
- Cabeçalho com nome do relatório e período
- Dados tabulares e gráficos quando aplicável
- Data de geração

### RF-102 · Filtros dos relatórios
Todos os relatórios devem suportar filtro por:
- Período (mês/ano ou intervalo)
- Origem (meus lançamentos / compartilhados comigo / ambos)
- Categoria

---

## 12. Entrada de Dados

### RF-110 · Lançamento manual
Descrito em cada módulo acima.

### RF-111 · Importação de arquivo
O sistema **[DEVE]** aceitar upload de arquivos **OFX** e **CSV**.

Fluxo de importação:
1. Usuário faz upload do arquivo
2. Sistema lê e lista as transações encontradas
3. Para cada transação, o sistema **sugere** categoria com base no histórico de descrições similares
4. Usuário revisa, ajusta categorias e forma de pagamento
5. Sistema verifica duplicatas (mesma data + valor + descrição já existente) e sinaliza
6. Usuário confirma — transações são importadas

### RN-110 · Detecção de duplicata
Uma transação é considerada duplicata se já existe no sistema um lançamento com:
- Mesma data
- Mesmo valor
- Descrição com similaridade ≥ 90%

O sistema sinaliza mas **não bloqueia** — o usuário decide se importa ou descarta.

---

## 13. Stack Tecnológica

### Backend
| Tecnologia | Função |
|---|---|
| **Django REST Framework** | API REST — regras de negócio, autenticação, endpoints |
| **PostgreSQL** | Banco de dados relacional principal |

### Frontend — Fase 1 (Mobile)
| Tecnologia | Função |
|---|---|
| **React Native + Expo** | Framework do app mobile (iOS e Android) |
| **TypeScript** | Linguagem base |
| **expo-camera / expo-barcode-scanner** | Leitura do QR Code da NFC-e e captura de foto do cupom |
| **React Navigation** | Navegação entre telas |

### Frontend — Fase 2 (Web)
| Tecnologia | Função |
|---|---|
| **React + TypeScript** | Framework e linguagem base do frontend web |
| **Tailwind CSS** | Estilização utilitária — layout, espaçamento, cores |
| **shadcn/ui** | Componentes funcionais prontos (tabelas, modais, formulários) |
| **React Bits** | Componentes animados e efeitos visuais — usado com moderação |

### Serviços externos
| Serviço | Função |
|---|---|
| **Consulta SEFAZ / Nuvem Fiscal** | Leitura dos dados da NFC-e a partir do QR Code |
| **Google Cloud Vision** | OCR de cupom físico (fallback) |

### Infraestrutura
| Tecnologia | Função |
|---|---|
| **Oracle Cloud** | Hospedagem do backend (Always Free tier) — uso pessoal |

### Critério de uso do React Bits (Fase 2)
O React Bits deve ser utilizado em elementos de **destaque visual e navegação** (ex: transições de página, indicadores de progresso, cards de resumo), e **evitado** em componentes de entrada de dados, tabelas e formulários, onde clareza e precisão têm prioridade sobre estética.

---

## 14. Requisitos Não Funcionais

### RNF-001 · Plataforma e infraestrutura
- **Fase 1: aplicação mobile (iOS e Android)** via React Native + Expo
- **Fase 2: aplicação web responsiva**, funcional em Chrome, Firefox e Safari
- O backend (Django REST + PostgreSQL) é compartilhado entre as duas fases desde o início
- **Hospedagem:** Oracle Cloud (Always Free tier) — uso pessoal
- Por ser uso pessoal, não há planos gratuito/pago
- Cada usuário é dono dos seus próprios dados; o compartilhamento entre usuários é item a item, via vínculo (ver RF-002)

### RNF-002 · Segurança
- Todas as comunicações via HTTPS
- Senhas armazenadas com hash (bcrypt ou argon2)
- Tokens de autenticação com expiração

### RNF-003 · Performance
- Telas principais devem carregar em menos de 2 segundos em conexão normal
- Importação de arquivos de até 1.000 transações deve ser processada em menos de 10 segundos

### RNF-004 · Dados
- Backup automático diário dos dados
- O usuário pode exportar todos os seus dados a qualquer momento (formato JSON ou CSV)

### RNF-005 · Idioma
- Interface em português brasileiro
- Formato de datas: DD/MM/AAAA
- Formato de valores: R$ 1.234,56

---

## Registro de Decisões

| Data | Decisão |
|---|---|
| Mai 2026 | **Modelo de usuários revisto: cada pessoa é um usuário independente (dados privados); compartilhamento item a item via vínculo (convite/aceite). Removido o conceito de conta familiar e de parceiro/papel.** |
| Mai 2026 | Categorias passam a ser **pessoais** (por usuário), não mais compartilhadas |
| Mai 2026 | Categorias: 12 pré-definidas (não excluíveis) + até 10 customizadas pelo usuário |
| Mai 2026 | Cancelamento de conta não previsto — sistema de uso pessoal |
| Mai 2026 | Hospedagem: Oracle Cloud (uso pessoal, sem planos ou multi-tenant) |
| Mai 2026 | Backend: Django REST Framework + PostgreSQL |
| Mai 2026 | **Ordem invertida: mobile primeiro (React Native + Expo), web na Fase 2** |
| Mai 2026 | Frontend web (Fase 2): React + TypeScript + Tailwind CSS + shadcn/ui + React Bits |
| Mai 2026 | **Nova feature: scanner de cupom fiscal (QR Code NFC-e + OCR fallback)** |
| Mai 2026 | Feature de scanner validada via MVP mobile separado antes da integração |

---

*Este documento é vivo. Referência: Documento de Visão v0.5*