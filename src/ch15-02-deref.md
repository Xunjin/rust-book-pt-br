## Tratando Ponteiros Inteligentes como Referências Normais com a Trait `Deref`

Implementar a trait `Deref` nos permite personalizar o comportamento do
_operador de desreferência_ (_dereference operator_), `*` (que é diferente do
operador de multiplicação ou de glob). Implementando a `Deref` de tal modo que o
ponteiro inteligente possa ser tratado como uma referência normal, podemos
escrever código que opere sobre referências e usar esse código com ponteiros
inteligentes também.

Primeiro vamos ver como o `*` funciona com referências normais, e então vamos
tentar definir nosso próprio tipo a la `Box<T>` e ver por que o `*` não funciona
como uma referência no nosso tipo recém-criado. Vamos explorar como a trait
`Deref` torna possível aos ponteiros inteligentes funcionarem de um jeito
similar a referências. E então iremos dar uma olhada na funcionalidade de
_coerção de desreferência_ (_deref coercion_) e como ela nos permite trabalhar
tanto com referências quanto com ponteiros inteligentes.

### Seguindo o Ponteiro até o Valor com `*`

Uma referência normal é um tipo de ponteiro, e um jeito de pensar sobre um
ponteiro é como uma seta até um valor armazenado em outro lugar. Na Listagem
15-6, nós criamos uma referência a um valor `i32` e em seguida usamos o operador
de desreferência para seguir a referência até o dado:

<span class="filename">Arquivo: src/main.rs</span>

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<span class="caption">Listagem 15-6: Usando o operador de desreferência para
seguir uma referência a um valor `i32`</span>

A variável `x` contém um valor `i32`, `5`. Nós setamos `y` igual a uma
referência a `x`. Podemos conferir (coloquialmente, "assertar") que `x` é igual
a `5`. Contudo, se queremos fazer uma asserção sobre o valor em `y`, temos que
usar `*y` para seguir a referência até o valor ao qual `y` aponta (por isso
"desreferência"). Uma vez que desreferenciamos `y`, temos acesso ao valor
inteiro ao qual `y` aponta para podermos compará-lo com `5`.

Se em vez disso tentássemos escrever `assert_eq!(5, y);`, receberíamos este erro
de compilação:

```text
erro[E0277]: a trait bound `{integer}: std::cmp::PartialEq<&{integer}>` não foi
satisfeita
 --> src/main.rs:6:5
  |
6 |     assert_eq!(5, y);
  |     ^^^^^^^^^^^^^^^^^ não posso comparar `{integer}` com `&{integer}`
  |
  = ajuda: a trait `std::cmp::PartialEq<&{integer}>` não está implementada para
  `{integer}`
```

Comparar um número com uma referência a um número não é permitido porque eles
são de tipos diferentes. Devemos usar `*` para seguir a referência até o valor
ao qual ela está apontando.

### Usando `Box<T>` como uma Referência

Podemos reescrever o código na Listagem 15-6 para usar um `Box<T>` em vez de uma
referência, e o operador de desreferência vai funcionar do mesmo jeito que na
Listagem 15-7:

<span class="filename">Arquivo: src/main.rs</span>

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<span class="caption">Listagem 15-7: Usando o operador de desreferência em um
`Box<i32>`</span>

A única diferença entre a Listagem 15-7 e a Listagem 15-6 é que aqui nós setamos
`y` para ser uma instância de um box apontando para o valor em `x` em vez de uma
referência apontando para o valor de `x`. Na última asserção, podemos usar o
operador de desreferência para seguir o ponteiro do box do mesmo jeito que
fizemos quando `y` era uma referência. A seguir, vamos explorar o que tem de
especial no `Box<T>` que nos permite usar o operador de desreferência, criando
nosso próprio tipo box.

### Definindo Nosso Próprio Ponteiro Inteligente

