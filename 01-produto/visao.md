# Documento de Visão — Sistema de Finanças Pessoais

> **Status:** Em construção · Versão 0.6  
> **Última atualização:** Maio de 2026  
> **Autor:** A definir

---

## 1. Visão Geral

Este sistema nasce da insatisfação com apps de finanças existentes que não entregam o nível de detalhamento e customização necessários para uma gestão financeira real. O objetivo é construir uma plataforma **pessoal**, onde cada centavo tem contexto, categoria e significado — e onde o usuário enxerga sua vida financeira de ponta a ponta, sem depender de planilhas. Cada pessoa é dona dos seus dados e pode **compartilhar lançamentos específicos** com quem quiser, item a item (ver seção 3).

### Problema Central

Os apps de mercado oferecem uma visão genérica das finanças. Eles não permitem:
- Categorizar com a granularidade que o usuário precisa
- Customizar a forma como os dados são organizados e visualizados
- Tratar o cartão de crédito com a complexidade que ele tem na vida real (fatura, fixos, variáveis, parcelamentos)
- Distinguir tipos de gastos fixos com categorias diferentes

O resultado: o usuário volta para a planilha. Este sistema é a planilha evoluída.

---

## 2. Plataformas

| Fase | Plataforma | Status |
|---|---|---|
| Fase 1 | Mobile (iOS / Android) | Prioridade inicial |
| Fase 2 | Web (browser) | Após consolidação do mobile |

A decisão por mobile primeiro é reforçada pela feature de **scanner de cupom fiscal**, que depende da câmera do celular — uma experiência naturalmente mobile. O backend é compartilhado entre as duas fases desde o início, então a versão web reaproveita toda a lógica já construída.

---

## 3. Usuários e Modelo de Compartilhamento

### Usuários independentes

Cada pessoa é um **usuário independente**, dono dos seus próprios dados. **Tudo é privado por padrão** — não existe uma conta familiar com acesso compartilhado, e não há papéis (titular/parceiro) nem hierarquia.

### Vínculo entre usuários

Duas pessoas que querem compartilhar gastos criam um **vínculo** entre si: um envia o convite por e-mail, o outro aceita. O vínculo é o que habilita o compartilhamento — sem ele, nada de um usuário é visível ao outro.

### Modelo de compartilhamento (item a item)

Mesmo com vínculo, **todos os lançamentos continuam pessoais por padrão**. Qualquer transação — gasto, receita, conta ou parcelamento — pode ser marcada como **compartilhada** com um vínculo, no momento do cadastro ou depois.

Ao compartilhar um lançamento, cada pessoa define **quanto vai pagar** daquele item — sem rateio fixo. Exemplos: 60/40, 100/0, ou qualquer divisão que faça sentido naquele momento. O item permanece pertencendo a quem o lançou; a outra pessoa o enxerga como "compartilhado comigo", com a sua porção.

Isso permite:
- Manter cada um dono e responsável pelos seus próprios dados
- Compartilhar apenas o que faz sentido, item a item
- Identificar claramente quanto cada um deve em cada compromisso compartilhado

---

## 4. Módulos do Sistema

### 4.1 · Renda / Receitas

Registro de toda entrada de dinheiro, incluindo:

- Salário (com data de recebimento esperada e real)
- Outras fontes de renda (freelance, aluguel, etc.)
- Renda variável (comissões, bônus)
- Flag de compartilhado (se a receita for conjunta)

**Comportamento esperado:**  
Ao marcar um salário como recebido, o sistema deve **atualizar automaticamente o status das contas vinculadas** — especialmente a fatura do cartão de crédito e os gastos fixos do mês (ver 4.3 e 4.4).

---

### 4.2 · Gastos do Dia a Dia

Lançamentos de gastos variáveis e não recorrentes:

- Data
- Valor
- Categoria e subcategoria customizáveis
- Forma de pagamento (dinheiro, pix, débito, crédito)
- Cartão utilizado (se for crédito)
- Pessoa (quem gastou)
- Flag de compartilhado + vínculo e rateio entre os dois (se aplicável)
- Descrição livre
- Tag / marcador opcional

**Compra detalhada (scanner de cupom):** para compras de supermercado, o usuário pode registrar a compra **item a item** sem digitar tudo. Escaneando o QR Code da nota fiscal (NFC-e), o sistema importa os itens automaticamente; quando não houver QR Code, uma foto do cupom é lida via OCR, e o que não for identificado fica para preenchimento manual. Essa feature é primariamente mobile.

---

### 4.3 · Gastos Fixos

Gastos que se repetem todo mês, divididos em **dois tipos com categorização diferente**:

