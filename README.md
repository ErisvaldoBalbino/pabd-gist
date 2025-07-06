# O Cartógrafo Digital - GiST para Dados Geoespaciais

[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![PostGIS](https://img.shields.io/badge/PostGIS-336791?style=for-the-badge&logo=postgresql&logoColor=white)](https://postgis.net/)

## O que é GiST?

**GiST (Generalized Search Tree)** é um framework de indexação flexível projetado para suportar tipos de dados complexos que não se encaixam bem na estrutura tradicional de índices B-Tree.

### Características Principais:
- **Framework** - Permite construção de diversos tipos de índices especializados
- **Ideal para dados multidimensionais** - Especialmente dados geoespaciais

## Como Funciona

1. **Nós Internos**: Contêm bounding boxes que englobam regiões
2. **Nós Folha**: Contêm os dados reais
3. **Busca**: Filtra regiões inteiras

Representações da estrutura de um índice GiST:

![image](https://github.com/user-attachments/assets/682adee4-5099-416a-bea1-53ad15eeb3d2)
![image](https://github.com/user-attachments/assets/6b7b5be7-8eb5-4019-b4c0-4fab40ade1cc)

![image](https://github.com/user-attachments/assets/fb2fc668-6715-426f-a8a3-c012fb3438e8)


## Utilizando o GiST

### 1. Instalar e habilitar a extensão

Tutorial para instalação: https://www.bostongis.com/PrinterFriendly.aspx?content_name=postgis_tut01


```sql
CREATE EXTENSION IF NOT EXISTS postgis;
```

### 2. Criar a tabela

```sql
CREATE TABLE coordenadas_geograficas (
    localizacao GEOGRAPHY(Point, 4326)
);
```

### 3. Popular a tabela

```sql
INSERT INTO coordenadas_geograficas (localizacao)
SELECT ST_SetSRID(
           ST_MakePoint(
               random() * 40.0 - 74.0,
               random() * 27.0 - 33.0
           ), 4326)
FROM generate_series(1, 5000000);
```

### 4. Testar a consulta sem o índice

```sql
EXPLAIN ANALYZE 
SELECT COUNT(*) 
FROM coordenadas_geograficas 
WHERE ST_DWithin(
    localizacao, 
    ST_SetSRID(ST_MakePoint(-46.63, -27.55), 4326), 
    0.009
);
```
Essa consulta conta quantos pontos existem dentro de uma área.

### 5. Criar o índice

```sql
CREATE INDEX idx_coordenadas_localizacao_gist ON coordenadas_geograficas USING GIST (localizacao);
```

### 6. Testar a consulta com o índice

```sql
EXPLAIN ANALYZE 
SELECT COUNT(*) 
FROM coordenadas_geograficas 
WHERE ST_DWithin(
    localizacao, 
    ST_SetSRID(ST_MakePoint(-46.63, -23.55), 4326),
    0.009
);
```

## Resultados de Performance

### Sem Índice

Parallel Seq Scan on coordenadas_geograficas

Execution Time: 676.931 ms

### Com Índice

Index Scan using idx_coordenadas_localizacao_gist on coordenadas_geograficas

Execution Time: 0.079 ms

Mais de **5000x** mais rápido!

## Referências

- [Documentação PostgreSQL GiST](https://www.postgresql.org/docs/current/gist.html)
- [Indexes in PostgreSQL — 5 (GiST)](https://medium.com/postgres-professional/indexes-in-postgresql-5-gist-86e19781b5db)
- [Cleaning out crocodiles teeth with PostgreSQL indexes](https://www.citusdata.com/blog/2017/10/17/tour-of-postgres-index-types/)
- [GiST Index Building in PostgreSQL 15](https://www.youtube.com/watch?v=TG28lRoailE)
