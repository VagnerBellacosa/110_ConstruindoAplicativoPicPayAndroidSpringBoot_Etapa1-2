# Conceitos

 







## Antes de começar

Este guia presume que você já conheça bem os conceitos inerentes à programação nativa e ao [desenvolvimento para Android](https://developer.android.com/develop?hl=pt-br).

## Introdução

Esta seção traz uma explicação de nível avançado sobre como o NDK funciona. O Android NDK é um conjunto de ferramentas que permitem incorporar C ou C++ (“código nativo”) em apps para Android. A capacidade de usar código nativo em apps para Android pode ser muito útil para desenvolvedores que querem:

- portar apps entre plataformas;
- reutilizar bibliotecas existentes ou fornecer as próprias para reutilização;
- melhorar o desempenho em determinados casos, em especial nos de computação intensa, como jogos.

## Como funciona

Esta seção apresenta os principais componentes usados na criação de um app nativo para Android e descreve o processo de compilação e empacotamento.

### Principais componentes

É necessário compreender os seguintes componentes durante a criação do seu app:

- Bibliotecas compartilhadas nativas: o NDK cria essas bibliotecas, ou arquivos `.so`, a partir do código-fonte C/C++.
- Bibliotecas estáticas nativas: o NDK também pode criar bibliotecas estáticas ou arquivos `.a` que podem ser vinculados a outras bibliotecas.
- Java Native Interface (JNI): a JNI é a interface de comunicação entre os componentes Java e C++. Este guia requer conhecimentos sobre JNI. Para mais informações sobre esse recurso, consulte a [Especificação da Java Native Interface](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html) (link em inglês).
- Interface Binária de App (ABI): a ABI define exatamente como deverá ser a interação entre o código de máquina do app e o sistema no momento da execução. O NDK compila arquivos `.so` de acordo com essas definições. Diferentes ABIs correspondem a diferentes arquiteturas: o NDK é compatível com ABI para ARM de 32 bits, AArch64, x86 e x86-64. Para mais informações, consulte [ABIs do Android](https://developer.android.com/ndk/guides/abis?hl=pt-br).
- Manifesto: se você estiver criando um app sem componente Java, precisará declarar a classe [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=pt-br) no [manifesto](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=pt-br). Consulte [Usar a interface native_activity.h](https://developer.android.com/ndk/guides/concepts?hl=pt-br#na) para ver mais detalhes sobre como fazer isso.

### Fluxo

O fluxo geral para desenvolver um app nativo para Android é o seguinte:

1. Projete o app e decida quais partes serão implementadas em Java e quais em código nativo.

   **Observação**: embora seja possível evitar totalmente o uso de Java, a estrutura Java para Android é útil para determinadas tarefas, como controlar a tela e a IU.

2. Crie um projeto de app Android normalmente.

3. Se você estiver criando um app sem componente Java, precisará declarar a classe [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=pt-br) no `AndroidManifest.xml`. Para mais informações, consulte [Atividades e apps nativos](https://developer.android.com/ndk/guides/concepts?hl=pt-br#naa).

4. Crie um arquivo `Android.mk` que descreva a biblioteca nativa, incluindo nome, sinalizações, bibliotecas vinculadas e arquivos de origem a serem compilados no diretório “JNI”.

5. Você também pode criar um arquivo `Application.mk` configurando as ABIs de destino, conjuntos de ferramentas, modo de liberação/depuração e STL. Para qualquer um desses itens que você não especificar, os valores padrão a seguir serão usados, respectivamente:

   - ABI: todas as ABIs não obsoletas
   - Modo: liberação
   - STL: system

6. Coloque o código-fonte nativo no diretório `jni` do projeto.

7. Use o ndk-build para compilar as bibliotecas nativas (`.so`, `.a`).

8. Crie o componente Java, produzindo o arquivo executável `.dex`.

9. Agrupe tudo em um arquivo APK, contendo `.so`, `.dex` e outros arquivos necessários para a execução do app.

## Atividades e aplicativos nativos

O Android SDK fornece uma classe auxiliar, [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=pt-br), que permite programar uma atividade completamente nativa. A [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=pt-br) lida com a comunicação entre a estrutura do Android e seu código nativo para que você não precise criar uma subclassificação nem chamar os métodos. Basta declarar o app como nativo no arquivo `AndroidManifest.xml` e começar a desenvolvê-lo.

Os apps Android que usam [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=pt-br) ainda podem ser executados na própria máquina virtual, no sandbox de outros apps. Portanto, você ainda pode acessar as APIs do framework do Android por meio do JNI. Porém, em certos casos, como para sensores, eventos de entrada e recursos, o NDK fornece interfaces nativas que você pode usar em vez de chamar o JNI. Para mais informações sobre essa compatibilidade, consulte [APIs nativas](https://developer.android.com/ndk/guides/stable_apis?hl=pt-br).

Independentemente de estar desenvolvendo ou não uma atividade nativa, recomendamos que você crie projetos com as ferramentas de compilação tradicionais do Android. Isso ajuda a garantir a compilação e o empacotamento de apps Android com a estrutura correta.

O Android NDK fornece duas possibilidades para implementar atividades nativas:

- O cabeçalho [native_activity.h](https://developer.android.com/ndk/reference/native__activity_8h?hl=pt-br) define a versão nativa da classe [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=pt-br). Ele contém a interface de callback e as estruturas de dados necessárias para criar a atividade nativa. Como a linha de execução principal do app lida com os callbacks, suas implementações não podem gerar bloqueios. Se isso acontecer, você poderá receber erros do tipo ANR (O app não está respondendo), porque a linha de execução principal não responderá até o retorno do callback.
- O arquivo `android_native_app_glue.h` define uma biblioteca auxiliar estática compilada sobre a interface [native_activity.h](https://developer.android.com/ndk/reference/native__activity_8h?hl=pt-br). Ele gera outra linha de execução, que gerencia elementos como callbacks ou eventos de entrada em um loop de evento. Mover esses eventos para uma linha de execução separada impede que qualquer callback bloqueie sua linha de execução principal.

A fonte `<ndk_root>/sources/android/native_app_glue/android_native_app_glue.c` também está disponível, permitindo que você modifique a implementação.

Para mais informações sobre como usar essa biblioteca estática, veja o exemplo de app de atividade nativa e a documentação correspondente. Para mais informações, leia também os comentários no arquivo `<ndk_root>/sources/android/native_app_glue/android_native_app_glue.h`.

### Usar a interface native_activity.h

Para implementar uma atividade nativa com a interface [native_activity.h](https://developer.android.com/ndk/reference/native__activity_8h?hl=pt-br):

1. Crie um diretório `jni/` no diretório raiz do projeto. Ele armazenará todos os códigos nativos.

2. Declare sua atividade nativa no arquivo `AndroidManifest.xml`.

   Como seu app não tem código em Java, defina `android:hasCode` como `false`.

   ```xml
   <application android:label="@string/app_name" android:hasCode="false">
   ```

   Defina o atributo `android:name` da tag de atividade como [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=pt-br).

   ```xml
   <activity android:name="android.app.NativeActivity"
             android:label="@string/app_name">
   ```

   **Observação**: você pode criar uma subclasse para [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=pt-br). Se fizer isso, use o nome da subclasse em vez de [NativeActivity](https://developer.android.com/reference/android/app/NativeActivity?hl=pt-br).

   O atributo `android:value` da tag `meta-data` especifica o nome da biblioteca compartilhada que contém o ponto de entrada para o app (como `main` em C/C++), omitindo o prefixo `lib` e o sufixo `.so` do nome da biblioteca.

   ```xml
   <manifest>
     <application>
       <activity>
         <meta-data android:name="android.app.lib_name"
                    android:value="native-activity" />
         <intent-filter>
           <action android:name="android.intent.action.MAIN" />
           <category android:name="android.intent.category.LAUNCHER" />
         </intent-filter>
       </activity>
     </application>
   </manifest>
   ```

3. Crie um arquivo para a atividade nativa e implemente a função nomeada na variável [ANativeActivity_onCreate](https://developer.android.com/ndk/reference/group___native_activity?hl=pt-br#ga02791d0d490839055169f39fdc905c5e). O app chama essa função quando a atividade nativa é iniciada. Essa função, análoga a `main` em C/C++, recebe um ponteiro para uma estrutura [NativeActivity](https://developer.android.com/ndk/reference/struct_a_native_activity?hl=pt-br), que contém ponteiros de função para as diferentes implementações de callback que você precisa escrever. Defina os ponteiros aplicáveis da função de callback em `ANativeActivity->callbacks` para as implementações dos callbacks.

4. Defina o campo `ANativeActivity->instance` como o endereço de qualquer instância de dados específicos que você quer usar.

5. Implemente tudo o que a atividade fará depois de iniciada.

6. Implemente os demais callbacks definidos em `ANativeActivity->callbacks`. Para saber mais sobre quando os callbacks são chamados, consulte [Gerenciamento do ciclo de vida da atividade](https://developer.android.com/training/basics/activity-lifecycle?hl=pt-br).

7. Desenvolva o restante do app.

8. Crie um `Android.mk file` no diretório `jni/` do projeto para descrever seu módulo nativo para o sistema de compilação. Para mais informações, consulte [Android.mk](https://developer.android.com/ndk/guides/android_mk?hl=pt-br).

9. Depois de criar um arquivo [Android.mk](https://developer.android.com/ndk/guides/android_mk?hl=pt-br), compile seu código nativo usando o comando `ndk-build`.

   ```
   cd <path>/<to>/<project>
   $NDK/ndk-build
   ```

10. Compile e instale seu projeto para Android normalmente. Se o código nativo estiver no diretório `jni/`, o script de compilação empacotará automaticamente os arquivos `.so` compilados a partir dele no APK.

## Outra amostra de código

Para fazer o download de amostras do NDK, consulte [Amostras do NDK](https://github.com/android/ndk-samples) (link em inglês).