   ![comercio](https://github.com/AndreFelipefer/COMERCIO_ELETRONICO/assets/129207232/3b9c9345-c416-4f29-a46c-cc91ab04e8a1) 
### O sistema envolve várias tabelas, incluindo "Produtos" para armazenar informações sobre produtos, "Pedidos" para registrar detalhes de pedidos, "Clientes" para manter informações de clientes e "Itens de Pedido" para registrar produtos em cada pedido. Além disso, foram estabelecidos relacionamentos essenciais entre "Pedidos" e "Clientes" para rastrear os pedidos de cada cliente, e entre "Itens de Pedido" e "Produtos" para associar produtos aos pedidos. Este sistema é projetado para facilitar a gestão e o rastreamento de pedidos em um ambiente de comércio eletrônico.

# Modelo Lógico:
![image](https://github.com/AndreFelipefer/COMERCIO_ELETRONICO/assets/129207232/17aefaf2-0ba1-4ad4-95ef-382956d198f4)


## Implemente uma stored procedure para permitir que os clientes adicionem produtos ao carrinho de compras.
```SQL
delimiter $
create procedure ListaPedidos(
pedido varchar(100),
quantidadePedido int
)
begin 
	insert into itens_de_pedido(pedido,quantidadePedido,Produtos_idProdutos) values (pedido,quantidadePedido,Produtos_idProdutos);
end$
delimiter ;
```
## Insert Conforme Procedure
```SQL
call ListaPedidos ('Caixa de Som', 3);
call ListaPedidos ('Mouse', 2);
call ListaPedidos ('Teclado', 1);
call ListaPedidos ('Headphones', 5);
call ListaPedidos ('Notebook', 1);
call ListaPedidos ('Smartphone', 2);
call ListaPedidos ('Tablet', 1);
call ListaPedidos ('TV', 5);
call ListaPedidos ('Geladeira', 1);
call ListaPedidos ('Fogão', 2);
call ListaPedidos ('Lava-louças', 1);
call ListaPedidos ('Máquina de lavar', 5);
call ListaPedidos ('Micro-ondas', 1);
call ListaPedidos ('Ar-condicionado', 2);
call ListaPedidos ('Lavadora de roupas', 1);
call ListaPedidos ('Secadora de roupas', 5);
call ListaPedidos ('Forno elétrico', 1);
```
## Crie uma stored procedure para processar pedidos, atualizando o estoque de produtos e registrando os detalhes do pedido.

```SQL
DELIMITER $

CREATE PROCEDURE ProcessarPedido(
    IN pClienteNome VARCHAR(100),
    IN pProdutoNome VARCHAR(100),
    IN pQuantidade INT
)
BEGIN
    DECLARE clienteId INT;
    DECLARE produtoId INT;
    DECLARE estoqueAtual INT;
    
    -- Obtém o ID do cliente
    SELECT idClientes INTO clienteId FROM Clientes WHERE nomeCliente = pClienteNome;
    
    -- Obtém o ID do produto
    SELECT idProdutos INTO produtoId FROM Produtos WHERE nome = pProdutoNome;
    
    -- Obtém o estoque atual do produto
    SELECT quantidadeEstoque INTO estoqueAtual FROM Produtos WHERE idProdutos = produtoId;
    
    -- Verifica se o estoque é suficiente
    IF estoqueAtual >= pQuantidade THEN
        -- Atualiza o estoque do produto
        UPDATE Produtos SET quantidadeEstoque = estoqueAtual - pQuantidade WHERE idProdutos = produtoId;
    
        -- Registra o pedido
        INSERT INTO Pedidos (data, clientePedido, status)
        VALUES (CURDATE(), pClienteNome, 'Enviado');  -- Status 'Enviado'
    
        -- Obtém o ID do último pedido inserido
        SET @pedidoId = LAST_INSERT_ID();
    
        -- Registra os detalhes do pedido
        INSERT INTO Itens_de_Pedido (pedido, quantidadePedido, Produtos_idProdutos)
        VALUES (@pedidoId, pQuantidade, produtoId);
    ELSE
        -- Caso o estoque não seja suficiente
        -- Você pode lidar com isso de acordo com sua lógica de negócios
        -- Pode lançar um erro, retornar uma mensagem, etc.
        -- Neste exemplo, apenas mostraremos uma mensagem de estoque insuficiente
        SELECT 'Estoque insuficiente para o produto selecionado.' AS Resultado;
    END IF;
    
END $

DELIMITER ;
```
## Chamando o Procedure para atualizar sistema:
```SQL
CALL ProcessarPedido('Antonio', 'Caixa de Som', 3);
```
