# Base de Conhecimento

## Dados Utilizados

Descreva se usou os arquivos da pasta `data`, por exemplo:

| Arquivo | Formato | Utilização no Agente |
|---------|---------|---------------------|
| `historico_de_atendimento/` | JSON / TXT | Contextualizar contatos anteriores e propostas já recusadas ou em andamento |
| `perfil_devedor.json` | JSON | Identificar renda líquida, despesas essenciais e capacidade real de pagamento |
| `linhas_de_credito.json` | JSON | Consultar critérios |
| `dividas_ativas.csv` | CSV | Listar credores, valores devidos, juros mensais, prazos e dias de atraso|



## Adaptações nos Dados

> Você modificou ou expandiu os dados mockados? Descreva aqui.
>
> eu tentei fazer algo diferente então mudei todas os dados 

[Os dados de referência de investimentos foram reformulados para cobrir o ciclo de endividamento:

 O antigo produtos_financeiros.json foi convertido em linhas_de_credito.json, reunindo regras dos métodos Avalanche e Bola de Neve, prazos de tolerância (corte de serviços, apreensão de garantia) e diretrizes de negociação.

Foco em Capacidade de Pagamento: O arquivo perfil_investidor.json tornou-se perfil_devedor.json, trocando tolerância a risco por margem disponível para amortização.

Extrato de Dívidas com Prazos: O transacoes.csv deu lugar a dividas_ativas.csv, incorporando taxas de juros mensais, existência de garantia alienada, data de vencimento e contagem de dias em atraso.]

---

## Estratégia de Integração

### Como os dados são carregados?
> Descreva como seu agente acessa a base de conhecimento.

inserindo os dados diretamente no prompt, ou carregar os arquivos via codigo

'''pytohn
import json
import pandas as pd

# 1. Carregar CSV (Dívidas Ativas)
dividas_df = pd.read_csv("data/dividas_ativas.csv")

# 2. Carregar JSONs de configuração e perfil
with open("data/perfil_devedor.json", "r", encoding="utf-8") as f:
    perfil_cliente = json.load(f)

with open("data/linhas_de_credito.json", "r", encoding="utf-8") as f:
    regras_credito = json.load(f)

with open("data/historico_de_atendimento/cli_1029.json", "r", encoding="utf-8") as f:
    historico_atendimento = json.load(f)]
'''

### Como os dados são usados no prompt?
> Os dados vão no system prompt? São consultados dinamicamente?

1. (Regras de negocio fixas)
na linhas_De_credito temos (metodos avalanche e bola de neve,limites e restricoes de comportamento) são inseridos no system no prompt e atuam como guardrail do agente.

2. Mensagem inicial(Dados Dinâmicos do cliente):
em perfil_devedor.json,dividas_ativas.csv e historico_De_atendimento o script python lê os arquivos e monta um bloco de texto com a situação atual do cliente esse bloco  e injetado dinamicamente junto com a mensagem do usuário no inicio da sessão


Dessa maneira a LLM consulta todo o historico e a lista de dividas 

---

## Exemplo de Contexto Montado

> Mostre um exemplo de como os dados são formatados para o agente.

```
[DADOS DO CLIENTE]
- Nome: Carlos Silva (ID: cli_1029)
- Renda Líquida Mensal: R$ 3.500,00
- Despesas Essenciais: R$ 2.600,00
- Capacidade Máxima para Pagamento de Dívidas: R$ 900,00/mês
- Objetivo Principal: Quitar dívidas sem perder patrimônio

[DÍVIDAS ATIVAS MAPEADAS (dividas_ativas.csv)]
1. Luz & Força Distribuidora
   - Saldo Devedor: R$ 350,00 | Taxa: 2.0% a.m. | Vencimento: 2026-03-03 | Atraso: 10 dias | Garantia: Não (Serviço Essencial)
2. AutoPrime Financiamentos
   - Saldo Devedor: R$ 12.000,00 | Taxa: 1.8% a.m. | Vencimento: 2026-01-15 | Atraso: 60 dias | Garantia: Sim (Veículo Alienado)
3. Banco Fortaleza
   - Saldo Devedor: R$ 3.200,00 | Taxa: 14.5% a.m. | Vencimento: 2026-01-30 | Atraso: 45 dias | Garantia: Não (Cartão Rotativo)
4. Crédito Fácil Financeira
   - Saldo Devedor: R$ 5.500,00 | Taxa: 4.2% a.m. | Vencimento: 2026-02-28 | Atraso: 15 dias | Garantia: Não (Empréstimo Pessoal)

[HISTÓRICO DE ATENDIMENTO RECENTE]
- 2026-02-10: Cliente relatou ligações de cobrança do Banco Fortaleza sobre o rotativo. Foi orientado a não aceitar proposta de parcelamento com juros altos antes de mapear os demais débitos.

[REGRAS E DIRETRIZES DE NEGOCIAÇÃO]
- Prioridade Legal Máxima: Contas de consumo essenciais e dívidas com garantia de bens.
- Regra de Risco: Dívida veicular com 60+ dias de atraso possui risco iminente de busca e apreensão.
- Método Sugerido para Redução de Custo: Avalanche (atacar primeiro a taxa de 14.5% a.m. do Banco Fortaleza).
- Limite de Comprometimento: As parcelas somadas nunca podem ultrapassar R$ 900,00/mês.
```
