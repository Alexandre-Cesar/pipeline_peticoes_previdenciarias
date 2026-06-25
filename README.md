# Desafio Técnico – Geração Automática de Petições Previdenciárias

## Objetivo

O objetivo deste projeto é automatizar o processamento de documentos utilizados em processos previdenciários, realizando a extração das informações relevantes, consolidando os dados em uma única estrutura e, por fim, gerando automaticamente uma petição inicial utilizando um modelo de linguagem (LLM).

A solução foi desenvolvida utilizando Databricks Community Edition, Apache Spark e Python.

---

# Ferramentas Utilizadas

## Databricks Community Edition

Utilizado como ambiente principal de desenvolvimento, responsável pela execução dos notebooks, processamento distribuído e armazenamento das tabelas.

## Apache Spark / PySpark

Responsável pelo processamento dos dados, transformação das informações extraídas e construção das tabelas Bronze, Silver e Gold.

## Regex

Toda a extração das informações dos documentos foi realizada através de expressões regulares.

Os padrões foram centralizados em um único arquivo de configuração (`documentos.json`), facilitando futuras manutenções sem necessidade de alterar o código.

## JSON

Após a consolidação dos dados, o registro é convertido para JSON para servir como entrada para o modelo de IA responsável pela geração da petição.

## OpenAI GPT

Responsável por gerar automaticamente uma nova petição utilizando os dados estruturados do processo e mantendo a estrutura do modelo de referência.

> **Observação:** A integração foi implementada, porém a execução da API não foi realizada devido à ausência de créditos disponíveis para utilização da API da OpenAI.

## python-docx

Biblioteca utilizada para gerar automaticamente o documento final em formato `.docx`.

---

# Raciocínio Utilizado

Inicialmente foi considerada a possibilidade de enviar diretamente todos os documentos para um modelo de IA.

Entretanto, essa abordagem apresentava algumas desvantagens:

* maior consumo de tokens;
* dificuldade de auditoria dos dados utilizados;
* baixa reutilização das informações extraídas;
* dependência integral do modelo para interpretar os documentos.

Por esse motivo, foi adotada uma arquitetura baseada em pipeline de dados.

Cada documento é processado individualmente, permitindo extrair apenas as informações relevantes através de Regex.

Posteriormente essas informações são consolidadas em uma única tabela (Gold), contendo todos os dados necessários para geração da petição.

Dessa forma, o modelo de IA recebe apenas dados estruturados, reduzindo o volume de informações enviadas e aumentando a previsibilidade da resposta.

---

# Fluxo do Pipeline

```text
PDFs
    │
    ▼
Extração do texto
    │
    ▼
Camada Bronze
(Texto bruto)
    │
    ▼
Regex
    │
    ▼
Camada Silver
(Tabelas específicas)
    │
    ▼
Junção dos documentos
    │
    ▼
Camada Gold
(Tabela consolidada)
    │
    ▼
JSON
    │
    ▼
LLM
    │
    ▼
Petição em DOCX
```

---

# Orquestração

O pipeline foi projetado para execução automática utilizando **Databricks Workflows**.

Cada notebook representa uma etapa independente do processamento.

Fluxo sugerido da orquestração:

1. Extração dos PDFs
2. Processamento do RG
3. Processamento do Comprovante de Residência
4. Processamento do Laudo Médico
5. Processamento da Carta de Indeferimento
6. Consolidação da camada Gold
7. Conversão para JSON
8. Geração da Petição
9. Geração do arquivo `.docx`

Caso novos documentos sejam adicionados ao ambiente, basta executar novamente o Workflow para que todas as tabelas sejam atualizadas automaticamente.

---

# O que Funcionou Bem

Durante o desenvolvimento alguns pontos se destacaram positivamente:

* Separação da solução em camadas (Bronze, Silver e Gold);
* Centralização de todos os Regex em um arquivo de configuração;
* Facilidade para adicionar novos tipos de documentos;
* Construção de uma tabela Gold consolidada contendo todas as informações necessárias para geração da petição;
* Geração automática do documento Word.

Essa arquitetura também facilita futuras integrações com outras soluções de IA, já que o modelo recebe um JSON estruturado ao invés dos documentos originais.

---

# Dificuldades Encontradas

A principal dificuldade foi relacionar os diferentes documentos.

Nem todos os documentos continham um identificador único (como CPF).

Foi necessário criar uma estratégia de relacionamento utilizando os dados disponíveis em cada documento, realizando a consolidação das informações antes da geração da tabela Gold.

Outro desafio foi lidar com pequenas variações nos layouts dos documentos, tornando necessário criar Regex mais robustos.

---

# Melhorias Futuras

Com mais tempo, algumas melhorias poderiam ser implementadas:

* Utilização de OCR especializado para documentos digitalizados;
* Validação automática dos dados extraídos (ex.: validação de CPF e datas);
* Substituição parcial das Regex por modelos de Document AI para maior flexibilidade;
* Integração completa com a API da OpenAI para geração automática da petição sem intervenção manual;
* Criação de testes automatizados para os Regex;
* Inclusão de monitoramento e tratamento de erros durante a execução do Workflow;
* Versionamento dos modelos de petição e dos prompts utilizados pelo LLM.

---

# Considerações Finais

A solução proposta prioriza a separação entre processamento de dados e geração de texto.

O pipeline produz uma camada estruturada contendo todas as informações relevantes do processo, permitindo que diferentes modelos de IA sejam utilizados futuramente sem necessidade de alterar o processamento dos documentos.

Essa abordagem torna a solução mais escalável, auditável e de fácil manutenção, além de reduzir significativamente o volume de dados enviados ao modelo de linguagem.
