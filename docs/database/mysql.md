## MySQL Database Server Para Estudos

### O aritgo está em desenvolvimento

O objetivo do criação do artigo é ajudar desenvolvedores que precisam de um servidor MySQL para estudos.

No caso do exemplo estamos com um servidor CentOS8, e todas as operações serão feitas remotamente.

**Não realize tais configurações para um banco de produção**

Parte do artigo pode ser feita em produção, tal configuração está longe de ter boas práticas.

Novamente, uso para estudos somente.

## Instale MySQL 8

```
dnf module -y install mysql:8.0
```

### Permitindo a conexão remota

Editar o arquivo /etc/my.cnf.d/mysql-server.cnf permitir a conexão remota:

```
vim /etc/my.cnf.d/mysql-server.cnf

# Adicionar a linha abaixo, que deve conter o IP do servidor de banco de dados
bind-address=192.168.0.201

```

## Configurando os serviços

```
systemctl restart mysqld.service

systemctl enable mysqld.service

systemctl start mysqld.service 
```

## Configurando o firewall

```
# Tanto faz confirar pela por ou pelo nome do serviço, mas em alguns casos pode ser melhor pelo nome do seriviço

firewall-cmd --add-port=3306/tcp --permanent

firewall-cmd --add-service=mysql --permanent 

firewall-cmd --reload
```

### Checando se de fato a porta está aberta

```
netstat -natp | grep 3306
```

## Criando os usuários

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



```
GRANT ALL PRIVILEGES ON *.* TO 'alan'@'%';

GRANT ALL PRIVILEGES ON *.* TO 'carlaalves'@'%';
```

### Recarregar os PRIVILEGES

```
FLUSH PRIVILEGES;
```

### Consulte as permissões

```
SHOW GRANTS FOR 'carlaalves'@'%';

SHOW GRANTS FOR 'alan'@'%';
```



## Testar

Hora de testar a conexão e se os usuários criados tem permissão de fato no banco de dados.

```
mysql -u carlaalves -h 192.168.0.201 -p

create database test_databases;
```
