## Certificados Digitais com openssl

Uma das formas mais comuns de criptografia hoje é a criptografia de chave pública. A criptografia de chave pública utiliza uma chave pública e uma chave privada . O sistema funciona criptografando informações usando a chave pública. As informações só podem ser descriptografadas usando a chave privada.

Um uso comum para criptografia de chave pública é criptografar o tráfego de aplicativos usando uma conexão **Secure Socket Layer (SSL)** ou **Transport Layer Security (TLS)**. Um exemplo: configurar o Apache para fornecer HTTPS , o protocolo HTTP sobre SSL/TLS. Isso permite criptografar o tráfego usando um protocolo que não fornece criptografia.

Um certificado é um método usado para distribuir uma chave pública e outras informações sobre um servidor e a organização responsável por ele. Os certificados podem ser assinados digitalmente por uma autoridade de certificação ou CA. Uma CA é um terceiro confiável que confirmou que as informações contidas no certificado são precisas.

## Tipos de certificados
Para configurar um servidor seguro usando criptografia de chave pública, na maioria dos casos, você envia sua solicitação de certificado (incluindo sua chave pública), prova de identidade de sua empresa e pagamento a uma CA. A CA verifica a solicitação de certificado e sua identidade e, em seguida, envia de volta um certificado para seu servidor seguro. Como alternativa, você pode criar seu próprio certificado autoassinado .

> **Nota**
>
> Observe que os certificados autoassinados não devem ser usados ​​na maioria dos ambientes de produção.

Continuando com o exemplo HTTPS, um certificado assinado por CA fornece dois recursos importantes que um certificado autoassinado não oferece:

Os navegadores (geralmente) reconhecem automaticamente a assinatura da CA e permitem que uma conexão segura seja feita sem avisar o usuário.

Quando uma CA emite um certificado assinado, está garantindo a identidade da organização que está fornecendo as páginas da web ao navegador.

A maioria dos softwares que oferecem suporte a SSL / TLS tem uma lista de CAs cujos certificados eles aceitam automaticamente. Se um navegador encontrar um certificado cuja autoridade de certificação não esteja na lista, o navegador pede ao usuário que aceite ou recuse a conexão. Além disso, outros aplicativos podem gerar uma mensagem de erro ao usar um certificado autoassinado.

O processo de obtenção de um certificado de uma CA é bastante fácil. Uma visão geral rápida é a seguinte:

1. Crie um par de chaves de criptografia pública e privada.

2. Crie uma solicitação de assinatura de certificado com base na chave pública. O pedido de certificado contém informações sobre o seu servidor e a empresa que o hospeda.

3. Envie a solicitação de certificado, juntamente com documentos que comprovem sua identidade, para uma CA. Não podemos dizer qual autoridade de certificação escolher. Sua decisão pode ser baseada em suas experiências anteriores, ou nas experiências de seus amigos ou colegas, ou puramente em fatores monetários. Depois de decidir sobre uma CA, você precisa seguir as instruções fornecidas sobre como obter um certificado deles.

4. Quando a CA está convencida de que você é realmente quem afirma ser, ela lhe envia um certificado digital.

5. Instale este certificado em seu servidor seguro e configure os aplicativos apropriados para usar o certificado.

## Gerando uma solicitação de assinatura de certificado (CSR)

Esteja você obtendo um certificado de uma CA ou gerando seu próprio certificado autoassinado, a primeira etapa é gerar uma chave.

Se o certificado for usado por daemons de serviço, como Apache, Postfix, Dovecot, etc., uma chave sem uma senha é geralmente apropriada. Não ter uma senha permite que os serviços sejam iniciados sem intervenção manual, geralmente a forma preferida de iniciar um daemon.

Esta seção cobrirá a geração de uma chave com uma frase-senha e outra sem. A chave sem senha será então usada para gerar um certificado que pode ser usado com vários daemons de serviço.

> **Atenção**
>
> Executar seu serviço seguro sem uma senha é conveniente porque você não precisará inserir a senha toda vez que iniciar seu serviço seguro. Mas é inseguro e comprometer a chave significa comprometer também o servidor.

Para gerar as chaves para a Solicitação de Assinatura de Certificado (CSR), execute o seguinte comando em um prompt de terminal:

```bash
openssl genrsa -des3 -out server.key 2048

Generating RSA private key, 2048 bit long modulus
..........................++++++
.......++++++
e is 65537 (0x10001)
Enter pass phrase for server.key:
```

Agora você pode inserir sua senha longa. Para melhor segurança, deve conter pelo menos oito caracteres. O comprimento mínimo ao especificar `-des3` é de quatro caracteres. Como prática recomendada, deve incluir números e/ou pontuação e não ser uma palavra em um dicionário. Lembre-se também de que sua senha é sensível a maiúsculas e minúsculas.

Digite novamente a frase secreta para verificar. Depois de redigitá-la corretamente, a chave do servidor é gerada e armazenada no arquivo `server.key`.

Agora crie a chave insegura, aquela sem senha, e embaralhe os nomes das chaves:

openssl rsa -in server.key -out server.key.insecure
mv server.key server.key.secure
mv server.key.insecure server.key
A chave insegura agora é nomeada server.keye você pode usar esse arquivo para gerar o CSR sem senha.

Para criar o CSR, execute o seguinte comando em um prompt de terminal:

