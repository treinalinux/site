# Programação com Ruby On Rails



## Primeiros passos



### Configuração do ambiente

Nosso ambiente inicial é baseado no vim, mas tudos as ferramentes podem ser usadas em outros editores e IDEs.


Para comentar um bloco de códigos no vim é super fácial, mas vamos adaptar com um Plug para não assustar quem começou no vim agora.
```
" No MacOS deve usar o fn \
Plug ‘tpope/vim-commentary’
noremap \ :Commentary<CR>
autocmd FileType ruby setlocal commentstring=#\ %s
```




### Qualidade de código