Vamos construir um smart pointer parecido com o tipo `Box<T>` fornecido pela
biblioteca padrão para vermos como ponteiros inteligentes, por padrão, se
comportam diferente de referências. Em seguida, veremos como adicionar a
habilidade de usar o operador de desreferência.

O tipo `Box<T>` no fim das contas é definido como uma struct-tupla (_tuple
struct_) de um elemento, então a Listagem 15-8 define um tipo `MeuBox<T>` da
mesma forma. Também vamos definir uma função `new` como a definida no `Box<T>`:

<span class="filename">Arquivo: src/main.rs</span>

```rust
struct MeuBox<T>(T);

impl<T> MeuBox<T> {
    fn new(x: T) -> MeuBox<T> {
        MeuBox(x)
    }
}
```

<span class="caption">Listagem 15-8: Definindo um tipo `MeuBox<T>`</span>

Definimos um struct chamado `MeuBox` e declaramos um parâmetro genérico `T`,
porque queremos que nosso tipo contenha valores de qualquer tipo. O tipo
`MeuBox` é uma struct-tupla de um elemento do tipo `T`. A função `MeuBox::new`
recebe um argumento do tipo `T` e retorna uma instância de `MeuBox` que contém o
valor passado.

Vamos tentar adicionar a função `main` da Listagem 15-7 à Listagem 15-8 e
alterá-la para usar o tipo `MeuBox<T>` que definimos em vez de `Box<T>`. O
código na Listagem 15-9 não irá compilar porque o Rust não sabe como
desreferenciar `MeuBox`:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let x = 5;
    let y = MeuBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<span class="caption">Listagem 15-9: Tentando usar o `MeuBox<T>` do mesmo jeito
que usamos referências e o `Box<T>`</span>

Aqui está o erro de compilação resultante:

```text
erro[E0614]: tipo `MeuBox<{integer}>` não pode ser desreferenciado
  --> src/main.rs:14:19
   |
14 |     assert_eq!(5, *y);
   |                   ^^
```

Nosso tipo `MeuBox<T>` não pode ser desreferenciado porque não implementamos
essa habilidade nele. Para habilitar desreferenciamento com o operador `*`,
temos que implementar a trait `Deref`.

### Implementando a Trait `Deref` para Tratar um Tipo como uma Referência

Conforme discutimos no Capítulo 10, para implementar uma trait, precisamos
prover implementações para os métodos exigidos por ela. A trait `Deref`,
disponibilizada pela biblioteca padrão, requer que implementemos um método
chamado `deref` que pega emprestado `self` e retorna uma referência para os
dados internos. A Listagem 15-10 contém uma implementação de `Deref` que
agrega à definição de `MeuBox`:

<span class="filename">Arquivo: src/main.rs</span>

```rust
use std::ops::Deref;

# struct MeuBox<T>(T);
impl<T> Deref for MeuBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```

<span class="caption">Listagem 15-10: Implementando `Deref` no
`MeuBox<T>`</span>

A sintaxe `type Target = T;` define um tipo associado para a trait `Deref` usar.
Tipos associados são um jeito ligeiramente diferente de declarar um parâmetro
genérico, mas você não precisa se preocupar com eles por ora; iremos cobri-los
em mais detalhe no Capítulo 19.

Nós preenchemos o corpo do método `deref` com `&self.0` para que `deref` retorne
uma referência ao valor que queremos acessar com o operador `*`. A função `main`
na Listagem 15-9 que chama `*` no valor `MeuBox<T>` agora compila e as asserções
passam!

Sem a trait `Deref`, o compilador só consegue desreferenciar referências `&`. O
método `deref` dá ao compilador a habilidade de tomar um valor de qualquer tipo
que implemente `Deref` e chamar o método `deref` para pegar uma referência `&`,
que ele sabe como desreferenciar.

Quando entramos `*y` na Listagem 15-9, por trás dos panos o Rust na verdade
rodou este código:

