# 📐 Measures.md  
## Medidas DAX + Script C# (Tabular Editor)  
### Projeto: Análise Executiva de Performance Comercial – Growth Wave

Este documento consolida **todas as medidas DAX utilizadas no projeto** e o **script em C#** aplicado no Tabular Editor para automatizar a criação das medidas.  
Ele faz parte da documentação técnica do repositório e demonstra domínio de boas práticas de modelagem, governança e produtividade em Power BI.

---

# 🧮 1. MEDIDAS DAX (DOCUMENTADAS)

---

## **Medidas DAX**
```DAX
TotalVendas =
SUM ( Vendas[ValorVenda] )

TotalCusto =
SUM ( Vendas[Custo] )

LucroTotal =
[TotalVendas] - [TotalCusto]

MargemMedia =
DIVIDE ( [LucroTotal], [TotalVendas] )

MargemMedia =
DIVIDE ( [LucroTotal], [TotalVendas] )

ComissaoValor =
SUMX (
    Vendas,
    Vendas[ValorVenda] * Vendas[Comissão (Percentual)] / 100
)

LucroPorProduto =
SUMX (
    VALUES ( Vendas[ID-Produto] ),
    CALCULATE ( SUM ( Vendas[ValorVenda] ) - SUM ( Vendas[Custo] ) )
)

RankFabricante =
RANKX (
    ALL ( Vendas[Fabricante] ),
    [TotalVendas],
    ,
    DESC,
    DENSE
)

TopN_Vendas =
IF (
    RANKX (
        ALLSELECTED ( Vendas[Fabricante] ),
        [TotalVendas],
        ,
        DESC,
        DENSE
    ) <= 5,
    [TotalVendas],
    BLANK ()
)

Outros_Vendas =
VAR TotalGeral = [TotalVendas]
VAR Top5 =
    CALCULATE (
        [TotalVendas],
        TOPN ( 5, VALUES ( Vendas[Fabricante] ), [TotalVendas], DESC )
    )
RETURN
    TotalGeral - Top5
```

  # 🧠 2. SCRIPT C# – TABULAR EDITOR  
### Automação para criação e atualização das medidas DAX
## **Medidas C#**
```csharp
using System.Linq;

// 1) Cria a tabela calculada "Medidas" se não existir
var tabela = Model.Tables.FirstOrDefault(t => t.Name == "Medidas");
if (tabela == null)
{
    // DATATABLE cria uma tabela pequena e inócua; será ocultada em seguida
    tabela = Model.AddCalculatedTable("Medidas", "DATATABLE(\"_placeholder\", INTEGER, {{1}})");
    tabela.IsHidden = true; // oculta a tabela para não aparecer em visuais
}
else
{
    // garante que a tabela esteja oculta se já existir
    tabela.IsHidden = true;
}

// 2) Helper para criar/atualizar medidas na tabela "Medidas"
Action<string, string, string> CriarMedida = (nome, expressao, formato) =>
{
    var existente = tabela.Measures.FirstOrDefault(m => m.Name == nome);
    if (existente != null)
    {
        existente.Expression = expressao;
        existente.FormatString = formato;
    }
    else
    {
        var m = tabela.AddMeasure(nome, expressao);
        m.FormatString = formato;
    }
};

// 3) Medidas (DAX sem '=' no início). Ajuste nomes de tabelas/colunas se necessário.
CriarMedida("TotalVendas", @"SUM(Vendas[ValorVenda])", "R$ #,0.00");
CriarMedida("TotalCusto", @"SUM(Vendas[Custo])", "R$ #,0.00");
CriarMedida("LucroTotal", @"[TotalVendas] - [TotalCusto]", "R$ #,0.00");
CriarMedida("MargemMedia", @"DIVIDE([LucroTotal], [TotalVendas])", "0.00%");

CriarMedida("ComissaoValor", @"SUMX(Vendas, Vendas[ValorVenda] * Vendas[Comissão (Percentual)] / 100)", "R$ #,0.00");

CriarMedida("LucroPorProduto",
@"SUMX(
    VALUES(Vendas[ID-Produto]),
    CALCULATE(SUM(Vendas[ValorVenda]) - SUM(Vendas[Custo]))
)", "R$ #,0.00");

CriarMedida("VendasYTD", @"TOTALYTD([TotalVendas], 'Calendario'[Data])", "R$ #,0.00");

CriarMedida("CrescimentoPeriodo",
@"VAR VendasAtual = [TotalVendas]
VAR VendasAnoAnterior = CALCULATE([TotalVendas], SAMEPERIODLASTYEAR('Calendario'[Data]))
RETURN DIVIDE(VendasAtual - VendasAnoAnterior, VendasAnoAnterior, 0)", "0.00%");

CriarMedida("RankFabricante", @"RANKX(ALL(Vendas[Fabricante]), [TotalVendas], , DESC, DENSE)", "0");

CriarMedida("TopN_Vendas",
@"IF(
    RANKX(ALLSELECTED(Vendas[Fabricante]), [TotalVendas], , DESC, DENSE) <= 5,
    [TotalVendas],
    BLANK()
)", "R$ #,0.00");

CriarMedida("Outros_Vendas",
@"VAR TotalGeral = [TotalVendas]
VAR Top5 = CALCULATE([TotalVendas], TOPN(5, VALUES(Vendas[Fabricante]), [TotalVendas], DESC))
RETURN TotalGeral - Top5", "R$ #,0.00");

// Observação: não chamamos Model.SaveChanges() aqui porque algumas builds não expõem esse método.
// Salve as alterações pela UI: Model -> Save Changes (Ctrl+S)
  ```

  # 🧩 3. DECISÕES ANALÍTICAS  
