## Kerberos

### Vamos usar uma versão com base no Debian, mas o conceito é o mesmo para Red Hat.


Kerberos é um sistema de autenticação de rede baseado no princípio de um terceiro confiável. As outras duas partes são o usuário e o serviço no qual o usuário deseja se autenticar. Nem todos os serviços e aplicativos podem usar Kerberos, mas para aqueles que podem, ele traz o ambiente de rede um passo mais perto de ser `Single Sign On (SSO)`.

Esta seção cobre a instalação e configuração de um servidor Kerberos e alguns exemplos de configurações de cliente.


## Visão geral

Se você for novo no Kerberos, há alguns termos que devem ser entendidos antes de configurar um servidor Kerberos. A maioria dos termos está relacionada a coisas com as quais você pode estar familiarizado em outros ambientes:

**Principal**: quaisquer usuários, computadores e serviços fornecidos por servidores precisam ser definidos como Principals Kerberos.

**Instances**: são usadas para entidades de serviço e entidades administrativas especiais.

**Realms**: O domínio exclusivo de controle fornecido pela instalação do Kerberos. Pense nele como o domínio ou grupo ao qual seus hosts e usuários pertencem. A convenção determina que o domínio deve estar em maiúsculas. Por padrão, o perola usará o domínio DNS convertido em maiúsculas (`TREINALINUX.COM`) como o realm.

**Key Distribution Center**: (`KDC`) consiste em três partes: um banco de dados de todos os principais, o servidor de autenticação e o servidor de concessão de tíquetes. Para cada realm, deve haver pelo menos um KDC.

**Ticket Granting Ticket**: emitido pelo Authentication Server (AS), o Ticket Granting Ticket (`TGT`) é criptografado na senha do usuário que é conhecida apenas pelo usuário e pelo KDC.

**Ticket Granting Server** : (`TGS`) emite tíquetes de serviço para clientes mediante solicitação.

**Tickets**: confirme a identidade dos dois principais. Um principal sendo um usuário e o outro um serviço solicitado pelo usuário. Os tickets estabelecem uma chave de criptografia usada para comunicação segura durante a sessão autenticada.

**Arquivos Keytab**: são arquivos extraídos do banco de dados principal KDC e contêm a chave de criptografia para um serviço ou host.

Para colocar as peças juntas, um Realm tem pelo menos um KDC, de preferência mais para redundância, que contém um banco de dados de Principals. Quando um usuário principal efetua login em uma estação de trabalho configurada para autenticação Kerberos, o KDC emite um Ticket Granting Ticket (TGT). 

Se as credenciais fornecidas pelo usuário corresponderem, o usuário será autenticado e poderá solicitar tíquetes para serviços Kerberizados do Servidor de concessão de tíquetes (TGS). 

Os tíquetes de serviço permitem que o usuário se autentique no serviço sem inserir outro nome de usuário e senha.


## Servidor Kerberos


### Instalação


Para esta discussão, criaremos um domínio MIT Kerberos com os seguintes recursos (edite-os para atender às suas necessidades):

```bash
Realm: TREINALINUX.COM

KDC primário: kdc01.treinalinux.com ( 192.168.0.241)

KDC secundário: kdc02.treinalinux.com ( 192.168.0.242)

Principal do usuário: perola

Administrador principal: perola/admin

```


> Nota
> 
> É altamente recomendável que seus usuários autenticados pela rede tenham seu uid em um intervalo diferente (digamos, começando em 5000) do que seus usuários locais.


Antes de instalar o servidor Kerberos, é necessário um servidor DNS configurado corretamente para o seu domínio. Como o domínio Kerberos por convenção corresponde ao nome de domínio, esta seção usa o `TREINALINUX.COM` domínio configurado na seção Servidor primário da documentação do DNS.

Além disso, o Kerberos é um protocolo sensível ao tempo. Portanto, se o horário do sistema local entre uma máquina cliente e o servidor for diferente em mais de cinco minutos (por padrão), a estação de trabalho não será capaz de se autenticar. Para corrigir o problema, todos os hosts devem ter sua hora sincronizada usando o mesmo servidor `Network Time Protocol (NTP)`. 


A primeira etapa na criação de um realm Kerberos é instalar os pacotes `krb5-kdc` e `krb5-admin-server`. De um terminal, digite:

```bash
sudo apt install krb5-kdc krb5-admin-server
```

