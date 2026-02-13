# 📊 Análise Executiva de Performance Comercial – Growth Wave
### Dashboard Executivo em Power BI

---

## 📌 Visão Geral do Projeto

Este repositório contém o desenvolvimento completo do dashboard **Análise Executiva de Performance Comercial**, criado em **Power BI** com o objetivo de transformar dados operacionais de vendas em **insights estratégicos** para tomada de decisão.

O projeto simula um ambiente real de gestão comercial, com foco em:

- Identificação de oportunidades de receita  
- Análise de lucratividade por categoria e produto  
- Avaliação de performance de vendedores e lojas  
- Entendimento dos fatores que influenciam o valor de venda  
- Acompanhamento da evolução temporal das vendas  

---

## 🎯 Objetivo de Negócio

O dashboard foi projetado para apoiar **diretores, gerentes comerciais e analistas de BI**, permitindo:

- Identificar **principais geradores de receita**  
- Avaliar **produtos e categorias mais lucrativas**  
- Entender **segmentos com maior impacto no valor de venda**  
- Monitorar **performance individual de vendedores**  
- Acompanhar **tendências temporais** de vendas, margem e lucro  

---

## 🗂 Estrutura do Dashboard

### 1. Índice (Resumo Executivo)
- Apresentação do projeto  
- Objetivo estratégico  
- Público-alvo  
- Nota do autor  
- Navegação para todas as páginas  

📸 ![alt text](Images/Indice.png)

---

### 2. Storytelling
Visão geral executiva com narrativa orientada a decisão.

**KPIs principais:**
- Total de Vendas  
- Lucro Total  
- Margem Total  
- Custo Total  
- Ticket Médio  
- Comissões Pagas  

**Visuais:**
- Linha temporal por Segmento  
- Rosca de Vendas por Categoria  
- Barras de Vendas por Fabricante  
- Ticket Médio por Fabricante  

📸 ![alt text](Images/Storytelling.png)

---

### 3. Influenciadores de Vendas
Página dedicada ao visual de IA do Power BI (Key Influencers).

**Principais fatores identificados:**
- Segmento **Corporativo**  
- Categoria **Celulares**  
- Fabricante **Dell**  
- Fabricante **Motorola**  

📸 ![alt text](Images/Influenciadores.png)

---

### 4. Vendas por Categoria
Análise de mix, lucratividade e evolução.

**Visuais:**
- Treemap de mix de vendas  
- Evolução temporal por categoria  
- Tabela com:
  - Total de Vendas  
  - Lucro Total  
  - Margem Média  
  - Ticket Médio  
- Lucro por Categoria (barras ou waterfall)

📸 ![alt text](Images/Vendas%20por%20Categoria.png)

---

### 5. Performance de Vendedores
Análise individual e por loja.

**Visuais:**
- Ranking de vendedores  
- Ticket médio por vendedor  
- Tabela detalhada com:
  - Total de vendas  
  - Lucro  
  - Margem  
  - Comissão  
- Indicadores agregados:
  - Total de vendas  
  - Ticket médio  
  - Total de lojas  
  - Total de vendedores  
  - Lucro total  
  - Custo total  
  - Comissões pagas  

📸 ![alt text](Images/Performance.png)

---

## 🧱 Modelagem de Dados

### Modelo Estrela

O modelo foi estruturado seguindo boas práticas de BI:

**Fato:**  
- `F_Vendas`  
  - ValorVenda  
  - Custo  
  - Comissão  
  - Data Venda  
  - IDs de Produto, Loja e Vendedor  

**Dimensões:**  
- `D_Produto`  
- `D_Tempo`  
- `D_Loja`  
- `D_Vendedor`  


---

### Relacionamentos

Todos os relacionamentos seguem cardinalidade **1:N**, com filtro indo da dimensão para a fato.

| Tabela Origem | Tabela Destino | Tipo | Direção |
|---------------|----------------|------|---------|
| D_Produto     | F_Vendas       | 1:N  | Single  |
| D_Tempo       | F_Vendas       | 1:N  | Single  |
| D_Loja        | F_Vendas       | 1:N  | Single  |
| D_Vendedor    | F_Vendas       | 1:N  | Single  |

---