### Critérios, escolhas de modelagem e lógica por trás das análises

Esta seção documenta **as decisões analíticas** tomadas durante o desenvolvimento do dashboard *Análise Executiva de Performance Comercial – Growth Wave*.  
O objetivo é registrar a lógica estratégica por trás das escolhas de métricas, visuais, filtros e estrutura narrativa, garantindo clareza para avaliadores técnicos, recrutadores e outros analistas.

---

## 🎯 3.1 Princípios que guiaram a análise

As decisões foram baseadas em quatro pilares:

1. **Clareza executiva**  
   - Priorizar indicadores que respondem perguntas de negócio.  
   - Evitar excesso de visuais ou métricas redundantes.  

2. **Consistência analítica**  
   - Mesmas medidas DAX utilizadas em todas as páginas.  
   - Padrões de cores e formatos replicados em todo o dashboard.  

3. **Navegação orientada à decisão**  
   - Estrutura em camadas: visão geral → diagnóstico → detalhe.  
   - Páginas independentes, mas conectadas por filtros sincronizados.  

4. **Performance e governança**  
   - Uso de medidas ao invés de colunas calculadas.  
   - Modelo estrela limpo e com relacionamentos unidirecionais.  

---

## 📊 3.2 Decisões sobre KPIs

### KPIs escolhidos:
- **Total de Vendas**  
- **Lucro Total**  
- **Margem (%)**  
- **Custo Total**  
- **Ticket Médio**  
- **Comissões Pagas**

### Justificativa:
- Representam **saúde financeira** e **eficiência comercial**.  
- São métricas universais em operações de varejo e distribuição.  
- Permitem análises comparativas entre categorias, produtos, lojas e vendedores.

---

## 🧱 3.3 Decisões sobre Modelagem

### Modelo Estrela
- 1 tabela fato (`F_Vendas`)  
- 4 dimensões principais (`D_Produto`, `D_Tempo`, `D_Loja`, `D_Vendedor`)  

**Por quê?**
- Facilita criação de medidas reutilizáveis.  
- Melhora performance do Power BI.  
- Evita ambiguidade de relacionamentos.  

### Relacionamentos unidirecionais
- Evita loops e ambiguidade.  
- Mantém o fluxo de filtro previsível.

---

## 📈 3.4 Decisões sobre Visuais

### Storytelling
- Linha temporal → tendência  
- Rosca → participação  
- Barras → ranking  
- KPIs → visão executiva  

**Motivo:**  
Cada visual responde uma pergunta específica de negócio.

---

### Influenciadores de Vendas
- Uso do visual de IA “Key Influencers”  
- Permite identificar fatores que aumentam o valor de venda  
- Complementa análises tradicionais com insights estatísticos

---

### Vendas por Categoria
- Treemap → mix  
- Linha → evolução  
- Tabela → detalhe  
- Waterfall → contribuição de lucro  

**Motivo:**  
Combinação ideal para entender **mix + tendência + lucratividade**.

---

### Performance de Vendedores
- Ranking → quem vende mais  
- Ticket médio → quem vende melhor  
- Tabela → detalhe operacional  
- KPIs → visão consolidada  

**Motivo:**  
Permite avaliar eficiência individual e por loja.

---

## 🎛 3.5 Decisões sobre Filtros e Interações

### Filtros sincronizados:
- Período  
- Loja  
- Vendedor  
- Categoria  

**Motivo:**  
Permite análises comparáveis entre páginas.

### Interações editadas:
- Treemap filtra todos os visuais  
- Ranking destaca séries no gráfico temporal  
- Tabelas não filtram KPIs (evita distorções)

---

## 🧠 3.6 Decisões sobre DAX

### Uso de:
- `DIVIDE()` → evita erros de divisão por zero  
- `SUMX()` → cálculos linha a linha  
- `RANKX()` → rankings dinâmicos  
- `TOTALYTD()` e `SAMEPERIODLASTYEAR()` → análises temporais  
- `ALL()` e `ALLSELECTED()` → controle de contexto  

**Motivo:**  
Boas práticas de DAX para garantir precisão e performance.

---

## 🚀 3.7 Decisões sobre Storytelling e UX

