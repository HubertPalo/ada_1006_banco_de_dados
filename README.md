# Avaliação Bancos de Dados I
Aluno: Darlinne Hubert Palo Soto

## Modelagem e normalização de bancos de dados relacionais

Certo dia, um dos gestores do banco em que você trabalha como cientista de dados procurou você pedindo ajuda para projetar um pequeno banco de dados com o objetivo de mapear os clientes da companhia pelos diferentes produtos financeiros que eles contrataram.

O gestor explicou que o banco tinha uma grande quantidade de clientes e oferecia uma variedade de produtos financeiros, como cartões de crédito, empréstimos, seguros e investimentos. No entanto, eles estavam tendo dificuldades para entender quais produtos eram mais populares entre os clientes e como esses produtos estavam interagindo entre si.

Como ponto de partida, o gestor deixou claro que um cliente pode contratar vários produtos diferentes ao passo que um mesmo produto pode também estar associado a vários clientes diferentes e elaborou um rústico esboço de banco de dados com duas tabelas, da seguinte forma:

Tabela 1

Nome da tabela: cliente
Colunas: codigo_cliente, nome_cliente, sobrenome_cliente, telefones_cliente, municipio_cliente, codigo_tipo_cliente, tipo_cliente
Tabela 2

Nome da tabela: produto
Colunas: codigo_produto, nome_produto, descricao_produto, codigo_tipo_produto, tipo_produto, codigo_diretor_responsavel, nome_diretor_responsavel, email_diretor_responsavel
1) Ainda sem fazer normalizações, apresente o modelo conceitual deste esboço oferecido pelo gestor, destacando atributos chaves e multivalorados, caso existam, e apresentando também a cardinalidade dos relacionamentos.

![alt text](https://github.com/HubertPalo/ada_1006_banco_de_dados/blob/main/modelo_conceitual.png?raw=true)

2) Agora apresente um modelo lógico que expresse as mesmas informações e relacionamentos descritos no modelo original, mas decompondo-os quando necessário para que sejam respeitadas as 3 primeiras formas normais. Destaque atributos chaves e multivalorados, caso existam, e apresente também a cardinalidade dos relacionamentos.

![alt text](https://github.com/HubertPalo/ada_1006_banco_de_dados/blob/main/modelo_logico.png?raw=true)


## Consultas SQL simples e complexas em um banco de dados postgres
Um exemplo de modelo de banco de dados com relacionamento muitos-para-muitos pode ser o de um e-commerce que tem produtos e categorias, onde um produto pode pertencer a várias categorias e uma categoria pode estar associada a vários produtos. Nesse caso, teríamos duas tabelas: "produtos" e "categorias", com uma tabela intermediária "produtos_categorias" para relacionar os produtos às suas categorias.

CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10, 2) NOT NULL,
);

CREATE TABLE categorias (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);

CREATE TABLE produtos_categorias (
    produto_id INTEGER REFERENCES produtos(id),
    categoria_id INTEGER REFERENCES categorias(id),
    PRIMARY KEY (produto_id, categoria_id)
);
Assim, usando o subconjunto da "structured query language" chamado de DQL, produza consultas postgress de modo a atender cada uma das seguintes solicitações:

3) Liste os nomes de todos os produtos que custam mais de 100 reais, ordenando-os primeiramente pelo preço e em segundo lugar pelo nome. Use alias para mostrar o nome da coluna nome como "Produto" e da coluna preco como "Valor". A resposta da consulta não deve mostrar outras colunas de dados.

```
SELECT
	nome as "Produto",
	preco as "Valor"
FROM produtos
WHERE preco > 100
ORDER BY "Valor" ASC, "Produto" ASC
```

4) Liste todos os ids e preços de produtos cujo preço seja maior do que a média de todos os preços encontrados na tabela "produtos".

```
SELECT id, preco FROM produtos
WHERE preco > (SELECT AVG(preco) FROM produtos)
```

