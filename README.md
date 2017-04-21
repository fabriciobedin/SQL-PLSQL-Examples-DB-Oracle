# ExampleSQL
Examples I'm seeing in class

### Criação de tabelas

<code>create table pessoas (</code><br>
<code>  codigo integer not null,</code><br>
<code>  fisicajuridica char(1) not null,</code><br>
<code>  nome varchar(50) not null,</code><br>
<code>  cidade integer not null,</code><br>
<code>  saldo numeric(7,2),</code><br>
<code>  data date default current_date,</code><br>
<code>  nota integer,</code><br>
<code>  primary key (codigo),</code><br>
<code>  foreign key (cidade) references cidades(codigo),</code><br>
<code>  foreign key (nota) references notas (numero) on delete cascade,</code><br>
<code>  check (fisicajuridica in ('F', 'J'))</code> --fisicajuridica vai aceitar apenas F ou J<br>
<code>);</code><br>

### Inserção de valores nas tabelas
<code>insert into pessoas (codigo, fisicajuridica, nome, cidade, saldo, nota)</code><br>
<code>       values (01, 'F', 'Fabricio Bedin', '05', '23856.25', '2545873221');</code><br>

### Alguns usos do SELECT

##### •Elabore uma lista de preços de produtos, consistindo de: código, descrição, localização e preço do produto. A lista deve ser ordenada pela descrição do produto.
<code>select codigo, descricao, localizacao, preco</code><br>
<code>from produtos</code><br>
<code>order by descricao;</code><br>

##### •Listar a descrição, localização dos produtos que estão com o estoque atual abaixo do valor mínimo estabelecido
<code>select descricao, localizacao</code><br>
<code>from produtos</code><br>
<code>where estoqueatual < estoqueminimo; </code><br>

##### •Listar o nome dos fornecedores do produto de código 2
<code>select pessoas.nome</code><br>
<code>from pessoas, produtosfornecedores</code><br>
<code>where pessoas.codigo = produtosfornecedores.fornecedor</code><br>
<code>and produtosfornecedores.produto = 2;</code><br>

##### •Listar o nome, telefone, cidade dos clientes que possuem parcelas ainda não pagas (com a data de pagamento vazia) juntamente com a data de vencimento e o valor destas parcelas
<code>select pessoas.nome,</code><br>
<code>       pessoas.telefone,</code><br>
<code>       cidades.cidade,</code><br>
<code>       cidades.uf,</code><br>
<code>       parcelas.vencimento,</code><br>
<code>       parcelas.valor</code><br>
<code>from pessoas, parcelas, notas, cidades</code><br>
<code>where pessoas.cidade = cidades.codigo</code><br>
<code>and   pessoas.codigo = notas.pessoa</code><br>
<code>and notas.numero = parcelas.nota</code><br>
<code>and parcelas.pagamento is null;</code><br>


```
select *
from parcelas
where pagamento is null
and vencimento between '2011-01-01' 
               and '2011-03-01';

select *
from parcelas
where pagamento is null
and vencimento >= '2011-01-01' 
and vencimento <= '2011-03-01';
```








