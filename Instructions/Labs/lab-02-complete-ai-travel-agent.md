---
lab:
  title: 'ラボ:Azure OpenAI と Semantic Kernel SDK を使用して AI エージェントを開発する'
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
* [最新の .NET 7.0 SDK](https://dotnet.microsoft.com/download/dotnet/7.0)
* Visual Studio Code 用の [C# 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

### 開発環境を準備する

これらの演習では、スターター プロジェクトを使用できます。 スターター プロジェクトを設定するには、次の手順を使用します。

> [!IMPORTANT]
> .NET Framework 8.0 だけでなく、C# 用の VS Code 拡張機能と NuGet パッケージ マネージャーもインストールされている必要があります。

1. `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/02/Lab-02-Starter.zip` にある ZIP ファイルをダウンロードします。

1. デスクトップ上のフォルダーなど、見つけやすく覚えやすい場所に ZIP ファイルの内容を展開します。

1. Visual Studio Code を開き、**[ファイル]** > **[フォルダーを開く]** を選択します。

1. 展開した **Starter** フォルダーに移動し、**[フォルダーの選択]** を選択します。

1. コード エディターで **Program.cs** ファイルを開きます。

## 演習 1:Semantic Kernel SDK を使用してプラグインを作成する

この演習では、大規模言語モデル (LLM) サービスのエンドポイントを作成します。 セマンティック カーネル SDK では、このエンドポイントを使用して LLM に接続し、プロンプトを実行します。 セマンティック カーネル SDK では、HuggingFace、OpenAI、Azure Open AI LLM がサポートされています。 この例では、Azure Open AI を使用します。

**演習のおおよその所要時間**:10 分

### タスク 1:Azure OpenAI リソースを作成する

1. [https://portal.azure.com](https://portal.azure.com) に移動します。

1. 既定の設定を使用して、新しい Azure OpenAI リソースを作成します。

    > [!NOTE]
    > Azure OpenAI リソースが既にある場合、この手順はスキップできます。

1. リソースが作成されたら、**[リソースに移動]** を選択します。

1. **[概要]** ページで、**[Azure OpenAI Studio に移動]** を選択します。

:::image type="content" source="../media/model-deployments.png" alt-text="Azure OpenAI デプロイ ページのスクリーンショット。":::

1. **[新しいデプロイの作成]**、**[モデルのデプロイ]** の順に選択します。

1. **[モデルの選択]** で **[gpt-35-turbo-16k]** を選択します。

    既定のモデル バージョンを使用します

1. デプロイの名前を入力します

1. デプロイが完了したら、Azure OpenAI リソースに戻ります。

1. **[リソース管理]** の下の **[キーとエンドポイント]** に移動します。

    これらの値は、次のタスクでカーネルを構築するために使用します。 必ずキーを秘密にして安全に保管してください。

1. Visual Studio Code で **Program.cs** ファイルに戻ります。

1. 以下の変数を、Azure OpenAI Service のデプロイ名、API キー、エンドポイントで更新します

    ```csharp
    string yourDeploymentName = "";
    string yourEndpoint = "";
    string yourApiKey = "";
    ```

    > [!NOTE]
    > Semantic Kernel SDK の一部の機能が動作するには、デプロイ モデルが "gpt-35-turbo-16k" である必要があります。

### タスク 2:ネイティブ関数を作成する

このタスクでは、基本通貨からターゲット通貨に金額を換算できるネイティブ関数を作成します。

1. **Plugins/ConvertCurrency** フォルダーに `CurrencyConverter.cs` という名前の新しいファイルを作成します

1. `CurrencyConverter.cs` ファイルに、次のコードを追加してプラグイン関数を作成します。

    ```c#
    using AITravelAgent;

    class CurrencyConverter
    {
        [KernelFunction, 
        Description("Convert an amount from one currency to another")]
        public static string ConvertAmount()
        {
            var currencyDictionary = Currency.Currencies;
        }
    }
    ```

    このコードでは、`KernelFunction` デコレーターを使用してネイティブ関数を宣言します。 `Description` デコレーターも使用して、関数の動作の説明を追加します。 `Currency.Currencies` を使用して、通貨とその為替レートの辞書を取得できます。 次に、特定の金額をある通貨から別の通貨に換算するためのロジックを追加します。

1. 次のコードを使用して、`ConvertAmount` 関数を変更します。

    ```c#
    [KernelFunction, Description("Convert an amount from one currency to another")]
    public static string ConvertAmount(
        [Description("The target currency code")] string targetCurrencyCode, 
        [Description("The amount to convert")] string amount, 
        [Description("The starting currency code")] string baseCurrencyCode)
    {
        var currencyDictionary = Currency.Currencies;
        
        Currency targetCurrency = currencyDictionary[targetCurrencyCode];
        Currency baseCurrency = currencyDictionary[baseCurrencyCode];

        if (targetCurrency == null)
        {
            return targetCurrencyCode + " was not found";
        }
        else if (baseCurrency == null)
        {
            return baseCurrencyCode + " was not found";
        }
        else
        {
            double amountInUSD = Double.Parse(amount) * baseCurrency.USDPerUnit;
            double result = amountInUSD * targetCurrency.UnitsPerUSD;
            return @$"${amount} {baseCurrencyCode} is approximately 
                {result.ToString("C")} in {targetCurrency.Name}s ({targetCurrencyCode})";
        }
    }
    ```

    このコードでは、`Currency.Currencies` 辞書を使用して、ターゲットの通貨と基本通貨の `Currency` オブジェクトを取得します。 次に、この `Currency` オブジェクトを使用して、金額を基本通貨からターゲットの通貨に換算します。 最後に、換算された金額の文字列を返します。 次に、プラグインをテストしてみましょう。

1. `Program.cs` ファイルで、次のコードを使用して、新しいプラグイン関数をインポートして呼び出します。

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var result = await kernel.InvokeAsync("CurrencyConverter", 
        "ConvertAmount", 
        new() {
            {"targetCurrencyCode", "USD"}, 
            {"amount", "52000"}, 
            {"baseCurrencyCode", "VND"}
        }
    );

    Console.WriteLine(result);
    ```

    このコードでは、`ImportPluginFromType` メソッドを使用してプラグインをインポートします。 次に、`InvokeAsync` メソッドを使用してプラグイン関数を呼び出します。 `InvokeAsync` メソッドは、プラグイン名、関数名、および辞書のパラメータを受け取ります。 最後に、結果をコンソールに出力します。 次に、コードを実行して機能することを確認します。

1. ターミナルに「`dotnet run`」と入力します。 次の出力が表示されます。

    ```output
    $52000 VND is approximately $2.13 in US Dollars (USD)
    ```

    ここで、プラグインが正しく機能したので、ユーザーが換算する通貨と金額を検出できる自然言語プロンプトを作成しましょう。

### タスク 3:プロンプトを使用してユーザー入力を解析する

このタスクでは、ユーザーの入力を解析して、ターゲットの通貨、基本通貨、換算する金額を特定するプロンプトを作成します。

1. **Prompts** フォルダーに、`GetTargetCurrencies` という名前の新しいフォルダーを作成します

1. `GetTargetCurrencies` フォルダーに、`config.json` という名前の新しいファイルを作成します

1. `config.json` ファイルに、次のテキストを入力します。

    ```output
    {
        "schema": 1,
        "type": "completion",
        "description": "Identify the target currency, base currency, and amount to convert",
        "execution_settings": {
            "default": {
                "max_tokens": 800,
                "temperature": 0
            }
        },
        "input_variables": [
            {
                "name": "input",
                "description": "Text describing some currency amount to convert",
                "required": true
            }
        ]
    }
    ```

1. `GetTargetCurrencies` フォルダーに、`skprompt.txt` という名前の新しいファイルを作成します

1. `skprompt.txt` ファイルに、次のテキストを入力します。

    ```html
    <message role="system">Identify the target currency, base currency, and 
    amount from the user's input in the format target|base|amount</message>

    For example: 

    <message role="user">How much in GBP is 750.000 VND?</message>
    <message role="assistant">GBP|VND|750000</message>

    <message role="user">How much is 60 USD in New Zealand Dollars?</message>
    <message role="assistant">NZD|USD|60</message>

    <message role="user">How many Korean Won is 33,000 yen?</message>
    <message role="assistant">KRW|JPY|33000</message>

    <message role="user">{{$input}}</message>
    <message role="assistant">target|base|amount</message>
    ```

### タスク 4:作業を確認する

このタスクでは、アプリケーションを実行し、コードが正しく機能することを確認します。 

1. 次のコードで `Program.cs` ファイルを更新して、新しいプロンプトをテストします。

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var result = await kernel.InvokeAsync(prompts["GetTargetCurrencies"],
        new() {
            {"input", "How many Australian Dollars is 140,000 Korean Won worth?"}
        }
    );

    Console.WriteLine(result);
    ```

1. ターミナルで「`dotnet run`」と入力します。 次の出力が表示されます。

    ```output
    AUD|KRW|140000
    ```

    > [!NOTE]
    > コードが期待した出力を生成しない場合は、**Solution** フォルダー内のコードを確認します。 より正確な結果を生成するには、`skprompt.txt` ファイル内のプロンプトを調整することが必要な場合があります。

これで、金額をある通貨から別の通貨に換算できるプラグインと、ユーザーの入力を `ConvertAmount` 関数が使用できる形式に解析するために使用できるプロンプトができました。 これにより、ユーザーは AI 旅行エージェントを使用して通貨の換算を簡単に実行できます。

## 演習 2:ユーザーの意図に基づいてプラグインの選択を自動化する

この演習では、ユーザーの意図を検出し、望ましいプラグインに会話をルーティングします。 提供されたプラグインを使用して、ユーザーの意図を取得できます。 それでは始めましょう。

**演習のおおよその所要時間**:10 分

### タスク 1:GetIntent プラグインを使用する

1. 次のコードを使用して、`Program.cs` ファイルを更新してください。

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    Console.WriteLine("What would you like to do?");
    var input = Console.ReadLine();

    var intent = await kernel.InvokeAsync<string>(
        prompts["GetIntent"], 
        new() {{ "input",  input }}
    );

    ```

    このコードでは、`GetIntent` プロンプトを使用してユーザーの意図を検出します。 次に、意図を `intent` という変数に保存します。 次に、意図を `CurrencyConverter` プラグインにルーティングします。

1. 次のコードを `Program.cs` ファイルに追加します。

    ```c#
    switch (intent) {
        case "ConvertCurrency": 
            var currencyText = await kernel.InvokeAsync<string>(
                prompts["GetTargetCurrencies"], 
                new() {{ "input",  input }}
            );
            var currencyInfo = currencyText!.Split("|");
            var result = await kernel.InvokeAsync("CurrencyConverter", 
                "ConvertAmount", 
                new() {
                    {"targetCurrencyCode", currencyInfo[0]}, 
                    {"baseCurrencyCode", currencyInfo[1]},
                    {"amount", currencyInfo[2]}, 
                }
            );
            Console.WriteLine(result);
            break;
        default:
            Console.WriteLine("Other intent detected");
            break;
    }
    ```

    `GetIntent` プラグインにより、次の値が返されます。ConvertCurrency、SuggestDestinations、SuggestActivities、Translate、HelpfulPhrases、Unknown。 `switch` ステートメントを使用して、ユーザーの意図を適切なプラグインにルーティングします。 
    
    ユーザーの意図が通貨の変換の場合は、`GetTargetCurrencies` プロンプトを使用して通貨情報を取得します。 その後、`CurrencyConverter` プラグインを使用して金額を変換します。

    次に、他の意図を処理するいくつかのケースを追加します。 ここでは、Semantic Kernel SDK の自動関数呼び出し機能を使用して、使用可能なプラグインに意図をルーティングしてみましょう。

1. `Program.cs` ファイルに次のコードを追加して、関数呼び出しの自動設定を作成してください。

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    OpenAIPromptExecutionSettings settings = new()
    {
        ToolCallBehavior = ToolCallBehavior.AutoInvokeKernelFunctions
    };

    Console.WriteLine("What would you like to do?");
    var input = Console.ReadLine();
    var intent = await kernel.InvokeAsync<string>(
        prompts["GetIntent"], 
        new() {{ "input",  input }}
    );
    ```

    次に、他の意図の switch ステートメントにケースを追加します。

1. 次のコードを使用して、`Program.cs` ファイルを更新してください。

    ```c#
    switch (intent) {
        case "ConvertCurrency": 
            // ...Code you entered previously...
            break;
        case "SuggestDestinations":
        case "SuggestActivities":
        case "HelpfulPhrases":
        case "Translate":
            var autoInvokeResult = await kernel.InvokePromptAsync(input!, new(settings));
            Console.WriteLine(autoInvokeResult);
            break;
        default:
            Console.WriteLine("Other intent detected");
            break;
    }
    ```

    このコードでは、`AutoInvokeKernelFunctions` 設定を使用して、カーネルで参照されている関数とプロンプトを自動的に呼び出します。 ユーザーの意図が通貨の変換の場合、`CurrencyConverter` プラグインがそのタスクを実行します。 
    
    ユーザーの意図が目的地またはアクティビティの提案の取得、フレーズの翻訳、ある言語での役に立つフレーズの取得の場合、`AutoInvokeKernelFunctions` 設定によりプロジェクト コードに含まれていた既存のプラグインが自動的に呼び出されます。

    これらの意図のいずれのケースにも該当しない場合は、ユーザーの入力を大規模言語モデル (LLM) へのプロンプトとして実行するコードを追加することもできます。

1. 既定のケースを以下のコードに置き換えてください。

    ```c#
    default:
        Console.WriteLine("Sure, I can help with that.");
        var otherIntentResult = await kernel.InvokePromptAsync(input!, new(settings));
        Console.WriteLine(otherIntentResult);
        break;
    ```

    ユーザーが別の意図を持っている場合、LLM はユーザーの要求を処理できます。 試してみましょう。

### タスク 2:作業を確認する

このタスクでは、アプリケーションを実行し、コードが正しく機能することを確認します。 

1. ターミナルで「`dotnet run`」と入力します。 ダイアログが表示されたら、次のプロンプトのようなテキストを入力してください。

    ```output
    What would you like to do?
    How many TTD is 50 Qatari Riyals?    
    ```

1. 次の応答のような出力が表示されます。

    ```output
    $50 QAR is approximately $93.10 in Trinidadian Dollars (TTD)
    ```

1. ターミナルで「`dotnet run`」と入力します。 ダイアログが表示されたら、次のプロンプトのようなテキストを入力してください。

    ```output
    What would you like to do?
    I want to go somewhere that has lots of warm sunny beaches and delicious, spicy food!
    ```

1. 次の応答のような出力が表示されます。

    ```output
    Based on your preferences for warm sunny beaches and delicious, spicy food, I have a few destination recommendations for you:

    1. Thailand: Known for its stunning beaches, Thailand offers a perfect combination of relaxation and adventure. You can visit popular beach destinations like Phuket, Krabi, or Koh Samui, where you'll find crystal-clear waters and white sandy shores. Thai cuisine is famous for its spiciness, so you'll have plenty of mouthwatering options to try, such as Tom Yum soup, Pad Thai, and Green Curry.

    2. Mexico: Mexico is renowned for its beautiful coastal regions and vibrant culture. You can explore destinations like Cancun, Playa del Carmen, or Tulum, which boast stunning beaches along the Caribbean Sea. Mexican cuisine is rich in flavors and spices, offering a wide variety of dishes like tacos, enchiladas, and mole sauces that will satisfy your craving for spicy food.

    ...

    These destinations offer a perfect blend of warm sunny beaches and delicious, spicy food, ensuring a memorable trip for you. Let me know if you need any further assistance or if you have any specific preferences for your trip!
    ```

1. ターミナルで「`dotnet run`」と入力します。 ダイアログが表示されたら、次のプロンプトのようなテキストを入力してください。

    ```output
    What would you like to do?
    Can you give me a recipe for chicken satay?

1. You should see a response similar to the following response:

    ```output
    Sure, I can help with that.
    Certainly! Here's a recipe for chicken satay:

    ...
    ```

    意図はデフォルトのケースにルーティングされ、LLM によりチキン サテのレシピの要求が処理されるはずです。

    > [!NOTE]
    > コードが期待した出力を生成しない場合は、**Solution** フォルダー内のコードを確認します。

次に、特定のプラグインに会話履歴を提供するようにルーティング ロジックを変更しましょう。 履歴を提供すると、プラグインはユーザーの要求に対するよりコンテキストに関連した応答を取得できます。

### タスク 3:プラグインのルーティングを完了する

この演習では、会話履歴を使って、大規模言語モデル (LLM) にコンテキストを提供します。 また、実際のチャットボットと同様に、ユーザーが会話を続けられるようにコードを調整します。 それでは作業を始めましょう。

1. `do`-`while` ループを使ってユーザーによる入力を受け入れるようにコードを変更します。

    ```c#
    string input;

    do 
    {
        Console.WriteLine("What would you like to do?");
        input = Console.ReadLine();

        // ...
    }
    while (!string.IsNullOrWhiteSpace(input));
    ```

    これで、ユーザーが空白行を入力するまで会話を続けることができます。

1. `SuggestDestinations` ケースを変更して、ユーザーの旅行に関する詳細をキャプチャします。

    ```c#
    case "SuggestDestinations":
        chatHistory.AppendLine("User:" + input);
        var recommendations = await kernel.InvokePromptAsync(input!);
        Console.WriteLine(recommendations);
        break;
    ```

1. 次のコードにより、`SuggestActivities` ケースで旅行の詳細を使います。

    ```c#
     case "SuggestActivities":
        var chatSummary = await kernel.InvokeAsync(
            "ConversationSummaryPlugin", 
            "SummarizeConversation", 
            new() {{ "input", chatHistory.ToString() }});
        break;
    ```

    このコードでは、組み込みの `SummarizeConversation` 関数を使って、ユーザーとのチャットを要約します。 次に、要約を使って、目的地でのアクティビティを提案しましょう。

1. 次のコードを使って、`SuggestActivities` ケースを拡張します。

    ```c#
    var activities = await kernel.InvokePromptAsync(
        input,
        new () {
            {"input", input},
            {"history", chatSummary},
            {"ToolCallBehavior", ToolCallBehavior.AutoInvokeKernelFunctions}
    });

    chatHistory.AppendLine("User:" + input);
    chatHistory.AppendLine("Assistant:" + activities.ToString());
    
    Console.WriteLine(activities);
    break;
    ```

    このコードでは、カーネル引数として `input` と `chatSummary` を追加します。 その後、カーネルはプロンプトを呼び出して、`SuggestActivities` プラグインにルーティングします。 また、ユーザーの入力とアシスタントの応答をチャット履歴に追加して、結果を表示します。 次に、`chatSummary` 変数を `SuggestActivities` プラグインに追加する必要があります。

1. **Prompts/SuggestActivities/config.json** に移動し、Visual Studio Code でファイルを開きます

1. `input_variables` の下に、チャット履歴の変数を追加します。

    ```json
    "input_variables": [
      {
          "name": "history",
          "description": "Some background information about the user",
          "required": false
      },
      {
          "name": "destination",
          "description": "The destination a user wants to visit",
          "required": true
      }
  ]
  ```

1. **Prompts/SuggestActivities/skprompt.txt** に移動してファイルを開きます

1. チャット履歴を使うためのプロンプトを追加します。

    ```html 
    You are an experienced travel agent. 
    You are helpful, creative, and very friendly. 
    Consider the traveler's background: {{$history}}
    ```

    プロンプトの残りの部分はそのままにしておきます。 これで、プラグインはチャット履歴を使って LLM にコンテキストを提供するようになります。

### タスク 4:作業を確認する

このタスクでは、アプリケーションを実行し、コードが正しく機能することを確認します。

1. 更新したスイッチ ケースを次のコードと比較します。

    ```c#
    case "SuggestDestinations":
            chatHistory.AppendLine("User:" + input);
            var recommendations = await kernel.InvokePromptAsync(input!);
            Console.WriteLine(recommendations);
            break;
    case "SuggestActivities":

        var chatSummary = await kernel.InvokeAsync(
            "ConversationSummaryPlugin", 
            "SummarizeConversation", 
            new() {{ "input", chatHistory.ToString() }});

        var activities = await kernel.InvokePromptAsync(
            input!,
            new () {
                {"input", input},
                {"history", chatSummary},
                {"ToolCallBehavior", ToolCallBehavior.AutoInvokeKernelFunctions}
        });

        chatHistory.AppendLine("User:" + input);
        chatHistory.AppendLine("Assistant:" + activities.ToString());
        
        Console.WriteLine(activities);
        break;
    ```

1. ターミナルで「`dotnet run`」と入力します。 メッセージが表示されたら、次のようなテキストを入力します。

    ```output
    What would you like to do?
    How much is 60 USD in new zealand dollars?
    ```

1. 次のような出力が表示されます。

    ```output
    $60 USD is approximately $97.88 in New Zealand Dollars (NZD)
    What would you like to do?
    ```

1. 次のようなコンテキスト キューを使って、目的地の候補のプロンプトを入力します。

    ```output
    What would you like to do?
    I'm planning an anniversary trip with my spouse, but they are currently using a wheelchair and accessibility is a must. What are some destinations that would be romantic for us?
    ```

1. 行きやすい目的地のおすすめ候補を含む出力を受け取るはずです。

1. 次のような、アクティビティの候補に関するプロンプトを入力します。

    ```output
    What would you like to do?
    What are some things to do in Barcelona?
    ```

1. 前のコンテキストに当てはまるおすすめ候補を受け取るはずです。次に示すのは、バルセロナでできるアクティビティの例です。

    ```output
    1. Visit the iconic Sagrada Família: This breathtaking basilica is an iconic symbol of Barcelona's architecture and is known for its unique design by Antoni Gaudí.

    2. Explore Park Güell: Another masterpiece by Gaudí, this park offers stunning panoramic views of the city, intricate mosaic work, and whimsical architectural elements.

    3. Visit the Picasso Museum: Explore the extensive collection of artworks by the iconic painter Pablo Picasso, showcasing his different periods and styles.
    ```

    > [!NOTE]
    > 期待した出力がコードで生成されない場合は、**Solution** フォルダー内のコードを確認できます。

さまざまなプロンプトとコンテキスト キューを使って、アプリケーションのテストを続けることができます。順調です。 LLM へのコンテキスト キューの提供と、ユーザーが会話を続けられるようにするコードの調整がうまくいきました。

### 確認

このラボでは、大規模言語モデル (LLM) サービス用のエンドポイントを作成し、Semantic Kernel オブジェクトを構築し、Semantic Kernel SDK を使用してプロンプトを実行し、Semantic Kernel 関数とプラグインを作成し、Semantic Kernel SDK の自動関数呼び出し機能を使用してユーザーの意図を適切なプラグインにルーティングしました。 また、会話履歴を使用して LLM にコンテキストを提供し、ユーザーが会話を続行できるようにしました。 以上でこのラボは完了です。