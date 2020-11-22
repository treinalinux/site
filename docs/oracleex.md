# Instalação do Oracle Database 18c Express Edition (XE) no CentOS 7

## Em construção.

Seja você um desenvolvedor, DBA, cientista de dados, educador ou apenas curioso sobre bancos de dados, o Oracle Database 18c Express Edition (XE) é a maneira ideal de começar. 

O mesmo Oracle Database avançado em que empresas de todo o mundo confiam, formatado para download simples, facilidade de uso e experiência completa. 

Você terá um Oracle Database para uso em qualquer ambiente e será capaz de incorporar e redistribuir, tudo completamente grátis!

**Então, vamos aprender como instalar o Oracle Database 18c Express Edition (XE) no Linux CentOS 7**

*Vamos começar...*


**1. Logar como super usuário:**

```bash
  sudo -s
```

**2. Configurar o arquivo hosts seu IP e com o nome do host (não usar o FQDN):**

```bash 
# Exemplo:
# IP          nome_somente
192.168.0.90  serverdb
```

**3. Atualizar e instalar pacotes necessários como pré-requesito:**

```bash
  yum update
```

```bash
  yum -y install binutils compat-libcap1 gcc gcc-c++ glibc glibc.i686 glibc-devel glibc.i686 ksh libaio libaio.i686 libaio-devel libaio-devel.i686 libgcc libgcc.i686 libstdc++ libstdc++l7.i686 libstdc++-devel libstdc++-devel.i686 compat-libstdc++-33 compat-libstdc++-33.i686 libXi libXi.i686 libXtst libXtst.i686 make sysstat
```


**4. Desabilitar o SELinux:**

```bash

  sestatus
```

```bash
  reboot 
```

**5. Instalação do Oracle Database 18c Express Edition (XE):**

```bash  
  curl -o oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm
```

```bash
  yum -y localinstall oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm 
```

```bash
  yum -y localinstall oracle-database-xe-18c-1.0-1.x86_64.rpm 
```


**6. Configuração inicial do banco de dados:**

```bash
  /etc/init.d/oracle-xe-18c configure
```
  
  **Resultado esperado deve ser igual o resultado abaixo para conectar no servidor Oracle:**
  
>  Criação do banco de dados concluída. Para obter detalhes, verifique os arquivos de log em:
 /opt/oracle/cfgtoollogs/dbca/XE.
Informações sobre o Banco de Dados:
Nome do Banco de Dados Global:XE
Identificador de Sistema (SID):XE
>
>Verifique o arquivo de log "/opt/oracle/cfgtoollogs/dbca/XE/XE.log" para obter mais detalhes.
>
>Connect to Oracle Database using one of the connect strings:
>     Pluggable database: oracledb.empresa.corp/XEPDB1
>     Multitenant container database: oracledb.empresa.corp
>Use https://localhost:5500/em to access Oracle Enterprise Manager for Oracle Database XE


**7. Instalando slqdeveloper e demais pacotes:**

```bash
  yum -y localinstall sqldeveloper-20.2.0.175.1842-20.2.0-175.1842.noarch.rpm jdk-8u271-linux-x64.rpm
```

```bash
 yum update 
```

```bash
  alternatives --config java
```

**8. Liberando as portas do firewall:**

```bash
  firewall-cmd --add-port=5500/tcp --permanent 
```

```bash
  firewall-cmd --add-port=1521/tcp --permanent 
```

```bash
  firewall-cmd --reload
```

```bash
  firewall-cmd --list-all
```


**9. Reiniciando e testando o banco de dados:**

```bash
  reboot 
```

```bash
  /etc/init.d/oracle-xe-18c start
```


**10. Adicionar o Oracle no path do **usuário oracle** e do **root** somente, pode usar o .bash_profile mesmo:**

```bash
umask 022
export ORACLE_SID=XE
export ORACLE_BASE=/opt/oracle/oradata
export ORACLE_HOME=/opt/oracle/product/18c/dbhomeXE
export PATH=$PATH:$ORACLE_HOME/bin
```


**Não esqueça de carreguar o configuração do arquivo com o comando source**


**11. Teste a conexão com sqlplus:**

```bash
	[root@oracledb ~]# sqlplus

	SQL*Plus: Release 18.0.0.0.0 - Production on Sat Nov 21 07:04:18 2020
	Version 18.4.0.0.0

	Copyright (c) 1982, 2018, Oracle.  All rights reserved.

	Enter user-name: system
	Enter password: 
	Horario do ultimo log-in bem-sucedido: Sab Nov 21 2020 07:03:22 -03:00

	Conectado a:
	Oracle Database 18c Express Edition Release 18.0.0.0.0 - Production
	Version 18.4.0.0.0

	SQL> select username from dba_users;
	...
	...
	...
	SQL> exit;
``` 


**Fontes:**
*oracle.com*
