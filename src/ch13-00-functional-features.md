# Functional Language Features: Iterators and Closures

# Recursos de linguagens funcionais: Iterators e Closures

Rust’s design has taken inspiration from many existing languages and
techniques, and one significant influence is *functional programming*.
Programming in a functional style often includes using functions as values by
passing them in arguments, returning them from other functions, assigning them
to variables for later execution, and so forth.

O design de Rust se inspirou em muitos idiomas e técnicas existentes, e uma
influência significante é *programação funcional*. Programar em um estilo
funcional geralmente inclui usar funções como valores passando-os como
argumentos, retornando-as de outras funções, atribuindo-os para variáveis para
serem executadas posteriormente, e assim por diante.

In this chapter, we won’t debate the issue of what functional programming is or
isn’t but will instead discuss some features of Rust that are similar to
features in many languages often referred to as functional.

Neste capítulo, nós não discutiremos a questão o que é programação funcional ou
não, porém vamos discutir alguns recursos do Rust que são semelhantes a
outros recursos em várias linguagens frequentemente referido como funcionais.

More specifically, we’ll cover:

Mais especificamente, abordaremos:

* *Closures*, a function-like construct you can store in a variable
* *Iterators*, a way of processing a series of elements
* How to use these two features to improve the I/O project in Chapter 12
* The performance of these two features (Spoiler alert: they’re faster than you
  might think!)

* *Closures*, Um construtor estilo função no qual você pode guardar em uma
variável;
* *Iterators*, Uma maneira de processar uma séries de elementos;
* Como usar esses 2 recursos para melhorar o projeto "I/O" do Capítulo 12;
* A performance destes dois recursos (Alerta de Spoiler: Eles são mais rápidos
do que você pensa!).

Other Rust features, such as pattern matching and enums, which we’ve covered in
other chapters, are influenced by the functional style as well. Mastering
closures and iterators is an important part of writing idiomatic, fast Rust
code, so we’ll devote this entire chapter to them.

Outros recursos da linguagem Rust, como "pattern matching" e "enums", os quais
são abordados em outros capítulos, são também influenciados pelo estilo
funcional. Maestria em "closures" e "iterators" é um aspecto importante para
escrita de Rust de forma idiomática e rápido.
