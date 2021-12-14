# Otimizar a velocidade do seu build

 







Longos tempos de compilação atrasam o processo de desenvolvimento. Esta página oferece algumas técnicas para ajudar você a resolver os gargalos de velocidade de compilação.

O processo geral para melhorar sua velocidade de compilação é o seguinte:

1. [Otimizar a configuração do build](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#optimize) realizando etapas que beneficiam imediatamente a maioria dos projetos do Android Studio.
2. [Criar um perfil para seu build](https://developer.android.com/studio/build/profile-your-build?hl=pt-br) para identificar e diagnosticar alguns dos gargalos mais complicados que podem ser específicos do projeto ou da estação de trabalho.

Ao desenvolver seu app, você precisa **implantá-lo em um dispositivo com o Android 7.0 (API de nível 24) ou uma versão mais recente**, sempre que possível. Versões mais recentes da plataforma Android implementam maneiras melhores para enviar atualizações ao app, como o [Android Runtime (ART)](https://source.android.com/devices/tech/dalvik/?hl=pt-br) e a compatibilidade nativa com [vários arquivos DEX](https://developer.android.com/studio/build/multidex?hl=pt-br).

**Observação:** depois do seu primeiro build limpo, você perceberá que os subsequentes (limpos e incrementais) têm um desempenho muito mais rápido, mesmo sem o uso de qualquer uma das otimizações descritas nesta página. Isso ocorre porque o daemon do Gradle tem um período de “aquecimento” para aumento de desempenho, semelhante a outros processos da JVM.

## Otimizar a configuração do seu build 

Siga estas dicas para aumentar a velocidade de criação do seu projeto do Android Studio.

### Manter suas ferramentas atualizadas 

As ferramentas do Android recebem otimizações de build e novos recursos em quase todas as atualizações. Algumas das dicas nesta página presumem que você está usando a versão mais recente. Para aproveitar as otimizações mais recentes, mantenha os seguintes itens atualizados:

- [O Android Studio e as ferramentas do SDK](https://developer.android.com/studio/intro/update?hl=pt-br)
- [O plug-in do Android para Gradle](https://developer.android.com/studio/releases/gradle-plugin?hl=pt-br)

### Criar uma variante de build para desenvolvimento 

**Observação**: esta seção se aplica a versões do Plug-in do Android para Gradle anteriores à 4.2. Para as versões 4.2 e mais recentes, recomendamos usar a [API Variant](https://developer.android.com/reference/tools/gradle-api/4.2/com/android/build/api/variant/Variant?hl=pt-br) para calcular códigos de versão usando tarefas do Gradle.

Muitas das configurações necessárias para [preparar o app para o lançamento](https://developer.android.com/studio/publish/preparing?hl=pt-br) não são exigidas durante o desenvolvimento do app. Ativar processos de compilação desnecessários reduz a velocidade dos builds incrementais e limpos. Por esse motivo, [configure uma variante de build](https://developer.android.com/studio/build/build-variants?hl=pt-br) que mantenha somente as configurações de build necessárias durante o desenvolvimento do app. O exemplo a seguir cria uma variação “dev” e uma “prod” (para as configurações da sua versão de lançamento):

[Groovy](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#kotlin)

```groovy
android {
    ...
    defaultConfig {...}
    buildTypes {...}
    productFlavors {
        // When building a variant that uses this flavor, the following configurations
        // override those in the defaultConfig block.
        dev {
            // To avoid using legacy multidex when building from the command line,
            // set minSdkVersion to 21 or higher. When using Android Studio 2.3 or higher,
            // the build automatically avoids legacy multidex when deploying to a device running
            // API level 21 or higher—regardless of what you set as your minSdkVersion.
            minSdkVersion 21
            versionNameSuffix "-dev"
            applicationIdSuffix '.dev'
        }

        prod {
            // If you've configured the defaultConfig block for the release version of
            // your app, you can leave this block empty and Gradle uses configurations in
            // the defaultConfig block instead. You still need to create this flavor.
            // Otherwise, all variants use the "dev" flavor configurations.
        }
    }
}
```

Se a configuração de build já usa as variações do produto para criar versões diferentes do seu app, você pode combinar as configurações "dev" e "prod" com essas variações, [usando as dimensões de variações](https://developer.android.com/studio/build/build-variants?hl=pt-br#flavor-dimensions). Por exemplo, se você já configurou uma variação "demo" (demonstração) e "full" (completa), poderá usar o exemplo de configuração a seguir para criar variações combinadas, como "devDemo" e "prodFull".

[Groovy](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#kotlin)

```groovy
android {
    ...
    defaultConfig {...}
    buildTypes {...}

    // Specifies the flavor dimensions you want to use. The order in which you
    // list each dimension determines its priority, from highest to lowest,
    // when Gradle merges variant sources and configurations. You must assign
    // each product flavor you configure to one of the flavor dimensions.

    flavorDimensions "stage", "mode"

    productFlavors {
        dev {
            dimension "stage"
            minSdkVersion 21
            versionNameSuffix "-dev"
            applicationIdSuffix '.dev'
            ...
        }

        prod {
            dimension "stage"
            ...
        }

        demo {
            dimension "mode"
            ...
        }

        full {
            dimension "mode"
            ...
        }
    }
}
```

### Sincronização de projeto de variante única 

[Sincronizar seu projeto](https://developer.android.com/studio/build?hl=pt-br#sync-files) com a configuração do build é uma etapa importante para permitir que o Android Studio entenda como o projeto está estruturado. No entanto, esse processo pode levar muito tempo em projetos grandes. Se o projeto usar diversas variantes de build, o Android Studio otimizará as sincronizações, limitando-as apenas à variante selecionada no momento.

**Esta otimização está ativada por padrão em todos os projetos e não é mais configurável no Android Studio 4.2 e versões mais recentes**.

Para ativar a otimização manualmente, é necessário usar o Android Studio 3.3 ou mais recente com o Plug-in do Android para Gradle 3.3.0 ou mais recente. Clique em **File > Settings > Experimental > Gradle** (**Android Studio > Preferences > Experimental > Gradle** em um Mac) e selecione a opção **Only sync the active variant**.

**Observação**: essa otimização é totalmente compatível com projetos que incluem as linguagens Java e C++, e é parcialmente compatível com Kotlin. Ao ativar a otimização para projetos com conteúdo em Kotlin, a sincronização do Gradle volta a usar variantes completas internamente.

### Evitar a compilação de recursos desnecessários 

Evite compilar e empacotar recursos que não estejam sendo testados (como localizações de idiomas extras e recursos de densidade de tela). Você pode fazer isso especificando apenas um recurso de idioma e uma densidade de tela para a variação “dev”, como mostrado no exemplo a seguir:

[Groovy](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#kotlin)

```groovy
android {
    ...
    productFlavors {
        dev {
            ...
            // The following configuration limits the "dev" flavor to using
            // English stringresources and xxhdpi screen-density resources.
            resConfigs "en", "xxhdpi"
        }
        ...
    }
}
```

### Desativar o Crashlytics para seus builds de depuração 

Se você não precisar gerar um [relatório do Crashlytics](https://docs.fabric.io/android/crashlytics/overview.html), agilize os builds de depuração desativando esse plug-in da seguinte maneira:

[Groovy](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#kotlin)

```groovy
android {
    ...
    buildTypes {
        debug {
            ext.enableCrashlytics = false
        }
    }
}
```

Você também precisa desativar o kit do Crashlytics no momento da execução para builds de depuração alterando a forma como inicializa a compatibilidade com Fabric no seu app, como mostrado abaixo:

[Kotlin](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#kotlin)[Java](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#java)

```kotlin
// Initializes Fabric for builds that don't use the debug build type.
Crashlytics.Builder()
        .core(CrashlyticsCore.Builder().disabled(BuildConfig.DEBUG).build())
        .build()
        .also { crashlyticsKit ->
            Fabric.with(this, crashlyticsKit)
        }
```

#### Desativar geração automática de ID de build 

Se quiser usar o Crashlytics com os builds de depuração, você ainda poderá agilizar os builds incrementais impedindo que o Crashlytics atualize recursos do app com o ID de build exclusivo dele em cada build. Como esse ID de build é armazenado em um arquivo de recurso ao qual o manifesto faz referência, a desativação da geração automática de IDs de build também permite [usar a opção Apply Changes alongside Crashlytics](https://developer.android.com/studio/run?hl=pt-br#apply-changes-libraries-plugins) para seus builds de depuração.

Para impedir que o Crashlytics atualize automaticamente seu ID de build, adicione o seguinte ao seu arquivo `build.gradle`:

[Groovy](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#kotlin)

```groovy
android {
    ...
    buildTypes {
        debug {
            ext.alwaysUpdateBuildId = false
        }
    }
}
```

Para saber mais sobre como otimizar seus builds ao usar o Crashlytics, leia a [documentação oficial](https://docs.fabric.io/android/crashlytics/build-tools.html).

### Usar valores estáticos de configuração do build com seu build de depuração 

Sempre use valores estáticos/codificados para propriedades que acessam o arquivo do manifesto ou os arquivos de recursos para seu tipo de build de depuração.

Por exemplo, usar códigos de versão dinâmicos, nomes de versão, recursos ou qualquer outra lógica de compilação que mude o arquivo de manifesto exige uma criação de app completa sempre que você quer executar uma alteração, mesmo que ela precise apenas de um hot swap. Se a sua configuração de build exigir essas propriedades dinâmicas, isole-as nas suas variantes de build de lançamento e mantenha os valores estáticos para os builds de depuração. Para ver um exemplo, consulte o arquivo `build.gradle` abaixo.

[Groovy](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#kotlin)

```groovy
int MILLIS_IN_MINUTE = 1000 * 60
int minutesSinceEpoch = System.currentTimeMillis() / MILLIS_IN_MINUTE

android {
    ...
    defaultConfig {
        // Making either of these two values dynamic in the defaultConfig will
        // require a full app build and reinstallation because the AndroidManifest.xml
        // must be updated.
        versionCode 1
        versionName "1.0"
        ...
    }

    // The defaultConfig values above are fixed, so your incremental builds don't
    // need to rebuild the manifest (and therefore the whole app, slowing build times).
    // But for release builds, it's okay. So the following script iterates through
    // all the known variants, finds those that are "release" build types, and
    // changes those properties to something dynamic.
    applicationVariants.all { variant ->
        if (variant.buildType.name == "release") {
            variant.mergedFlavor.versionCode = minutesSinceEpoch;
            variant.mergedFlavor.versionName = minutesSinceEpoch + "-" + variant.flavorName;
        }
    }
}
```

### Usar versões de dependência estáticas 

Quando você declarar dependências nos seus arquivos `build.gradle`, evite o uso de números de versão com um sinal de soma no final, como `'com.android.tools.build:gradle:2.+'`. O uso de números de versão dinâmicos pode causar atualizações inesperadas, dificuldade para resolver diferenças de versão e builds mais lentos causados pelo Gradle ao verificar se há atualizações. Em vez disso, use números de versão estáticos/codificados.

### Criar módulos de biblioteca 

Procure no seu app um código que possa ser convertido em um [módulo de biblioteca do Android](https://developer.android.com/studio/projects/android-library?hl=pt-br). Modularizar o código dessa maneira permite que o sistema compile somente os módulos que você modificar e armazene em cache essas saídas para builds futuros. Isso também torna a [execução de projetos paralelos](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:parallel_execution) (link em inglês) mais eficaz quando essa otimização é ativada.

### Criar tarefas para lógica de compilação personalizada 

Depois que você [criar um perfil de compilação](https://developer.android.com/studio/build/profile-your-build?hl=pt-br), se ele mostrar que uma parte relativamente grande do tempo de compilação é passada na fase “Configuring Projects”, analise os scripts `build.gradle` e procure um código que possa ser incluído em uma tarefa personalizada do Gradle. Quando alguma lógica de compilação é movida para uma tarefa, ela é executada somente quando necessário. Os resultados podem ser armazenados em cache para builds futuros, e essa lógica de compilação se qualificará para execução em paralelo se você ativar a [execução de projeto em paralelo](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:parallel_execution) (link em inglês). Para saber mais, leia a [documentação oficial do Gradle](https://docs.gradle.org/current/userguide/more_about_tasks.html) (link em inglês).

**Dica:** caso seu build inclua um grande número de tarefas personalizadas, organize seus arquivos `build.gradle` [criando classes de tarefas personalizadas](https://docs.gradle.org/current/userguide/custom_tasks.html) (link em inglês). Adicione suas classes ao diretório `project-root/buildSrc/src/main/groovy/`. O Gradle vai incluí-las automaticamente no caminho de classe de todos os arquivos `build.gradle` do seu projeto.

### Converter imagens em WebP 

[WebP](https://developers.google.com/speed/webp/?hl=pt-br) (link em inglês) é um formato de arquivo de imagem que proporciona compactação com perda (como JPEG) e transparência (como PNG), mas com mais qualidade do que JPEG ou PNG. A redução dos tamanhos de arquivos de imagem, sem precisar realizar compactação do tempo de compilação, pode agilizar seus builds, principalmente se o app usa muitos recursos de imagem. Entretanto, você poderá perceber um pequeno aumento no uso de CPU do dispositivo ao descompactar as imagens WebP. É fácil [converter as imagens em WebP](https://developer.android.com/studio/write/convert-webp?hl=pt-br#convert_images_to_webp) usando o Android Studio.

### Desativar o processamento de PNG 

Se você não puder (ou não quiser) [converter suas imagens PNG em WebP](https://developer.android.com/studio/write/convert-webp?hl=pt-br#convert_images_to_webp), ainda será possível agilizar seu build desativando a compactação automática de imagens sempre que criar seu app. Caso esteja usando o [plug-in do Android 3.0.0 ](https://developer.android.com/studio/releases/gradle-plugin?hl=pt-br#3-0-0)ou versão mais recente, o processamento de PNG estará desativado por padrão somente para o tipo de build de "depuração". Para desativar essa otimização para outros tipos de build, adicione o seguinte ao seu arquivo `build.gradle`:

[Groovy](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#groovy)[Kotlin](https://developer.android.com/studio/build/optimize-your-build?hl=pt-br#kotlin)

```groovy
android {
    buildTypes {
        release {
            // Disables PNG crunching for the release build type.
            crunchPngs false
        }
    }

// If you're using an older version of the plugin, use the
// following:
//  aaptOptions {
//      cruncherEnabled false
//  }
}
```

Como tipos de build ou variações de produto não definem essa propriedade, é necessário definir a propriedade manualmente para `true` durante a criação da versão de lançamento do app.

### Usar processadores de anotações incrementais 

O Plug-in do Android para Gradle 3.3.0 e versões mais recentes melhoram a compatibilidade com o [processamento de anotações incrementais](https://docs.gradle.org/4.10.1/userguide/java_plugin.html#sec:incremental_annotation_processing). Portanto, para melhorar as velocidades dos builds em incrementos, atualize o Plug-in do Android para Gradle e use apenas processadores incrementais de anotações sempre que possível.

**Observação:** esse recurso é compatível com as versões 4.10.1 e mais recentes do Gradle, exceto o Gradle 5.1. Consulte o [problema 8194 do Gradle](https://github.com/gradle/gradle/issues/8194) (link em inglês).

Para começar, consulte a tabela a seguir de processadores de anotações mais usados que são compatíveis com o processamento incremental de anotações. Para ver uma lista mais completa, consulte [Estado da compatibilidade nos processadores de anotações mais usados](https://docs.gradle.org/current/userguide/java_plugin.html#state_of_support_in_popular_annotation_processors) (link em inglês). Alguns dos processadores de anotações podem exigir outras etapas para permitir a otimização. Portanto, leia a documentação de cada um deles.

Além disso, se você usar o Kotlin no app, será necessário usar o [kapt 1.3.30 ou versões mais recentes](https://kotlinlang.org/docs/reference/kapt.html#incremental-annotation-processing-since-1330) para oferecer compatibilidade com processadores incrementais de anotações no seu código Kotlin. Leia a documentação oficial e veja se é necessário ativar manualmente esse comportamento.

Se você precisar usar um ou mais processadores de anotações que não sejam compatíveis com builds incrementais, o processamento de anotações não será incremental. No entanto, se o projeto estiver usando o kapt, a compilação Java ainda será incremental.

#### *Compatibilidade com o processamento incremental de anotações* 

| Nome do projeto |          Nome da classe do processador de anotações          | Compatível desde…                                            |
| :-------------: | :----------------------------------------------------------: | :----------------------------------------------------------- |
| **DataBinding** |  android.databinding.annotationprocessor.ProcessDataBinding  | AGP 3.5                                                      |
|    **Room**     |                 androidx.room.RoomProcessor                  | 2.3.0-alpha02  2.20: use a [opção `room.incremental`](https://developer.android.com/jetpack/androidx/releases/room?hl=pt-br#compiler-options). |
| **ButterKnife** |          butterknife.compiler.ButterKnifeProcessor           | 10.2.0                                                       |
|    **Glide**    | com.bumptech.glide.annotation.compiler.GlideAnnotationProcessor | 4.9.0                                                        |
|   **Dagger**    |          dagger.internal.codegen.ComponentProcessor          | 2.18                                                         |
|  **Lifecycle**  |            androidx.lifecycle.LifecycleProcessor             | 2.2.0-alpha02                                                |
| **AutoService** |    com.google.auto.service.processor.AutoServiceProcessor    | 1.0-rc7                                                      |
|   **Dagger**    |          dagger.android.processor.AndroidProcessor           | 2.18                                                         |
|    **Realm**    |              io.realm.processor.RealmProcessor               | 5.11.0                                                       |
|   **Lombok**    |  lombok.launch.AnnotationProcessorHider$AnnotationProcessor  | 1.16.22                                                      |
|   **Lombok**    |   lombok.launch.AnnotationProcessorHider$ClaimingProcessor   | 1.16.22                                                      |



### Configurar o coletor de lixo da JVM

O desempenho do build pode ser melhorado configurando o coletor de lixo otimizado da JVM usado pelo Gradle. Enquanto o JDK 8 é configurado para usar o coletor de lixo em paralelo por padrão, o JDK 9 e versões mais recentes são configurados para usar [o coletor de lixo do G1](https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-ED3AB6D3-FD9B-4447-9EDF-983ED2F7A573).

Para potencialmente melhorar o desempenho do build, recomendamos que você [teste seus builds do Gradle](https://developer.android.com/studio/build/profile-your-build?hl=pt-br) com o coletor de lixo em paralelo. Em `gradle.properties`, defina:

```
org.gradle.jvmargs=-XX:+UseParallelGC
```

Se já houver outras opções definidas nesse campo, adicione uma nova opção:

```
org.gradle.jvmargs=-Xmx1536m -XX:+UseParallelGC
```

Para medir a velocidade do build com diferentes configurações, consulte [Criar perfil para seu build](https://developer.android.com/studio/build/profile-your-build?hl=pt-br#profiling_different_memorycpu_settings).