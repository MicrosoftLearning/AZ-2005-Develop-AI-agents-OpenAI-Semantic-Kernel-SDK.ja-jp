---
lab:
  title: 'ラボ: Azure OpenAI と Semantic Kernel SDK を使用して AI エージェントを開発する'
  module: 'Module 01: Build your kernel'
---

# ラボ:AI 旅行エージェントを完成させる
# 受講生用ラボ マニュアル

このラボでは、Semantic Kernel SDK を使用して AI 旅行エージェントを完成させます。 大規模言語モデル (LLM) サービス用のエンドポイントを作成し、Semantic Kernel 関数を作成し、Semantic Kernel SDK の自動関数呼び出し機能を使用して、提供された事前構築済みプラグインを含む適切なプラグインにユーザーの意図をルーティングします。 また、会話履歴を使用して LLM にコンテキストを提供し、ユーザーが会話を続行できるようにします。

## 課題シナリオ

あなたは、顧客向けにカスタマイズされた旅行体験の作成を専門とする旅行代理店の開発者です。 あなたは、顧客が旅行先の詳細を知り、旅行のアクティビティを計画するのに役立つ AI 旅行エージェントの作成を任されました。 この AI 旅行エージェントは、通貨の換算、目的地とアクティビティの提案、さまざまな言語での役に立つフレーズの提供、フレーズの翻訳を行うことができる必要があります。 また、この AI 旅行エージェントは、会話履歴を使用して、ユーザーの要求に対してコンテキストを考慮した応答を提供できる必要もあります。

## 目標

このラボを完了することで、以下のことを達成できます。

* 大規模言語モデル (LLM) サービス用のエンドポイントを作成する
* Semantic Kernel オブジェクトを構築する
* Semantic Kernel SDK を使用してプロンプトを実行する
* Semantic Kernel 関数とプラグインを作成する
* Semantic Kernel SDK の自動関数呼び出し機能を使用する

## ラボのセットアップ

### 前提条件

演習を完了するには、次の項目がシステムにインストールされている必要があります。

