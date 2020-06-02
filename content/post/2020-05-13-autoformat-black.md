+++
title = "Autoformatação de Código ― Quando e Como"
date = 2020-05-13T20:49:47-03:00
description = "Uma descrição da minha jornada desde odiar formatadores automáticos até usá-los em tudo"
draft = false
toc = true
categories = ["python", "estilo de código"]
tags = ["black", "python"]
images = [] # overrides site-wide open graph image
[[copyright]]
  owner = "Felipe Bidu"
  date = "2020"
  license = "cc-by-nc-sa-4.0"
+++

{{% hackcss-alert type="info" %}}
<strong>TL; DR:</strong> Use o índice para ir direto ao ponto que você quer, por
exemplo [Reformatando com Black!]({{< relref "2020-05-13-autoformat-black.md#reformatando-com-black" >}})
{{% /hackcss-alert %}}

## Introdução

Fazer um código legível e bonito é algo importante no nosso dia-a-dia, especialmente
quando colaboramos com demais colegas. Um código que é mal formatado e mal estruturado
é cansativo de ler e difícil de entender o que ele faz.

A formatação de um código é só um pedaço da história, mas um pedaço muito importante.
Ela não é a única responsável pela legibilidade do código ― um projeto mal estruturado,
com classe e funções com responsabilidades mal definidas pode até ser bem formatado,
mas será difícil de compreender.

No começo de minha carreira, eu não gostava muito da ideia de formatação automática
de código. Por um lado, eu acho que eu era um pouco orgulhoso em relação à isso
e não estava afim de perder "controle". Por outro lado, mais de 10 anos atrás,
essas ferramentas não eram exatamente boas, pelo menos não as que eu conhecia.

<!--more-->

Quanto ao primeiro argumento, hoje eu vejo essa resistência à perda do "controle"
como algo meio bobo. A formatação do código por mais importante que seja, não é
exatamente o foco do trabalho de um desenvolvedor ou sua finalidade. O centro
do nosso trabalho é a abstração de problemas do mundo real para dentro de um
algoritmo e a solução desse problema através de ferramentas computacionais, levando
em consideração as restrições de projeto que temos.

Além disso, se nosso código deve seguir um livro de estilo padronizado, qual é o mérito
em gastar tempo e esforço para cumprir um padrão definido? Humanos são bons em
criar, em fazer coisas novas. É aí que mora nosso diferencial, o que for automatizável,
podemos deixar que o computador faça por nós e nos deixe mais livres.

