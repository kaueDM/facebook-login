# Login do Facebook + Firebase no React Native
>Esse tutorial leva em consideração que o ambiente de desenvolvimento está devidamente configurado.
>
>Docs: [React Native - Get Started](https://facebook.github.io/react-native/docs/getting-started.html)
## Android
### _Inicializando o projeto_

```
react-native init random_app
```

Aguarde a instalação de todos os pacotes e então abra o projeto no **Android Studio**, selecionando a 
pasta _android_ dentro do diretório do projeto. Aguarde a _build_ ser executada.


### _Integrando a SDK do Facebook_

A instalação e configuração da SDK pode ser automática ou manual. Dentro de testes executados, a forma automática apresentou
falhas diversas vezes, portanto será adotado o método manual.

Primeiro, instale a SDK no projeto, executando o comando

```
npm install --save react-native-fbsdk
```

No Android Studio, abra o arquivo **settings.gradle** |  _Gradle Scripts/settings.gradle_
>Caso os arquivos não estejam visíveis, pressione ALT+1 para abrir a guia _Project_

Adicione o seguinte código acima de **include ':app'**

```
include ':react-native-fbsdk'
project(':react-native-fbsdk').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-fbsdk/android')
```

No arquivo no nível da aplicação **build.gradle** | _Gradle Scripts/build.gradle(Module: yourAppName)_

inclua o seguinte trecho dentro de **dependencies**

```
compile project(':react-native-fbsdk')
```

No arquivo **MainApplication.java** | _yourAppName/java/com.app/MainApplication.java_

faça a importação dos componentes necessários no topo do código

>**Importante:**
>
>Sim, vão aparecer diversos erros. Calma, isso vai ser corrigido logo :)

```
import com.facebook.CallbackManager;
import com.facebook.FacebookSdk;
import com.facebook.reactnative.androidsdk.FBSDKPackage;
import com.facebook.appevents.AppEventsLogger;
```

Dentro da classe MainApplication | _public class MainApplication extends Application implements ReactApplication{lorem ipsum}_

crie uma instância da variável _CallbackManager_ e seu getter

```
private static CallbackManager mCallbackManager = CallbackManager.Factory.create();

	protected static CallbackManager getCallbackManager() {
		return mCallbackManager;
	}
```

No método _onCreate()_ adicione as seguintes linhas:

```
FacebookSdk.sdkInitialize(getApplicationContext());
	//O AppEventsLogger vai gerar logs dos eventos. Mais detalhes na documentação da SDK
	AppEventsLogger.activateApp(this);
```

E para finalizar as alterações nesse arquivo, registre o pacote no método _getPackages()_
>Lembre-se de usar vírgula pra separar os pacotes!

```
@Override
	protected List<ReactPackage> getPackages() {
		return Arrays.<ReactPackage>asList(
			new MainReactPackage(),
			new FBSDKPackage(mCallbackManager) //<~~ Adicione essa linha
		);
	}
```

Agora, no arquivo **MainActivity.java** | _yourAppName/java/com.app/MainActivity.java_

faça a seguinte importação caso ela não exista:

```
import android.content.Intent
```

Dentro da classe _MainActivity_, faça o _Override_ do método **onActivityResult()**
```
@Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        MainApplication.getCallbackManager().onActivityResult(requestCode, resultCode, data);
    }
```

Para concluir esta etapa, acesse o menu superior e encontre a opção **Build > Clean Project**
Aguarde os arquivos _gradle_ sincronizarem e o projeto será atualizado. No final deste processo,
**TODOS** os erros que você encontrou até agora no código vão (ou deveriam) sumir. Caso algum persista ou o 
processo de _build_ pare no meio, revise **TODAS** as alterações feitas nos arquivos. Provavelmente algo ficou 
errado.

Inicie o emulador de sua preferência, abra um console dentro da pasta do projeto e execute:
```
react-native run-android
```

Se nada pegar fogo, ótimo.
Se algum erro aparecer durante o processo, leia o _log_, o problema estará explícito. Provavelmente será algum erro nos arquivos
_MainActivity_ ou _MainApplication_.

Se seu projeto passou pelo debug e pelo packager. Parabéns! A etapa mais 'complicada' foi concluída.

>Alguns tutoriais por aí indicam para adicionar _mavenCentral()_ ao arquivo **build.gradle** do nível do projeto. Essa configuração
>é desnecessária segundo pesquisa (leia-se _StackOverflow_), pois o repositório _jCenter()_ integra o _mavenCentral()_

### _Criando um aplicativo no Facebook_

Para prosseguir, você vai precisar de um Facebook App.
Acesse o site [Facebook Developers](https://developers.facebook.com/), cadastre-se ou faça login.

Crie um novo aplicativo. Para isso, insira as informações básicas no formulário e clique em **Create App ID**
Na próxima tela, **Product Setup**, procure por **Facebook Login** e clique no **Get Started** ao lado do mesmo.

Selecione a plataforma. No nosso caso, Android. Nas próximas etapas, você fará a configuração do seu aplicativo.

* **1. Download the Facebook SDK for Android**: Só continue para o próximo passo;

* **2. Import the Facebook SDK**: Também continue para o próximo passo;

* **3. Tell Us about Your Android Project**: No arquivo **AndroidManifest.xml** | _yourAppName/manifests/AndroidManifest.xml_ , você vai
encontrar as informações necessárias. Em **Package Name**, coloque o valor de **package** do XML. Se o nome do seu projeto for _pudim_, o seu
Package Name provavelmente será **com.pudim** . Seguindo essa linha de raciocínio, no campo **Default Activity Class Name** o valor será **com.pudim.MainActivity** .
Salve e clique na opção **Use this package name** no popup que vai abrir.

* **4. Add Your Development and Release Key Hashes**: Observe o código que está na sua tela.  
Execute o que corresponde ao seu sistema operacional e ao seu objetivo (se é uma chave de **desenvolvimento** ou **release**) no console. Caso algum erro apareça, execute novamente, 
mas dessa vez **DENTRO da pasta bin do Java SDK**. No Windows isso resolve o problema. Usa outro SO? Testa aí :D
Se persistir com erros, apele para o **Google**. Não obtive nenhum, então não posso te ajudar.

O resultado dessa execução será um hash de 28 caracteres. Copie isso do console e insira no campo **Key Hashes**. _Save'n'Go_;

* **5. Enable Single Sign On for Your App**: Ative e prossiga;

* **6. Add Your Facebook App ID**: Siga as instruções, adicionando o código mostrado aos arquivos correspondentes do seu projeto;

* **7. Include FacebookActivity in your AndroidManifest.xml**: Faça o mesmo que o passo acima;

* **8. Enable Chrome Custom Tabs**: Opcional. Leia com atenção e altere os arquivos necessários se preferir;

* **9. Log App Events**: Já fizemos isso, lembra? Vai para o próximo;

Nos passos seguintes, só vá clicando em **Continue**. Faremos isso no Javascript. Clicando em **Dashboard**, você verá o status do seu Facebook App. Adivinha só: Não tem nada acontecendo.

### _Criando um botão de Login_



## iOS
404