No final da instalação, você será solicitado a fornecer o nome do host para os servidores Kerberos e Admin, que podem ou não ser o mesmo servidor, para o realm. Como vamos criar o realm e, portanto, esses servidores, digite o nome completo do host deste servidor.


> Nota
> 
> Por padrão, o domínio é criado a partir do nome de domínio do KDC.


Em seguida, crie o novo domínio com o kdb5_newrealmutilitário:

```bash
sudo krb5_newrealm
```

Ele solicitará uma senha mestra do banco de dados, que é usada para criptografar o banco de dados local. Escolha uma senha segura: sua força não é verificada para você.


## Configuração


As perguntas feitas durante a instalação são usadas para configurar os arquivos `/etc/krb5.conf` e `/etc/krb5kdc/kdc.conf`. O primeiro é usado pelas bibliotecas kerberos 5, e o último configura o KDC. Se você precisar ajustar as configurações do Key Distribution Center (KDC), simplesmente edite o arquivo e reinicie o daemon krb5-kdc. Se você precisar reconfigurar o Kerberos do zero, talvez para alterar o nome do realm, você pode fazer isso digitando

```bash
sudo dpkg-reconfigure krb5-kdc
````

> Nota
> 
> A página de manual de `krb5.conf` está no pacote `krb5-doc`.

Assim que o KDC estiver funcionando corretamente, um usuário administrador - o administrador principal - é necessário. É recomendado usar um nome de usuário diferente do seu nome de usuário diário. Usando o `kadmin.local` utilitário em um prompt de terminal, digite:

```bash
$ sudo kadmin.local