* [Visual Studio Code](https://code.visualstudio.com)
* [最新の .NET 8.0 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
* Visual Studio Code 用の [C# 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

### 開発環境を準備する

これらの演習では、スターター プロジェクトを使用できます。 スターター プロジェクトを設定するには、次の手順を使用します。

> [!IMPORTANT]
> .NET Framework 8.0 だけでなく、C# 用の VS Code 拡張機能と NuGet パッケージ マネージャーもインストールされている必要があります。

1. 次の URL を新しいブラウザー ウィンドウに貼り付けます。
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/02/Lab-02-Starter.zip`

1. ページの右上にある [<kbd>...</kbd>] ボタンをクリックして zip ファイルをダウンロードするか、<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>S</kbd> を押します。

1. デスクトップ上のフォルダーなど、見つけやすく覚えやすい場所に ZIP ファイルの内容を展開します。

1. Visual Studio Code を開き、**[ファイル]** > **[フォルダーを開く]** を選択します。

1. 展開した **Starter** フォルダーに移動し、**[フォルダーの選択]** を選択します。

1. コード エディターで **Program.cs** ファイルを開きます。

> [!NOTE]
> フォルダー内のファイルを信頼するように求められたら、**[はい、作成者を信頼します]** を選択します。

## 演習 1:Semantic Kernel SDK を使用してプラグインを作成する

この演習では、大規模言語モデル (LLM) サービスのエンドポイントを作成します。 セマンティック カーネル SDK では、このエンドポイントを使用して LLM に接続し、プロンプトを実行します。 セマンティック カーネル SDK では、HuggingFace、OpenAI、Azure Open AI LLM がサポートされています。 この例では、Azure Open AI を使用します。

**演習のおおよその所要時間**:10 分

### タスク 1:Azure OpenAI リソースを作成する

1. [https://portal.azure.com](https://portal.azure.com) に移動します。

1. 既定の設定を使用して、新しい Azure OpenAI リソースを作成します。

    > [!NOTE]
    > Azure OpenAI リソースが既にある場合、この手順はスキップできます。

1. リソースが作成されたら、**[リソースに移動]** を選択します。

1. **[概要]** ページで、**[Go to Azure AI Foundry Portal]** を選択します。

1. **[新しいデプロイの作成]**、**[基本モデルから]** の順に選択します。

1. モデルの一覧で **gpt-35-turbo-16k** を選択します。

1. **[確認]** を選択します

1. デプロイの名前を入力し、既定のオプションのままにします。

1. デプロイが完了したら、Azure portal の Azure OpenAI リソースに戻ります。

1. **[リソース管理]** の下の **[キーとエンドポイント]** に移動します。

    次のタスクでこのデータを使用してカーネルを構築します。 必ずキーを秘密にして安全に保管してください。

### タスク 2: ネイティブ プラグインを作成する

このタスクでは、基本通貨からターゲット通貨に金額を換算できるネイティブ関数プラグインを作成します。

1. Visual Studio Code プロジェクトに戻ります。

1. **appsettings.json** ファイルを開き、使用する Azure OpenAI サービスのモデル ID、エンドポイント、API キーで値を更新します。

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. **Plugins/ConvertCurrency** フォルダーにある **CurrencyConverter.cs** という名前の新しいファイルに移動します。

1. **CurrencyConverter.cs** ファイルに、次のコードを追加してプラグイン関数を作成します。

    ```c#
    class CurrencyConverter
    {
        [KernelFunction("convert_currency")]
        [Description("Converts an amount from one currency to another, for example USD to EUR")]
        public static decimal ConvertCurrency(decimal amount, string fromCurrency, string toCurrency)
        {
            decimal exchangeRate = GetExchangeRate(fromCurrency, toCurrency);
            return amount * exchangeRate;
        }
    }
    ```

    このコードでは、**KernelFunction** デコレーターを使用して、ネイティブ関数を宣言します。 また、**Description** デコレーターを使用して、関数の動作の説明を追加します。 次に、与えられた金額をある通貨から別の通貨に換算するロジックを追加します。

1. **Program.cs** ファイルで、次のコードを使用して、新しいプラグインをインポートします。

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    ```

    次に、プラグインをテストしてみましょう。

1. **Program.cs** ファイルを右クリックし、[統合ターミナルで開く] をクリックします。

1. ターミナルに「`dotnet run`」と入力します。 

    通貨換算を要求するプロンプトを入力します (例: 10 米国ドルは香港ドルでいくらですか?)。

    次のような出力が表示されるはずです。

    ```output
    Assistant: 10 USD is equivalent to 77.70 Hong Kong dollars (HKD).
    ```

## 演習 2: Handlebars プロンプトを作成する

この演習では、Handlebars プロンプトから関数を作成します。 この関数は LLM に対して、ユーザーの旅行プランを作成するように指示します。 それでは始めましょう。

**演習のおおよその所要時間**:10 分

### タスク 1: Handlebars プロンプトから関数を作成する

1. **Program.cs** ファイルに次の `using` ディレクティブを追加します。

    `using Microsoft.SemanticKernel.PromptTemplates.Handlebars;`

1. **Program.cs** ファイルを次のコードで更新します。

    ```c#
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();

    string hbprompt = """
        <message role="system">Instructions: Before providing the the user with a travel itenerary, ask how many days their trip is</message>
        <message role="user">I'm going to {{city}}. Can you create an itenerary for me?</message>
        <message role="assistant">Sure, how many days is your trip?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">
        """;
    ```

    このコードでは、Handlebars テンプレート フォーマットを使用して、少数ショット プロンプトを作成します。 このプロンプトは、旅行プランを作成する前に、ユーザーからより多くの情報を取得するようにモデルを誘導します。

1. **Program.cs** ファイルに次のコードを追加します。

    ```c#
    // Create the prompt template config using handlebars format
    var templateFactory = new HandlebarsPromptTemplateFactory();
    var promptTemplateConfig = new PromptTemplateConfig()
    {
        Template = hbprompt,
        TemplateFormat = "handlebars",
        Name = "GetItenerary",
    };

    // Create a plugin from the prompt
    var promptFunction = kernel.CreateFunctionFromPrompt(promptTemplateConfig, templateFactory);
    var iteneraryPlugin = kernel.CreatePluginFromFunctions("TravelItenerary", [promptFunction]);

    // Add the new plugin to the kernel
    kernel.Plugins.Add(iteneraryPlugin);
    ```

    このコードでは、プロンプトから Handlebars テンプレート構成を作成します。 そして、プロンプトのプラグイン関数を作成し、カーネルに追加します。 これで、関数を呼び出す準備が整いました。

1. ターミナルに「`dotnet run`」と入力してコードを実行します。

    プランについて LLM に指示する次の入力を試してみてください。

    ```output
    Assistant: How may I help you?
    User: I'm going to Hong Kong, can you create an itenerary for me?
    Assistant: Sure! How many days will you be staying in Hong Kong?
    User: 10
    Assistant: Great! Here's a 10-day itinerary for your trip to Hong Kong:
    ...
    ```

    これで、AI 旅行アシスタントの出発点ができました。 プロンプトとプラグインを使用して、さらに機能を追加しましょう

1.  **Program.cs** ファイルに次のコードを追加します。

    ```c#
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();
    kernel.ImportPluginFromType<FlightBookingPlugin>();
    ```

    このプラグインは、模擬の詳細を含む **flights.json** ファイルを使用してフライト予約をシミュレーションします。 次に、アシスタントにいくつかのシステム プロンプトを追加します。

1.  **Program.cs** ファイルに次のコードを追加します。

    ```c#
    // Setup the assistant chat
    var history = new ChatHistory();
    history.AddSystemMessage("The current date is 01/10/2025");
    history.AddSystemMessage("You are a helpful travel assistant.");
    history.AddSystemMessage("Before providing destination recommendations, ask the user about their budget.");
    ```

    これらのプロンプトは、スムーズなユーザー エクスペリエンスを作成し、フライト予約プラグインをシミュレーションするのに役立ちます。 コードをテストする準備ができました。

1. ターミナルで「`dotnet run`」と入力します。

    次のプロンプトをいくつか入力してみてください。

    ```output
    1. Can you give me some destination recommendations for Europe?
    2. I want to go to Barcelona, can you create an itenerary for me?
    3. How many Euros is 100 USD?
    4. Can you book me a flight to Barcelona?
    ```

    他の入力を試して、旅行アシスタントがどのように応答するかを確認してください。

## 演習 3: アクションにユーザーの同意を要求する

この演習では、エージェントがユーザーに代わってフライトを予約できるようにする前に、ユーザーの承認を要求するフィルター呼び出し関数を追加します。 それでは始めましょう。

### タスク 1: 関数呼び出しフィルターを作成する

1. **PermissionFilter.cs** という名前の新しいファイルを作成します。

1. その新しいファイルに次のコードを入力します。

    ```c#
    #pragma warning disable SKEXP0001 
    using Microsoft.SemanticKernel;
    
    public class PermissionFilter : IFunctionInvocationFilter
    {
        public async Task OnFunctionInvocationAsync(FunctionInvocationContext context, Func<FunctionInvocationContext, Task> next)
        {
            
        }
    }
    ```

    >[!NOTE] 
    > Semantic Kernel SDK のバージョン 1.30.0 では、関数フィルターは変更できてしまうため、警告して禁止する必要があります。 

    このコードでは、`IFunctionInvocationFilter` インターフェイスを実装します。 `OnFunctionInvocationAsync` メソッドは、AI エージェントから関数が呼び出されるたびに毎回呼び出されます。

1. `book_flight` 関数が呼び出されたときに検出する次のコードを追加します。

    ```c#
    if ((context.Function.PluginName == "FlightBooking" && context.Function.Name == "book_flight"))
    {
    
    }

    await next(context);
    ```

    このコードでは、`FunctionInvocationContext` を使用して、呼び出されたプラグインと関数を特定します。

1. 次のロジックを追加して、ユーザーがフライトを予約できるアクセス許可を要求します。

    ```c#
    if ((context.Function.PluginName == "FlightBooking" && context.Function.Name == "book_flight"))
    {
        Console.WriteLine("System Message: The agent requires an approval to complete this operation. Do you approve (Y/N)");
        Console.Write("User: ");
        string shouldProceed = Console.ReadLine()!;

        if (shouldProceed != "Y")
        {
            context.Result = new FunctionResult(context.Result, "The operation was not approved by the user");
            return;
        }
    }

    await next(context);
    ```

1. **Program.cs** ファイルに移動します。

1. 次のコードを使用して、カーネルにアクセス許可フィルターを追加します。

    ```c#
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();
    kernel.ImportPluginFromType<FlightBookingPlugin>();
    kernel.FunctionInvocationFilters.Add(new PermissionFilter());
    ```

1. ターミナルで「`dotnet run`」と入力します。

    フライトを予約するためのプロンプトを入力します。 次のような応答が表示されます。

    ```output
    User: Find me a flight to Tokyo on the 19
    Assistant: I found a flight to Tokyo on the 19th of January. The flight is with Air Japan and the price is $1200.
    User: Y
    System Message: The agent requires an approval to complete this operation. Do you approve (Y/N)
    User: N
    Assistant: I'm sorry, but I am unable to book the flight for you.
    ```

    エージェントは、予約に移る前にユーザーの承認を要求するはずです。

### 確認

このラボでは、大規模言語モデル (LLM) サービスのエンドポイントを作成し、Semantic Kernel オブジェクトを構築し、Semantic Kernel SDK を使用してプロンプトを実行しました。 また、プラグインを作成し、システム メッセージを利用してモデルを誘導しました。 以上でこのラボは完了です。