```rust,ignore
*(y.deref())
```

O Rust substitui o operador `*` com uma chamada ao método `deref` e em seguida
uma desreferência comum, de modo que nós programadores não precisamos pensar
sobre se temos ou não que chamar o método `deref`. Essa funcionalidade do Rust
nos permite escrever código que funcione identicamente quando temos uma
referência comum ou um tipo que implementa `Deref`.

O fato de o método `deref` retornar uma referência ao valor, e a desreferência
comum fora dos parênteses em `*(y.deref())` ainda ser necessária, é devido ao
sistema de posse (_ownership_). Se o método `deref` retornasse o valor
diretamente em vez de uma referência ao valor, o valor seria movido para fora do
`self`. Nós não queremos tomar posse do valor interno do `MeuBox<T>` neste e na
maioria dos casos em que usamos o operador de desreferência.

Note que o `*` é substituído por uma chamada ao método `deref` e então uma
chamada ao `*` apenas uma vez, cada vez que digitamos um `*` no nosso código.
Como a substituição do `*` não entra em recursão infinita, nós terminamos com o
dado do tipo `i32`, que corresponde ao `5` em `assert_eq!` na Listagem 15-9.

### Coerções de Desreferência Implícitas com Funções e Métodos

_Coerção de desreferência_ (_deref coercion_) é uma conveniência que o Rust
aplica a argumentos de funções e métodos. A coerção de desreferência converte
uma referência a um tipo que implementa `Deref` em uma referência a um tipo ao
qual a `Deref` pode converter o tipo original. A coerção de desreferência
acontece automaticamente quando passamos uma referência ao valor de um tipo
específico como argumento a uma função ou método e esse tipo não corresponde ao
tipo do parâmetro na definição da função ou método. Uma sequência de chamadas ao
método `deref` converte o tipo que providenciamos no tipo que o parâmetro exige.

A coerção de desreferência foi adicionada ao Rust para que programadores
escrevendo chamadas a métodos e funções não precisassem adicionar tantas
referências e desreferências explícitas com `&` e `*`. A funcionalidade de
coerção de desreferência também nos permite escrever mais código que funcione
tanto com referências quanto com ponteiros inteligentes.

Para ver a coerção de desreferência em ação, vamos usar o tipo `MeuBox<T>` que
definimos na Listagem 15-8 e também a implementação de `Deref` que adicionamos
na Listagem 15-10. A Listagem 15-11 mostra a definição de uma função que tem um
parâmetro do tipo string slice:

<span class="filename">Arquivo: src/main.rs</span>

```rust
fn ola(nome: &str) {
    println!("Olá, {}!", nome);
}
```

<span class="caption">Listagem 15-11: Uma função `ola` que tem um parâmetro
`nome` do tipo `&str`</span>

Podemos chamar a função `ola` passando uma string slice como argumento, por
exemplo `ola("Rust");`. A coerção de desreferência torna possível chamar `ola`
com uma referência a um valor do tipo `MeuBox<String>`, como mostra a Listagem
15-12:

<span class="filename">Arquivo: src/main.rs</span>

```rust
# use std::ops::Deref;
#
# struct MeuBox<T>(T);
#
# impl<T> MeuBox<T> {
#     fn new(x: T) -> MeuBox<T> {
#         MeuBox(x)
#     }
# }
#
# impl<T> Deref for MeuBox<T> {
#     type Target = T;
#
#     fn deref(&self) -> &T {
#         &self.0
#     }
# }
#
# fn ola(name: &str) {
#     println!("Olá, {}!", name);
# }
#
fn main() {
    let m = MeuBox::new(String::from("Rust"));
    ola(&m);
}
```

<span class="caption">Listagem 15-12: Chamando `ola` com uma referência a um
valor `MeuBox<String>`, o que só funciona por causa da coerção de
desreferência</span>

