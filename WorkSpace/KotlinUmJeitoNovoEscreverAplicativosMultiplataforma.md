![Aprenda Kotlin](https://abacomm.com.br/wp-content/uploads/2018/05/aprenda-kotlin-862x575.jpg)

# Kotlin: Um jeito novo de escrever aplicativos multiplataforma.

------

**D**esde que foi criado pela JetBrains em 2011, o Kotlin vem ganhando notoriedade entre a comunidade de desenvolvedores, principalmente depois que o Google apresentou a linguagem como alternativa para o desenvolvimento de aplicações nativas na plataforma Android, onde a mesma passou a fazer parte integral do Android Studio desde sua versão 3.0 em diante. O principal objetivo do projeto – segundo seus criadores – era criar uma linguagem que fosse moderna e que atendesse às expectativas dos desenvolvedores em ambientes de desenvolvimento igualmente modernos. O Kotlin bebe muito na fonte de linguagens como Scala e, até mesmo, o próprio Java. No entanto, tenta eliminar aquilo que enxergamos como desvantagem nesses ambientes.



Outros pontos importantes sobre a linguagem:

- **Código conciso:** Segundo testes, códigos escritos em Kotlin resultam em 30% menos código do que em Java;
- **Null-safety:** O compilador do Kotlin detecta erros de NullPointerException que em Java são detectados apenas em runtime;
- **Interoperabilidade:** Kotlin permite o uso de bibliotecas Java e também pode se mesclar com códigos escritos em Java dentro do mesmo projeto!
- **Aplicações web:** O Kotlin é interpretado na JVM mas também pode ser compilado em código JavaScript, o que significa que pode ser utilizada para o desenvolvimento Frontend! (A implementação atual corresponde ao código ECMAScript 5 mas já há planos para suportar a versão 6 em breve);
- **Múltiplos paradigmas:** Kotlin permite a escrita de código em padrão OOP ao mesmo tempo em que suporta o paradigma funcional.

Nesse rápido tutorial nós vamos aprender conceitos básicos da linguagem para que você se familiarize com sua sintaxe e, quem sabe, sinta-se compelido à utilizar o Kotlin em seus projetos Android. Vamos lá?

### Mão na massa

------

Meu intuito aqui é tentar fornecer uma visão geral da sintaxe e principais conceitos para que você possa começar à se aventurar com a linguagem. Eu vou assumir que você já tem intimidade com alguma linguagem de programação, como Java ou qualquer outra. Portanto, vou pular os aspectos mais básicos e focar apenas no objetivo primário da nossa proposta.

Existem várias maneiras de se [escrever código Kotlin](https://www.devmedia.com.br/guia/linguagem-kotlin/40739), como utilizar a **IntelliJ IDE**, o já mencionado **Android Studio** ou fazendo o download direto do seu compilador. Mas para manter esse tutorial o mais simples possível, iremos utilizar apenas o seu ótimo editor online: o **Kotlin Playground**.

[![Kotlin Playground](https://abacomm.com.br/wp-content/uploads/2018/05/site-kotlin-playground.png)](https://abacomm.com.br/wp-content/uploads/2018/05/site-kotlin-playground.png)

Ao lado esquerdo do Playground nós temos uma lista de códigos com exemplos de uso da linguagem que vão desde os mais simples até sugestões de problemas e bugs que podem ser resolvidos (o que é ótimo para quem quer se aprofundar com exemplos e desafios reais). Mas vamos focar no primeiro dos exemplos, o simples e clássico “Hello World”:

```
fun main(args: Array) {
    println("Hello, world!”)
}
```

A primeira coisa que chama a atenção é a palavra **fun**, responsável por definir uma nova função no Kotlin. O **main** é a função principal, que serve de ponto de entrada de qualquer programa. **args** representa o parâmetro principal da função (caso exista) e o **println** é uma função de linha de comando que escreve o resultado na tela e posiciona o cursor na linha seguinte.

Vejamos como seria a mesma função **main** escrita em Java:

```
public static void main(String[] args) {
    System.out.println("Hello World!”);
}
```

Mesmo com um exemplo tão simples quanto esse, já conseguimos notar a diferença de sintaxe e uma ligeira redução na quantidade de código necessário.

Vamos nos aprofundar nos conceitos por trás dessa sintaxe para entender suas vantagens.

### Variáveis e objetos

------

O Kotlin é uma linguagem de tipagem estática, porém, o compilador pode inferir o tipo da variável em alguns cenários em que o mesmo for omitido. Tipos primitivos em Kotlin são os mesmos presentes no Java (Int, Long, Double, Float, Short e Byte). Porém, diferente do Java, Kotlin não faz casting implícito.

Podemos declarar variáveis de duas formas: **var** para variáveis e **val** para constantes. Vejamos:

```
fun main(args: Array) {
    val a : Int = 10     // tipo inteiro
    val b = 10           // tipo inteiro
    val c : Float        // tipo float
    val d                // erro de compilação
    b = c                // erro de compilação
}
```

Observe que nas três primeiras linhas temos exemplos de como as variáveis podem ser declaradas: Na variável **a** temos o tipo inteiro declarado de maneira estática, enquanto na variável **b** ele é implícito, uma vez que foi omitido. Na variável **c** nós declaramos o seu tipo, porém, não inicializamos com nenhum valor.

Na variável **d** temos um erro de compilação porque, sem declarar o valor, o compilador não consegue inferir nenhum tipo à variável, enquanto que na última linha simulamos o já citado problema de casting implícito que aqui não é suportado.

No entanto, o Kotlin suporta métodos de casting explícito, como **toByte**, **toLong**, **toChar**, etc. O código abaixo, por exemplo, compila sem maiores problemas:

```
c = b.toFloat()
```

#### Strings

Similar aos tipos primitivos, podemos declarar uma String de maneira explícita ou implícita. As novidades aqui ficam por conta do chamado **Raw String** (algo como “String bruta”) que pode aceitar tanto espaços em branco quanto quebras de linhas, diminuindo assim a necessidade de concatenação e o **String Template**, que é a técnica onde inserimos uma variável (ou expressão) dentro de uma String.

Vejamos os exemplos abaixo:

```
fun main(args: Array) {
	val message = """
   	Hi, there!
        
   	Are you enjoying the tut?
    Let us know in the comments section below.
    
    Best regards,
	""" // -> raw string 1
    
    val author = """
	Carlos Cabral
    """ // -> raw string 2
    
    println("Message is: $message $author") // -> string template
    
}
```

### Controle de fluxo

------

Condicionais em Kotlin funcionam de maneira bem similar ao Java com uma diferença interessante: Eles também podem retornar valores que podem ser atribuídos à variáveis, veja:

```
var isOdd = if ((7 % 2) == 0) true else false
println(isOdd) // false
```

É importante ressaltar que nesse tipo de operação a cláusula **else** é sempre obrigatória, pois é preciso garantir que algum valor será atribuído:

```
var isOdd = if ((7 % 2) == 0) true 
println(isOdd) // erro de compilação
```

Em Kotlin a cláusula **switch** foi substituída por **when**, onde o teste dos valores é feito por uma seta no lugar do **case**, conforme abaixo:

```
fun main(args: Array) {
    var x = (Math.random() * 10).toInt()
    println(x)
    when (x) {
        0 -> print("x is 0")
        1 -> print("x is 1")
        2 -> print("x is 2")
        3,4,5 -> print("x is between 3 or 5")
        else -> { 
            print("x is equal or greater than 6")
        }
    }
}
```

Percebam que cada teste é executado de maneira exclusiva, sem a necessidade de se usar o **break**. O default é a cláusula **else**, executada quando nenhuma das condições acima são atendidas.



Saiba como contratar o seu aplicativo da forma correta!

Baixe agora o PDF gratuitamente.

[Clique aqui](https://abacomm.com.br/inbound/wb7iuv8)



### Estruturas de repetição

------

Kotlin, assim como Java, utiliza **while**, **do-while** e **for** como blocos de repetição. Os dois primeiros funcionam exatamente da mesma maneira como no Java, porém, o **for** se comporta de maneira similar à um **for each** do JavaScript:

```
for (item in collection) print(item)
```

As Strings também são tratadas como objetos de iteração, onde cada letra é considerado um caracter isolado:

```
var company = "abacomm"
for(letter in company)
   println(letter)

// a
// b
// a
// c
// o
// m
// m
```

Também podemos utilizar a propriedade indices para acessar o índice correspondente à cada letra do array de objetos, vejamos:

```
for(index in company.indices)
   print(index) // 0123456
```

Ou podemos também retornar uma coleção de tuplas com o método **whithindex**:

```
for((index, letter) in company.withIndex())
   print("$index:($letter) ") // 0:(a) 1:(b) 2:(a) 3:(c) 4:(o) 5:(m) 6:(m)
```

Podemos também utilizar a expressão **range**, que promove a iteração entre números ou caracteres dentro de um loop **for** utilizando a palavra reservada in. O **range** pode ser representado pelo operador de dois pontos, veja:

```
for(i in 1..5)
   println(i)

// 1
// 2
// 3
// 4
// 5
```

Ou podemos excluir o último número utilizando **until**:

```
for(i in 1 until 5)
   println(i)

// 1
// 2
// 3
// 4
```

Também podemos iterar de maneira reversa substituindo o **..** pela expressão **downTo**:

```
for(i in 5 downTo 1)
   print("$i, ") // 5 4 3 2 1
```

Que também funciona com strings:

```
for(i in 'z' downTo 'a')
   print("$i ") // z y x w v u t s r q p o n m l k j i h g f e d c b a  
```

Podemos também trabalhar com **range** em conjunto com a expressão **step** para iterar com intervalos arbitrários:

```
for(i in 1..20 step 4)
   println(i)

// 1
// 5
// 9
// 13
// 17
```

### Lambda

------

É muito comum utilizar funções que se comportam como variáveis em linguagens funcionais, como, por exemplo, retornar uma função ao invés de um valor ou passar uma função como parâmetro de uma outra função. Em Kotlin esse tipo de comportamento é chamado de **expressões lambda** (lambda expressions). Considere o código abaixo:

```
val performCalc = {
   println(5 + 4) // 9
}
performCalc()
```

Expressões lambda são denotadas por chaves. No código acima nós invocamos a expressão chamado o método **performCalc**. Perceba que ao invés de **func** estamos utilizando **val**.

Podemos também passar parâmetros para uma expressão lambda:

```
val performCalc = { number1: Int, number2: Int ->
   println(number1 + number2) // 9
}
performCalc(5, 4)
```

No exemplo acima, o sinal de **->** indica o fim da lista de parâmetros e o início do corpo da função. O compilador fica responsável por inferir o valor retornado.

Além dos tipos existentes presentes em uma linguagem, temos também o tipo Função. Considere o exemplo abaixo:

```
val f: (String) -> Int
```

A variável **f** representa uma função que recebe um parâmetro do tipo String e retorna um inteiro. Em Kotlin podemos atribuir uma expressão lambda à funções do tipo Função, conforme abaixo:

```
var f: (String) -> Int = { x -> x.toInt() * x.toInt() }
print(f(“2")) // 4
```

A partir do exemplo acima podemos atribuir a variável **f** à uma nova expressão lambda, que vai gerar um resultado diferente ao ser invocado novamente.

### Convertendo de Java para Kotlin

------

Talvez uma das características mais interessantes da linguagem seja sua capacidade de converter código Java de maneira muito transparente. Isso significa que você pode copiar códigos Java de seus projetos existentes e convertê-los para a sintaxe do Kotlin imediatamente, o que pode ser uma excelente mão na roda para acelerar seu aprendizado. Você pode, inclusive, fazer isso pelo editor online clicando no botão **Converter para Kotlin**, conforme imagem abaixo:

Em seguida, basta colar seu código Java no bloco da esquerda e depois clicar no botão pra ver a mágica acontecendo:

[![Imagem converter Java para Kotlin](https://abacomm.com.br/wp-content/uploads/2018/05/converter-java-kotlin.png)](https://abacomm.com.br/wp-content/uploads/2018/05/converter-java-kotlin.png)

> O código Java do exemplo acima foi retirado do site [Java Coding Samples](https://www.cs.utexas.edu/~scottm/cs307/javacode/codeSamples/Factorial.java).

### Conclusão

------

Com características de linguagem funcional, promessa de interoperabilidade entre outras linguagens da JVM e facilidade de uso, o Kotlin vem adquirindo cada vez mais popularidade entre os desenvolvedores. Soma-se à isso o bom suporte oferecido pela JetBrains e você tem a oportunidade de desenvolver seu app em um ambiente menos verboso, mais produtivo e muito mais divertido!

Aqui na [Abacomm](https://www.abacomm.com.br/) temos testado o Kotlin em ambientes de produção com resultados muito positivos. E você, já o adotou em seus projetos? Sinta-se à vontade para compartilhar sua experiência nos comentários.