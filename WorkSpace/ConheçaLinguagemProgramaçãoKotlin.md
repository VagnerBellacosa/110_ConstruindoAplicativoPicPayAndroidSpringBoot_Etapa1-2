# Conheça a linguagem de programação Kotlin

 







[Kotlin](https://kotlinlang.org/) é uma linguagem de programação amplamente usada por desenvolvedores Android em qualquer lugar. Este tópico funciona como um curso intensivo do Kotlin para você começar a usá-lo rapidamente.

## Declaração de variável

O Kotlin usa duas palavras-chave diferentes para declarar variáveis: `val` e `var`.

- Use `val` para uma variável cujo valor nunca muda. Não é possível reatribuir um valor a uma variável que tenha sido declarada usando `val`.
- Use `var` para uma variável cujo valor possa ser mudado.

No exemplo abaixo, `count` é uma variável do tipo `Int` que recebe um valor inicial de `10`:

```kotlin
var count: Int = 10
```

`Int` é um tipo que representa um número inteiro, um dos muitos tipos numéricos que podem ser representados em Kotlin. Assim como acontece com outras linguagens, você também pode usar `Byte`, `Short`, `Long`, `Float` e `Double`, dependendo dos seus dados numéricos.

A palavra-chave `var` significa que você pode reatribuir valores a `count` conforme necessário. Por exemplo, você pode mudar o valor de `count` de `10` para `15`:

```kotlin
var count: Int = 10
count = 15
```

No entanto, alguns valores não podem ser mudados. Considere um `String` chamado `languageName`. Se você quiser garantir que `languageName` sempre tenha o valor "Kotlin", poderá declarar `languageName` usando a palavra-chave `val`:

```kotlin
val languageName: String = "Kotlin"
```

Essas palavras-chave permitem que você seja explícito sobre o que pode ser mudado. Use-as em seu favor conforme necessário. Se uma referência de variável precisar ser reatribuível, declare-a como `var`. Do contrário, use `val`.

### Inferência de tipo

Continuando com o exemplo anterior, quando você atribui um valor inicial a `languageName`, o compilador Kotlin pode inferir o tipo com base no tipo do valor atribuído.

Como o valor de `"Kotlin"` é do tipo `String`, o compilador infere que `languageName` também é um `String`. O Kotlin é uma linguagem *estática*. Isso significa que o tipo é resolvido no momento da compilação e nunca muda.

No exemplo a seguir, `languageName` é inferido como `String`. Portanto, não é possível chamar nenhuma função que não faça parte da classe `String`:

```kotlin
val languageName = "Kotlin"
val upperCaseName = languageName.toUpperCase()

// Fails to compile
languageName.inc()
```

`toUpperCase()` é uma função que só pode ser chamada em variáveis do tipo `String`. Como o compilador Kotlin inferiu `languageName` como `String`, você pode chamar `toUpperCase()` com segurança. `inc()`, entretanto, é uma função de operador `Int`, por isso não pode ser chamada em `String`. A abordagem do Kotlin para a inferência de tipos oferece concisão e segurança de tipos.

### Segurança nula

Em algumas linguagens, uma variável de tipo de referência pode ser declarada sem fornecer um valor explícito inicial. Nesses casos, as variáveis geralmente contêm um valor nulo. Por padrão, as variáveis do Kotlin não podem reter valores nulos. Isso significa que o snippet a seguir é inválido.

```kotlin
// Fails to compile
val languageName: String = null
```

Para que uma variável mantenha um valor nulo, ela precisa ser do tipo *anulável*. Você pode especificar uma variável como sendo anulável, usando um sufixo do tipo com `?`, conforme mostrado neste exemplo a seguir.

```kotlin
val languageName: String? = null
```

Com um tipo `String?`, você pode atribuir um valor `String` ou `null` a `languageName`.

Você precisa lidar com variáveis anuláveis com cuidado ou corre o risco de ter um `NullPointerException`. Em Java, por exemplo, se você tentar invocar um método em um valor nulo, seu programa falhará.

O Kotlin fornece uma série de mecanismos para trabalhar com segurança com variáveis anuláveis. Para ver mais informações, consulte [Padrões comuns do Kotlin no Android: anulação](https://developer.android.com/kotlin/common-patterns?hl=pt-br#nullability) (link em inglês).

## Condicionais

O Kotlin apresenta vários mecanismos para implementar a lógica condicional. O mais comum deles é uma *instrução if-else*. Se uma expressão entre parênteses ao lado de uma palavra-chave `if` for avaliada como `true`, o código dentro dessa ramificação (ou seja, o código imediatamente seguinte que é encapsulado entre chaves) será executado. Caso contrário, será executado o código dentro da ramificação `else`.

```kotlin
if (count == 42) {
    println("I have the answer.")
} else {
    println("The answer eludes me.")
}
```

Você pode representar várias condições usando `else if`. Isso permite representar uma lógica mais granular e complexa em uma única instrução condicional, conforme mostrado neste exemplo:

```kotlin
if (count == 42) {
    println("I have the answer.")
} else if (count > 35) {
    println("The answer is close.")
} else {
    println("The answer eludes me.")
}
```

As instruções condicionais são úteis para representar a lógica com estado, mas você pode se repetir ao gravá-las. No exemplo acima, você simplesmente imprime um `String` em cada ramificação. Para evitar essa repetição, o Kotlin oferece *expressões condicionais*. O último exemplo pode ser regravado da seguinte forma:

```kotlin
val answerString: String = if (count == 42) {
    "I have the answer."
} else if (count > 35) {
    "The answer is close."
} else {
    "The answer eludes me."
}

println(answerString)
```

Implicitamente, cada ramificação condicional retorna o resultado da expressão na linha final, de modo que não é necessário usar uma palavra-chave `return`. Como o resultado das três ramificações é do tipo `String`, o resultado da expressão if-else também é do tipo `String`. Neste exemplo, `answerString` recebe um valor inicial do resultado da expressão if-else. A inferência de tipos pode ser usada para omitir a declaração de tipo explícito para `answerString`, mas geralmente é uma boa ideia incluí-la para fins de clareza.

**Observação:** o Kotlin não inclui um [operador ternário](https://en.wikipedia.org/wiki/%3F:) tradicional, favorecendo o uso de expressões condicionais.

Conforme a complexidade da instrução condicional aumenta, é recomendável substituir a expressão if-else por uma expressão *when*, conforme mostrado neste exemplo:

```kotlin
val answerString = when {
    count == 42 -> "I have the answer."
    count > 35 -> "The answer is close."
    else -> "The answer eludes me."
}

println(answerString)
```

Cada ramificação em uma expressão `when` é representada por uma condição, uma seta (`->`) e um resultado. Se a condição no lado esquerdo da seta for avaliada como verdadeira, o resultado da expressão no lado direito será retornado. Observe que a execução não passa de uma ramificação para a próxima. O código no exemplo de expressão `when` é funcionalmente equivalente ao do exemplo anterior, mas é mais fácil de ler.

As condicionais do Kotlin destacam um dos recursos mais avançados, a *transmissão inteligente*. Em vez de usar o operador de chamada segura ou o operador de declaração não nulo para trabalhar com valores anuláveis, você pode verificar se uma variável contém uma referência a um valor nulo usando uma instrução condicional, conforme mostrado neste exemplo:

```kotlin
val languageName: String? = null
if (languageName != null) {
    // No need to write languageName?.toUpperCase()
    println(languageName.toUpperCase())
}
```

Na ramificação condicional, `languageName` pode ser tratado como não anulável. O Kotlin é inteligente o suficiente para reconhecer que a condição para executar a ramificação é que `languageName` não contenha um valor nulo. Portanto, você não precisa tratar `languageName` como anulável nessa ramificação. Essa transmissão inteligente funciona para verificações nulas, [verificações de tipo](https://kotlinlang.org/docs/reference/typecasts.html#is-and-is-operators) ou qualquer condição que satisfaça a um [contrato](https://kotlinlang.org/docs/reference/whatsnew13.html#contracts).

## Funções

Você pode agrupar uma ou mais expressões em uma *função*. Em vez de repetir a mesma série de expressões sempre que precisar de um resultado, você pode unir as expressões em uma função e chamar essa função.

Para declarar uma função, use a palavra-chave `fun` seguida pelo nome da função. Em seguida, defina os tipos de entrada que sua função assume, se houver, e declare o tipo de saída retornada. No corpo de uma função, você define expressões que são chamadas quando sua função é invocada.

Com base nos exemplos anteriores, veja uma função completa do Kotlin:

```kotlin
fun generateAnswerString(): String {
    val answerString = if (count == 42) {
        "I have the answer."
    } else {
        "The answer eludes me"
    }

    return answerString
}
```

A função no exemplo acima tem o nome `generateAnswerString`. Não é necessária nenhuma entrada. Ela gera um resultado do tipo `String`. Para chamar uma função, use o nome dela, seguido pelo operador de invocação (`()`). No exemplo abaixo, a variável `answerString` é inicializada com o resultado de `generateAnswerString()`.

```kotlin
val answerString = generateAnswerString()
```

As funções podem receber argumentos como entrada, conforme mostrado neste exemplo:

```kotlin
fun generateAnswerString(countThreshold: Int): String {
    val answerString = if (count > countThreshold) {
        "I have the answer."
    } else {
        "The answer eludes me."
    }

    return answerString
}
```

Ao declarar uma função, você pode especificar qualquer número de argumentos e os tipos. No exemplo acima, `generateAnswerString()` leva um argumento chamado `countThreshold` do tipo `Int`. Dentro da função, você pode se referir ao argumento usando o nome dele.

Ao chamar essa função, você precisa incluir um argumento nos parênteses da chamada da função:

```kotlin
val answerString = generateAnswerString(42)
```

### Como simplificar declarações de função

`generateAnswerString()` é uma função bastante simples. A função declara uma variável e, em seguida, retorna imediatamente. Quando o resultado de uma única expressão é retornado de uma função, você pode ignorar a declaração de uma variável local retornando diretamente o resultado da expressão if-else contida na função, conforme mostrado neste exemplo:

```kotlin
fun generateAnswerString(countThreshold: Int): String {
    return if (count > countThreshold) {
        "I have the answer."
    } else {
        "The answer eludes me."
    }
}
```

Você também pode substituir a palavra-chave de retorno pelo operador de atribuição:

```kotlin
fun generateAnswerString(countThreshold: Int): String = if (count > countThreshold) {
        "I have the answer"
    } else {
        "The answer eludes me"
    }
```

### Funções anônimas

Nem todas as funções precisam de um nome. Algumas funções são identificadas mais diretamente por suas entradas e saídas. Essas funções são chamadas de *funções anônimas*. Você pode manter uma referência a uma função anônima usando essa referência para chamar a função anônima posteriormente. Você também pode passar a referência no seu app, como acontece com outros tipos de referência.

```kotlin
val stringLengthFunc: (String) -> Int = { input ->
    input.length
}
```

Assim como as funções nomeadas, as funções anônimas podem conter qualquer número de expressões. O valor retornado da função é o resultado da expressão final.

No exemplo acima, `stringLengthFunc` contém uma referência a uma função anônima que usa um `String` como entrada e retorna o comprimento da entrada `String` como saída do tipo `Int`. Por esse motivo, o tipo da função é denotado como `(String) -> Int`. No entanto, esse código não invoca a função. Para recuperar o resultado da função, você precisa invocá-lo como faria com uma função nomeada. Forneça um `String` ao chamar `stringLengthFunc`, conforme mostrado neste exemplo:

```kotlin
val stringLengthFunc: (String) -> Int = { input ->
    input.length
}

val stringLength: Int = stringLengthFunc("Android")
```

### Funções de ordem superior

Uma função pode usar outra função como um argumento. As funções que usam outras funções como argumentos são chamadas de *funções de ordem superior*. Esse padrão é útil para a comunicação entre componentes, da mesma forma que você pode usar uma interface de callback em Java.

Veja um exemplo de uma função de ordem superior:

```kotlin
fun stringMapper(str: String, mapper: (String) -> Int): Int {
    // Invoke function
    return mapper(str)
}
```

A função `stringMapper()` usa um `String` junto com uma função que deriva um valor `Int` de um `String` transmitido nela.

Você pode chamar `stringMapper()` passando um `String` e uma função que satisfaça o outro parâmetro de entrada, ou seja, uma função que usa um `String` como entrada e gera um `Int`, conforme mostrado neste exemplo:

```kotlin
stringMapper("Android", { input ->
    input.length
})
```

Se a função anônima for o *último* parâmetro definido em uma função, você poderá transmiti-la para fora dos parênteses usados para chamar a função, conforme mostrado neste exemplo:

```kotlin
stringMapper("Android") { input ->
    input.length
}
```

As funções anônimas podem ser encontradas em toda a biblioteca padrão do Kotlin. Para mais informações, consulte [Funções de ordem superior e lambdas](https://kotlinlang.org/docs/reference/lambdas.html) (link em inglês).

## Classes

Todos os tipos mencionados até agora estão integrados à linguagem de programação Kotlin. Se quiser adicionar um tipo personalizado, você poderá definir uma classe usando a palavra-chave `class`, conforme mostrado neste exemplo:

```kotlin
class Car
```

### Propriedades

As classes representam o estado usando propriedades. Uma [propriedade](https://kotlinlang.org/docs/reference/properties.html) é uma variável de nível de classe que pode incluir um getter, um setter e um campo de backup. Como um carro precisa de rodas para dirigir, você pode adicionar uma lista de objetos `Wheel` como uma propriedade de `Car`, conforme mostrado neste exemplo:

```kotlin
class Car {
    val wheels = listOf<Wheel>()
}
```

Observe que `wheels` é um `public val`, o que significa que `wheels` pode ser acessado de fora da classe `Car` e não pode ser reatribuído. Se você quiser ter uma instância de `Car`, primeiro é necessário chamar seu construtor. A partir daí, você pode acessar qualquer uma das propriedades acessíveis.

```kotlin
val car = Car() // construct a Car
val wheels = car.wheels // retrieve the wheels value from the Car
```

Se você quiser personalizar suas rodas, poderá definir um construtor personalizado que especifica como as propriedades de classe são inicializadas:

```kotlin
class Car(val wheels: List<Wheel>)
```

No exemplo acima, o construtor de classe usa um `List<Wheel>` como argumento do construtor e usa esse argumento para inicializar a propriedade `wheels`.

### Funções de classe e encapsulamento

As classes usam funções para modelar o comportamento. As funções podem modificar o estado, ajudando você a expor somente os dados que quer expor. Esse controle de acesso faz parte de um conceito maior orientado a objetos, conhecido como *encapsulamento*.

No exemplo a seguir, a propriedade `doorLock` é mantida privada em qualquer item fora da classe `Car`. Para desbloquear o carro, você precisa chamar a função `unlockDoor()` transmitindo uma chave válida, conforme mostrado neste exemplo:

```kotlin
class Car(val wheels: List<Wheel>) {

    private val doorLock: DoorLock = ...

    fun unlockDoor(key: Key): Boolean {
        // Return true if key is valid for door lock, false otherwise
    }
}
```

Se quiser personalizar a forma como uma propriedade é referenciada, você poderá fornecer getter e setter personalizados. Por exemplo, se quiser expor o getter de uma propriedade ao restringir o acesso a setter, você poderá designar esse setter como `private`:

```kotlin
class Car(val wheels: List<Wheel>) {

    private val doorLock: DoorLock = ...

    var gallonsOfFuelInTank: Int = 15
        private set

    fun unlockDoor(key: Key): Boolean {
        // Return true if key is valid for door lock, false otherwise
    }
}
```

Com uma combinação de propriedades e funções, você pode criar classes que modelam todos os tipos de objeto.

## Interoperabilidade

Uma das características mais importantes do Kotlin é a interoperabilidade fluida com Java. Como o código Kotlin é compilado até o bytecode da JVM, seu código Kotlin pode ser chamado diretamente no código Java e vice versa. Isso significa que você pode aproveitar bibliotecas Java já existentes diretamente do Kotlin. Além disso, a maioria das APIs do Android é gravada em Java, e você pode chamá-las diretamente do Kotlin.

## A seguir

Kotlin é uma linguagem flexível e pragmática com compatibilidade e dinâmica crescentes. Encorajamos você a testá-la se ainda não tiver feito isso. Para as próximas etapas, confira a [documentação oficial do Kotlin](https://kotlinlang.org/) (link em inglês) com o guia sobre como aplicar [padrões comuns do Kotlin](https://developer.android.com/kotlin/common-patterns?hl=pt-br) nos seus apps para Android.