Aqui estamos chamando a função `ola` com o argumento `&m`, que é uma referência
a um valor `MeuBox<String>`. Como implementamos a trait `Deref` em `MeuBox<T>`
na Listagem 15-10, o Rust pode transformar `&MeuBox<String>` em `&String`
chamando `deref`. A biblioteca padrão provê uma implementação de `Deref` para
`String` que retorna uma string slice, documentada na API de `Deref`. O Rust
chama `deref` de novo para transformar o `&String` em `&str`, que corresponde à
definição da função `ola`.

Se o Rust não implementasse coerção de desreferência, teríamos que escrever o
código na Listagem 15-13 em vez do código na Listagem 15-12 para chamar `ola`
com um valor do tipo `&MeuBox<String>`:

<span class="filename">Arquivo: src/main.rs</span>

```rust
# use std::ops::Deref;
#
# struct MeuBox<T>(T);
#
# impl<T> MeuBox<T> {
#     fn new(x: T) -> MeuBox<T> {
#         MeuBox(x)
#     }
# }
#
# impl<T> Deref for MeuBox<T> {
#     type Target = T;
#
#     fn deref(&self) -> &T {
#         &self.0
#     }
# }
#
# fn ola(name: &str) {
#     println!("Olá, {}!", name);
# }
#
fn main() {
    let m = MeuBox::new(String::from("Rust"));
    ola(&(*m)[..]);
}
```

<span class="caption">Listagem 15-13: O código que teríamos que escrever se o
Rust não tivesse coerção de desreferência</span>

O `(*m)` desreferencia o `MeuBox<String>` em uma `String`. Então o `&` e o
`[..]` obtêm uma string slice da `String` que é igual à string inteira para
corresponder à assinatura de `ola`. O código sem coerção de desreferência é mais
difícil de ler, escrever e entender com todos esses símbolos envolvidos. A
coerção de desreferência permite que o Rust lide com essas conversões
automaticamente para nós.

Quando a trait `Deref` está definida para os tipos envolvidos, o Rust analisa os
tipos e usa `Deref::deref` tantas vezes quanto necessário para chegar a uma
referência que corresponda ao tipo do parâmetro. O número de vezes que
`Deref::deref` precisa ser inserida é resolvido em tempo de compilação, então
não existe nenhuma penalidade em tempo de execução para tomar vantagem da
coerção de desreferência.

### Como a Coerção de Desreferência Interage com a Mutabilidade

De modo semelhante a como usamos a trait `Deref` para redefinir `*` em
referências imutáveis, o Rust provê uma trait `DerefMut` para redefinir `*` em
referências mutáveis.

O Rust faz coerção de desreferência quando ele encontra tipos e implementações
de traits em três casos:

- De `&T` para `&U` quando `T: Deref<Target=U>`;
- De `&mut T` para `&mut U` quando `T: DerefMut<Target=U>`;
- De `&mut T` para `&U` quando `T: Deref<Target=U>`.

Os primeiros dois casos são o mesmo exceto pela mutabilidade. O primeiro caso
afirma que se você tem uma `&T`, e `T` implementa `Deref` para algum tipo `U`,
você pode obter um `&U` de maneira transparente. O segundo caso afirma que a
mesma coerção de desreferência acontece para referências mutáveis.

O terceiro caso é mais complicado: o Rust também irá coagir uma referência
mutável a uma imutável. Mas o contrário _não_ é possível: referências imutáveis
nunca serão coagidas a referências mutáveis. Por causa das regras de empréstimo,
se você tem uma referência mutável, ela deve ser a única referência àqueles
dados (caso contrário, o programa não compila). Converter uma referência mutável
a uma imutável nunca quebrará as regras de empréstimo. Converter uma referência
imutável a uma mutável exigiria que houvesse apenas uma referência imutável
àqueles dados, e as regras de empréstimo não garantem isso. Portanto, o Rust não
pode assumir que converter uma referência imutável a uma mutável seja possível.