5) Para cada categoria, mostre o preço médio do conjunto de produtos a ela associados. Caso uma categoria não tenha nenhum produto a ela associada, esta categoria não deve aparecer no resultado final. A consulta deve estar ordenada pelos nomes das categorias.

```
SELECT
	cat.nome as categoria,
	products_grouped.avg_preco as preco_medio
FROM
(
	SELECT
		AVG(prod.preco) as avg_preco,
		produtos_categorias.categoria_id as cat_id
	FROM
	produtos_categorias
	LEFT JOIN produtos as prod
	ON produtos_categorias.produto_id = prod.id
	GROUP BY produtos_categorias.categoria_id
) as products_grouped
LEFT JOIN categorias as cat
ON products_grouped.cat_id = cat.id
ORDER BY cat.nome ASC
```

## Inserções, alterações e remoções de objetos e dados em um banco de dados postgres
Você está participando de um processo seletivo para trabalhar como cientista de dados na Ada, uma das maiores formadoras do país em áreas correlatadas à tecnologia. Dividido em algumas etapas, o processo tem o objetivo de avaliar você nos quesitos Python, Machine Learning e Bancos de Dados. Ainda que os dois primeiros sejam o cerne da sua atuação no dia-a-dia, considera-se que Bancos de Dados também constituem um requisito importante e, por isso, esta etapa pode ser a oportunidade que você precisava para se destacar dentre os seus concorrentes, demonstrando um conhecimento mais amplo do que os demais.

6) Com o objetivo de demonstrar o seu conhecimento através de um exemplo contextualizado com o dia-a-dia da escola, utilize os comandos do subgrupo de funções DDL para construir o banco de dados simples abaixo, que representa um relacionamento do tipo 1,n entre as entidades "aluno" e "turma":

Tabela 1

Nome da tabela: aluno
Colunas da tabela: id_aluno (INT), nome_aluno (VARCHAR), aluno_alocado (BOOLEAN), id_turma (INT)

Tabela 2

Nome da tabela: turma
Colunas da tabela: id_turma (INT), código_turma (VARCHAR), nome_turma (VARCHAR)

```
CREATE TABLE turma(
	id_turma SERIAL PRIMARY KEY,
	codigo_turma VARCHAR(20) NOT NULL,
	nome_turma VARCHAR(100) NOT NULL
)
```
```
CREATE TABLE aluno(
	id_aluno SERIAL PRIMARY KEY,
	nome_aluno VARCHAR(100) NOT NULL,
	aluno_alocado BOOL,
	id_turma INTEGER REFERENCES turma(id_turma)
)
```

7) Agora que você demonstrou que consegue ser mais do que um simples usuário do banco de dados, mostre separadamente cada um dos códigos DML necessários para cumprir cada uma das etapas a seguir:

a) Inserir pelo menos duas turmas diferentes na tabela de turma;

```
INSERT INTO turma(codigo_turma, nome_turma)
VALUES
('T001', 'Turma 1'),
('T002', 'Turma 2')
```

b) Inserir pelo menos 1 aluno alocado em cada uma destas turmas na tabela aluno (todos com NULL na coluna aluno_alocado);

```
INSERT INTO aluno(nome_aluno, aluno_alocado, id_turma)
VALUES
('Aluno 1 emturma 1', NULL, 1),
('Aluno 2 emturma 2', NULL, 2)
```

c) Inserir pelo menos 2 alunos não alocados em nenhuma turma na tabela aluno (todos com NULL na coluna aluno_alocado);

```
INSERT INTO aluno(nome_aluno, aluno_alocado, id_turma)
VALUES
('Aluno 3 semturma', NULL, NULL),
('Aluno 4 semturma', NULL, NULL)
```

d) Atualizar a coluna aluno_alocado da tabela aluno, de modo que os alunos associados a uma disciplina recebam o valor True e alunos não associdos a nenhuma disciplina recebam o falor False para esta coluna.

```
UPDATE aluno
SET aluno_alocado = CASE WHEN id_turma IS NULL THEN FALSE ELSE TRUE END
```
