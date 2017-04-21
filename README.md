# ExampleSQL
Examples I'm seeing in class

### Criação de tabelas
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


### Inserção de valores nas tabelas
```
insert into pessoas (codigo, fisicajuridica, nome, cidade, saldo, nota)
       values (01, 'F', 'Fabricio Bedin', '05', '23856.25', '2545873221');
```

### Alguns usos do SELECT

##### •Elabore uma lista de preços de produtos, consistindo de: código, descrição, localização e preço do produto. A lista deve ser ordenada pela descrição do produto.
```
select codigo, descricao, localizacao, preco</code><br>
from produtos
order by descricao;
```

##### •Listar a descrição, localização dos produtos que estão com o estoque atual abaixo do valor mínimo estabelecido
```
select descricao, localizacao
from produtos
where estoqueatual < estoqueminimo; 
```

##### •Listar o nome dos fornecedores do produto de código 2
```
select pessoas.nome
from pessoas, produtosfornecedores
where pessoas.codigo = produtosfornecedores.fornecedor
and produtosfornecedores.produto = 2;
```

##### •Listar o nome, telefone, cidade dos clientes que possuem parcelas ainda não pagas (com a data de pagamento vazia) juntamente com a data de vencimento e o valor destas parcelas
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








