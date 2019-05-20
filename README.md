# Tutorial React-native/Dynatrace


## Entendedo implementação

1. Injetaremos o agente do dynatrace (que faz a instrumentação **automatica**) nos modulos nativos gerados pela aplicação em react-native.
2. Caso a instrumentação automatica não atenda todas as suas necessidades de análise, usaremos a api do pluguin do dynatrace para instrumentar nosso app de forma **manual**.

## Antes de começar

1. Ejetar Create React Native APP (CRNA) ou Expo :

Para podermos fazer a instrumentação do dynatrace em um app react-native, antes precisamos ejetar o mesmo :

> `"Ejecting" is the process of setting up your own custom builds for your CRNA app. It can be necessary to do if you have needs that aren't covered by CRNA, but please note that aside from the use of version control systems (git, hg, etc.) it is not reversible.`
> `-` react-native documentation.

Lembrando que esse procedimento e **irreversivel**.

2. Verifique se tanto a pasta **ios** quanto a pasta **android** encontram-se no diretório raiz do app.




## Para o Android

### implementação **Automatica**

Siga as instruções em [help.dynatrace.com/android](https://help.dynatrace.com/user-experience-monitoring/mobile-apps/how-do-i-enable-user-experience-monitoring-for-android-apps/) para instrumentar o seu Android App.

> O grandle ja vem configurado como ferramenta de build padrao no react-native-cli. Esse e o caminho mais rapido para injetar o dynatrace.

### implementação **Manual**

**A implementação manual depende da automatica.**

Copiar os seguintes arquivos para o seu projeto:

* Copie o pluguin/arquivo java para: `android/app/src/main/java/com/dynatrace/plugin/`



Se você está usando o Grandle para como build manager do seu app, copie a seguinte linha para o seu arquivo `android/app/build.gradle`:

````
dependencies {
  implementation 'com.dynatrace.agent:agent-android:+'
}
````

Se você está usando a linha de comando para instrumentar seu aplicativo, copie `Dynatrace.jar` na pasta `android/app/libs/`.

Modifique `android/app/src/main/java/.../MainApplication.java` para carregar `com.dynatrace.plugin.DynatraceReactPackage`:

```
...
@Override
protected List<ReactPackage> getPackages() {
  return Arrays.<ReactPackage>asList(
    new MainReactPackage(),
    new com.dynatrace.plugin.DynatraceReactPackage());
}
...
```

### Modificar arquivos no React Native - Javascript 

* Adicione `Dynatrace.js` e `dynafetch.js` para o diretorio do projeto.

* Importe os arquivos `.js` node seu codido :

```
import Dynatrace from './Dynatrace.js';
import dynafetch from './dynafetch.js';


```



Instancie um novo objeto `Dynatrace` .

```
const dt = new Dynatrace();
```

No seu codigo, toda vez que quizer iniciar uma nova acao do usuario:

```
var action = dt.enterAction('Touch on Settings');
```

`enterAction()` retorna um inteiro.

Quando você quer parar a acao do usuario:

```
global.dt.leaveAction(action);
```

Para ter vizibilidade das requisicoes, foi criada um englobador da `fetch()` chamado `dynafetch()`. Tudo que essa funcao faz e criar uma acao de usuario em torno de uma call `fetch()` .

Exemplo:

```
componentDidMount() {
  return dynafetch(SERVERURL)
    .then((response) => response.json())
    .then((responseJson) => {
      this.setState({
        response: responseJson
      });
    })
    .catch((error) => {
      this.setState({
        error: error.message
      });
    });
}
```



## Para o iOS

### Intrumentacao **Automatica**

Siga as instrucoes em [help.dynatrace.com/ios](https://www.dynatrace.com/support/doc/appmon/user-experience-management/mobile-uem/how-to-instrument-an-ios-app/auto-instrumentation-for-ios/) para a auto-instrumentação do ios.

> Por padrao, o Cocoapods somente vem configurado em projetos ejetados do expo-cli. Para instrumentar um projeto ios, pode-se optar por tanto iniciar o Cocoapods ([pod init](https://cocoapods.org/#get_started)) quanto adicionar o Dynatrace.framework no projeto de forma direta no XCode.

### instrumentação **Manual**

Copie os seguinte arquivos para `ios/APPNAME` (o mesmo diretorio que contem `AppDelegate.h` e `AppDelegate.m`) e adicione eles ao seu projeto no XCode:

* `Dynatrace.h`
* `DynatraceModule.h`
* `DynatraceModule.m`

* Adicione `Dynatrace.framework` do ADK em Embedded Binaries.
* Adiciona as libs necessarais conforme documentacao ([Passo 3.4](https://www.dynatrace.com/support/doc/appmon/user-experience-management/mobile-uem/how-to-instrument-an-ios-app/ios-manual-setup/)).
* você pode pular o passo `Other Linker Flags` uma vez que o react-native ja faz esse trabalho.
* definir `Strip Style` para `Debugging Symbols`.

Dynatrace usa chaves diferentes que o App Mon no `Info.plist`. Set Strip Style to Debugging Symbols. Para saber as chaves corretas, basta criar um novo aplicativo mobile no Dynatrace e selecionar Apple IOS. La ira mostrar os valores dos `DTXAgentEnvironment` e `DTXApplicationID`.

Quando editar o arquivo `Info.plist`, pode ser uma boa idéia setar as seguintes chaves:

* `DTXLogLevel` para `ALL` (note que isso somente sera aplicado para non-production apps - você deve usar opcoes menos verbosas para aplicativos em producao).
* `DTXSendEmptyAutoAction` para `YES`

Copie os arquivos `Dynatrace.js` e `dynafetch.js` para o seu projeto se você não fez isso ainda.

Continue com o tutorial de *Modificar arquivos no React Native - Javascript* descrito acima.