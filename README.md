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
