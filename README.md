
# BDCD - Trabalho 2

### Grupo

Jonathan Braian Dias Vaz / RA - 790870

Fabrício Rodrigues dos Santos / RA - 790994

### Script para importação de base

```cypher
// NOTE: The following script syntax is valid for database version 5.0 and above.

:param {
  // Define the file path root and the individual file names required for loading.
  // https://neo4j.com/docs/operations-manual/current/configuration/file-locations/
  file_path_root: 'https://raw.githubusercontent.com/devfabri/BDCDT2/refs/heads/main/',
  file_0: 'temas.csv',
  file_1: 'deputados.csv',
  file_2: 'proposicoes.csv',
  file_3: 'partidos.csv',
  file_4: 'proposicoes_temas.csv',
  file_5: 'proposicoes_autores.csv',
  file_6: 'deputados_partidos.csv'
};

// CONSTRAINT creation
// -------------------
//
// Create node uniqueness constraints, ensuring no duplicates for the given node label and ID property exist in the database. This also ensures no duplicates are introduced in future.
//
CREATE CONSTRAINT `cod_Tema_uniq` IF NOT EXISTS
FOR (n: `Tema`)
REQUIRE (n.`cod`) IS UNIQUE;
CREATE CONSTRAINT `id_Deputado_uniq` IF NOT EXISTS
FOR (n: `Deputado`)
REQUIRE (n.`id`) IS UNIQUE;
CREATE CONSTRAINT `id_Proposicao_uniq` IF NOT EXISTS
FOR (n: `Proposicao`)
REQUIRE (n.`id`) IS UNIQUE;
CREATE CONSTRAINT `id_Partido_uniq` IF NOT EXISTS
FOR (n: `Partido`)
REQUIRE (n.`id`) IS UNIQUE;

:param {
  idsToSkip: []
};

// NODE load
// ---------
//
// Load nodes in batches, one node label at a time. Nodes will be created using a MERGE statement to ensure a node with the same label and ID property remains unique. Pre-existing nodes found by a MERGE statement will have their other properties set to the latest values encountered in a load file.
//
// NOTE: Any nodes with IDs in the 'idsToSkip' list parameter will not be loaded.
LOAD CSV WITH HEADERS FROM ($file_path_root + $file_0) AS row
WITH row
WHERE NOT row.`cod` IN $idsToSkip AND NOT toInteger(trim(row.`cod`)) IS NULL
CALL (row) {
  MERGE (n: `Tema` { `cod`: toInteger(trim(row.`cod`)) })
  SET n.`cod` = toInteger(trim(row.`cod`))
  SET n.`nome` = row.`nome`
} IN TRANSACTIONS OF 10000 ROWS;

LOAD CSV WITH HEADERS FROM ($file_path_root + $file_1) AS row
WITH row
WHERE NOT row.`id` IN $idsToSkip AND NOT toInteger(trim(row.`id`)) IS NULL
CALL (row) {
  MERGE (n: `Deputado` { `id`: toInteger(trim(row.`id`)) })
  SET n.`id` = toInteger(trim(row.`id`))
  SET n.`nome` = row.`nome`
  SET n.`siglaPartido` = row.`siglaPartido`
  SET n.`siglaUf` = row.`siglaUf`
  SET n.`idLegislatura` = toInteger(trim(row.`idLegislatura`))
  SET n.`urlFoto` = row.`urlFoto`
} IN TRANSACTIONS OF 10000 ROWS;

LOAD CSV WITH HEADERS FROM ($file_path_root + $file_2) AS row
WITH row
WHERE NOT row.`id` IN $idsToSkip AND NOT toInteger(trim(row.`id`)) IS NULL
CALL (row) {
  MERGE (n: `Proposicao` { `id`: toInteger(trim(row.`id`)) })
  SET n.`id` = toInteger(trim(row.`id`))
  SET n.`siglaTipo` = row.`siglaTipo`
  SET n.`codTipo` = toInteger(trim(row.`codTipo`))
  SET n.`numero` = toInteger(trim(row.`numero`))
  SET n.`ano` = toInteger(trim(row.`ano`))
  SET n.`ementa` = row.`ementa`
} IN TRANSACTIONS OF 10000 ROWS;

LOAD CSV WITH HEADERS FROM ($file_path_root + $file_3) AS row
WITH row
WHERE NOT row.`id` IN $idsToSkip AND NOT toInteger(trim(row.`id`)) IS NULL
CALL (row) {
  MERGE (n: `Partido` { `id`: toInteger(trim(row.`id`)) })
  SET n.`id` = toInteger(trim(row.`id`))
  SET n.`sigla` = row.`sigla`
  SET n.`nome` = row.`nome`
} IN TRANSACTIONS OF 10000 ROWS;


// RELATIONSHIP load
// -----------------
//
// Load relationships in batches, one relationship type at a time. Relationships are created using a MERGE statement, meaning only one relationship of a given type will ever be created between a pair of nodes.
LOAD CSV WITH HEADERS FROM ($file_path_root + $file_4) AS row
WITH row 
CALL (row) {
  MATCH (source: `Proposicao` { `id`: toInteger(trim(row.`idProposicao`)) })
  MATCH (target: `Tema` { `cod`: toInteger(trim(row.`codTema`)) })
  MERGE (source)-[r: `ABORDA`]->(target)
  SET r.`idProposicao` = toInteger(trim(row.`idProposicao`))
  SET r.`codTema` = toInteger(trim(row.`codTema`))
} IN TRANSACTIONS OF 10000 ROWS;

LOAD CSV WITH HEADERS FROM ($file_path_root + $file_5) AS row
WITH row 
CALL (row) {
  MATCH (source: `Deputado` { `id`: toInteger(trim(row.`idAutor`)) })
  MATCH (target: `Proposicao` { `id`: toInteger(trim(row.`idProposicao`)) })
  MERGE (source)-[r: `CRIA`]->(target)
  SET r.`idProposicao` = toInteger(trim(row.`idProposicao`))
  SET r.`idAutor` = toInteger(trim(row.`idAutor`))
} IN TRANSACTIONS OF 10000 ROWS;

LOAD CSV WITH HEADERS FROM ($file_path_root + $file_6) AS row
WITH row 
CALL (row) {
  MATCH (source: `Deputado` { `id`: toInteger(trim(row.`idDeputado`)) })
  MATCH (target: `Partido` { `id`: toInteger(trim(row.`idPartido`)) })
  MERGE (source)-[r: `PERTENCE_A`]->(target)
  SET r.`idDeputado` = toInteger(trim(row.`idDeputado`))
  SET r.`idPartido` = toInteger(trim(row.`idPartido`))
} IN TRANSACTIONS OF 10000 ROWS;

```
