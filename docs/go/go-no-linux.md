i## Instalação do go no Linux


- 1 - Baixar e instalar
- 2 - Colocar no PATH
- 3 - Recarregar o .profile
- 4 - Criar a estrutura de pastas
- 5 - Configurar o vim

### Baixando e instalando o go

```bash
wget -c https://dl.google.com/go/go1.16.linux-amd64.tar.gz -O - | sudo tar -xz -C /usr/local
```

### Configurando o go no PATH
```
echo "export PATH=$PATH:/usr/local/go/bin" >> .profile
```

**Caso queria alterar para todos o usuários:**

```
sudo tee -a /etc/profile <<< "export PATH=$PATH:/usr/local/go/bin"
```

### Recarregando o .profile
```
source .profile
```

ou

```
source /etc/profile
```

**No caso de usar outro shell padrão pode ser necessário outra configuração**


```
echo "export PATH=$PATH:/usr/local/go/bin" >> .zshrc
```


### Criando a estrutura de pasta para o go

Antes de avançar vamos instalar alguns pacotes necessários:

```bash
sudo apt install vim-gocomplete golint gocode vim-fugitive vim vim-scripts tree
```

Por padrão o go espera uma pasta com a estrutura abaixo criada no seu diretório pessoal:

```
mkdir -vp go/{bin,pkg,src}
```

Veja a estrutura de pastas do go com o tree:

```
tree go

go
├── bin
├── pkg
└── src

3 directories, 0 files

```


### Configurando o vim

Agora abra um arquivo de teste com o nome main.go com o vim:

```go
package main

import "fmt"

func main() {
	fmt.Println("vim-go")
}
```

Dentro do vim instale os binários do go para o vim-go, use o comando no vim abaixo, isso pode demorar alguns minutos:

```
:GoInstallBinaries
```

Se você listar o conteúdo da pasta go, agora vai ver que as pastas que foram criar em go estão com diversas outras pastas em módulos.


Pode sair do vim sem salvar o arquivo ```ESC:q```.

Veja novamente a estrutura de pasta do go com o tree, no meu caso deu *7574 directories, 22998 files*:

```
tree go

go
├── ...
├── ...
└── ...

7574 directories, 22998 files

```


### Vamos testar

Por padrão seu projeto deveria ser criado na pasta go/src, mas isso pode ser um limitador e causador de problemas para projetos muito grandes, por isso o comando ```go mod init``` vai nos ajudar bastante.

Crie uma pasta de **teste** em algum diretóio de sua preferência, depois entre na pasta e execute:

```
go mod init teste

go: creating new go.mod: module teste
```

Ele vai criar um arquivo chamado **go.mod** com o seguinte conteúdo:

```
module teste

go 1.16
```


Caso queria testar diretamente com um repositório do github, vá no github crie um repositório com o **nome do seu projeto** e depois faça um clone para sua computudor.

Agora copie do seu github conforme o exemplo abaixo, entre na pasta do repositório clonado **nome do seu projeto** e execute: 

```bash
go mod init github.com/treinalinux/nome-do-seu-projeto

go: creating new go.mod: module github.com/treinalinux/nome-do-seu-projeto
```

Se tudo ocorreu como esperado é só executar o comando abaixo dentro do diretório para ver o repositório listado **github.com/treinalinux/nome-do-seu-projeto**:

```
go list -m all

github.com/treinalinux/nome-do-seu-projeto
```

Tudo pronto para que você possa começar sua jornada no golang, não esqueça do ```Hello, World!```.

