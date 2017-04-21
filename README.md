## Criação de tabelas
```
create table pessoas (
  codigo integer not null,
  fisicajuridica char(1) not null,
  nome varchar(50) not null,
  cidade integer not null,
  saldo numeric(7,2),
  data date default current_date,
  nota integer,
  primary key (codigo),
  foreign key (cidade) references cidades(codigo),
  foreign key (nota) references notas (numero) on delete cascade,
  check (fisicajuridica in ('F', 'J')) --fisicajuridica vai aceitar apenas F ou J
);
```


## Inserção de valores nas tabelas
```
insert into pessoas (codigo, fisicajuridica, nome, cidade, saldo, nota)
       values (01, 'F', 'Fabricio Bedin', '05', '23856.25', '2545873221');
```

## Alguns usos do SELECT

* Elabore uma lista de preços de produtos, consistindo de: código, descrição, localização e preço do produto. A lista deve ser ordenada pela descrição do produto.
```
select codigo, descricao, localizacao, preco</code><br>
from produtos
order by descricao;
```

* Listar a descrição, localização dos produtos que estão com o estoque atual abaixo do valor mínimo estabelecido
```
select descricao, localizacao
from produtos
where estoqueatual < estoqueminimo; 
```

* Listar o nome dos fornecedores do produto de código 2
```
select pessoas.nome
from pessoas, produtosfornecedores
where pessoas.codigo = produtosfornecedores.fornecedor
and produtosfornecedores.produto = 2;
```

* Listar o nome, telefone, cidade dos clientes que possuem parcelas ainda não pagas (com a data de pagamento vazia) juntamente com a data de vencimento e o valor destas parcelas
```
select pessoas.nome,
       pessoas.telefone,
       cidades.cidade,
       cidades.uf,
       parcelas.vencimento,
       parcelas.valor
from pessoas, parcelas, notas, cidades
where pessoas.cidade = cidades.codigo
and   pessoas.codigo = notas.pessoa
and notas.numero = parcelas.nota
and parcelas.pagamento is null;
```
* Listar os itens da nota de número _ _ _ _ _ com as seguintes informações: Código do produto, descrição do produto, quantidade, preço unitário e valor total do item (este calculado de preço unitário x quantidade)
```
select produtos.codigo, 
       produtos.descricao, 
       itensnotas.quantidade,
       itensnotas.precounitario,
       itensnotas.quantidade * itensnotas.precounitario as total
from produtos, itensnotas
where produtos.codigo = itensnotas.produto
and itensnotas.nota = 1;
```

* Qual o valor total da nota anterior ?
```
select sum(itensnotas.quantidade * itensnotas.precounitario) as total
from itensnotas
where itensnotas.nota = 1
```

* Selecionar o nome e telefone de todas as pessoas que possuam a palavra ‘Fab’ em qualquer parte do nome
```
select nome, telefone
from pessoas
where upper(nome) like '%FAB%';
```

* Obter a descrição de cada categoria e a contagem de produtos distintos que cada uma possui
```
select categorias.categoria, count(*)
from categorias, produtos
where produtos.categoria = categorias.codigo
group by categorias.categoria;
```

## Atualizar valores

* Atualizar o endereço, telefone, código da cidade, cep e email de uma pessoa já existente no banco de dados.
```
update pessoas
set endereco = 'Rua XYZ',
    telefone = '11111111',
    cidade = '1',
    cep = '99999-999',
    email = 'seila@com.com'
where codigo = 1;
```

* Atualizar o preço dos produtos da categoria de código de sua escolha, reajustandoos em 20%
```
update produtos
set preco = preco * 1.2
where categoria = 9;
```

* Atualizar o valor de todas as parcelas vencidas há mais de 30 dias em 5%
```
update parcelas
set valor = valor * 1.05
where vencimento < current_date - 30
and pagamento is null;
```

## Remover Tabelas e Valores

* Excluir uma tabela
```
drop table pessoas;
```

* Excluir todas a parcelas pagas do cliente de código de sua escolha.
```
delete from parcelas
where pagamento is not null
and nota in ( select numero from notas
              where pessoa = 1);
```

## Alterar estrutura da tabela

* Alterar a estrutura da tabela pessoas, acrescentando as colunas pessoaContato e website, ambas do tipo caracteres.
```
alter table pessoas
add pessoacontato varchar(40),
add website varchar(100);
```

* Acrescentar na tabela de categorias uma restrição de forma que não sejam permitidas duplicidades na descrição
```
alter table categorias
add unique (categoria);
```

## Visões

* Crie uma visão contendo o nome do projeto, sua duração e o código do departamento responsável.
```
create view view1 as
select projetos.nome,
projetos.duracao,
departamentos.codigo
from projetos, departamentos, funcionarios, funcionariosprojetos
where funcionarios.codigodepartamento = departamentos.codigo
and funcionarios.codigo = funcionariosprojetos.codigofuncionario
and funcionariosprojetos.codigoprojeto = projetos.codigo;
```
* Crie uma visão contendo o código e nome do empregado e o nome de seus dependentes
```
create view view2 as
select funcionarios.codigo,
funcionarios.nome as funcionario,
dependentes.nome as dependente
from funcionarios, dependentes
where funcionarios.codigo = dependentes.codigofuncionario;
```

* Crie uma visão sobre a visão 2 que contenha o nome do empregado e o número de dependentes que ele possui.
```
create view view3 as
select view2.funcionario, count(*)
from view2
group by view2.funcionario;
```

* Crie uma visão contendo o nome do empregado e o número total de horas que ele trabalhou em projetos
```
create view view4 as
select funcionarios.nome,
sum(funcionariosprojetos.horasalocadas) as horas
from funcionarios, funcionariosprojetos
where funcionarios.codigo = funcionariosprojetos.codigofuncionario
group by funcionarios.nome;
```

* Crie uma visão que mostre o nome dos departamento e o nome dos projetos pelos quais eles são responsáveis (utilize a visão do exercício 1).
```
create view view5 as
select departamentos.departamento,
view1.nome as projeto
from departamentos, view1
where departamentos.codigo = view1.codigo;
```




## Procedures


```
create table triangulos (
id integer not null primary key,
vlr1 numeric (5),
vlr2 numeric (5),
vlr3 numeric (5),
tipo varchar(20)
);

create sequence seq_triangulos;

CREATE OR REPLACE PROCEDURE inseretriangulo (a IN NUMBER, b IN NUMBER, c IN NUMBER) IS
  triangulo_invalido EXCEPTION;
  tipo varchar(20);
BEGIN
    -- testa condicao de existencia do triangulo
    if ((a+b<c) or (a+c<b) or (b+c<a)) then
        raise triangulo_invalido;
    end if;

    -- classifica o triangulo
    if ( a = b and a = c) then
        tipo :=  'Equilatero';
    elsif (a <> b and a<>c and b<>c) then
        tipo := 'Escaleno';
    else 
        tipo := 'Isosceles';
    end if;
    insert into triangulos values (seq_triangulos.nextval, a, b, c, tipo);
    EXCEPTION
      WHEN triangulo_invalido THEN
        raise_application_error(-20001, 'Valores nao formam triangulo');
END inseretriangulo; 
/

show errors

execute inseretriangulo(10,10,10);
execute inseretriangulo(10,10,20);
execute inseretriangulo(10,20,30);
-- erro
execute inseretriangulo(10,20,50);

select * from triangulos;
```


```
```


```
```



