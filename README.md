# cbh-codes

Comitês das Bacias Hidrográficas (CBHs) do Brasil, códigos e legislação de criação

.. EM CONSTRUCAO ... Ver planilha aberta colaborativa [AQUI!](https://docs.google.com/spreadsheets/d/1VqObVuwNMn15vVn5DJ_pJqalX2nnmjHJVPKDUNDcArk/)

Instalação com sql-unifier: acrescentar `"datasets-br/city-codes":null`.


```sql
CREATE VIEW dataset.vw_cbh_revisada AS  
  SELECT *, CASE
      WHEN char_length(codigo) <5 THEN substr(codigo,1,3)||'0'||substr(codigo,4)
      ELSE codigo
    END as cod2
  FROM dataset.vw_cbh
;
CREATE VIEW dataset.vw_cbh2 AS
  SELECT f.*, c.uf, c.nome, c.norma_criacao
  FROM  dataset.vw_cbh_forum f LEFT JOIN dataset.vw_cbh_revisada c
        ON c.cod2=f.codigo --WHERE c.codigo IS NULL
;

CREATE VIEW dataset.vw_cbh2_final AS
  SELECT codigo, 'br;'||lower(uf) as jurisdicao,
        sigla as abreviacao, nome_completo,
        CASE WHEN nome_completo!=nome THEN nome ELSE NULL END as nome_alternativo,
        norma_criacao
  FROM dataset.vw_cbh2  order by 1;
COPY (select * from  dataset.vw_cbh2_final) TO '/tmp/cbh_completo.csv' CSV HEADER;

-- -- -- --

-- ausencias
SELECT codigo
FROM dataset.vw_cbh2
WHERE uf is  null;  -- 43 items ausentes de vw_cbh

-- diferenças de nome
SELECT nome, nome_completo
FROM dataset.vw_cbh2
WHERE uf is not null AND  nome!=nome_completo; -- 171 inconsistências.

```