- Paleta de cores consistente  
- Ícones para navegação lateral  
- Títulos claros e objetivos  
- Notas explicativas em cada página  
- Layout limpo e com hierarquia visual  

**Motivo:**  
Melhorar a experiência do usuário e facilitar leitura executiva.

---

## 📌 3.8 Resumo das decisões

- Modelo estrela para governança  
- Medidas DAX padronizadas  
- Visuais escolhidos com propósito claro  
- Filtros sincronizados entre páginas  
- Estrutura narrativa orientada à tomada de decisão  
- Script C# para produtividade e padronização  

---

# 🧱 4. BOAS PRÁTICAS DE MODELAGEM  
### Diretrizes aplicadas no projeto para garantir performance, governança e escalabilidade

Esta seção documenta as boas práticas adotadas na construção do modelo semântico do Power BI, garantindo que o dashboard seja eficiente, sustentável e fácil de evoluir.

---

## 🧩 4.1 Estrutura em Modelo Estrela

O projeto segue rigorosamente o padrão **Star Schema**, composto por:

- **1 tabela fato**: F_Vendas  
- **4 dimensões**: Produto, Tempo, Loja, Vendedor  

### Benefícios:
- Reduz ambiguidade de relacionamentos  
- Melhora performance de cálculos DAX  
- Facilita criação de medidas reutilizáveis  
- Suporta escalabilidade do modelo  

---

## 🔄 4.2 Relacionamentos Unidirecionais

Todos os relacionamentos foram configurados como:

- **1:N (um para muitos)**  
- **Direção de filtro: Dimensão → Fato**

### Motivos:
- Evita loops e ambiguidade  
- Mantém o fluxo de filtro previsível  
- Melhora performance em visuais complexos  

---

## 🧮 4.3 Medidas ao invés de Colunas Calculadas

Sempre que possível, foram utilizadas **medidas DAX**, não colunas calculadas.

### Vantagens:
- Medidas são calculadas sob demanda  
- Reduz tamanho do modelo  
- Evita recomputações desnecessárias  
- Permite reutilização em múltiplos visuais  

---

## 🎛 4.4 Uso de Funções DAX Otimizadas

- `DIVIDE()` para evitar erros de divisão  
- `SUMX()` apenas quando necessário  
- `CALCULATE()` para manipular contexto  
- `ALL()` e `ALLSELECTED()` para controle de filtros  
- Funções de tempo (`TOTALYTD`, `SAMEPERIODLASTYEAR`) para análises temporais  

---

## 🎨 4.5 Padronização de Formatos

- Valores monetários → `R$ #,0.00`  
- Percentuais → `0.00%`  
- Rankings → sem casas decimais  

Essa padronização foi automatizada via **script C# no Tabular Editor**.

---

## ⚙️ 4.6 Governança e Organização

- Criação da tabela **Medidas** para centralizar todas as métricas  
- Tabela oculta para não poluir o painel de campos  
- Nomeação clara e consistente das medidas  
- Separação lógica entre medidas simples e derivadas  

---

## 🚀 4.7 Performance

- Evitou-se uso de `SUMX` sobre tabelas grandes  
- Relacionamentos otimizados  
- Filtros sincronizados apenas quando necessário  
- Visuais limitados a TopN para evitar sobrecarga  

---

# 📘 5. CONCLUSÃO DA DOCUMENTAÇÃO  
### Encerramento e visão geral do projeto

O projeto **Análise Executiva de Performance Comercial – Growth Wave** demonstra a aplicação prática de:

- Modelagem dimensional (Star Schema)  
- Construção de KPIs estratégicos  
- Uso avançado de DAX  
- Automação via Tabular Editor (C#)  
- Design de dashboards executivos  
- Storytelling orientado à tomada de decisão  

---

## 🎯 Impacto do Projeto

O dashboard permite que gestores:

- Identifiquem oportunidades de crescimento  
- Avaliem lucratividade por categoria e produto  
- Entendam fatores que influenciam o valor de venda  
- Acompanhem performance de vendedores e lojas  
- Tomem decisões baseadas em dados, não em intuição  

---

## 🧠 Aprendizados

Durante o desenvolvimento, foram reforçados conceitos como:

- Importância de um modelo estrela bem estruturado  
- Poder do DAX para análises avançadas  
- Valor da automação para produtividade  
- Necessidade de narrativa clara em dashboards executivos  

---

## 🚀 Evoluções Futuras

- Inclusão de metas (Budget vs. Actual)  
- Implementação de RLS (Row-Level Security)  
- Conexão com banco de dados em tempo real  
- Publicação no Power BI Service com alertas  
- Expansão para análise de clientes e churn  

---

## 📎 Finalização

Este documento consolida toda a lógica técnica, analítica e estrutural do projeto, servindo como referência para:

- Recrutadores  
- Avaliadores técnicos  
- Outros analistas de BI  
- Evolução futura do dashboard  

A documentação está completa e pronta para ser adicionada ao repositório GitHub.

