# Pipeline de Big Data: Livros Kindle (Kaggle) → Parquet → MongoDB

**Avaliação N3 – Desenvolvimento com Linguagens Disruptivas I**

**Equipe:** Yasmin Ferreira, Paula Haas, Gustavo Correia da Silveira e Marcella Cagliari.

**Dataset:** [Amazon Kindle Books Dataset 2023 (130k books)](https://www.kaggle.com/datasets/asaniczka/amazon-kindle-books-dataset-2023-130k-books)

## O que o pipeline faz

1. **Captura:** baixa o CSV do Kaggle para `landing/` (o arquivo original nunca é alterado).
2. **Leitura:** Spark lê o CSV com os tipos das colunas declarados.
3. **Validação:** registros inválidos são enviados para `rejected/`, acompanhados do motivo da rejeição.
4. **Transformação:** cria campos como `publication.year`, `pricing.priceRange`, `rating.hasReviews`, `author.authorKey` e `ingestion.runId/loadedAt`.
5. **Parquet:** grava os dados em `curated/books/`, particionados por ano de publicação.
6. **MongoDB:** carrega os livros na coleção `books` (upsert pelo `asin`) e gera `categories` e `authors` por agregação.
7. **Índices e log:** cria índices e registra a execução na coleção `ingestion_runs`.

## Requisitos

Para executar o projeto no Windows, é necessário instalar:

- **Python 3.12:** [Download oficial](https://www.python.org/downloads/release/python-31210/)
- **Java JDK 17 (Temurin):** [Download](https://adoptium.net/temurin/releases/)
- **Docker Desktop:** [Download](https://www.docker.com/products/docker-desktop/)
- **Hadoop para Windows:** disponibilizar `winutils.exe` e `hadoop.dll` na pasta `C:\hadoop\bin`.

Também é necessário ter o Docker Desktop instalado e em execução.

## Como executar no Windows

### 1. Clonar o repositório

```powershell
git clone https://github.com/Calangogu1/kindle-pipeline.git
cd kindle-pipeline
```

### 2. Configurar o Hadoop

No PowerShell, execute:

```powershell
$env:HADOOP_HOME = "C:\hadoop"
$env:hadoop_home = "C:\hadoop"
$env:PATH += ";C:\hadoop\bin"
```

### 3. Iniciar o MongoDB

Com o Docker Desktop aberto, execute:

```powershell
docker run -d --name mongo-kindle -p 27017:27017 -v mongo-kindle-data:/data/db mongo:7
```

Caso o contêiner já exista, utilize:

```powershell
docker start mongo-kindle
```

### 4. Criar e ativar o ambiente virtual

```powershell
py -3.12 -m venv .venv
```

Ative o ambiente:

```powershell
.venv\Scripts\Activate.ps1
```

### 5. Instalar as dependências

```powershell
pip install -r requirements.txt
```

### 6. Executar o pipeline

```powershell
python pipeline.py
```

Na primeira execução, o pipeline realiza o download do dataset automaticamente, caso necessário.

### 7. Executar as consultas de demonstração

```powershell
python consultas.py
```

As consultas apresentam os resultados armazenados no MongoDB.

**Observação:** se o download automático falhar, baixe o CSV manualmente pelo Kaggle, descompacte e coloque o arquivo dentro da pasta `landing/`. Depois, execute novamente `python pipeline.py`.

## Utilizando MongoDB Atlas

Também é possível utilizar o MongoDB Atlas no lugar do Docker.

No PowerShell, configure a variável de ambiente:

```powershell
$env:MONGO_URI = "mongodb+srv://USUARIO:SENHA@cluster.mongodb.net"
```

Depois execute:

```powershell
python pipeline.py
```

## Saídas esperadas

| Local | Resultado |
|---|---|
| Terminal | Etapas 1/7 até 7/7 e resumo da execução |
| `landing/` | Arquivo CSV original |
| `curated/books/` | Arquivos Parquet particionados por ano |
| `rejected/` | CSV com registros rejeitados e motivo da rejeição |
| MongoDB (`kindle`) | Coleções `books`, `categories`, `authors` e `ingestion_runs` |
| `python consultas.py` | Consultas de demonstração e tabelas com os resultados |

O pipeline pode ser executado várias vezes. O mecanismo de upsert utilizando o campo `asin` evita a duplicação de livros.

## Estrutura do projeto

```text
kindle-pipeline/
├── pipeline.py
├── consultas.py
├── requirements.txt
├── landing/
├── curated/
│   └── books/
└── rejected/
```

As pastas `landing/`, `curated/` e `rejected/` são utilizadas para armazenar os dados durante a execução do pipeline.
