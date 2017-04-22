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
## Subconsultas e Agrupamentos / Conjuntos e Agregação
* Obter as cidades que não possuem clientes relacionados
```
select codigo, cidade, uf
from cidades
EXCEPT
select cidades.codigo, cidades.cidade, cidades.uf
from cidades, pessoas
where cidades.codigo = pessoas.cidade
```
ou
```
select cidades.codigo, cidades.cidade, cidades.uf
from cidades
where cidades.codigo NOT IN ( select cidade from pessoas);
```

* Obter produtos que não possuem vendas
```
select codigo, descricao
from produtos
EXCEPT
select itensnotas.produto, produtos.descricao
from itensnotas, produtos, notas
where itensnotas.produto = produtos.codigo
and   notas.operacao = 1;
```

* Obter o número de itens que a nota de número 1 possui
```
select count(*)
from itensnotas
where nota = 1;
```

* Obter o preço médio dos produtos da categoria 9
```
select avg(preco)
from produtos
where categoria = 9;
```

* Obter a descrição do produto mais caro
```
select produtos.descricao
from produtos
where preco = ( select max(preco)
		            from produtos);
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
## Sequences
```
create sequence seq_gera1 start 100;
create sequence seq_gera2 increment by 2 maxvalue 9 cycle nocache;

select nextval('seq_gera1')||'-'||nextval('seq_gera2')
from dual;

create table teste (
codigo varchar(5) primary key,
descricao varchar(30)
);

insert into teste values (seq_gera1.nextval||'-'||seq_gera2.nextval, 'alguma coisa');
```


## Procedures

* Procedure PAR ou IMPAR
```
create or replace procedure parimpar (valor in number) as
 tipo VARCHAR2(5);
BEGIN
    IF MOD(valor,2) = 0 THEN
      tipo := 'PAR';
    ELSE
      tipo := 'IMPAR';
    END IF;
    DBMS_OUTPUT.PUT_LINE ('O numero  ' || valor || ' e´ ' || tipo);
END;
/
```

* Procedure PAR
```
create or replace procedure par(valor integer) as
 erro EXCEPTION;
BEGIN
    IF MOD(valor,2) = 0 THEN
      DBMS_OUTPUT.PUT_LINE ('Numero par recebido');
    ELSE
      raise erro;
    END IF;

    EXCEPTION
      WHEN erro THEN
        raise_application_error( -20001, 'Erro! Numero impar recebido');
END;
/
```

* Elabore um procedimento armazenado inseretriangulo(vlr1, vlr2, vlr3) que recebe valores para os lados de um triângulo e a seguir insere na tabela abaixo os valores dos lados dos triangulos e o tipo do triângulo formado (equilátero, isósceles, escaleno).
*Triangulos 
id integer
vlr1 number
vlr2 number
vlr3 number
tipo varchar
Caso os valores fornecidos para os lados não permitam a formação de um triângulo, o usuário deverá receber uma mensagem indicativa.*

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
### Procedure Banco
* O esquema abaixo representa de forma parcial uma agência bancária. A respeito deste esquema, desenvolva os seguintes procedures:
*Correntista = #Código, nome<br>
Contas = #Número, saldo, @Código do correntista<br>
Movimentaçao=#Número, data, valor, natureza (débito/crédito), @numero da conta*<br>

*Observações:
a cada operação realizada (saque, depósito, etc) um registro de movimentação deve ser inserido automaticamente pelas funções na tabela de Movimentação, descrevendo a operação se houve débito ou crédito
não poderá haver saque de valor superior ao disponível no saldo da conta*

```
create sequence seq_correntistas start with 100 increment by 1;
create sequence seq_contas start with 1000 increment by 1;
create sequence seq_movimentacoes;

create table correntistas (
codigo integer primary key,
nome varchar(50)
);

create table contas (
numero integer primary key,
saldo numeric(8,2),
correntista integer,
foreign key (correntista) references correntistas(codigo)
);

create table movimentacoes (
numero integer primary key,
data date,
valor numeric (7,2),
natureza char(1),
conta integer,
constraint ck_movimentacoes check (natureza in ('D','C')),
constraint fk_movimentacoes foreign key (conta) references contas(numero)
);
```


* Insere um novo cliente e abre uma conta associada ao mesmo
*a) abreConta(nome)*
```
CREATE or replace procedure abreconta(varnome in string) as
begin
	insert into correntistas(codigo, nome) values (seq_correntistas.nextval, varnome);
	insert into contas(numero, correntista, saldo) values (seq_contas.nextval, seq_correntistas.currval, 0);
end;
/
```
```
exec abreconta('Jose');
exec abreconta('Maria');
select * from correntistas;
select * from contas;
```

* Deposita valor em uma conta
*b) deposita(Numero conta, valor)*
```
CREATE or replace procedure deposita(vdestino in integer, vvalor in real) as
begin
	insert into movimentacoes (numero, data, valor, natureza, conta) 
  values (seq_movimentacoes.nextval, current_date, vvalor, 'C', vdestino);
	
	update contas
	set saldo = saldo + vvalor
	where numero = vdestino;
end;
/
```
```
exec deposita (1001, 500.75);
exec deposita (1002, 2355.85);
exec deposita (1001, 100.00);
select * from movimentacoes;
select * from contas;
```

* Saca valor de uma conta
*c) saca(Numero conta, valor)*
```
CREATE or replace procedure saca(vorigem in integer, vvalor in real) as
vsaldo real;
saldo_insuficiente EXCEPTION;
begin
  -- obtem saldo da conta a sacar
	select saldo into vsaldo
  from contas 
  where numero = vorigem;

  -- testa se saque permitido
	if (vsaldo < vvalor) then
    raise saldo_insuficiente;   
	end if;

  -- insere historico da movimentacao
	insert into movimentacoes (numero, data, valor, natureza, conta) 
  values (seq_movimentacoes.nextval, current_date, vvalor, 'D', vorigem);

  -- atualiza o saldo da conta
	update contas
	set saldo = saldo - vvalor
	where numero = vorigem;
  
  EXCEPTION
    WHEN saldo_insuficiente THEN
      raise_application_error(-20001, 'Saldo insuficiente para realizar saque');
    WHEN no_data_found THEN
      raise_application_error(-20002, 'Conta nao encontrada');
end;
/
```
```
exec saca (1001, 5000);
exec saca (1002, 310);
```

* Transfere valor de uma conta de origem para uma conta de destino
*d) transfere(Conta origem, conta destino, valor)*
```
CREATE or replace procedure transfere(vorigem in int,vdestino in int, vvalor in real) as
begin
	saca(vorigem, vvalor);
	deposita(vdestino, vvalor);
end;
/
```
```
exec transfere(1001,1002, 100);
exec transfere(1002,1001, 200);

```