#### Tipo A — Compromissos Fixos (valor fixo mensal)
Exemplos: aluguel, financiamento, plano de saúde, escola.
- Valor não varia
- Têm data de vencimento
- Podem ser pagos via cartão ou débito/pix
- Status: **pendente → pago / atrasado**
- Flag de compartilhado + vínculo e rateio entre os dois (se aplicável)

#### Tipo B — Recorrentes Estimados (valor pode variar)
Exemplos: conta de luz, água, internet, mercado estimado.
- Valor pode oscilar mês a mês
- Lançamento mensal com valor real
- Comparação com média histórica dos últimos meses

**Comportamento esperado:**  
O usuário deve conseguir dar **check individual** em cada gasto fixo conforme for pagando. A tela do mês deve mostrar claramente "o que já saiu" e "o que ainda vai sair".

---

### 4.4 · Cartão de Crédito

Este é um dos módulos mais críticos do sistema. O cartão não é apenas uma forma de pagamento — ele tem uma lógica própria de fatura, ciclo de vencimento e composição de gastos.

#### Múltiplos cartões

O sistema suporta **múltiplos cartões** cadastrados simultaneamente. Para cada cartão:

- Nome / apelido (ex: "Nubank", "Inter")
- Limite total
- Dia de fechamento da fatura
- Dia de vencimento
- Status: ativo / inativo (cartões reserva de emergência podem ser cadastrados como inativos mas monitorados)

#### Estrutura da Fatura

Cada fatura mensal deve detalhar os gastos em três categorias:

| Categoria | Descrição |
|---|---|
| **Gastos fixos no cartão** | Assinaturas, planos, cobranças recorrentes cobradas no cartão |
| **Parcelamentos** | Compras parceladas com número de parcela atual / total |
| **Gastos variáveis** | Compras avulsas do mês |

#### Funcionalidades esperadas

- Visualização da fatura atual com **subtotal por categoria**
- Controle de **limite disponível × utilizado**
- Cada gasto fixo recorrente deve ter status de **check** (foi cobrado este mês?)
- Parcelamentos devem mostrar: valor da parcela · parcela atual / total · valor total da compra
- Ao registrar o recebimento do salário, o sistema indica se a **fatura está coberta** pelo saldo disponível
- Visão consolidada de todos os cartões em um painel único

---

### 4.5 · Dívidas e Parcelamentos

Visão consolidada de tudo que está sendo pago ao longo do tempo:

- Compras parceladas no cartão
- Financiamentos (carro, imóvel, etc.)
- Empréstimos
- Dívidas informais (se necessário)

Para cada dívida / parcelamento:

- Valor total
- Valor já pago
- Valor restante
- Número de parcelas restantes
- Projeção de quitação (mês/ano)
- Juros (se aplicável)
- Flag de compartilhado + vínculo e rateio entre os dois (se aplicável)

---

### 4.6 · Metas de Economia

Objetivos financeiros com acompanhamento de progresso:

- Nome da meta (ex: "Viagem para Europa", "Reserva de emergência")
- Valor alvo
- Valor já guardado
- Data alvo
- Contribuições mensais planejadas × realizadas
- Indicador visual de progresso (%)

---

### 4.7 · Investimentos

O módulo de investimentos terá **duas fases**:

#### Fase 1 — Controle de Aportes (escopo atual)
Foco em registrar quanto está sendo guardado/investido ao longo do tempo:

- Tipo de investimento (renda fixa, ações, FIIs, cripto, tesouro, poupança, etc.)
- Valor aportado e data
- Histórico de aportes mensais
- Total acumulado por tipo e geral

#### Fase 2 — Rendimento Automático (futuro)
Quando for o momento, o sistema calculará automaticamente o rendimento com base nas taxas de cada tipo de investimento. Esta fase fica em standby até que o controle de aportes esteja maduro.

---

### 4.8 · Patrimônio Líquido

Visão consolidada de tudo que o usuário possui e deve:

```
Patrimônio Líquido = Ativos − Passivos
```

**Ativos:**
- Saldo em conta corrente / poupança
- Total investido (valor dos aportes — Fase 1)
- Bens (imóveis, veículos — valor estimado, inserido manualmente)
- Outros

**Passivos:**
- Dívidas e parcelamentos em aberto
- Fatura(s) do cartão não pagas

A atualização é automática conforme os outros módulos são alimentados. Bens de valor estimado são atualizados manualmente pelo usuário.

---

## 5. Entrada de Dados

O sistema suporta **dois modos de lançamento**:

| Modo | Descrição |
|---|---|
| **Manual** | Usuário digita cada lançamento diretamente no sistema |
| **Importação** | Upload de arquivo OFX / CSV gerado pelo banco ou cartão |

Na importação, o sistema deve:
- Sugerir categorias com base no histórico de lançamentos anteriores
- Permitir que o usuário revise e ajuste antes de confirmar
- Detectar e sinalizar possíveis duplicatas

