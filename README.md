# Projeto de Automação de Benefícios de RH com IA

## Visão Geral

Este projeto automatiza o processo de cálculo de benefícios de Vale-Refeição (VR) para funcionários, desde a extração e limpeza de dados de várias fontes de planilhas até a consolidação, análise com IA para aplicar regras de negócio e, finalmente, a exportação dos resultados para uma planilha formatada. A solução foi projetada para ser modular, configurável e robusta, utilizando um banco de dados PostgreSQL para gerenciamento de dados e a API Gemini do Google para lógica de negócios complexa.

## Fluxo de Trabalho

O processo de ponta a ponta é orquestrado da seguinte forma:

1.  **Download e Extração de Dados**: O sistema baixa um arquivo .zip de um URL especificado, o extrai e padroniza os nomes dos arquivos de planilha resultantes.
2.  **Limpeza de Nomes de Colunas**: As colunas em cada planilha são limpas e padronizadas para garantir a consistência para o processamento do banco de dados.
3.  **Sincronização com o Banco de Dados**: As planilhas limpas são usadas para criar tabelas em um banco de dados PostgreSQL. Os dados de cada planilha são então inseridos nas tabelas correspondentes.
4.  **Consolidação de Dados**: Os dados de várias tabelas são consolidados em uma única tabela principal, unindo informações de funcionários ativos, de férias, demitidos e recém-contratados.
5.  **Análise e Exclusão com IA**: Um agente de IA (Gemini) analisa os dados consolidados com base em regras predefinidas (por exemplo, excluindo diretores, estagiários ou funcionários em licença) e identifica os funcionários que não são elegíveis para o benefício. Esses registros são então removidos da tabela consolidada.
6.  **Geração de SQL com IA para Cálculo de Benefícios**: Um segundo agente de IA gera uma consulta SQL para calcular os valores do benefício de VR para os funcionários elegíveis restantes. Esta consulta insere os resultados em uma tabela `vr_mensal`.
7.  **Exportação de Dados**: Os dados finais da tabela `vr_mensal` são exportados para uma nova planilha do Excel, que é baseada em um modelo predefinido, preenchendo os valores calculados e resumindo o custo total.

## Módulos e Classes

O projeto é dividido em várias classes, cada uma com uma responsabilidade específica:

| Classe | Descrição |
| :--- | :--- |
| `ZipDownloader` | Responsável por baixar, extrair e limpar os nomes das planilhas de um arquivo zip. |
| `DatabaseConfig` | Gerencia a configuração da conexão com o banco de dados (host, nome do banco, usuário, senha, porta). |
| `DBUtils` | Fornece funções de utilidade para interagir com o banco de dados, como verificar a existência de tabelas, descartar tabelas e executar consultas SQL. |
| `ColumnNameCleaner` | Carrega planilhas do Excel, limpa e padroniza os nomes de suas colunas e salva as planilhas limpas em um novo diretório. |
| `DataSync` | Sincroniza os dados das planilhas do Excel com o banco de dados PostgreSQL, criando tabelas e inserindo os dados. |
| `DataConsolidator`| Consolida dados de várias tabelas em uma única tabela (`consolidado`) e prepara o ambiente do banco de dados para os cálculos. |
| `DataFrameConsolidado` | Um componente para encapsular o acesso ao dataframe da tabela `consolidado` no banco de dados. |
| `GeminiAgent` | Um cliente para interagir com a API generativa de IA do Google (Gemini), com gerenciamento de taxa de chamadas. |
| `ExclusionAgent` | Usa o `GeminiAgent` para analisar os dados dos funcionários e identificar quais devem ser excluídos com base em um conjunto de regras. |
| `VrMensalSqlAgent` | Usa o `GeminiAgent` para gerar dinamicamente a consulta SQL necessária para calcular os benefícios e popular a tabela `vr_mensal`. |
| `DataExport` | Exporta os dados calculados da tabela `vr_mensal` para um arquivo Excel formatado, usando um modelo. |
| `Infrastructure` | Orquestra a execução das diferentes fases do pipeline de processamento de dados (download, limpeza, sincronização, etc.). |

## Pré-requisitos

- Python 3.x
- Bibliotecas Python:
  - `pandas`
    - `psycopg2-binary`
      - `gdown`
        - `openpyxl`
          - `xlrd`
            - `google-generativeai`

            ## Configuração

            1.  **Variáveis de Ambiente**: As credenciais do banco de dados e a chave da API do Google devem ser configuradas como segredos no ambiente (por exemplo, usando `userdata` no Google Colab).
                - `SUPABASE_HOST`
                    - `SUPABASE_NAME`
                        - `SUPABASE_USER`
                            - `SUPABASE_PASS`
                                - `SUPABASE_PORT`
                                    - `GOOGLE_API_KEY`
                                        - `GOOGLE_GENERATIVE_MODEL` (opcional)
                                            - `GOOGLE_GENAI_RATE_LIMIT` (opcional)
                                                - `GOOGLE_GENAI_RATE_LIMIT_WINDOW` (opcional)

                                                2.  **Configuração do Fluxo de Trabalho**: A execução de cada etapa principal do processo pode ser habilitada ou desabilitada no bloco de execução principal, modificando os dicionários `infra_config`, `exclusion_config`, `vr_sql_agent_config` e `export_config`.

                                                ## Como Usar

                                                1.  Configure as credenciais do banco de dados e da API como segredos.
                                                2.  Certifique-se de que o arquivo de modelo do Excel (`VR_MENSAL_05.2025.xlsx`) esteja no local esperado (`/content/planilhas/`).
                                                3.  Execute o script Python. O script executará o fluxo de trabalho completo, desde o download dos dados até a exportação do relatório final.
                                                4.  O arquivo de saída, `VR MENSAL 05.2025.xlsx`, será salvo no diretório `/content/resultado`.
