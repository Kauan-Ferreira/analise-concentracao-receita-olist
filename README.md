📊 Análise de Concentração de Receita (Curva ABC) - Olist E-commerce
Este projeto analisa o banco de dados público do e-commerce brasileiro Olist para identificar a concentração de faturamento do negócio. Através da aplicação do Princípio de Pareto (Curva ABC), o objetivo é responder a uma pergunta de negócio fundamental:

🎯 Quais categorias de produtos realmente sustentam a receita da empresa e devem ser priorizadas em estratégias de marketing, estoque e logística?

O pipeline integra extração de dados via Pandas, processamento de regras de negócio nativamente em banco de dados relacional (SQL) e visualização de dados focada em storytelling executivo.

🛠️ Tecnologias Utilizadas
🐍 Python (Pandas): Extração, limpeza e modelagem inicial dos dados (ETL).

💾 SQL (SQLite): Cálculos analíticos avançados, Window Functions e categorização condicional.

📈 Matplotlib: Visualização de dados avançada com foco na proporção Data-Ink e storytelling.

🧠 O Problema e as Decisões de Negócio
Para garantir que a análise refletisse a realidade financeira da empresa e não apenas intenções de compra, tomei decisões estritas de tratamento e engenharia de dados antes do cálculo da Curva ABC:

Filtro de Receita Real (Status do Pedido): Filtrei a base para considerar exclusivamente pedidos com status delivered. Pedidos cancelados, em processamento ou devolvidos inflariam o faturamento e levariam a decisões de negócio equivocadas.

Tratamento de Dados Faltantes: Categorias nulas (cerca de 1,8% da base) foram removidas. Optou-se por descartar esses registros em vez de agrupá-los em "Outros" para evitar distorções no ranqueamento das categorias nomeadas.

Processamento Híbrido (Pandas + SQL): O cruzamento das tabelas brutas (Orders, Items e Products) foi feito no Pandas por eficiência no manuseio de múltiplos arquivos .csv. Porém, a carga cognitiva da análise (cálculo de faturamento acumulado e percentual relativo) foi delegada ao SQL utilizando CTEs e Window Functions (SUM() OVER()). Isso simula um ambiente real onde as regras de negócio rodam otimizadas no lado do banco de dados.

Design da Visualização (Top 20): Embora a base contenha mais de 70 categorias, o gráfico final foi restrito ao Top 20. Do ponto de vista executivo, plotar a "cauda longa" (Classe C) inteira compromete a legibilidade e desvia a atenção das categorias da Classe A, que são o verdadeiro foco do relatório.

📊 Arquitetura e Fluxo de Dados (ETL)
Extract & Transform: Leitura dos arquivos CSV brutos, merge relacional via Pandas e agregação de volume de vendas e faturamento total por categoria.

Load: Inserção do dataframe processado no banco de dados SQLite local (banco_olist.db) de forma segura utilizando Context Managers (with).

Query Analytics: Execução da query de Curva ABC, classificando as categorias em:

🥇 Classe A: Responsáveis por até 80% do faturamento acumulado.

🥈 Classe B: Responsáveis pelos próximos 15% (de 80% a 95%).

🥉 Classe C: Os 5% finais da cauda longa.

💡 Principais Insights
O modelo comprovou uma alta concentração de receita típica do Princípio de Pareto. Das 73 categorias validadas no portfólio da Olist:

Apenas 16 categorias (aproximadamente 22% do catálogo) são responsáveis por sustentar 80% de todo o faturamento da empresa (Classe A).

Categorias como Beleza & Saúde, Relógios & Presentes e Cama, Mesa & Banho lideram a geração de caixa, indicando onde os esforços de retenção e negociação com fornecedores devem ser intensificados.

⚙️ Como reproduzir este projeto
1. Clone este repositório:

Bash
git clone https://github.com/Kauan-ferreira/seu-repositorio.git
2. Certifique-se de baixar os datasets originais do Kaggle (Olist) e alocá-los na pasta data/raw/.

3. Instale as dependências:

Bash
pip install pandas matplotlib
4. Execute o notebook/script Python para gerar automaticamente o banco de dados banco_olist.db e o gráfico executivo final.
