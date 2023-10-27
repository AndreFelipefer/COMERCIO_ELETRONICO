   ![comercio](https://github.com/AndreFelipefer/COMERCIO_ELETRONICO/assets/129207232/3b9c9345-c416-4f29-a46c-cc91ab04e8a1) 
### O sistema envolve várias tabelas, incluindo "Produtos" para armazenar informações sobre produtos, "Pedidos" para registrar detalhes de pedidos, "Clientes" para manter informações de clientes e "Itens de Pedido" para registrar produtos em cada pedido. Além disso, foram estabelecidos relacionamentos essenciais entre "Pedidos" e "Clientes" para rastrear os pedidos de cada cliente, e entre "Itens de Pedido" e "Produtos" para associar produtos aos pedidos. Este sistema é projetado para facilitar a gestão e o rastreamento de pedidos em um ambiente de comércio eletrônico.

# Modelo Lógico:
![image](https://github.com/AndreFelipefer/COMERCIO_ELETRONICO/assets/129207232/17aefaf2-0ba1-4ad4-95ef-382956d198f4)

## Criando as tabelas -
```SQL
CREATE TABLE produtos (
  id INT NOT NULL AUTO_INCREMENT,
  nome VARCHAR(255) NOT NULL,
  descricao VARCHAR(255),
  preco DECIMAL(10,2) NOT NULL,
  quantidade_estoque INT NOT NULL,
  PRIMARY KEY (id)
);

CREATE TABLE pedidos (
  id INT NOT NULL AUTO_INCREMENT,
  data DATETIME NOT NULL,
  cliente_id INT NOT NULL,
  status VARCHAR(255) NOT NULL,
  PRIMARY KEY (id),
  FOREIGN KEY (cliente_id) REFERENCES clientes (id)
);

CREATE TABLE clientes (
  id INT NOT NULL AUTO_INCREMENT,
  nome VARCHAR(255) NOT NULL,
  endereco_entrega VARCHAR(255),
  telefone VARCHAR(255),
  email VARCHAR(255),
  PRIMARY KEY (id)
);

CREATE TABLE itens_pedido (
  pedido_id INT NOT NULL,
  produto_id INT NOT NULL,
  quantidade INT NOT NULL,
  PRIMARY KEY (pedido_id, produto_id),
  FOREIGN KEY (pedido_id) REFERENCES pedidos (id),
  FOREIGN KEY (produto_id) REFERENCES produtos (id)
);
```
## Criando os relacionamentos
```SQL
ALTER TABLE pedidos ADD FOREIGN KEY (cliente_id) REFERENCES clientes (id);

ALTER TABLE itens_pedido ADD FOREIGN KEY (produto_id) REFERENCES produtos (id);
```
## Implementando as stored procedures
### Adicionar produto ao carrinho
```SQL
delimiter $
CREATE PROCEDURE adicionar_produto_ao_carrinho (
  IN id_produto INT,
  IN quantidade INT
)
BEGIN

  INSERT INTO itens_pedido ( produto_id, quantidade)
  VALUES (id_produto, quantidade);

END$
delimiter ;
```

## Processar pedido
```SQL
delimiter $

CREATE PROCEDURE processar_pedido (
  IN id_cliente INT,
  IN data DATETIME,
  IN status VARCHAR(255)
)
BEGIN

  DECLARE id_pedido INT;

  -- Incrementar o id do pedido
  SET id_pedido = (SELECT MAX(id) + 1 FROM pedidos);

  -- Inserir os detalhes do pedido
  INSERT INTO pedidos (id, data, cliente_id, status)
  VALUES (id_pedido, data, id_cliente, status);

  -- Atualizar o estoque dos produtos
  UPDATE produtos
  SET quantidade_estoque = quantidade_estoque - 
    (SELECT quantidade FROM itens_pedido WHERE pedido_id = id_pedido);

END$

delimiter ;
```
## Calcular o total de um pedido 
```SQL
delimiter $

CREATE PROCEDURE calcular_total_pedido (
  IN id_pedido INT
)
BEGIN
  DECLARE total DECIMAL(10,2);

  -- Obter o total do pedido
  SELECT SUM(produto.preco * itens_pedido.quantidade) INTO total
  FROM itens_pedido
  INNER JOIN produtos ON produtos.id = itens_pedido.produto_id
  WHERE itens_pedido.pedido_id = id_pedido;

  -- Retornar o total
  SELECT total;
END$
delimiter ;
```

# Implementando as views
## Histórico de pedidos
```SQL
CREATE VIEW historico_pedidos AS
SELECT
  pedidos.id,
  pedidos.data,
  pedidos.cliente_id,
  pedidos.status,
  produtos.nome AS produto,
  itens_pedido.quantidade AS quantidade
FROM pedidos
INNER JOIN itens_pedido ON itens_pedido.pedido_id = pedidos.id
INNER JOIN produtos ON produtos.id = itens_pedido.produto_id;
```

## Produtos disponíveis
```SQL
CREATE VIEW produtos_disponiveis AS
SELECT
  produtos.id,
  produtos.nome,
  produtos.descricao,
  produtos.preco,
  produtos.quantidade_estoque
FROM produtos
WHERE produtos.quantidade_estoque > 0;
```
