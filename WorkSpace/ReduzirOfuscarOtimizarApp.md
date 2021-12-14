# Reduzir, ofuscar e otimizar o app

 







Para que o app seja o menor possível, ative a *redução*. Isso remover código e recursos não utilizados do seu build de lançamento. Ativando a redução, você também aproveita o *ofuscamento*, que reduz os nomes das classes e dos membros do seu app, e a *otimização*, que aplica estratégias mais agressivas para reduzir ainda mais o tamanho do app. Esta página descreve como o R8 realiza essas tarefas de tempo de compilação para seu projeto e como é possível personalizá-las.

Quando você cria o projeto usando o [Plug-in do Android para Gradle 3.4.0](https://developer.android.com/studio/releases/gradle-plugin?hl=pt-br#3-4-0) ou mais recente, o ProGuard não é mais usado para realizar a otimização do código durante o tempo de compilação. Em vez disso, o plug-in usará o *compilador R8* para processar as seguintes tarefas de tempo de compilação:

- **Redução de código (ou tree shaking):** detecta e remove com segurança classes, campos, métodos e atributos não utilizados do app e das dependências de biblioteca dele, o que a torna uma ferramenta valiosa para superar o [limite de 64 mil referências](https://developer.android.com/studio/build/multidex?hl=pt-br). Por exemplo, caso você use somente algumas APIs de uma dependência de biblioteca, a redução pode identificar o código da biblioteca que seu app *não* usa e remover apenas esse código do app. Para saber mais, vá para a seção sobre como [reduzir seu código](https://developer.android.com/studio/build/shrink-code?hl=pt-br#shrink-code).
- **Redução de recursos:** remove recursos não utilizados do app empacotado, incluindo os que estiverem em dependências da biblioteca do app. Ele trabalha em conjunto com a redução de código, de forma que, depois que um código não utilizado é removido, qualquer recurso não referenciado também pode ser removido com segurança. Para saber mais, vá para a seção sobre como [reduzir os recursos](https://developer.android.com/studio/build/shrink-code?hl=pt-br#shrink-resources).
- **Ofuscação:** encurta o nome de classes e membros, o que resulta em arquivos DEX menores. Para saber mais, acesse a seção sobre como [ofuscar o código](https://developer.android.com/studio/build/shrink-code?hl=pt-br#obfuscate).
- **Otimização:** inspeciona e reescreve o código para reduzir ainda mais o tamanho dos arquivos DEX do app. Por exemplo, se o R8 detectar que a ramificação `else {}` de determinada instrução "if/else" nunca é usada, ele removerá o código da ramificação `else {}`. Para saber mais, acesse a seção sobre [otimização de código](https://developer.android.com/studio/build/shrink-code?hl=pt-br#optimization).

Ao criar a versão de lançamento do seu app, o R8 executa automaticamente, por padrão, as tarefas de tempo de compilação descritas acima. No entanto, é possível desativar algumas tarefas ou personalizar o comportamento do R8 por meio dos arquivos de regras do ProGuard. Na verdade, o **R8 funciona com todos os arquivos de regras existentes do ProGuard**. Desse modo, a atualização do Plug-in do Android para Gradle para usar o R8 não exige que você altere as regras existentes.

## Ativar redução, ofuscação e otimização

Quando você usa o Android Studio 3.4 ou o Plug-in do Android para Gradle 3.4.0 e versões mais recentes, o R8 é o compilador padrão que converte o bytecode Java do projeto no formato DEX, que é executado na plataforma Android. No entanto, quando você cria um novo projeto usando o Android Studio, a redução, a ofuscação e a otimização de código não são ativadas por padrão. Isso ocorre porque essas otimizações em tempo de compilação aumentam o tempo de compilação do projeto e podem introduzir bugs se você não [personalizar suficientemente o código a ser mantido](https://developer.android.com/studio/build/shrink-code?hl=pt-br#keep-code).

Portanto, é melhor ativar essas tarefas de tempo de compilação ao criar a versão final do app que você testou antes de publicar. Para ativar a redução, ofuscação e otimização, inclua o seguinte no seu arquivo `build.gradle` no nível do projeto.

[Groovy](https://developer.android.com/studio/build/shrink-code?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/shrink-code?hl=pt-br#kotlin)

```kotlin
android {
    buildTypes {
        getByName("release") {
            // Enables code shrinking, obfuscation, and optimization for only
            // your project's release build type.
            isMinifyEnabled = true

            // Enables resource shrinking, which is performed by the
            // Android Gradle plugin.
            isShrinkResources = true

            // Includes the default ProGuard rules files that are packaged with
            // the Android Gradle plugin. To learn more, go to the section about
            // R8 configuration files.
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt")),
                "proguard-rules.pro"
            )
        }
    }
    ...
}
```

## Arquivos de configuração do R8

O R8 usa arquivos de regras do ProGuard para modificar o comportamento padrão e entender melhor a estrutura do app, como as classes que servem como pontos de entrada para o código do app. Embora você possa modificar alguns arquivos de regras, algumas regras podem ser geradas automaticamente por ferramentas de tempo de compilação, como AAPT2, ou herdadas das dependências da biblioteca do app. A tabela abaixo descreve as fontes dos arquivos de regras do ProGuard que o R8 usa.

| Fonte                                   | Local                                                        | Descrição                                                    |
| --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Android Studio                          | `<module-dir>/proguard-rules.pro`                            | Quando você cria um novo módulo usando o Android Studio, o ambiente de desenvolvimento integrado cria um arquivo `proguard-rules.pro` no diretório raiz desse módulo.Por padrão, esse arquivo não aplica regras. Portanto, inclua suas regras do ProGuard aqui, como suas [regras keep personalizadas](https://developer.android.com/studio/build/shrink-code?hl=pt-br#keep-code). |
| Plug-in do Android Gradle               | Gerado pelo Plug-in do Android para Gradle no tempo de compilação. | O Plug-in do Android para Gradle gera `proguard-android-optimize.txt`, que inclui regras úteis para a maioria dos projetos Android e permite [anotações `@Keep*`](https://developer.android.com/reference/androidx/annotation/Keep?hl=pt-br).Por padrão, quando um novo módulo é criado usando o Android Studio, o arquivo `build.gradle` do módulo inclui esse arquivo de regras no build de lançamento.**Observação:** o Plug-in do Android para Gradle inclui outros arquivos de regras do ProGuard predefinidos, mas é recomendável que você use o `proguard-android-optimize.txt`. |
| Dependências de biblioteca              | Bibliotecas AAR: `<library-dir>/proguard.txt`Bibliotecas JAR: `<library-dir>/META-INF/proguard/` | Se uma biblioteca AAR for publicada com o próprio arquivo de regras do ProGuard e você incluir essa AAR como uma dependência no tempo de compilação, o R8 aplicará automaticamente as regras dele ao compilar o projeto.O uso de arquivos de regras empacotados com bibliotecas AAR é útil se determinadas regras keep são necessárias para que a biblioteca funcione corretamente, ou seja, se o desenvolvedor da biblioteca tiver executado as etapas de solução de problemas para você.No entanto, como as regras do ProGuard são *aditivas*, algumas regras incluídas na dependência da biblioteca AAR não podem ser removidas e podem afetar a compilação de outras partes do app. Por exemplo, se uma biblioteca inclui uma regra para desativar as otimizações de código, essa regra desativa as otimizações para todo o projeto. |
| Android Asset Packaging Tool 2 (AAPT2)  | Após criar seu projeto com `minifyEnabled true`: `<module-dir>/build/intermediates/proguard-rules/debug/aapt_rules.txt` | O AAPT2 gera regras keep com base em referências a classes no manifesto, nos layouts e em outros recursos do app. Por exemplo, o AAPT2 inclui uma regra keep para cada atividade registrada no manifesto do app como um ponto de entrada. |
| Arquivos de configuração personalizados | Por padrão, quando você cria um novo módulo usando o Android Studio, o ambiente de desenvolvimento integrado cria `<module-dir>/proguard-rules.pro` para você incluir suas regras. | Você pode [incluir configurações adicionais](https://developer.android.com/studio/build/shrink-code?hl=pt-br#add-configuration), e o R8 as aplica no tempo de compilação. |

Quando você define a propriedade `minifyEnabled` como `true`, o R8 combina regras de todas as fontes disponíveis listadas acima. É importante lembrar disso ao [resolver problemas com o R8](https://developer.android.com/studio/build/shrink-code?hl=pt-br#troubleshoot), porque outras dependências de tempo de compilação, como dependências de biblioteca, podem introduzir mudanças desconhecidas no comportamento do R8.

Para gerar um relatório completo de todas as regras que o R8 aplica ao criar seu projeto, inclua o seguinte no arquivo `proguard-rules.pro` do seu módulo:

```
// You can specify any path and filename.
-printconfiguration ~/tmp/full-r8-config.txt
```

### Incluir configurações adicionais

Quando você cria um novo projeto ou módulo usando o Android Studio, o ambiente de desenvolvimento integrado cria um arquivo `<module-dir>/proguard-rules.pro` para incluir suas próprias regras. Você também pode incluir mais regras de outros arquivos, adicionando-as à propriedade `proguardFiles` no arquivo `build.gradle` do módulo.

Por exemplo, você pode adicionar regras específicas para cada variante de compilação incluindo outra propriedade `proguardFiles` no bloco `productFlavor` correspondente. O arquivo do Gradle a seguir adiciona `flavor2-rules.pro` à variação `flavor2` do produto. Agora, `flavor2` usa todas as três regras do ProGuard porque as do bloco `release` também são aplicadas.

[Groovy](https://developer.android.com/studio/build/shrink-code?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/shrink-code?hl=pt-br#kotlin)

```kotlin
android {
    ...
    buildTypes {
        getByName("release") {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                // List additional ProGuard rules for the given build type here. By default,
                // Android Studio creates and includes an empty rules file for you (located
                // at the root directory of each module).
                "proguard-rules.pro"
            )
        }
    }
    flavorDimensions.add("version")
    productFlavors {
        create("flavor1") {
            ...
        }
        create("flavor2") {
            proguardFile("flavor2-rules.pro")
        }
    }
}
```

## Reduzir seu código

A redução de código com R8 é ativada por padrão quando você define a propriedade `minifyEnabled` como `true`.

A redução de código (também conhecida como tree shaking) é o processo de remoção do código que o R8 determina não ser necessário no tempo de execução. Esse processo pode reduzir muito o tamanho do app se, por exemplo, ele inclui muitas dependências de biblioteca, mas utiliza apenas uma pequena parte da funcionalidade delas.

Para reduzir o código do app, o R8 primeiro determina todos os pontos de entrada no código do app com base no [conjunto combinado de arquivos de configuração](https://developer.android.com/studio/build/shrink-code?hl=pt-br#configuration-files). Esses pontos de entrada incluem todas as classes que a Plataforma Android pode usar para abrir as atividades ou os serviços do app. A partir de cada ponto de entrada, o R8 inspeciona o código do app para criar um gráfico de todos os métodos, variáveis de membros e outras classes que o app possa acessar no tempo de execução. O código que não está conectado a esse gráfico é considerado *inacessível* e pode ser removido do app.

A Figura 1 mostra um app com uma dependência de biblioteca de tempo de execução. Ao inspecionar o código do app, o R8 determina que os métodos `foo()`, `faz()` e `bar()` podem ser acessados no ponto de entrada do `MainActivity.class`. No entanto, a classe `OkayApi.class` ou o método `baz()` nunca são usados pelo app durante a execução. Por esse motivo, o R8 remove esse código ao reduzir o app.

![img](https://developer.android.com/studio/images/build/r8/tree-shaking.png?hl=pt-br)

**Figura 1.** No tempo de compilação, o R8 cria um gráfico com base nas regras keep combinadas do projeto para determinar o código inacessível.



O R8 determina pontos de entrada por meio de regras `-keep` nos [arquivos de configuração do R8](https://developer.android.com/studio/build/shrink-code?hl=pt-br#configuration-files) do projeto. Isso significa que as regras keep especificam as classes que não podem ser descartadas pelo R8 ao reduzir seu app, e o R8 considera essas classes como possíveis pontos de entrada no app. O Plug-in do Android para Gradle e o AAPT2 geram automaticamente as regras keep que são exigidas pela maior parte dos projetos de app, como atividades, visualizações e serviços do app. No entanto, se você precisar personalizar esse comportamento padrão com mais regras keep, leia a seção sobre como [personalizar o código a ser mantido](https://developer.android.com/studio/build/shrink-code?hl=pt-br#keep-code).

Se, em vez disso, você tiver interesse apenas em reduzir o tamanho dos recursos do app, acesse a seção sobre como [reduzir os recursos](https://developer.android.com/studio/build/shrink-code?hl=pt-br#shrink-resources).

### Personalizar o código a ser mantido

Para a maior parte das situações, o arquivo de regras padrão do ProGuard (`proguard-android- optimize.txt`) é suficiente para que o R8 remova somente o código não utilizado. Entretanto, algumas situações são difíceis para o R8 analisar corretamente, e ele pode remover códigos de que seu app precisa. Alguns dos exemplos de quando códigos podem ser removidos incorretamente incluem:

- Quando seu app chama um método da Java Native Interface (JNI)
- Quando seu aplicativo procura código em tempo de execução (por exemplo, com reflexão)

Os testes do app costumam revelar todos os erros causados por código removido indevidamente, mas você também pode inspecionar o código removido [gerando um relatório do código removido](https://developer.android.com/studio/build/shrink-code?hl=pt-br#usage).

Para corrigir erros e forçar o R8 a manter determinados códigos, adicione uma linha [`-keep`](https://www.guardsquare.com/en/products/proguard/manual/usage#keepoptions) (link em inglês) no arquivo de regras do ProGuard. Exemplo:

```
-keep public class MyClass
```

Como alternativa, você pode adicionar a anotação [`@Keep`](https://developer.android.com/reference/androidx/annotation/Keep?hl=pt-br) ao código que prefere manter. Adicionar `@Keep` a uma classe manterá toda a classe intacta. Adicionar essa anotação a um método ou campo manterá o método/campo (e o nome), bem como o nome da classe, intactos. Essa anotação estará disponível quando você estiver usando a [biblioteca AndroidX Annotations](https://developer.android.com/reference/androidx/annotation/package-summary?hl=pt-br) e quando incluir o arquivo de regras do ProGuard que acompanha o Plug-in do Android para Gradle, conforme descrito na seção sobre como [ativar a redução](https://developer.android.com/studio/build/shrink-code?hl=pt-br#enable).

Há muitos fatores a serem considerados ao usar a opção `-keep`. Para saber mais sobre como personalizar seu arquivo de configuração, leia o [Manual do ProGuard](https://www.guardsquare.com/en/products/proguard/manual/usage). A seção [Solução de problemas](https://www.guardsquare.com/en/products/proguard/manual/troubleshooting) descreve outros problemas comuns que podem ser encontrados quando o código é removido (links em inglês).

## Remover bibliotecas nativas

Por padrão, as bibliotecas de código nativo são removidas nos builds de lançamento do seu app. Isso consiste na remoção da tabela de símbolos e das informações de depuração contidas nas bibliotecas nativas usadas pelo app. A remoção de bibliotecas de código nativo resulta em uma economia significativa de tamanho. No entanto, é impossível diagnosticar falhas no Google Play Console devido à falta de informações (como nomes de classe e função).

### Compatibilidade com falhas nativas

O Google Play Console informa falhas nativas em [Android vitals](https://developer.android.com/topic/performance/vitals/crash?hl=pt-br#diagnose-crashes). Em poucas etapas, você pode gerar e fazer upload de um arquivo de símbolos de depuração nativo para seu app. Esse arquivo permite o stack trace de falhas nativas simbolizadas (que incluem nomes de classe e função) no Android vitals para ajudar você a depurar seu app na produção. Essas etapas variam dependendo da versão do Plug-in do Android para Gradle usada no projeto e do resultado de compilação do projeto.

**Observação:** para restaurar os nomes de símbolos nos relatórios de erros, use a [ferramenta ndk-stack](https://developer.android.com/ndk/guides/ndk-stack?hl=pt-br), que vem empacotada com o Android NDK.

#### Versão do Plug-in do Android para Gradle: 4.1 ou mais recente

Se o projeto criar um Android App Bundle, você poderá incluir automaticamente o arquivo de símbolos de depuração nativo nele. Para incluir esse arquivo em builds de lançamento, adicione o seguinte ao arquivo `build.gradle` do seu app:

```
android.buildTypes.release.ndk.debugSymbolLevel = { SYMBOL_TABLE | FULL }
```

Selecione o nível do símbolo de depuração:

- Use `SYMBOL_TABLE` para conseguir os nomes de função nos stack traces simbólicos do Play Console. Esse nível é compatível com [Tombstones](https://source.android.com/devices/tech/debug?hl=pt-br).
- Use `FULL` para conseguir os nomes de função, arquivos e números de linha nos stack traces simbólicos do Play Console.

**Observação:** há um limite de 300 MB para o arquivo de símbolos de depuração nativo. Se o tamanho dos símbolos de depuração for muito grande, use `SYMBOL_TABLE` em vez de `FULL` para diminuir o tamanho do arquivo.

Se o projeto cria um APK, use a configuração de compilação `build.gradle` mostrada anteriormente para gerar o arquivo de símbolos de depuração nativo separadamente. [Faça o upload do arquivo de símbolos de depuração nativo](https://support.google.com/googleplay/android-developer/answer/9848633?hl=pt-br#upload_file) manualmente para o Google Play Console. Como parte do processo de compilação, o Plug-in do Android para Gradle gera o arquivo no seguinte local do projeto:

```
app/build/outputs/native-debug-symbols/variant-name/native-debug-symbols.zip
```

#### Plug-in do Android para Gradle 4.0 ou versões anteriores (e outros sistemas de compilação)

Como parte do processo de compilação, o Plug-in do Android para Gradle mantém uma cópia das bibliotecas sem símbolos de depuração em um diretório do projeto. Essa estrutura de diretório é semelhante a esta:

```
app/build/intermediates/cmake/universal/release/obj/
├── armeabi-v7a/
│   ├── libgameengine.so
│   ├── libothercode.so
│   └── libvideocodec.so
├── arm64-v8a/
│   ├── libgameengine.so
│   ├── libothercode.so
│   └── libvideocodec.so
├── x86/
│   ├── libgameengine.so
│   ├── libothercode.so
│   └── libvideocodec.so
└── x86_64/
    ├── libgameengine.so
    ├── libothercode.so
    └── libvideocodec.so
```

**Observação:** se você usar outro sistema de compilação, poderá modificá-lo para armazenar bibliotecas sem símbolos de depuração em um diretório com a estrutura necessária acima.

1. Compacte o conteúdo do diretório:

   ```
   cd app/build/intermediates/cmake/universal/release/obj
   zip -r symbols.zip .
   ```

2. [Faça upload do arquivo `symbols.zip`](https://support.google.com/googleplay/android-developer/answer/9848633?hl=pt-br#upload_file) manualmente para o Google Play Console.

**Observação:** o limite para o arquivo de símbolos de depuração é de 300 MB. Se o arquivo é muito grande, é provável que os arquivos `.so` tenham uma tabela de símbolos (nomes de função), além de informações de depuração DWARF (nomes de arquivos e linhas de código). Eles não são necessários para simbolizar o código. Você pode removê-los executando o seguinte comando:
`$OBJCOPY --strip-debug lib.so lib.so.sym`
, em que `$OBJCOPY` aponta para a versão específica da ABI que está sendo removida (por exemplo, `ndk-bundle/toolchains/aarch64-linux-android-4.9/prebuilt/linux-x86_64/bin/aarch64-linux-android-objcopy`).

## Reduzir recursos

A redução de recursos funciona apenas em conjunto com a redução de código. Depois que o redutor de código remove todos os códigos não utilizados, o redutor de recursos pode identificar quais recursos ainda são usados pelo app. Isso é válido especialmente ao adicionar bibliotecas de código que contêm recursos: é necessário remover o código não utilizado da biblioteca para que as referências aos recursos da biblioteca sejam removidas e, dessa forma, o redutor de recursos possa removê-los.

Para ativar a redução de recursos, defina a propriedade `shrinkResources` como `true` no arquivo `build.gradle` (junto a `minifyEnabled` para a redução de código). Exemplo:

[Groovy](https://developer.android.com/studio/build/shrink-code?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/shrink-code?hl=pt-br#kotlin)

```kotlin
android {
    ...
    buildTypes {
        getByName("release") {
            isShrinkResources = true
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

Caso ainda não tenha criado o app usando `minifyEnabled` para a redução do código, tente fazer isso antes de ativar `shrinkResources`, porque pode ser necessário editar o arquivo `proguard-rules.pro` para armazenar classes ou métodos criados ou invocados de maneira dinâmica antes de começar a remover recursos.

### Personalizar quais recursos manter

Se quiser manter ou descartar recursos específicos, crie um arquivo XML no projeto com uma tag `<resources>` e especifique os recursos a serem mantidos no atributo `tools:keep` e os que serão descartados no atributo `tools:discard`. Os dois atributos aceitam uma lista separada por vírgulas de nomes de recursos. Você pode usar o caractere asterisco como curinga.

Exemplo:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*"
    tools:discard="@layout/unused2" />
```

Salve esse arquivo nos recursos do projeto como, por exemplo, em `res/raw/keep.xml`. O build não empacota esse arquivo no app.

Pode parecer bobagem especificar quais recursos descartar quando se poderia simplesmente excluí-los, mas essa ferramenta pode ser útil ao usar variantes de build. Por exemplo, é possível colocar todos os seus recursos no diretório comum do projeto e criar um arquivo `keep.xml` diferente para cada variante de compilação quando um recurso é aparentemente usado no código (e, portanto, não foi removido pelo redutor), mas você sabe que esse recurso não será usado pela variante de compilação em questão. Também é possível que as ferramentas de compilação identifiquem incorretamente um recurso como necessário. Isso pode acontecer porque o compilador adiciona os IDs de recursos inline, e o analisador de recursos pode desconhecer a diferença entre um recurso realmente referenciado e um número inteiro no código que tem acidentalmente o mesmo valor.

### Ativar verificações de referências rígidas

Normalmente, o redutor de recursos pode determinar com precisão se um recurso está sendo usado. No entanto, se o código chamar `Resources.getIdentifier()` (ou se uma das bibliotecas fizer isso, como a biblioteca [AppCompat](https://developer.android.com/topic/libraries/support-library/features?hl=pt-br#v7-appcompat) faz), isso significa que o código está procurando nomes de recursos com base em strings geradas de forma dinâmica. Ao fazer isso, o redutor de recursos se comporta de maneira defensiva por padrão e marca todos os recursos que tenham um formato de nome correspondente como potencialmente utilizado e indisponível para remoção.

O código a seguir, por exemplo, faz com que todos os recursos com o prefixo `img_` sejam marcados como utilizados.

[Kotlin](https://developer.android.com/studio/build/shrink-code?hl=pt-br#kotlin)[Java](https://developer.android.com/studio/build/shrink-code?hl=pt-br#java)

```kotlin
val name = String.format("img_%1d", angle + 1)
val res = resources.getIdentifier(name, "drawable", packageName)
```

O redutor de recursos também examina todas as constantes de strings no código, bem como em vários recursos `res/raw/`, procurando URLs de recursos com formato semelhante a `file:///android_res/drawable//ic_plus_anim_016.png`. Se ele encontra strings como essa ou outras que pareçam poder ser usadas para construir URLs como esse, elas não são removidas.

Esses são exemplos do modo de redução segura que é ativado por padrão. No entanto, você pode desativar essa abordagem de "melhor prevenir do que remediar" e especificar que o redutor de recursos precisa manter apenas os recursos que com certeza estão sendo usados. Para fazer isso, defina `shrinkMode` como `strict` no arquivo `keep.xml`, da seguinte forma:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:shrinkMode="strict" />
```

Se você ativar o modo de redução rígida e o código também referir-se a recursos com strings geradas dinamicamente, como mostrado acima, precisará manter esses recursos manualmente usando o atributo `tools:keep`.

### Remover recursos alternativos não utilizados

O redutor de recursos do Gradle remove apenas os recursos não referenciados pelo código do seu app, o que significa que ele não removerá [recursos alternativos](https://developer.android.com/guide/topics/resources/providing-resources?hl=pt-br#AlternativeResources) para diferentes configurações de dispositivo. Se necessário, você pode usar a propriedade `resConfigs` do Plug-in do Android para Gradle para remover os arquivos de recursos alternativos que não são usados pelo app.

Por exemplo, se você estiver usando uma biblioteca que contenha recursos de idioma (como AppCompat ou Google Play Services), seu app conterá todas as strings de idiomas traduzidas para as mensagens nessas bibliotecas, independentemente do restante do seu app estar traduzido ou não para os mesmos idiomas. Se quiser manter apenas os idiomas com os quais o app é compatível oficialmente, use a propriedade `resConfig` para especificá-los. Os recursos para idiomas não especificados serão removidos.

O snippet a seguir mostra como limitar seus recursos de idioma para apenas inglês e francês:

[Groovy](https://developer.android.com/studio/build/shrink-code?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/shrink-code?hl=pt-br#kotlin)

```kotlin
android {
    defaultConfig {
        ...
        resourceConfigurations.addAll(listOf("en", "fr"))
    }
}
```

Ao lançar um app usando o formato Android App Bundle, por padrão, apenas os idiomas configurados no dispositivo de um usuário serão transferidos por download ao instalar o app. Da mesma forma, somente os recursos que correspondem à densidade da tela do dispositivo e às bibliotecas nativas correspondentes da ABI do dispositivo serão incluídas no download. Para ver mais informações, consulte a [configuração do Android App Bundle](https://developer.android.com/guide/app-bundle/configure-base?hl=pt-br#disable_config_apks).

Para apps legados lançados com APKs (criados antes de agosto de 2021), você pode personalizar a densidade da tela ou os recursos da ABI que serão incluídos no seu APK [criando vários APKs](https://developer.android.com/studio/build/configure-apk-splits?hl=pt-br), cada um deles para uma configuração de dispositivo diferente.

### Mesclar recursos duplicados

Por padrão, o Gradle também mescla recursos com nomes idênticos, como drawables com o mesmo nome que possam estar em pastas de recursos diferentes. Esse comportamento não é controlado pela propriedade `shrinkResources` e não pode ser desativado, porque ele é necessário para evitar erros quando vários recursos possuem o mesmo nome procurado pelo código.

A mesclagem de recursos ocorre apenas quando dois ou mais recursos compartilham um nome, tipo ou qualificador de recurso idêntico. O Gradle seleciona qual arquivo considera ser a melhor escolha entre as cópias, com base na ordem de prioridade descrita abaixo, e transmite apenas esse recurso ao AAPT para distribuição no artefato final.

O Gradle procura recursos duplicados nos seguintes locais:

- Nos recursos principais associados ao conjunto de origem principal, normalmente localizados em `src/main/res/`
- Nas sobreposições de variações, dos tipos e variações de compilação
- Nas dependências de biblioteca do projeto

O Gradle combina os recursos duplicados na seguinte ordem de prioridade (crescente):

Dependências → Principal → Variação de compilação → Tipo de compilação

Por exemplo, se um recurso duplicado aparece tanto nos recursos principais quanto em uma variação de compilação, o Gradle seleciona o recurso na variação de compilação.

Se recursos idênticos aparecerem no mesmo conjunto de origem, o Gradle não poderá mesclá-los e emitirá um erro de mesclagem de recursos. Isso poderá ocorrer se você definir vários conjuntos de origem na propriedade `sourceSet` do arquivo `build.gradle`, por exemplo, se `src/main/res/` e `src/main/res2/` tiverem recursos idênticos.

## Ofuscar o código

O objetivo da ofuscação é reduzir o tamanho do app encurtando os nomes das classes, dos métodos e dos campos dele. Veja a seguir um exemplo de ofuscação usando o R8.

```
androidx.appcompat.app.ActionBarDrawerToggle$DelegateProvider -> a.a.a.b:
androidx.appcompat.app.AlertController -> androidx.appcompat.app.AlertController:
    android.content.Context mContext -> a
    int mListItemLayout -> O
    int mViewSpacingRight -> l
    android.widget.Button mButtonNeutral -> w
    int mMultiChoiceItemLayout -> M
    boolean mShowTitle -> P
    int mViewSpacingLeft -> j
    int mButtonPanelSideLayout -> K
```

Embora a ofuscação não remova o código do seu app, economias significativas de tamanho podem ser vistas em apps com arquivos DEX que indexam muitas classes, métodos e campos. No entanto, como a ofuscação renomeia diferentes partes do seu código, algumas tarefas, como a inspeção de stack traces, exigem outras ferramentas. Para entender o stack trace após a ofuscação, leia a próxima seção sobre como [decodificar um stack trace ofuscado](https://developer.android.com/studio/build/shrink-code?hl=pt-br#decode-stack-trace).

Além disso, se o código depende de nomenclatura previsível para os métodos e as classes do app (ao usar reflexão, por exemplo), você precisa tratar essas assinaturas como pontos de entrada e especificar regras keep para elas, conforme descrito na seção sobre como [personalizar os códigos a serem mantidos](https://developer.android.com/studio/build/shrink-code?hl=pt-br#keep-code). As regras keep instruem o R8 não só a manter esse código no DEX final do seu app, mas também a manter a nomenclatura original.

### Decodificar um stack trace ofuscado

Depois que o R8 ofusca seu código, é difícil (se não impossível) entender um stack trace, porque os nomes de classes e métodos podem ter sido modificados. Além de renomear, o R8 também pode modificar os números de linha presentes nos stack traces para conseguir mais economia de tamanho ao escrever os arquivos DEX. Felizmente, o R8 cria um arquivo `mapping.txt` sempre que é executado, contendo os nomes de classes, métodos e campos ofuscados mapeados para os nomes originais. Esse arquivo de mapeamento também contém informações para mapear os números de linha de volta para os números originais de linha do arquivo de origem. O R8 salva o arquivo no diretório `<module-name>/build/outputs/mapping/<build-type>/`.

**Cuidado:** o arquivo `mapping.txt` gerado pelo R8 é substituído sempre que você cria seu projeto. Por esse motivo, salve uma cópia sempre que publicar uma nova versão. Manter uma cópia do arquivo `mapping.txt` para cada build de lançamento permitirá depurar problemas caso um usuário envie um stack trace ofuscado de uma versão anterior do app.

Para garantir que os novos rastreamentos de stack traces não sejam ambíguos, a regra a seguir precisa ser adicionada ao arquivo `proguard-rules.pro` do módulo:

```
-keepattributes LineNumberTable,SourceFile
-renamesourcefileattribute SourceFile
```

O atributo `LineNumberTable` é necessário para diferenciar as posições otimizadas nos métodos. O atributo `SourceFile` é necessário para exibir números de linha impressos em um stack trace em um dispositivo ou uma máquina virtual. `-renamesourcefileattribute` definirá o arquivo de origem em stack traces como apenas `SourceFile`. O nome real do arquivo de origem não é necessário para o novo rastreamento, já que o arquivo de mapeamento contém o arquivo de origem.

É possível fazer upload do arquivo `mapping.txt` para cada versão do app ao publicá-lo no Google Play. Ao publicar usando Android App Bundles, esse arquivo é incluído automaticamente como parte do conteúdo do pacote de app. O Google Play desofuscará os novos stack traces de problemas informados pelos usuários para que você possa analisá-los no Play Console. Para saber mais, consulte o artigo da Central de Ajuda sobre como [desofuscar stack traces de falhas](https://support.google.com/googleplay/android-developer/answer/6295281?hl=pt-br).

Para decodificar um stack trace ofuscado, use a ferramenta de linha de comando [retrace](https://developer.android.com/studio/command-line/retrace?hl=pt-br), que faz parte do [pacote de ferramentas de linha de comando](https://developer.android.com/studio?hl=pt-br#cmdline-tools).

## Otimização de código

Para reduzir ainda mais seu app, o R8 inspeciona o código de forma mais profunda para remover mais códigos não utilizados ou, quando possível, reescrever o código para torná-lo menos detalhado. Veja a seguir alguns exemplos dessas otimizações:

- Caso seu código nunca receba a ramificação `else {}` de determinada instrução "if/else", o R8 poderá remover o código da ramificação `else {}`.
- Caso seu código chame um método em apenas um local, o R8 poderá remover o método e incorporá-lo no site de chamada única.
- Se o R8 determinar que uma classe tem apenas uma subclasse, e a classe em si não for instanciada (por exemplo, uma classe base abstrata usada apenas por uma classe de implementação concreta), o R8 poderá combinar as duas classes e remover uma classe do app.
- Para saber mais, leia os [posts do blog sobre otimização do R8](https://jakewharton.com/blog/) (em inglês) de Jake Wharton.

O R8 não permite desativar ou ativar otimizações discretas nem modificar o comportamento de uma otimização. Na verdade, o R8 ignora todas as regras do ProGuard que tentam modificar as otimizações padrão, como `-optimizations` e `- optimizationpasses`. Essa restrição é importante porque, como o R8 continua a melhorar, manter um comportamento padrão para otimizações ajuda a equipe do Android Studio a solucionar facilmente os problemas que podem ocorrer.

### Ativar otimizações mais agressivas

O R8 inclui um conjunto de outras otimizações que não estão ativadas por padrão. Para ativar essas outras otimizações, inclua o seguinte no arquivo `gradle.properties` do projeto:

```
android.enableR8.fullMode=true
```

Como as otimizações extras fazem o R8 se comportar de maneira diferente do ProGuard, elas podem exigir que você inclua mais regras do ProGuard para evitar problemas de tempo de execução. Por exemplo, digamos que seu código referencie uma classe por meio da API Java Reflection. Por padrão, o R8 considera que você pretende examinar e manipular objetos dessa classe no tempo de execução, mesmo que o código não faça isso, e mantém automaticamente a classe e o inicializador estático correspondente.

No entanto, ao usar o "modo completo", o R8 não faz essa suposição, e, se o R8 declarar que seu código nunca usa a classe durante o tempo de execução, ele removerá a classe do DEX final do app. Ou seja, se você quiser manter a classe e o inicializador estático correspondente, precisará incluir uma regra keep no arquivo de regras.

Se houver algum problema ao usar o "modo completo" do R8, consulte a [página de perguntas frequentes do R8](https://r8.googlesource.com/r8/+/refs/heads/master/compatibility-faq.md) (link em inglês) para encontrar uma possível solução. Caso não consiga resolver o problema, [informe um bug](https://issuetracker.google.com/issues/new?component=326788&%3Btemplate=1025938&hl=pt-br) (link em inglês).

## Resolver problemas com o R8

Esta seção descreve algumas estratégias para solucionar problemas ao ativar redução, ofuscação e otimização usando o R8. Se você não encontrar uma solução para seu problema abaixo, leia também a [página de perguntas frequentes do R8](https://r8.googlesource.com/r8/+/refs/heads/master/compatibility-faq.md) e o [guia de solução de problemas do ProGuard](https://www.guardsquare.com/en/products/proguard/manual/troubleshooting) (links em inglês).

### Gerar um relatório de código removido (ou mantido)

Para ajudar a resolver problemas específicos do R8, pode ser útil ver um relatório sobre todo o código removido do app pelo R8. Adicione `-printusage <output-dir>/usage.txt` ao arquivo de regras personalizadas para cada módulo que você quer gerar um relatório. Quando você [ativa o R8](https://developer.android.com/studio/build/shrink-code?hl=pt-br#enable) e cria seu app, o R8 gera um relatório com o caminho e o nome do arquivo especificados. O relatório do código removido é semelhante ao seguinte:

```
androidx.drawerlayout.R$attr
androidx.vectordrawable.R
androidx.appcompat.app.AppCompatDelegateImpl
    public void setSupportActionBar(androidx.appcompat.widget.Toolbar)
    public boolean hasWindowFeature(int)
    public void setHandleNativeActionModesEnabled(boolean)
    android.view.ViewGroup getSubDecor()
    public void setLocalNightMode(int)
    final androidx.appcompat.app.AppCompatDelegateImpl$AutoNightModeManager getAutoNightModeManager()
    public final androidx.appcompat.app.ActionBarDrawerToggle$Delegate getDrawerToggleDelegate()
    private static final boolean DEBUG
    private static final java.lang.String KEY_LOCAL_NIGHT_MODE
    static final java.lang.String EXCEPTION_HANDLER_MESSAGE_SUFFIX
...
```

Se, em vez disso, você quiser ver um relatório dos pontos de entrada que o R8 determina a partir das regras keep do projeto, inclua `-printseeds <output-dir>/seeds.txt` no arquivo de regras personalizadas. Quando você [ativa o R8](https://developer.android.com/studio/build/shrink-code?hl=pt-br#enable) e cria seu app, o R8 gera um relatório com o caminho e o nome do arquivo especificados. O relatório de pontos de entrada mantidos é semelhante ao seguinte:

```
com.example.myapplication.MainActivity
androidx.appcompat.R$layout: int abc_action_menu_item_layout
androidx.appcompat.R$attr: int activityChooserViewStyle
androidx.appcompat.R$styleable: int MenuItem_android_id
androidx.appcompat.R$styleable: int[] CoordinatorLayout_Layout
androidx.lifecycle.FullLifecycleObserverAdapter
...
```

### Solucionar problemas com a redução de recursos

Quando você reduz recursos, a janela **Build** ![img](https://developer.android.com/studio/images/buttons/window-build-2x.png?hl=pt-br) mostra um resumo dos recursos removidos do app. Você precisa primeiro clicar em **Toggle view** ![img](https://developer.android.com/studio/images/buttons/build-toggle_view-2x.png?hl=pt-br) no lado esquerdo da janela para exibir a saída de texto detalhada do Gradle. Exemplo:

```
:android:shrinkDebugResources
Removed unused resources: Binary resource data reduced from 2570KB to 1711KB: Removed 33%
:android:validateDebugSigning
```

O Gradle também cria um arquivo de diagnóstico chamado `resources.txt` em `<module-name>/build/outputs/mapping/release/` (a mesma pasta dos arquivos de saída do ProGuard). Esse arquivo traz detalhes como quais recursos fazem referência a outros e quais são utilizados ou removidos.

Por exemplo: para descobrir por que `@drawable/ic_plus_anim_016` ainda está no app, abra o arquivo `resources.txt` e pesquise o nome desse arquivo. Para descobrir se ele é referenciado em outro recurso, faça o seguinte:

```
16:25:48.005 [QUIET] [system.out] &#64;drawable/add_schedule_fab_icon_anim : reachable=true
16:25:48.009 [QUIET] [system.out]     &#64;drawable/ic_plus_anim_016
```

Agora é preciso saber por que `@drawable/add_schedule_fab_icon_anim` está acessível. Se você procurar de baixo para cima, verá que esse recurso está listado em "The root reachable resources are:". Isso significa que há uma referência de código para `add_schedule_fab_icon_anim`, ou seja, o ID do R.drawable foi encontrado em código acessível.

Se você não estiver usando a verificação rígida, os IDs de recursos poderão ser marcados como alcançáveis se houver constantes de strings que pareçam poder ser usadas para construir nomes para recursos dinamicamente carregados. Nesse caso, se você pesquisar no resultado de compilação pelo nome do recurso, poderá ver uma mensagem como esta:

```
10:32:50.590 [QUIET] [system.out] Marking drawable:ic_plus_anim_016:2130837506
    used because it format-string matches string pool constant ic_plus_anim_%1$d.
```

Se vir uma dessas strings e tiver certeza de que ela não está sendo usada para carregar o recurso em questão de forma dinâmica, você poderá usar o atributo `tools:discard` para instruir o sistema de compilação a removê-la, conforme descrito na seção sobre como [personalizar quais recursos serão mantidos](https://developer.android.com/studio/build/shrink-code?hl=pt-br#keep-resources).