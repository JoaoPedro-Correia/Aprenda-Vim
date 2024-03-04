# Ch03. Procurando Arquivos

O objetivo desse capítulo é dar uma introdução de como realizar buscas rápidas no Vim. Ser capaz de pesquisar rapidamente é um ótima maneira de impulsionar sua produtividade no Vim. Quando eu descobri como pesquisar arquivos rapidamente, decidi usar o Vim como editor principal.

Esse capítulo é dividido em duas partes: como procurar sem plugins e como buscar usando o pluguin [fzf.vim](https://github.com/junegunn/fzf.vim)

## Abrindo e Editando Arquivos

Ao abrir um arquivo no Vim, você pode usar o comando `:edit`

```
:edit file.txt
```

Se `file.txt` existir, ele será aberto no buffer. Caso `file.txt` não exista, o arquivo será criado em um buffer novo.

*Autocomplete* com `<Tab>` funciona no comando `:edit`. Por exemplo, se seu arquivo está dentro de um diretório de controlador de usuários do controlador de aplicativos [Rails](https://rubyonrails.org/) `./app/controllers/users_controllers.rb` você pode usar `<Tab>` para expandir os termos rapidamente:

```
:edit a<Tab>c<Tab>u<Tab>
```

`:edit` aceita wildcards como argumento. `*` corresponde a qualquer arquivo no diretório. Se você só está buscando por arquivos com extensão `.yml` no diretório corrente: 

```
:edit *.yml<Tab>
```
Vim dará a você uma lista de todos os arquivos `.yml` no diretório atual para escolher.

Você pode usar `**` para procurar recursivamente. Se você quer procurar por todos os arquivos `*.md` no seu projeto, mas não sabe em qual diretório buscar, você pode fazer isso:

```
:edit **/*.md<Tab>
```

`:edit` pode ser usado para rodar `netrw`, que é o explorador de arquivos integrado do Vim. Para fazer isso, forneça um argumento de diretório ao comando `:edit` em vez do arquivo.

```
:edit .
:edit test/unit/
```

## Procurando Arquivos Usando o Comando Find
Você pode encontrar arquivos com o comando `:find`. Por exemplo:

```
:find package.json
:find app/controllers/users_controller.rb
```
Autocomplete também funciona com `:find`:

```
:find p<Tab>                " to find package.json
:find a<Tab>c<Tab>u<Tab>    " to find app/controllers/users_controller.rb
```
Como você pode notar, o comando `:find` é parecido com o `:edit`. Então qual é a diferença?

## Comandos Find e Path

A diferença é que `:find` encotra arquivo pelo `path`, `:edit` não. Vamos aprender um pouco sobre o `path`. Uma vez que você aprende a modificar o seu `path`, o comando `:find` se torna uma ferramenta de busca poderosa. Cheque o que o seu `path` é com o comando: 

```
:set path?
```

Por padrão, provavelmente deve aparecer algo como isso:

```
path=.,/usr/include,,
```

- `.` procura no diretório do arquivo corrente no qual foi aberto
- `,` procura no diretório corrente. 
- `/usr/include` é tipicamente o diretório onde os arquivos das bibliotecas em C se encontram. 

Os dois primeiros são importantes para o nosso contexto e o terceiro pode ser ignorado por agora. O ponto principal aqui é que você pode madificar o seu próprio `path`, que é onde o Vim ira procurar por arquivos.  Vamos assumir essa é sua estrutura de projeto:  

```
app/
  assets/
  controllers/
    application_controller.rb
    comments_controller.rb
    users_controller.rb
    ...
```

Se você está querendo ir para `users_controller.rb` a partir do diretório raiz, você vai precisar ir através desses diversos diretórios (e pressionar uma quantidade considerável de tabs). Depois quando você estiver trabalhando com framework, 90% do seu tempo vai ser gasto em um diretório específico. Nessa situação, você só vai se importar em ir ao diretório `controllers/` digitando menos.

Você precisa adicionar o caminho `app/controllers/` no `path` corrente. Como você pode fazer isso:

```
:set path+=app/controllers/
```

Agora que seu `path` está atualizado, quando você digitar `:find u<Tab>`, Vim vai procurar dentro do diretório `app/controllers/` por arquivos que começam com 'u'.

Se o arquivo fica dentro de outro subdiretório de `controllers/`, como `app/controllers/account/users_controller.rb`, o Vim não será capaz de encontrar o arquivo. Para ser possível, você precisa adicionar `:set path+=app/controllers/**` para o autocomplete encontrar o `users_controller.rb`. Isso é ótimo! Agora você pode encontrar o `users_controller.rb` com 1 tab em vez de 3.

Você pode estar pensando em adicionar diretórios de projetos inteiros, para quando você pressionar `tab` o Vim já procura em todos os subdiretórios por esse arquivo, exemplo:

```
:set path+=$PWD/**
```

`$PWD` é o diretório de trabalho atual. Se você tentar adicionar projetos inteiros no `path` esperando que todos os aquivos sejam alcançados pressionando `tab`, embora isso possa funcionar para pequenos projetos, fazer isso torna a sua busca significativamente mais lenta se você tem um grande número de arquivos no seu projeto. Eu recomendo fazer isso só para os diretórios/arquivo mais visitados.

Você pode adicionar o `set path+={seu-path-aqui}` no arquivo vimrc. Atualizar o `path` vai levar só alguns segundos e fazendo isso pode te salvar por muito tempo.

## Procurando Arquivo Usando o Comando Grep

If you need to find in files (find phrases in files), you can use grep. Vim has two ways of doing that:

- Internal grep (`:vim`. Yes, it is spelled `:vim`. It is short for `:vimgrep`).
- External grep (`:grep`).

Let's go through internal grep first. `:vim` has the following syntax:

```
:vim /pattern/ file
```

- `/pattern/` is a regex pattern of your search term.
- `file` is the file argument. You can pass multiple arguments. Vim will search for the pattern inside the file argument. Similar to `:find`, you can pass it `*` and `**` wildcards.

For example, to look for all occurrences of "breakfast" string inside all ruby files (`.rb`) inside `app/controllers/` directory:

```
:vim /breakfast/ app/controllers/**/*.rb
```

After running that, you will be redirected to the first result. Vim's `vim` search command uses `quickfix` operation. To see all search results, run `:copen`. This opens a `quickfix` window. Here are some useful quickfix commands to get you productive immediately:

```
:copen        Open the quickfix window
:cclose       Close the quickfix window
:cnext        Go to the next error
:cprevious    Go to the previous error
:colder       Go to the older error list
:cnewer       Go to the newer error list
```

To learn more about quickfix, check out `:h quickfix`.

You may notice that running internal grep (`:vim`) can get slow if you have a large number of matches. This is because Vim loads each matching file into memory, as if it were being edited. If Vim finds a large number of files matching your search, it will load them all and therefore consume a large amount of memory.

Let's talk about external grep. By default, it uses `grep` terminal command. To search for "lunch" inside a ruby file inside `app/controllers/` directory, you can do this:

```
:grep -R "lunch" app/controllers/
```

Note that instead of using `/pattern/`, it follows the terminal grep syntax `"pattern"`. It also displays all matches using `quickfix`.

Vim defines the `grepprg` variable to determine which external program to run when running the `:grep` Vim command so that you don't have to close Vim and invoke the terminal `grep` command. Later, I will show you how to change the default program invoked when using the `:grep` Vim command.

## Navegando Por Aquivos Usando o Comando Netrw

`netrw` is Vim's built-in file explorer. It is useful to see a project's hierarchy. To run `netrw`, you need these two settings in your `.vimrc`:

```
set nocp
filetype plugin on
```

Since `netrw` is a vast topic, I will only cover the basic usage, but it should be enough to get you started. You can start `netrw` when you launch Vim by passing it a directory as a parameter instead of a file. For example:

```
vim .
vim src/client/
vim app/controllers/
```

To launch `netrw` from inside Vim, you can use the `:edit` command and pass it a directory parameter instead of a filename:

```
:edit .
:edit src/client/
:edit app/controllers/
```

There are other ways to launch `netrw` window without passing a directory:

```
:Explore     Starts netrw on current file
:Sexplore    No kidding. Starts netrw on split top half of the screen
:Vexplore    Starts netrw on split left half of the screen
```

You can navigate `netrw` with Vim motions (motions will be covered in depth in a later chapter). If you need to create, delete, or rename a file or directory, here is a list of useful `netrw` commands:

```
%    Create a new file
d    Create a new directory
R    Rename a file or directory
D    Delete a file or directory
```

`:h netrw` is very comprehensive. Check it out if you have time.

If you find `netrw` too bland and need more flavor, [vim-vinegar](https://github.com/tpope/vim-vinegar) is a good plugin to improve `netrw`. If you're looking for a different file explorer, [NERDTree](https://github.com/preservim/nerdtree) is a good alternative. Check them out!

## Fzf

Now that you've learned how to search files in Vim with built-in tools, let's learn how to do it with plugins.

One thing that modern text editors get right and that Vim didn't is how easy it is to find files, especially via fuzzy search. In this second half of the chapter, I will show you how to use [fzf.vim](https://github.com/junegunn/fzf.vim) to make searching in Vim easy and powerful.

## Configuração

First, make sure you have [fzf](https://github.com/junegunn/fzf) and [ripgrep](https://github.com/BurntSushi/ripgrep) downloaded. Follow the instruction on their github repo. The commands `fzf` and `rg` should now be available after successful installs.

Ripgrep is a search tool much like grep (hence the name). It is generally faster than grep and has many useful features. Fzf is a general-purpose command-line fuzzy finder. You can use it with any commands, including ripgrep. Together, they make a powerful search tool combination.

Fzf does not use ripgrep by default, so we need to tell fzf to use ripgrep by defining a `FZF_DEFAULT_COMMAND` variable. In my `.zshrc` (`.bashrc` if you use bash), I have these:

```
if type rg &> /dev/null; then
  export FZF_DEFAULT_COMMAND='rg --files'
  export FZF_DEFAULT_OPTS='-m'
fi
```

Pay attention to `-m` in `FZF_DEFAULT_OPTS`. This option allows us to make multiple selections with `<Tab>` or `<Shift-Tab>`. You don't need this line to make fzf work with Vim, but I think it is a useful option to have. It will come in handy when you want to perform search and replace in multiple files which I'll cover in just a little. The fzf command accepts many more options, but I won't cover them here. To learn more, check out [fzf's repo](https://github.com/junegunn/fzf#usage) or `man fzf`. At minimum you should have `export FZF_DEFAULT_COMMAND='rg'`.

After installing fzf and ripgrep, let's set up the fzf plugin. I am using [vim-plug](https://github.com/junegunn/vim-plug) plugin manager in this example, but you can use any plugin managers.

Add these inside your `.vimrc` plugins. You need to use [fzf.vim](https://github.com/junegunn/fzf.vim) plugin (created by the same fzf author).

```
call plug#begin()
Plug 'junegunn/fzf.vim'
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
call plug#end()
```

After adding these lines, you will need to open `vim` and run `:PlugInstall`. It will install all plugins that are defined in your `vimrc` file and are not installed. In our case, it will install `fzf.vim` and `fzf`. 

For more info about this plugin, you can check out [fzf.vim repo](https://github.com/junegunn/fzf/blob/master/README-VIM.md).

## Sintaxe do Fzf

To use fzf efficiently, you should learn some basic fzf syntax. Fortunately, the list is short:

- `^` is a prefix exact match. To search for a phrase starting with "welcome": `^welcome`.
- `$` is a suffix exact match. To search for a phrase ending with "my friends": `friends$`.
- `'` is an exact match. To search for the phrase "welcome my friends": `'welcome my friends`.
- `|` is an "or" match. To search for either "friends" or "foes": `friends | foes`.
- `!` is an inverse match. To search for phrase containing "welcome" and not "friends": `welcome !friends`

You can mix and match these options. For example, `^hello | ^welcome friends$` will search for the phrase starting with either "welcome" or "hello" and ending with "friends".

## Encontrando Arquivos

To search for files inside Vim using fzf.vim plugin, you can use the `:Files` method. Run `:Files` from Vim and you will be prompted with fzf search prompt.

Since you will be using this command frequently, it is good to have this mapped to a keyboard shortcut. I map mine to `Ctrl-f`. In my vimrc, I have this:

```
nnoremap <silent> <C-f> :Files<CR>
```

## Encontrando no Arquivo

To search inside files, you can use the `:Rg` command.

Again, since you will probably use this frequently, let's map it to a keyboard shortcut. I map mine to `<Leader>f`. The `<Leader>` key is mapped to `\` by default.

```
nnoremap <silent> <Leader>f :Rg<CR>
```

## Outras Buscas

Fzf.vim provides many other search commands. I won't go through each one of them here, but you can check them out [here](https://github.com/junegunn/fzf.vim#commands).

Here's what my fzf maps look like:

```
nnoremap <silent> <Leader>b :Buffers<CR>
nnoremap <silent> <C-f> :Files<CR>
nnoremap <silent> <Leader>f :Rg<CR>
nnoremap <silent> <Leader>/ :BLines<CR>
nnoremap <silent> <Leader>' :Marks<CR>
nnoremap <silent> <Leader>g :Commits<CR>
nnoremap <silent> <Leader>H :Helptags<CR>
nnoremap <silent> <Leader>hh :History<CR>
nnoremap <silent> <Leader>h: :History:<CR>
nnoremap <silent> <Leader>h/ :History/<CR>
```

## Substituindo o Comando Grep Pelo Rg

As mentioned earlier, Vim has two ways to search in files: `:vim` and `:grep`. `:grep` uses external search tool that you can reassign using the `grepprg` keyword. I will show you how to configure Vim to use ripgrep instead of terminal grep when running the `:grep` command.

Now let's setup `grepprg` so that the `:grep` Vim command uses ripgrep. Add this in your vimrc:

```
set grepprg=rg\ --vimgrep\ --smart-case\ --follow
```

Feel free to modify some of the options above! For more information on what the options above mean, check out `man rg`.

After you updated `grepprg`, now when you run `:grep`, it runs `rg --vimgrep --smart-case --follow` instead of `grep`.  If you want to search for "donut" using ripgrep, you can now run a more succinct command `:grep "donut"` instead of `:grep "donut" . -R` 

Just like the old `:grep`, this new `:grep` also uses quickfix to display results.

You might wonder, "Well, this is nice but I never used `:grep` in Vim, plus can't I just use `:Rg` to find phrases in files? When will I ever need to use `:grep`?

That is a very good question. You may need to use `:grep` in Vim to do search and replace in multiple files, which I will cover next.

## Procurando e Substituindo em Multiplos Arquivos

Modern text editors like VSCode makes it very easy to search and replace a string across multiple files. In this section, I will show you two different methods to easily do that in Vim.

The first method is to replace *all* matching phrases in your project. You will need to use `:grep`. If you want to replace all instances of "pizza" with "donut", here's what you do:

```
:grep "pizza"
:cfdo %s/pizza/donut/g | update
```

Let's break down the commands:

1. `:grep pizza` uses ripgrep to search for all instances of "pizza" (by the way, this would still work even if you didn't reassign `grepprg` to use ripgrep. You would have to do `:grep "pizza" . -R` instead of `:grep "pizza"`).
2. `:cfdo` executes any command you pass to all files in your quickfix list. In this case, your command is the substitution command `%s/pizza/donut/g`. The pipe (`|`) is a chain operator. The `update` command saves each file after substitution. I will cover the substitute command in more depth in a later chapter.

The second method is to search and replace in selected files. With this method, you can manually choose which files you want to perform select-and-replace on. Here is what you do:

1. Clear your buffers first. It is imperative that your buffer list contains only the files you want to apply the replace on. You can either restart Vim or run `:%bd | e#` command (`%bd` deletes all the buffers and `e#` opens the file you were just on).
2. Run `:Files`.
3. Select all files you want to perform search-and-replace on. To select multiple files, use `<Tab>` / `<Shift-Tab>`. This is only possible if you have the multiple flag (`-m`) in `FZF_DEFAULT_OPTS`.
4. Run `:bufdo %s/pizza/donut/g | update`. The command `:bufdo %s/pizza/donut/g | update` looks similar to the earlier `:cfdo %s/pizza/donut/g | update` command. The difference is instead of substituting all quickfix entries (`:cfdo`), you are substituting all buffer entries (`:bufdo`).

## Learn Search the Smart Way

Searching is the bread-and-butter of text editing. Learning how to search well in Vim will improve your text editing workflow significantly.

Fzf.vim is a game-changer. I can't imagine using Vim without it. I think it is very important to have a good search tool when starting Vim. I've seen people struggling to transition to Vim because it seems to be missing critical features modern text editors have, like an easy and powerful search feature. I hope this chapter will help you to make the transition to Vim easier. 

You also just saw Vim's extensibility in action - the ability to extend search functionality with a plugin and an external program. In the future, keep in mind of what other features you wish to extend Vim with. Chances are, it's already in Vim, someone has created a plugin or there is a program for it already. Next, you'll learn about a very important topic in Vim: Vim grammar.
