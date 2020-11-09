# Servidores DHCP no Linux

## Dynamic Host Configuration Protocol (DHCP)

### Usamos um Debian para o exemplo

O **Dynamic Host Configuration Protocol** ou **protocolo de configuração dinâmica de hosts** (`DHCP`) é um serviço de rede que permite que computadores host recebam configurações automaticamente de um servidor em vez de configurar manualmente cada host de rede. 

Os computadores configurados para serem clientes DHCP não têm controle sobre as configurações que recebem do servidor DHCP, e a configuração é transparente para o usuário do computador.

As configurações mais comuns fornecidas por um servidor DHCP para clientes DHCP incluem:

Endereço IP e máscara de rede

Endereço IP do gateway padrão a ser usado

Endereços IP dos servidores DNS a serem usados

No entanto, um servidor DHCP também pode fornecer propriedades de configuração, como:


* Host Name

* Domain Name

* Time Server

* Print Server


A vantagem de usar DHCP é que as mudanças na rede, por exemplo, uma mudança no endereço do servidor DNS, só precisam ser mudadas no servidor DHCP e todos os hosts da rede serão reconfigurados na próxima vez que seus clientes DHCP fizerem polling do servidor DHCP . Como vantagem adicional, também é mais fácil integrar novos computadores à rede, pois não há necessidade de verificar a disponibilidade de um endereço IP. Os conflitos na alocação de endereços IP também são reduzidos.

Um servidor DHCP pode fornecer definições de configuração usando os seguintes métodos:

* Manual allocation (MAC address)
Este método envolve o uso de DHCP para identificar o endereço de hardware exclusivo de cada placa de rede conectada à rede e, em seguida, fornecer continuamente uma configuração constante cada vez que o cliente DHCP faz uma solicitação ao servidor DHCP usando aquele dispositivo de rede. Isso garante que um endereço específico seja atribuído automaticamente a essa placa de rede, com base em seu endereço MAC.

* Dynamic allocation (address pool)
Neste método, o servidor DHCP atribuirá um endereço IP de um pool de endereços (às vezes também chamado de intervalo ou escopo) por um período de tempo ou concessão, que é configurado no servidor ou até o cliente informa ao servidor que não precisa mais do endereço. Desta forma, os clientes receberão suas propriedades de configuração dinamicamente e por ordem de chegada. Quando um cliente DHCP não está mais na rede por um período especificado, a configuração expira e é liberada de volta para o pool de endereços para uso por outros clientes DHCP. Dessa forma, um endereço pode ser alugado ou usado por um período de tempo. Após esse período, o cliente deve renegociar o aluguel com o servidor para manter o uso do endereço.

* Automatic allocation
Usando este método, o DHCP atribui automaticamente um endereço IP permanentemente a um dispositivo, selecionando-o em um conjunto de endereços disponíveis. Normalmente, o DHCP é usado para atribuir um endereço temporário a um cliente, mas um servidor DHCP pode permitir um tempo de concessão infinito.

Os dois últimos métodos podem ser considerados “automáticos” porque em cada caso o servidor DHCP atribui um endereço sem necessidade de intervenção extra. A única diferença entre eles está em quanto tempo o endereço IP é alugado, em outras palavras, se o endereço de um cliente varia com o tempo. O servidor DHCP que o Ubuntu disponibiliza é o dhcpd (daemon de protocolo de configuração de host dinâmico), que é fácil de instalar e configurar e será iniciado automaticamente na inicialização do sistema.


## Instalação

Em um prompt de terminal, digite o seguinte comando para instalar o dhcpd:

```bash
sudo apt install isc-dhcp-server
```

> NOTA: as mensagens do dhcpd estão sendo enviadas para o syslog. Procure por mensagens de diagnóstico.


## Configuração


Você provavelmente precisará alterar a configuração padrão editando `/etc/dhcp/dhcpd.conf` para atender às suas necessidades e configuração particular.

Geralmente, o que você deseja fazer é atribuir um endereço IP aleatoriamente. Isso pode ser feito com as seguintes configurações:

```bash
# minimal sample /etc/dhcp/dhcpd.conf
default-lease-time 600;
max-lease-time 7200;

subnet 192.168.1.0 netmask 255.255.255.0 {
 range 192.168.1.150 192.168.1.200;
 option routers 192.168.1.254;
 option domain-name-servers 192.168.1.1, 192.168.1.2;
 option domain-name "mydomain.example";
}
```

Isso fará com que o servidor DHCP forneça aos clientes um endereço IP no intervalo 192.168.1.150-192.168.1.200. Ele concederá um endereço IP por 600 segundos se o cliente não solicitar um período de tempo específico. Caso contrário, o aluguel máximo (permitido) será de 7200 segundos. O servidor também “aconselhará” o cliente a usar 192.168.1.254 como o gateway padrão e 192.168.1.1 e 192.168.1.2 como seus servidores DNS.

Você também pode precisar editar `/etc/default/isc-dhcp-server`para especificar as interfaces que o dhcpd deve ouvir.

```bash
INTERFACESv4="eth4"
```

Depois de alterar os arquivos de configuração, você deve reiniciar o serviço dhcpd:

```bash
sudo systemctl restart isc-dhcp-server.service
```

**Fontes:**
A página Wiki do Ubuntu dhcp3-server tem mais informações.

Para obter mais `/etc/dhcp/dhcpd.conf` opções, consulte a página do manual dhcpd.conf.