---

## 6. Notificações e Alertas

### Canais suportados

| Canal | Status |
|---|---|
| E-mail | ✅ Implementado na Fase 1 |
| Google Calendar | 🔄 Tentativa de integração na Fase 1 |
| WhatsApp | ⏳ Futuro |

### Tipos de alertas previstos

- Fatura do cartão vencendo (antecedência configurável pelo usuário)
- Conta fixa a pagar (antecedência configurável pelo usuário)
- Salário recebido — resumo do mês aberto
- Meta de economia atingida
- Gasto fora do padrão histórico (alerta de anomalia)

A antecedência de cada tipo de alerta é **configurada pelo próprio usuário** nas preferências do sistema.

---

## 7. Relatórios e Histórico

O sistema terá uma camada de relatórios para análise do comportamento financeiro ao longo do tempo. Todos os relatórios poderão ser **exportados em PDF**.

### Relatórios previstos

- **Resumo mensal:** receitas, gastos fixos, gastos variáveis, economia do mês
- **Comparativo mês a mês:** evolução de cada categoria ao longo do tempo
- **Gastos por categoria:** onde o dinheiro foi (gráfico de pizza ou barras)
- **Evolução do patrimônio:** linha do tempo do patrimônio líquido
- **Relatório de poupança:** quanto foi guardado × quanto foi planejado guardar
- **Relatório de parcelamentos:** projeção de compromissos futuros mês a mês

---

## 7. Stack Tecnológica

| Camada | Tecnologia |
|---|---|
| **Backend** | Django REST Framework |
| **Banco de dados** | PostgreSQL |
| **Mobile (Fase 1)** | React Native + Expo + TypeScript |
| **Web (Fase 2)** | React + TypeScript + Tailwind CSS + shadcn/ui + React Bits |
| **Serviços externos** | Consulta SEFAZ / Nuvem Fiscal (NFC-e) · Google Vision (OCR) |
| **Infraestrutura** | Oracle Cloud (Always Free) |

---

## 8. Experiência Esperada

- **Visão mensal como centro:** o mês é a unidade principal. O usuário entra e vê o status do mês atual.
- **Check e progresso:** poder marcar contas como pagas e acompanhar o mês em tempo real.
- **Sem surpresas:** o sistema deve deixar claro o que ainda vai sair antes do fim do mês.
- **Customização total:** categorias, subcategorias e tags são criadas pelo usuário, sem limitações.
- **Compartilhamento sob controle:** cada um é dono dos seus dados; o que é dividido aparece via vínculo, com a flag de compartilhado e a divisão livre de valores como centro da lógica.

---

## 9. O que este sistema NÃO é (por enquanto)

- Não faz integração automática com bancos (Open Finance / Pluggy)
- Não é uma ferramenta de contabilidade empresarial
- Não automatiza pagamentos
- Não calcula rendimento de investimentos automaticamente (Fase 2)

---

## Registro de Decisões

| Data | Decisão |
|---|---|
| Mai 2026 | **Usuários independentes: cada pessoa tem seus próprios dados (privados). Compartilhamento item a item via vínculo (convite/aceite). Removido o conceito de conta familiar e de parceiro/papel.** |
| Mai 2026 | Suporte a múltiplos cartões (mín. 2); cartões inativos podem ser cadastrados para emergência |
| Mai 2026 | Modelo de compartilhamento via flag por lançamento — padrão é pessoal |
| Mai 2026 | Rateio de gastos compartilhados é livre — cada pessoa define quanto vai pagar por item |
| Mai 2026 | Plataformas: mobile primeiro (React Native + Expo), web na Fase 2 |
| Mai 2026 | Investimentos: Fase 1 foca em controle de aportes; rendimento automático fica para Fase 2 |
| Mai 2026 | Notificações: e-mail (Fase 1) + tentativa de Google Calendar; WhatsApp para o futuro |
| Mai 2026 | Antecedência dos alertas é configurável pelo próprio usuário |
| Mai 2026 | Relatórios exportáveis em PDF |
| Mai 2026 | Haverá histórico comparativo e relatórios de gastos e poupança |
| Mai 2026 | Backend: Django REST Framework + PostgreSQL |
| Mai 2026 | Mobile (Fase 1): React Native + Expo; Web (Fase 2): React + TypeScript + Tailwind + shadcn/ui + React Bits |
| Mai 2026 | Nova feature: scanner de cupom fiscal (QR Code NFC-e + OCR fallback), validada via MVP mobile |
| Mai 2026 | Infraestrutura: Oracle Cloud (Always Free) |

---

*Este documento é vivo. Será atualizado conforme as decisões forem tomadas.*