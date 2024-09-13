
# Estudo ETL com Oracle

Este projeto visa estudar e praticar o processo de ETL (Extração, Transformação e Carga) utilizando Oracle e suas ferramentas. O foco é entender como manipular dados de várias fontes, transformá-los e carregá-los de forma eficiente no banco de dados Oracle.

## 1. Fundamentos de ETL

- **Extração (E):** Como extrair dados de várias fontes (bancos de dados, arquivos, APIs).
- **Transformação (T):** Como limpar, padronizar, agregar e transformar os dados.
- **Carga (L):** Como carregar os dados transformados para um destino, como o Oracle Database.

## 2. Ferramentas e Tecnologias

- **Oracle Data Integrator (ODI):** Ferramenta principal de ETL da Oracle. Familiarize-se com a interface e os principais componentes, como Mappings, Interfaces e Packages.
- **SQL e PL/SQL:** Essenciais para transformar e manipular dados diretamente no Oracle.
- **Oracle SQL Loader:** Utilizado para carregar grandes volumes de dados de arquivos externos para o Oracle Database.

## 3. Estudos e Exercícios Práticos

### Exercício 1: Extração de Dados

Crie uma tabela no Oracle para armazenar os dados extraídos de um arquivo CSV de clientes:

```sql
CREATE TABLE clientes (
    ID NUMBER PRIMARY KEY,
    Nome VARCHAR2(100),
    Email VARCHAR2(100),
    Data_Cadastro DATE
);
```

Utilize o SQL Loader para carregar os dados do arquivo `clientes.csv` para a tabela criada. Crie um arquivo de controle `.ctl`:

```bash
LOAD DATA
INFILE 'clientes.csv'
INTO TABLE clientes
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
(ID, Nome, Email, Data_Cadastro "TO_DATE(:Data_Cadastro, 'YYYY-MM-DD')")
```

Execute o SQL Loader:

```bash
sqlldr userid=user/password control=clientes.ctl
```

### Exercício 2: Transformação de Dados

Transforme a coluna `Nome` para letras maiúsculas e calcule o número de dias desde o `Data_Cadastro`:

```sql
SELECT ID, 
       UPPER(Nome) AS Nome_Maiusculo, 
       Email, 
       ROUND(SYSDATE - Data_Cadastro) AS Dias_Desde_Cadastro
FROM clientes;
```

Crie uma nova tabela para armazenar os dados transformados:

```sql
CREATE TABLE clientes_transformados AS
SELECT ID, 
       UPPER(Nome) AS Nome_Maiusculo, 
       Email, 
       ROUND(SYSDATE - Data_Cadastro) AS Dias_Desde_Cadastro
FROM clientes;
```

### Exercício 3: Carga de Dados

Carregue os dados atualizados de um novo arquivo CSV (`clientes_novos.csv`) utilizando SQL Loader:

```bash
LOAD DATA
INFILE 'clientes_novos.csv'
INTO TABLE clientes_transformados
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
(ID, Nome_Maiusculo, Email, Dias_Desde_Cadastro)
```

Execute o SQL Loader:

```bash
sqlldr userid=user/password control=clientes_novos.ctl
```

### Exercício 4: Automação de Carga

Automatize o processo de ETL utilizando PL/SQL:

```plsql
CREATE OR REPLACE PROCEDURE etl_clientes IS
BEGIN
    -- Extração de dados
    INSERT INTO clientes_staging (ID, Nome, Email, Data_Cadastro)
    SELECT ID, Nome, Email, Data_Cadastro
    FROM clientes_externos;

    -- Transformação de dados
    INSERT INTO clientes_transformados (ID, Nome_Maiusculo, Email, Dias_Desde_Cadastro)
    SELECT ID, UPPER(Nome), Email, ROUND(SYSDATE - Data_Cadastro)
    FROM clientes_staging;

    -- Carga de dados
    COMMIT;
END;
/
```

Esse procedimento pode ser agendado para execução periódica com o Oracle **DBMS_SCHEDULER**.

### Exercício 5: Monitoramento de Performance

Monitore a performance das consultas ETL utilizando a visão `V$SQL`:

```sql
SELECT sql_id, 
       executions, 
       elapsed_time/1000000 AS elapsed_seconds, 
       cpu_time/1000000 AS cpu_seconds, 
       disk_reads, 
       buffer_gets 
FROM V$SQL 
WHERE sql_text LIKE '%clientes%';
```

Gere um relatório AWR:

```sql
EXEC DBMS_WORKLOAD_REPOSITORY.create_snapshot;
SELECT * FROM TABLE(DBMS_WORKLOAD_REPOSITORY.awr_report_text(
    (SELECT dbid FROM v$database), 
    1, 
    (SELECT MIN(snap_id) FROM dba_hist_snapshot), 
    (SELECT MAX(snap_id) FROM dba_hist_snapshot)
));
```

## 4. Melhores Práticas

- **Particionamento de Dados:** Em ETL, o particionamento de grandes tabelas pode melhorar o desempenho e facilitar a manutenção.
- **Manuseio de Erros:** Configure logs e manuseio de exceções em PL/SQL para tratar erros durante a extração ou carga de dados.
- **Tuning e Indexação:** Otimize consultas e operações de carga utilizando índices adequados e práticas de tuning.

## Conclusão

Este estudo aborda a prática de processos ETL no Oracle, fornecendo uma base sólida para entender e aplicar técnicas de extração, transformação e carga de dados, utilizando SQL, PL/SQL, SQL Loader e ferramentas de monitoramento.
