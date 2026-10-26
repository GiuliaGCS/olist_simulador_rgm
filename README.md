# 📦 Simulador de Preços e Margem (RGM) para Vendedores Autônomos

> Projeto Hands-On — MBA Data Engineering · Universidade Presbiteriana Mackenzie
> Base de dados: [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle)

![status](https://img.shields.io/badge/status-em%20desenvolvimento-yellow)
![python](https://img.shields.io/badge/python-3.10%2B-blue)
![arquitetura](https://img.shields.io/badge/arquitetura-Bronze%20%7C%20Silver%20%7C%20Gold-orange)

---

## 📌 Sumário

1. [Contexto e problema](#-contexto-e-problema)
2. [Objetivo e proposta de valor](#-objetivo-e-proposta-de-valor)
3. [Módulos do produto](#-módulos-do-produto)
4. [Bases de dados](#-bases-de-dados)
5. [Arquitetura de dados](#-arquitetura-de-dados)
6. [Ferramentas e tecnologias](#-ferramentas-e-tecnologias)
7. [Estrutura do repositório](#-estrutura-do-repositório)
8. [Como executar](#-como-executar-instruções-preliminares)
9. [Resultados preliminares da EDA](#-resultados-preliminares-da-eda)
10. [Cronograma (Sprints)](#-cronograma-sprints)
11. [Equipe](#-equipe)
12. [Licença e créditos dos dados](#-licença-e-créditos-dos-dados)

---

## 🎯 Contexto e problema

Pequenos lojistas de marketplace frequentemente definem o preço de venda "no achômetro", sem considerar frete, parcelamento, distância logística e risco de atraso. O resultado é perda de margem, avaliações negativas e, em muitos casos, o encerramento do negócio por falta de planejamento.

Este projeto usa o ecossistema de dados da Olist (2016–2018) para mostrar, com dados reais, como uma **cultura data driven** ajuda na saúde financeira e operacional de um e-commerce.

## 🚀 Objetivo e proposta de valor

Transformar os dados da Olist em uma **plataforma de consultoria online e inteligência automatizada** para vendedores autônomos.

O vendedor informa **categoria, preço de venda e região** e recebe um **Laudo de Viabilidade Instantâneo**, baseado no comportamento real de milhares de lojistas, com alertas sobre:

- risco de margem;
- peso do frete no preço final;
- risco de atraso logístico.

**Oportunidade adicional:** incluir um *DE-PARA* para simulação de cenários da **Reforma Tributária**.

## 🧩 Módulos do produto

| # | Módulo | O que entrega | Tabelas principais |
|---|--------|---------------|--------------------|
| 1 | **Simulador de Precificação e Margem (RGM)** | Calculadora de margem líquida por categoria e alerta de sensibilidade ao frete | `order_items`, `order_payments`, `products` |
| 2 | **Radar de Oportunidades (Categoria × Geografia)** | Mapa de calor logístico, distância média (Haversine) e análise de saturação | `products`, `category_translation`, `customers`, `sellers`, `geolocation` |
| 3 | **Motor Preditivo de Riscos Logísticos (ML)** | Termômetro de envio (verde/amarelo/vermelho) com probabilidade de atraso por rota | `orders`, `order_items`, `sellers`, `customers` |
| 4 | **Sentimento e Retenção (NLP)** | Checklist de qualidade a partir de termos recorrentes em avaliações baixas | `order_reviews` |
| 5 | **Diagnóstico 360°** | Nota de saúde do e-commerce e 3 ações prioritárias, em PDF ou dashboard | Todas as tabelas |

## 🗂 Bases de dados

Os dados são os CSVs públicos do Kaggle (Olist). **Os arquivos não são versionados neste repositório** (veja `data/README.md`).

| Arquivo | Descrição |
|---------|-----------|
| `olist_orders_dataset.csv` | Pedidos e datas de cada etapa (compra, aprovação, postagem, entrega, estimativa) |
| `olist_order_items_dataset.csv` | Itens por pedido: produto, vendedor, preço e frete |
| `olist_order_payments_dataset.csv` | Formas de pagamento, parcelas e valores |
| `olist_order_reviews_dataset.csv` | Notas e comentários dos clientes |
| `olist_customers_dataset.csv` | Localização do cliente (cidade, estado, CEP) |
| `olist_sellers_dataset.csv` | Localização do vendedor |
| `olist_products_dataset.csv` | Categoria, dimensões e peso dos produtos |
| `olist_geolocation_dataset.csv` | Latitude e longitude por prefixo de CEP |
| `product_category_name_translation.csv` | Tradução das categorias (PT → EN) |

**DataFrame mestre (EDA):** a união das tabelas resultou em **119.143 linhas × 40 colunas**.

**Variáveis derivadas:**

- `delivery_time_days`: dias entre a compra e a entrega ao cliente;
- `delivery_delay_days`: diferença entre entrega real e estimada (negativo = entrega antecipada);
- `total_item_value`: preço do produto + frete.

## 🏗 Arquitetura de dados

Arquitetura em camadas (Medallion):

```
 CSVs (Kaggle)
      │
      ▼
┌───────────┐    ┌───────────┐    ┌───────────┐    ┌────────────────────────┐
│  BRONZE   │ -> │  SILVER   │ -> │   GOLD    │ -> │ Consumo                │
│ dado bruto│    │ limpo,    │    │ tabelas   │    │ Dashboard · Simulador  │
│ (raw)     │    │ tipado,   │    │ analíticas│    │ Modelo de ML · Laudo   │
│           │    │ joins     │    │ e features│    │                        │
└───────────┘    └───────────┘    └───────────┘    └────────────────────────┘
```

- **Bronze:** cópia fiel dos CSVs, sem alterações.
- **Silver:** conversão de datas, tratamento de nulos e duplicados, padronização, joins entre tabelas.
- **Gold:** tabelas finais por módulo (margem por categoria, rotas e distâncias, features de ML, indicadores do diagnóstico).

## 🛠 Ferramentas e tecnologias

- **Google Colab** — exploração e prototipagem
- **Databricks** — processamento e camadas Bronze/Silver/Gold
- **Git / GitHub Actions** — versionamento e CI/CD
- **Python** — pandas, scikit-learn, XGBoost, matplotlib/plotly, NLP
- **Trello** — gestão do Kanban e das sprints

## 📁 Estrutura do repositório

```
.
├── src/                # código-fonte (pipelines, features, modelos, app)
│   ├── ingestion/      # leitura dos CSVs → Bronze
│   ├── transformation/ # Bronze → Silver → Gold
│   ├── features/       # variáveis derivadas e engenharia de features
│   ├── models/         # modelos preditivos (atraso logístico, NLP)
│   └── app/            # simulador / dashboard
├── notebooks/          # EDA e experimentos (Colab / Databricks)
├── data/               # dados locais (não versionados)
│   ├── bronze/
│   ├── silver/
│   └── gold/
├── docs/               # documentação, arquitetura, apresentações, dicionário de dados
├── infra/              # infraestrutura como código e pipelines de CI/CD
├── .github/workflows/  # GitHub Actions
├── .gitignore
├── requirements.txt
└── README.md
```

## ▶️ Como executar (instruções preliminares)

> ⚠️ Em construção — será atualizado conforme o desenvolvimento avançar.

1. **Clonar o repositório**
   ```bash
   git clone https://github.com/<organizacao-ou-usuario>/<nome-do-repositorio>.git
   cd <nome-do-repositorio>
   ```
2. **Criar o ambiente virtual e instalar dependências**
   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```
3. **Baixar os dados** do [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) e colocar os 9 CSVs em `data/bronze/`.
4. **Executar os notebooks** em `notebooks/`, começando pela EDA.

## 📊 Resultados preliminares da EDA

- **Preço vs. frete:** preço médio de R$ 120,64 (mediana R$ 74,90) e frete médio de cerca de R$ 20,03. Em produtos de ticket baixo, o frete chega perto de 30% do valor pago.
- **Pagamentos:** cartão de crédito domina (quase 80 mil transações), seguido pelo boleto (~20 mil), o que mostra a dependência do parcelamento.
- **Logística:** tempo médio de entrega de 12,48 dias; atraso médio de -11,35 dias, ou seja, as estimativas têm uma "folga" de segurança.
- **Satisfação:** correlação de -0,30 entre tempo de entrega e nota; acima de 15 dias de espera, as avaliações de 1 estrela aumentam bastante.
- **Geografia e demanda:** SP concentra os compradores, seguido por RJ e MG. As categorias líderes são Cama, Mesa e Banho e Saúde e Beleza.

**Conclusão:** o sucesso do vendedor depende de três pilares: **precificação correta**, **controle geográfico** e **cumprimento do SLA de entrega**.

## 📅 Cronograma (Sprints)

| Sprint | Período | Foco |
|--------|---------|------|
| 1 | 06/10 – 12/10 | Definição do problema, seleção das bases, arquitetura inicial, ferramentas, Kanban e repositório |
| 2 | 13/10 – 19/10 | Modelagem das camadas, CI/CD e IaC, EDA inicial, protótipo de dashboard |
| 3 | 20/10 – 26/10 | MVP e requisitos, ambiente técnico, camada de dados, modelo preditivo (versão inicial) |
| 4 | 27/10 – 09/11 | Finalização do desenvolvimento, análise de resultados, testes, dashboard final |
| 5 | 10/11 – 16/11 | Revisão do escopo, documentação final e apresentação |

Acompanhe o Kanban no [Trello do projeto](https://trello.com/invite/b/6ad7e8d5b2fbe988950e3811/ATTI46fc8d1c2565216a02b99b5c5f70bb0d53AE229C/projeto-rgm-simulador-de-precos-mba-data-engineering).

## 👥 Equipe

- Maria Laura
- Giulia Santiago
- Luana Rodrigues
- Vinicius

## 📄 Licença e créditos dos dados

- Dados: *Brazilian E-Commerce Public Dataset by Olist*, disponibilizado no Kaggle sob a licença **CC BY-NC-SA 4.0**. Uso exclusivamente acadêmico e não comercial.
- Código: projeto acadêmico do MBA em Data Engineering — Mackenzie.
