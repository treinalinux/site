# Servidores DNS no Linux

**Aviso importante, como estmos migrando os arquivos dos posts do site, vai ser comum você ver posts assim....**

## Em construção.

**O que é um servidor DNS?**

## Servidor DNS no Linux

### Bind 

**Principais arquivos de configuração:**

* hosts 
* resolv.conf
* named.conf - Principal arquivo de configuração (Red Hat)
* named.conf - Principal arquivo de configuração com includes (Debian)
* named.conf.options - Opções Globais (Debian)
* named.conf.local - Para configuração de Zonas (Debian)
* named.conf.default.zones - Para configuração de Zonas padrão (Debian)


---
## Tipos de Name Severs

**autoritativo**

Os servidores de nomes autorizados respondem a registros de recursos que fazem parte de suas zonas apenas. Esta categoria inclui servidores de nomes primários (master) e secundários (slaves).

**recursivo**

Os servidores de nomes recursivos oferecem serviços de resolução, mas não são autorizados para nenhuma zona. As respostas para todas as resoluções são armazenadas em cache em uma memória por um período fixo de tempo, que é especificado pelo registro de recurso recuperado.



## Tipos de sevidores DNS

Os tipos de servidores DNS são:

Caching Nameserver

Primary Server

Secundary Server


## Tipos de zonas


## O que são domínios


## Tipos de registros


## Ferramentas de Consulta
