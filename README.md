# ExampleSQL
Examples I'm seeing in class

### Criação de tabelas

<code>create table pessoas (<br>
  codigo integer not null,<br>
  fisicajuridica char(1) not null,<br>
  nome varchar(50) not null,<br>
  cidade integer not null,<br>
  saldo numeric(7,2),<br>
  data date default current_date,<br>
  nota integer,<br>
  primary key (codigo),<br>
  foreign key (cidade) references cidades(codigo),<br>
  foreign key (nota) references notas (numero) on delete cascade,<br>
  check (fisicajuridica in ('F', 'J')) --fisicajuridica vai aceitar apenas F ou J<br>
);</code><br>
