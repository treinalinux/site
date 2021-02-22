## MySQL Database Server Para Estudos

### O aritgo está em desenvolvimento

O objetivo do criação do artigo é ajudar desenvolvedores que precisam de um servidor MySQL para estudos.

No caso do exemplo estamos com um servidor CentOS8, e todas as operações de uso serão feitas remotamente pelo desenvolvedor.

**Não realize tais configurações em um banco de dados de produção**

Parte do artigo pode ser feita em produção, mas tal configuração está longe de ter boas práticas em um servidor de banco de dados. Novamente, o uso é somente para estudos.

## Instale MySQL 8

```
dnf module -y install mysql:8.0
```

### Permitindo a conexão remota

Editar o arquivo /etc/my.cnf.d/mysql-server.cnf permitir a conexão remota:

```
vim /etc/my.cnf.d/mysql-server.cnf

# Adicionar a linha abaixo, que deve conter o IP do servidor de banco de dados que receberá a conexão.
bind-address=192.168.0.201

```

## Configurando os serviços

```
systemctl restart mysqld.service

systemctl enable mysqld.service

systemctl start mysqld.service 
```

## Configurando o firewall

Tanto faz configurar pela porta ou pelo nome do serviço, mas em alguns casos pode ser melhor pelo nome do serviço.

```
firewall-cmd --add-port=3306/tcp --permanent

firewall-cmd --add-service=mysql --permanent 

firewall-cmd --reload
```

### Checando se de fato a porta está aberta

```
netstat -natp | grep 3306
```

## Criando os usuários

A criação dos usuário da forma que está sendo feita vai permitir a conexão do usuário criado a partir de qualque computador com rota para tal servidor de banco de dados.

```
mysql -u root -p
```

```
CREATE USER 'carlaalves'@'%' IDENTIFIED BY '1234@Carla';
```

```
CREATE USER 'alan'@'%' IDENTIFIED BY '1234@Alan';
```


##  Concedendo permissões totais no MySQL

Agora vamos conceder totais privilégios em qualquer banco de dados e de qualquer computador  com rota para tal servidor.

```
GRANT ALL PRIVILEGES ON *.* TO 'alan'@'%';

GRANT ALL PRIVILEGES ON *.* TO 'carlaalves'@'%';
```

### Recarregar os PRIVILEGES

Recarregar os PRIVILEGES é um passo importante para carregar a nova configuração.

```
FLUSH PRIVILEGES;
```

### Consulte as permissões

Aqui vamos checar quais permissão temos no servidor.

```
SHOW GRANTS FOR 'carlaalves'@'%';

SHOW GRANTS FOR 'alan'@'%';
```



## Testar

Hora de testar a conexão e se os usuários criados tem permissão de fato no banco de dados.

O teste abaixo vai testar com conexão com o banco de dados através do IP de exemplo 192.168.0.201 pela usuária carlaalves.


```
mysql -u carlaalves -h 192.168.0.201 -p

create database test_databases;
```

Caso tenha algum erro, favor rever os passos informados.

Basicamente montei tal ambiente de estudos para minha esposa Carla, de forma a facilitar a conexão pelo MySQL Workbench. Como o objetivo dela é desenvolver aplicação que vão se conectar no banco de dados e fazer CRUD para inserir, atualizar e deletar informações, ela não precisa perder tem a infraestrutura.

Outro ponto importante é que o SELinux está desabilidado.

O ambiente também poderia ser feito via Docker, mas estamos com um servidor em uma máquina virtual.

Espero que ajude outros estudantes.

Att, Alan Alves.