Authenticating as principal root/admin@TREINALINUX.COM with password.
kadmin.local: addprinc perola/admin
WARNING: no policy specified for perola/admin@TREINALINUX.COM; defaulting to no policy
Enter password for principal "perola/admin@TREINALINUX.COM": 
Re-enter password for principal "perola/admin@TREINALINUX.COM": 
Principal "perola/admin@TREINALINUX.COM" created.
kadmin.local: quit
```

No exemplo acima, perola é o Principal , */admin* é uma Instância desse principal e @ `TREINALINUX.COM` significa o realm. O Principal “todos os dias” , também conhecido como o principal do usuário , seria `perola@TREINALINUX.COM`, e deveria ter apenas direitos de usuário normais.


> Nota
> 
> Substitua EXAMPLE.COM e perola com seu domínio e nome de usuário de administrador.

Em seguida, o novo usuário administrador precisa ter as permissões apropriadas da `Lista de Controle de Acesso (ACL)`. As permissões são configuradas no arquivo `/etc/krb5kdc/kadm5.acl`:

```bash
perola/admin@TREINALINUX.COM        *
```

Esta entrada concede ao **perola/admin** a habilidade de realizar qualquer operação em todos os principais no realm. Você pode configurar principais com privilégios mais restritivos, o que é conveniente se você precisar de um administrador principal que a equipe júnior possa usar em clientes Kerberos. Consulte a página do manual kadm5.acl para obter detalhes.

> Nota
> 
> O privilégio de extração não está incluído no privilégio curinga; deve ser atribuído explicitamente. seu privilégio permite ao usuário extrair chaves do banco de dados e deve ser manuseado com muito cuidado para evitar a divulgação de chaves importantes como aquelas de `kadmin/*` ou `krbtgt/*` principais. Consulte a página do manual `kadm5.acl` para obter detalhes.

Agora reinicie o *krb5-admin-server* para que a nova ACL tenha efeito:

```bash
sudo systemctl restart krb5-admin-server.service
````

O novo usuário principal pode ser testado usando o utilitário kinit:

```bash
$ kinit perola/admin
Password for perola/admin@TREINALINUX.COM:
````

Depois de inserir a senha, use o utilitário klist para ver informações sobre o **Ticket Granting Ticket (TGT)**:

```bash
$ klist
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: perola/admin@TREINALINUX.COM

Valid starting     Expires            Service principal
09/08/20 20:19:42  09/09/20 06:19:42  krbtgt/TREINALINUX.COM@TREINALINUX.COM
        renew until 09/09/20 20:19:42
```

Onde o nome do arquivo do cache krb5cc_1000é composto pelo prefixo krb5cc_e pelo id do usuário (uid), que neste caso é 1000.

kinit irá inspecionar `/etc/krb5.conf` para descobrir qual KDC entrar em contato e seu endereço. O KDC também pode ser encontrado por meio de pesquisas DNS para registros TXT e SRV especiais. Você pode adicionar esses registros à sua treinalinux.comzona DNS:

```bash
_kerberos._udp.TREINALINUX.COM.     IN SRV 1  0 88  kdc01.treinalinux.com.
_kerberos._tcp.TREINALINUX.COM.     IN SRV 1  0 88  kdc01.treinalinux.com.
_kerberos._udp.TREINALINUX.COM.     IN SRV 10 0 88  kdc02.treinalinux.com. 
_kerberos._tcp.TREINALINUX.COM.     IN SRV 10 0 88  kdc02.treinalinux.com. 
_kerberos-adm._tcp.TREINALINUX.COM. IN SRV 1  0 749 kdc01.treinalinux.com.
_kpasswd._udp.TREINALINUX.COM.      IN SRV 1  0 464 kdc01.treinalinux.com.
```


> Nota
> 
> Substitua `TREINALINUX.COM`, `kdc01` e `kdc02` com seu nome de domínio, KDC primário e KDC secundário.


Uma maneira muito rápida e útil de solucionar o que `kinit` está acontecendo é definir a variável de ambiente `KRB5_TRACE` como um arquivo ou stderr, e ele mostrará informações extras. A saída é bastante detalhada e não será mostrada totalmente aqui:

```bash
$ KRB5_TRACE=/dev/stderr kinit perola/admin
[2898] 1585941845.278578: Getting initial credentials for perola/admin@TREINALINUX.COM
[2898] 1585941845.278580: Sending unauthenticated request
[2898] 1585941845.278581: Sending request (189 bytes) to TREINALINUX.COM
[2898] 1585941845.278582: Resolving hostname kdc01.treinalinux.com
(...)
```

Seu novo Kerberos Realm agora está pronto para autenticar clientes.


## KDC Secundário


Depois de ter um Centro de distribuição de chaves (KDC) em sua rede, é uma boa prática ter um KDC secundário, caso o principal se torne indisponível. Além disso, se você tiver clientes Kerberos que estão em redes diferentes (possivelmente separados por roteadores usando NAT), é aconselhável colocar um KDC secundário em cada uma dessas redes.


> Nota
> 
> O mecanismo de replicação nativo explicado aqui depende de um cronjob e, essencialmente, despeja o banco de dados no primário e o carrega de volta no secundário. Você pode querer dar uma olhada em como usar o backend kldap, que pode usar o mecanismo de replicação OpenLDAP. Isso é explicado mais adiante.


Primeiro, instale os pacotes e, quando perguntado pelos nomes dos servidores Kerberos e Admin, digite o nome do KDC primário:

```bash
sudo apt install krb5-kdc krb5-admin-server
```

Depois de instalar os pacotes, crie os principais de host para ambos os KDCs. Em um prompt de terminal, digite:

```bash
kadmin -q "addprinc -randkey host/kdc01.treinalinux.com"
kadmin -q "addprinc -randkey host/kdc02.treinalinux.com"
```

> Nota
>
> O padrão do comando kadmin é usar um principal como username/admin@TREINALINUX.COM, onde username é seu usuário shell atual. Se você precisar substituir isso, use-p principal-you-want

Certifique-se o principal que você está usando tem o extra de extract-keys privilégio em kdc01 no arquivo /etc/krb5kdc/kadm5.acl. Algo assim:

```bash
perola/admin@TREINALINUX.COM *e
```

Onde “*” significa todos os privilégios (exceto extract-keys) e significa exatamente extract-keys.

Extraia o arquivo keytab :

```bash
kadmin -q "ktadd -norandkey -k keytab.kdc02 host/kdc02.treinalinux.com"
```

Agora deve haver um keytab.kdc02no diretório atual, mova o arquivo para /etc/krb5.keytab:

```bash
sudo mv keytab.kdc02 /etc/krb5.keytab
sudo chown root:root /etc/krb5.keytab
```

> Nota
> 
> Se o caminho para o keytab.kdc02arquivo for diferente, ajuste de acordo.

Além disso, você pode listar os principais em um arquivo Keytab, que pode ser útil na solução de problemas, usando o utilitário klist:

```bash
sudo klist -k /etc/krb5.keytab
```

A opção -k indica que o arquivo é um arquivo keytab.

Em seguida, deve haver um kpropd.aclarquivo em cada KDC que liste todos os KDCs para o Realm. Por exemplo, no KDC primário e secundário, crie /etc/krb5kdc/kpropd.acl:

```bash
host/kdc01.treinalinux.com@TREINALINUX.COM
host/kdc02.treinalinux.com@TREINALINUX.COM
```

Crie um banco de dados vazio no KDC secundário :

```bash
sudo kdb5_util -s create
```

Agora instale o daemon kpropd, que escuta as conexões do utilitário kprop do kdc primário:

```bash
sudo apt install krb5-kpropd
```

O serviço estará em execução logo após a instalação.

De um terminal no KDC primário , crie um arquivo de despejo do banco de dados principal:

```bash
sudo kdb5_util dump /var/lib/krb5kdc/dump
````

Ainda no KDC primário , extraia seu arquivo keytab e copie-o para /etc/krb5.keytab:

```bash
kadmin -q "ktadd -k keytab.kdc01 host/kdc01.treinalinux.com"
sudo mv keytab.kdc01 /etc/krb5.keytab
sudo chown root:root /etc/krb5.keytab
```

> Nota
> 
> Agora você pode remover o extract-keysprivilégio de este principal em kdc01's /etc/krb5kdc/kadm5.aclarquivo

No KDC primário , execute o kproputilitário para enviar o dump do banco de dados feito antes para o KDC secundário:

```bash
$ sudo kprop -r TREINALINUX.COM -f /var/lib/krb5kdc/dump kdc02.treinalinux.com
Database propagation to kdc02.treinalinux.com: SUCCEEDED
```

Observe a mensagem `SUCCEEDED` , que sinaliza que a propagação funcionou. Se houver uma mensagem de erro, verifique `/var/log/syslog` o KDC secundário para obter mais informações.

Você também pode criar um cron job para atualizar periodicamente o banco de dados no KDC secundário. Por exemplo, o seguinte fará push do banco de dados a cada hora:

```bash
# m h  dom mon dow   command
0 * * * * root /usr/sbin/kdb5_util dump /var/lib/krb5kdc/dump && /usr/sbin/kprop -r TREINALINUX.COM -f /var/lib/krb5kdc/dump kdc02.treinalinux.com
````

De volta ao KDC secundário , crie um arquivo stash para conter a chave mestra Kerberos:

```bash
sudo kdb5_util stash
```

Finalmente, inicie o krb5-kdcdaemon no KDC Secundário:

```bash
sudo systemctl start krb5-kdc.service
```

> Nota
> 
> O KDC Secundário não executa um servidor de administração, uma vez que é uma cópia somente leitura

De agora em diante, você pode especificar os dois servidores KDC /etc/krb5.conf para o domínio TREINALINUX.COM, em qualquer host participante deste domínio (incluindo kdc01e kdc02), mas lembre-se de que só pode haver um servidor de administração e é aquele em execução kdc01:

```bash
[realms]
    TREINALINUX.COM = {
            kdc = kdc01.treinalinux.com
            kdc = kdc02.treinalinux.com
            admin_server = kdc01.treinalinux.com
    }
```


O KDC secundário agora deve ser capaz de emitir tíquetes para o Reino. Você pode testar isso parando o krb5-kdcdaemon no KDC primário e usando kinit para solicitar um tíquete. Se tudo correr bem, você deve receber um tíquete do KDC Secundário. Caso contrário, verifique `/var/log/syslog` e `/var/log/auth.log` no KDC secundário.

## Cliente Kerberos Linux

Esta seção cobre a configuração de um sistema Linux como cliente Kerberos. Isso permitirá o acesso a quaisquer serviços kerberizados, uma vez que o usuário tenha feito login com sucesso no sistema.

Observe que o Kerberos sozinho não é suficiente para que um usuário exista em um sistema Linux. Ou seja, não podemos simplesmente apontar o sistema para um servidor Kerberos e esperar que todos os principais do Kerberos sejam capazes de fazer o login no sistema Linux, simplesmente porque esses usuários não existem localmente. 

O Kerberos fornece apenas autenticação: ele não conhece grupos de usuários, uids e gids do Linux, diretórios pessoais, etc.

Normalmente, outra fonte de rede é usada para essas informações, como um servidor LDAP ou Windows e, antigamente, NIS foi usado para isso também.


## Instalação

Se você tiver usuários locais que correspondem aos principais em um realm Kerberos e quiser apenas mudar a autenticação de local para remoto usando Kerberos, siga esta seção. Este não é um cenário muito comum, mas serve para destacar a separação entre a autenticação do usuário e as informações do usuário (nome completo, uid, gid, diretório inicial, grupos, etc). 

Se você quiser apenas pegar os tíquetes e usá-los, basta instalar krb5-user e executar kinit.

Vamos usar sssdum truque para que ele busque as informações do usuário nos arquivos do sistema local, em vez de uma fonte remota, o que é o caso comum.

Para instalar os pacotes, digite o seguinte em um prompt de terminal:

```bash
sudo apt install krb5-user sssd-krb5
```

Serão solicitados os endereços de seus KDCs e servidores de administração. Se você tem seguido este capítulo até agora, os KDCs serão: kdc01.treinalinux.com kdc02.treinalinux.com (separados por espaço)

E o administrador do servidor será: kdc01.treinalinux.com. Lembre-se de que kdc02 é uma cópia somente leitura do KDC primário, portanto, não executa um servidor de administração.

> Nota
> 
> Se você adicionou os registros SRV apropriados ao DNS, nenhum desses prompts precisará de resposta.

## Configuração

Se você perdeu as perguntas anteriormente, você pode reconfigurar o pacote para enchê-los novamente: sudo dpkg-reconfigure krb5-config.

Você pode testar a configuração do Kerberos solicitando um tíquete usando o kinitutilitário. Por exemplo:

```bash
$ kinit perola/admin@TREINALINUX.COM
Password for perola/admin@TREINALINUX.COM:
```

> Nota
>
> kinit não precisa que o principal exista como um usuário local no sistema. Na verdade, você pode usar kinit para o principal que quiser. Se você não especificar um, a ferramenta usará o nome de usuário de quem estiver executando kinit.

Já que estamos nisso, vamos também criar um principal não administrativo para perola:

```bash
$ kadmin -q "addprinc perola"
Authenticating as principal perola/admin@TREINALINUX.COM with password.
Password for perola/admin@TREINALINUX.COM: 
WARNING: no policy specified for perola@TREINALINUX.COM; defaulting to no policy
Enter password for principal "perola@TREINALINUX.COM": 
Re-enter password for principal "perola@TREINALINUX.COM": 
Principal "perola@TREINALINUX.COM" created.
```

A única configuração restante agora é para sssd. Crie o arquivo `/etc/sssd/sssd.conf` com o seguinte conteúdo:

```bash
[sssd]
config_file_version = 2
services = pam
domains = treinalinux.com

[pam]

[domain/treinalinux.com]
id_provider = proxy
proxy_lib_name = files
auth_provider = krb5
krb5_server = kdc01.treinalinux.com,kdc01.treinalinux.com
krb5_kpasswd = kdc01.treinalinux.com
krb5_realm = TREINALINUX.COM
```

A configuração acima usará kerberos para autenticação `(auth_provider)`, mas usará os usuários do sistema local para informações de usuário e grupo `(id_provider)`.

Ajuste as permissões do arquivo de configuração e comece sssd:

```bash
$ sudo chown root:root /etc/sssd/sssd.conf
$ sudo chmod 0600 /etc/sssd/sssd.conf
$ sudo systemctl start sssd
```

Apenas por ter instalado sssde suas dependências, o PAM já estará configurado para uso sssd, com um fallback para a autenticação do usuário local. Para experimentá-lo, se esta for uma estação de trabalho, simplesmente alterne os usuários (na GUI), ou abra um terminal de login (`CTRL-ALT- <número>`), ou gere um shell de login com sudo logine tente fazer login usando o nome de um diretor de kerberos. Lembre-se de que este usuário já deve existir no sistema local:

```bash
$ sudo login
focal-krb5-client login: perola
Password: 
Welcome to perola Focal Fossa (development branch) (GNU/Linux 5.4.0-21-generic x86_64)

(...)

Last login: Thu Apr  30 23:49:50 UTC 2020 from 192.168.0.120 on pts/0
$ klist
Ticket cache: FILE:/tmp/krb5cc_1000_NlfnSX
Default principal: perola@TREINALINUX.COM

Valid starting     Expires            Service principal
09/08/20 20:30:19  09/09/20 06:30:19  krbtgt/TREINALINUX.COM@TREINALINUX.COM
	renew until 09/09/20 20:30:19
```

E você terá um tíquete Kerberos logo após o login.

## Glossário

**default_realm = TREINALINUX.COM** - Identifica o domínio Kerberos padrão para o cliente. Defina seu valor para o seu reino Kerberos. 

Se isso não for especificado e a pesquisa de registro TXT estiver habilitada (consulte Usando DNS), essas informações serão usadas para determinar o domínio padrão. 

Se esta tag não estiver definida neste arquivo de configuração e não houver informações de DNS encontradas, um erro será retornado.

**default_keytab_name = FILE:/etc/krb5.keytab** - Esta relação especifica o nome do keytab padrão a ser usado por servidores de aplicativos, como telnetd e rlogind. O padrão é /etc/krb5.keytab.


**dns_lookup_realm = true** - Indica se os registros TXT do DNS devem ser usados para determinar o domínio Kerberos de um host. 

Habilitar essa opção pode permitir um ataque de redirecionamento, em que as respostas DNS falsificadas persuadem um cliente a se autenticar no domínio errado, ao falar com o host errado (falsificando ainda mais registros DNS ou interceptando o tráfego da rede). 

Dependendo de como o software cliente gerencia os nomes de host, no entanto, ele já pode estar vulnerável a tais ataques. 

Estamos procurando maneiras possíveis de minimizar ou eliminar essa exposição. Por enquanto, incentivamos os sites mais aventureiros a tentar usar DNS seguro. 

Se esta opção não for especificada, mas dns_fallback for, esse valor será usado. Se nenhuma opção for especificada, o comp    ortamento dependerá das opções de tempo de configuração; se nenhum for fornecido, o padrão é desabilitar esta opção. Se o suporte DNS não estiver compilado, essa entrada não terá efeito. 


**dns_lookup_kdc = true** - Indica se os registros SRV do DNS devem ser usados para localizar os KDCs e outros servidores para um realm, se eles não estiverem listados nas informações do realm. (Observe que a entrada admin_server deve estar no arquivo, porque a implementação do DNS para ele está incompleta.) 

Habilitar essa opção abre um tipo de ataque de negação de serviço, se alguém falsificar os registros DNS e redirecionar você para outro servidor. 

No entanto, não é pior do que uma negação de serviço, porque aquele KDC falso será incapaz de decodificar qualquer coisa que você enviar (além da solicitação de tíquete inicial, que não tem dados criptografados), e qualquer coisa que o KDC falso enviar não será confiável sem verificação usando algum segredo que ele não saberá. 

Se esta opção não for especificada, mas dns_fallback for, esse valor será usado. 

Se nenhuma opção for especificada, o comportamento dependerá das opções de tempo de configuração; se nenhum for fornecido, o padrão é habilitar esta opção. 

Se o suporte DNS não estiver compilado, essa entrada não terá efeito.


**forwardble = true** - Se este sinalizador for *true*, os tíquetes iniciais serão encaminhados por padrão, se permitido pelo KDC. O valor padrão é *false*.


**rdns = false** - Se este sinalizador for *true*, a consulta reversa de nome será usada além da consulta direta de nome para canonizar nomes de host para uso em nomes de entidades de serviço. Se dns_canonicalize_hostname for definido como false, este sinalizador não terá efeito. O valor padrão é *true*.


**renew_lifetime = 7d** - O valor dessa tag é a vida útil renovável padrão para os tickets iniciais. O valor padrão da tag é 0.


**ticket_lifetime = 24h** - Define a vida útil padrão para solicitações de tickets iniciais. O valor padrão é 1 dia.


**[domain_realm]** - A seção **[domain_realm]** fornece uma tradução de um nome de domínio ou nome de host para um nome de domínio Kerberos. 

O nome da marca pode ser um nome de host ou nome de domínio, onde os nomes de domínio são indicados por um prefixo de um ponto (.). 

O valor da relação é o nome do domínio Kerberos para esse host ou domínio específico. Uma relação de nome de host fornece implicitamente a relação de nome de domínio correspondente, a menos que uma relação de nome de domínio explícita seja fornecida. 

O domínio Kerberos pode ser identificado na seção de domínios ou usando registros SRV do DNS. 

Os nomes de host e nomes de domínio devem estar em letras minúsculas.


### Fonte:
*Consulte o site do MIT Kerberos.*