```bash
openssl req -new -key server.key -out server.csr
```

Ele solicitará que você insira a senha. Se você inserir a senha correta, será solicitado que você insira o nome da empresa, o nome do site, a identificação do e-mail, etc. Depois de inserir todos esses detalhes, seu CSR será criado e armazenado no server.csrarquivo.

Agora você pode enviar este arquivo CSR a uma CA para processamento. A CA usará este arquivo CSR e emitirá o certificado. Por outro lado, você pode criar um certificado autoassinado usando este CSR.

Criação de um certificado autoassinado
Para criar o certificado autoassinado, execute o seguinte comando em um prompt de terminal:

```bash
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

O comando acima solicitará que você insira a senha. Depois de inserir a senha correta, seu certificado será criado e armazenado no server.crtarquivo.

> **Atenção**
>
> Se o seu servidor seguro for usado em um ambiente de produção, você provavelmente precisará de um certificado assinado pela CA. Não é recomendado usar certificado autoassinado.

## Instalando o Certificado
Você pode instalar o arquivo de chave `server.key` e o arquivo de certificado `server.crt`, ou o arquivo de certificado emitido por sua CA, executando os seguintes comandos em um prompt de terminal:

```bash
sudo cp server.crt /etc/ssl/certs
sudo cp server.key /etc/ssl/private
```

Agora, basta configurar qualquer aplicativo, com a capacidade de usar criptografia de chave pública, para usar o certificado e os arquivos de chave . Por exemplo, o Apache pode fornecer HTTPS, Dovecot pode fornecer IMAPS e POP3S, etc.

## Autoridade de Certificação
Se os serviços em sua rede exigirem mais do que alguns certificados autoassinados, pode valer a pena o esforço adicional para configurar sua própria Autoridade de Certificação (CA) interna. O uso de certificados assinados por sua própria CA permite que os vários serviços que usam os certificados confiem facilmente em outros serviços usando certificados emitidos pela mesma CA.

Primeiro, crie os diretórios para conter o certificado CA e os arquivos relacionados:

```bash
sudo mkdir /etc/ssl/CA
sudo mkdir /etc/ssl/newcerts
```

A CA precisa de alguns arquivos adicionais para operar, um para rastrear o último número de série usado pela CA, cada certificado deve ter um número de série exclusivo e outro arquivo para registrar quais certificados foram emitidos:

```bash
sudo sh -c "echo '01' > /etc/ssl/CA/serial"
sudo touch /etc/ssl/CA/index.txt
```

O terceiro arquivo é um arquivo de configuração da CA. Embora não seja estritamente necessário, é muito conveniente ao emitir vários certificados. Editar `/etc/ssl/openssl.cnf`, e na mudança **[CA_default]** :

```bash
dir             = /etc/ssl               # Where everything is kept
database        = $dir/CA/index.txt      # database index file.
certificate     = $dir/certs/cacert.pem  # The CA certificate
serial          = $dir/CA/serial         # The current serial number
private_key     = $dir/private/cakey.pem # The private key
```

Em seguida, crie o certificado raiz autoassinado:

```bash
openssl req -new -x509 -extensions v3_ca -keyout cakey.pem -out cacert.pem -days 3650
````

Em seguida, você será solicitado a inserir os detalhes sobre o certificado.

Agora instale o certificado raiz e a chave:

```bash
sudo mv cakey.pem /etc/ssl/private/
sudo mv cacert.pem /etc/ssl/certs/
```

Agora você está pronto para começar a assinar certificados. O primeiro item necessário é uma solicitação de assinatura de certificado (CSR), consulte Gerando uma solicitação de assinatura de certificado (CSR) para obter detalhes. Depois de ter um CSR, digite o seguinte para gerar um certificado assinado pela CA:

```bash
sudo openssl ca -in server.csr -config /etc/ssl/openssl.cnf
```

Depois de inserir a senha para a chave CA, você será solicitado a assinar o certificado e, novamente, a confirmar o novo certificado. Em seguida, você deverá ver uma quantidade relativamente grande de saída relacionada à criação do certificado.

Agora deve haver um novo arquivo `/etc/ssl/newcerts/01.pem`, contendo a mesma saída. Copie e cole tudo que comece com a linha: **----- BEGIN CERTIFICATE -----** e continuando pela linha: **---- END CERTIFICATE -----** linhas para um arquivo nomeado após o nome do host do servidor onde o certificado será instalado. Por exemplo mail.example.com.crt, é um bom nome descritivo.

Certificados subseqüentes serão nomeados `02.pem`, `03.pem` etc.

> **Nota**
>
> Substitua **mail.example.com.crt** pelo seu próprio nome descritivo.

Por fim, copie o novo certificado para o host que precisa dele e configure os aplicativos apropriados para usá-lo. O local padrão para instalar os certificados é `/etc/ssl/certs`. Isso permite que vários serviços usem o mesmo certificado sem permissões de arquivo excessivamente complicadas.

Para aplicativos que podem ser configurados para usar um certificado CA, você também deve copiar o `/etc/ssl/certs/cacert.pem` arquivo para o diretório `/etc/ssl/certs/` em cada servidor.

**Fontes:**

[Network Security with OpenSSL: Cryptography for Secure Communications](https://amzn.to/3eFYRJK)