Já em relação ao segundo argumento, hoje nós temos diversas ferramentas de
formatação automática que são cada vez mais robustas. O foco do restante desse
artigo é o [Black](https://github.com/psf/black), um formatador automático para
Python.

## Black

A instalação do Black pode ser feita usando `pip install black` . Se seu
ambiente estiver bem configurado, você conseguirá digitar `black --help` em seu
terminal e ver algumas de suas opções de uso. Caso você queira, pode explorar
o funcionamento dele no [Black Playground](https://black.now.sh).

O Black permite diversas formas de execução. Podemos, por exemplo, invocá-lo direto
da linha de comando com uma string de código:

```
black -c "x=2  * 3+4"
```

E ele te retornará a expressão propriamente formatada, `x = 2 * 3 + 4` com os
espaços corretos.

## Explorando Erros com Pylint

Considere agora esse código, bem mal formatado mas ainda assim é válido:

```python  {linenos=table}
"""
Módulo que oferece um fatorial iterativo
"""

def fatorial(n):

    result=1

    for i in(range(1, n+1)):
        result*=i
    return result

print(  fatorial(   3))
print(fatorial(2     ))
print(fatorial(10     ))

```

Uma forma de avaliarmos a situação desse código é usando o `pylint` . Esse arquivo
está salvo em minha máquina como "black_sample01.py". Rodando `pylint black_sample01.py` ,
nos dá a seguinte resposta:

{{% hackcss-card header="Resultado do PyLint" %}}

```

************* Module black_sample01
black_sample01.py:2:10: C0326: Exactly one space required around assignment

    result=1

          ^ (bad-whitespace)

black_sample01.py:7:0: C0325: Unnecessary parens after 'in' keyword (superfluous-parens)
black_sample01.py:8:14: C0326: Exactly one space required around assignment

        result*=i
              ^^ (bad-whitespace)

black_sample01.py:11:5: C0326: No space allowed after bracket
print(  fatorial(   3))

     ^ (bad-whitespace)

black_sample01.py:11:16: C0326: No space allowed after bracket
print(  fatorial(   3))

                ^ (bad-whitespace)

black_sample01.py:12:21: C0326: No space allowed before bracket
print(fatorial(2     ))

                     ^ (bad-whitespace)

black_sample01.py:13:0: C0304: Final newline missing (missing-final-newline)
black_sample01.py:13:22: C0326: No space allowed before bracket
print(fatorial(10     ))

                      ^ (bad-whitespace)

black_sample01.py:1:0: C0114: Missing module docstring (missing-module-docstring)
black_sample01.py:1:0: C0103: Argument name "n" doesn't conform to snake_case naming style (invalid-name)
black_sample01.py:1:0: C0116: Missing function or method docstring (missing-function-docstring)

--------------------------------------------------------------------
Your code has been rated at -3.75/10

```

{{% /hackcss-card %}}

Existem erros de parênteses desnecessários, erros de espaço em branco e várias
linhas desnecessárias também. Podemos agora explorar o `black` .

## Verificando Possíveis Mudanças

Chamando o black com a flag `--check` , ele apenas nos conta se o arquivo seria
reformatado ou não. Isso é útil para usar em servidores de CI e evitar que arquivos
mal formatados subam para a base:

{{% hackcss-card %}}

```

$ black --check black_sample01.py

would reformat black_sample01.py
Oh no! 💥 💔 💥
1 file would be reformatted.

```

{{% /hackcss-card %}}

(_Mais sobre CI e ferramentas assim em breve!_)

Nós podemos usar a flag `--diff` para ver as mudanças que seriam feitas:

{{% hackcss-card header= `black --diff` %}}

```

$ black --diff black_sample01.py

--- black_sample01.py	2020-05-20 13:52:23.495800 +0000
+++ black_sample01.py	2020-05-20 13:55:56.786380 +0000
@@ -2, 19 +2, 16 @@
 Módulo que oferece um fatorial iterativo
 """

 def fatorial(n):

*    result=1
*    result = 1

-
-
-

*    for i in(range(1, n+1)):
*        result*=i
*    for i in range(1, n + 1):
*        result *= i

     return result

+print(fatorial(3))
+print(fatorial(2))
+print(fatorial(10))

-print(  fatorial(   3))
-print(fatorial(2     ))
-print(fatorial(10     ))
reformatted black_sample01.py
All done! ✨ 🍰 ✨
1 file reformatted.

```

{{% /hackcss-card %}}

Honestamente, é meio raro eu usar essas duas flags no meu dia-a-dia. Uma situação
em que eu costumo usar a `--diff` é quando eu estou trabalhando em uma base de
código pela primeira vez e estou pensando se vou ou não sugerir uma mudança de
estilo. Rodar o `--diff` me permite estimar o quão trabalhoso seria reformatar
a base toda. Já a flag `--check` eu uso em servidores de CI (Integração Contínua)
mesmo.

## Reformatando com Black!

E finalmente a reformatação é bem direta. Basta invocar o `black` sem flags
e com o arquivo:

```

$ black black_sample01.py

reformatted black_sample01.py
All done! ✨ 🍰 ✨
1 file reformatted.

```

O arquivo em si será modificado:

```python {linenos=table}

"""
Módulo que oferece um fatorial iterativo
"""

def fatorial(n):
    result = 1

    for i in range(1, n + 1):
        result *= i
    return result

print(fatorial(3))
print(fatorial(2))
print(fatorial(10))

```

## Usando Black no dia-a-dia

Okay, formatamos _um_ arquivo! Mas como garantir isso ao longo do processo?
Eu gosto de usar três abordagens diferentes:

* Direto no meu editor de texto
* Como um hook _pre-commit_ no git
* No servidor de CI

A parte do hook e do CI merecem um artigo só pra elas. Quanto ao editor, seja
lá qual for seu editor, ele
[provavelmente já tem uma integração com o Black](https://github.com/psf/black/blob/master/docs/editor_integration.md).

Ultimamente, tenho usado o [VSCode](https://code.visualstudio.com/) como editor
principal. A configuração no VSCode pode ser global ou específica de projeto.
Uma vez habilitado, você pode autoformatar seu código apertando `CTRL + Shift + I`

### Configuração Global

No caso da configuração global, abra as configurações em `File > Preferences > Settings` ou digite `CTRL + ,` . Na barra de busca, digite "python formatting provider".
Você deverá ver essa opção:

{{< figure

    src="/img/vscode-black.png"
    title="Configuração do Black no VSCode"

>}}

Selecione "black" na lista e pronto!

### Configuração Local

A configuração local de projetos é feita através do arquivo `.vscode/settings.json` .
Basta adicionar nele a chave `"python.formatting.provider": "black"` para habilitar
o black no seu projeto atual.

## Autoformatar ao salvar ou não?

Tudo isso que a gente fez habilitou a invocação `manual` do Black ― você precisa
apertar `CTRL + Shift + I` para formatar o código. O VSCode e muitos outros
editores dão uma outra opção ― autoformatar ao salvar.

Particularmente, eu deixo essa opção desativada nas configurações globais. Muitas
vezes eu trabalho em código que envolve outros colegas, em projetos que não usam
black ou que simplesmente possuem um padrão de formatação diferente. Autoformatar
nesses casos é ruim.

Projetos com código "feio" ou seguindo um padrão de formatação diferente, podem
ser problemáticos. Mas mais problemático ainda é quebrar a consistência interna
do projeto. Usar `CamelCase` para dar nomes em variáveis não é comum em Python
e o `PyLint` vai reclamar disso. Porém, se você está trabalhando em um projeto
que usa CamelCase para tudo, mantenha a consistência e ― talvez ― proponha a
mudança de padrão em um segundo momento. Mas isso é assunto pra outro post!
