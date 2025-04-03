---
lab:
  title: 'ラボ: Azure OpenAI と Semantic Kernel SDK を使用して AI エージェントを開発する'
  module: 'Module 01: Build your kernel'
---

# ラボ: AI 音楽レコメンデーション アシスタントを作成する
# 受講生用ラボ マニュアル

このラボでは、ユーザーの音楽ライブラリを管理し、カスタマイズされた曲やコンサートのレコメンデーションを提供できる AI アシスタントのコードを作成します。 Semantic Kernel SDK を使用して AI アシスタントを構築し、それを大規模言語モデル (LLM) サービスに接続します。 Semantic Kernel SDK を使用すると、LLM サービスと対話し、カスタマイズされたレコメンデーションをユーザーに提供できるスマート アプリケーションを作成できます。

## 課題シナリオ

あなたは国際的なオーディオ ストリーミング サービスの開発者です。 サービスを AI と統合して、よりパーソナライズされたエクスペリエンスをユーザーに提供する職務を担っています。 ユーザーの視聴履歴や好みに基づいて、曲や今後のアーティストのツアーを AI で推奨できるようにする必要があります。 あなたは Semantic Kernel SDK を使用して、大規模言語モデル (LLM) サービスと対話できる AI アシスタントを構築することにしました。

## 目標

このラボを完了することで、以下のことを達成できます。

* 大規模言語モデル (LLM) サービス用のエンドポイントを作成する
* Semantic Kernel オブジェクトを構築する
* Semantic Kernel SDK を使用してプロンプトを実行する
* Semantic Kernel 関数とプラグインを作成する
* 自動関数呼び出しを有効にしてタスクを自動化する

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
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/01/Lab-01-Starter.zip`

1. ページの右上にある [`...`] ボタンをクリックして zip ファイルをダウンロードするか、<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>S</kbd> を押します。

1. デスクトップ上のフォルダーなど、見つけやすく覚えやすい場所に ZIP ファイルの内容を展開します。

1. Visual Studio Code を開き、**[ファイル]** > **[フォルダーを開く]** を選択します。

1. 展開した **Starter** フォルダーに移動し、**[フォルダーの選択]** を選択します。

1. コード エディターで **Program.cs** ファイルを開きます。

> [!NOTE]
> フォルダー内のファイルを信頼するように求められたら、**[はい、作成者を信頼します]** を選択します。 

## 演習 1:Semantic Kernel SDK を使用してプロンプトを実行する

この演習では、大規模言語モデル (LLM) サービスのエンドポイントを作成します。 セマンティック カーネル SDK では、このエンドポイントを使用して LLM に接続し、プロンプトを実行します。 セマンティック カーネル SDK では、HuggingFace、OpenAI、Azure Open AI LLM がサポートされています。 この例では、Azure Open AI を使用します。

**演習のおおよその所要時間**:10 分

### タスク 1:Azure OpenAI リソースを作成する

1. [https://portal.azure.com](https://portal.azure.com) に移動します。

1. 既定の設定を使用して、新しい Azure OpenAI リソースを作成します。

    > [!NOTE]
    > Azure OpenAI リソースが既にある場合、この手順はスキップできます。

1. リソースが作成されたら、**[リソースに移動]** を選択します。

1. **[概要]** ページで、**[Azure AI Foundry ポータルに移動]** を選択します。

1. **[新しいデプロイの作成]**、**[基本モデルから]** の順に選択します。

1. モデルの一覧で **gpt-35-turbo-16k** を選択します。

1. **[確認]** を選択します

1. デプロイの名前を入力し、既定のオプションのままにします。

1. デプロイが完了したら、Azure portal の Azure OpenAI リソースに戻ります。

1. **[リソース管理]** の下の **[キーとエンドポイント]** に移動します。

    次のタスクでこのデータを使用してカーネルを構築します。 必ずキーを秘密にして安全に保管してください。

### タスク 2:カーネルを構築する

この演習では、最初の Semantic Kernel SDK プロジェクトをビルドする方法について学習します。 学習するのは、新しいプロジェクトを作成し、Semantic Kernel SDK の NuGet パッケージを追加して、Semantic Kernel SDK への参照を追加する方法です。 それでは始めましょう。

1. Visual Studio Code プロジェクトに戻ります。

1. **appsettings.json** ファイルを開き、お使いの Azure OpenAI Services のモデル ID、エンドポイント、API キーで値を更新します。

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. **[ターミナル]** > **[新しいターミナル]** を選択してターミナルを開きます。

1. ターミナルで、次のコマンドを実行して Semantic Kernel SDK をインストールします。

    `dotnet add package Microsoft.SemanticKernel --version 1.30.0`

1. 1. 次の `using` ディレクティブを **Program.cs** ファイルに追加します。

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.ChatCompletion;
    using Microsoft.SemanticKernel.Connectors.OpenAI;
    ```

1. **Create a kernel builder with Azure OpenAI chat completion** (Azure OpenAI チャットによる候補を使ってカーネル ビルダーを作成する) というコメントの下に次のコードを追加します。

    ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
    ```

1. **Build the kernel** (カーネルを作成する) というコメントの下に次のコードを追加して、カーネルを作成します。

    ```c#
    // Build the kernel
    var kernel = builder.Build();
    ```

1. **Verify the endpoint and run a prompt** (エンドポイントを確認しプロンプトを実行する) というコメントの下に次のコードを追加します。

    ```c#
    // Verify the endpoint and run a prompt
    var result = await kernel.InvokePromptAsync("Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. **Starter** フォルダーを右クリックし、**[統合ターミナルで開く]** を選択します。

1. ターミナルで「`dotnet run`」と入力して、コードを実行します。

    世界で最も有名なミュージシャンの上位 5 人を含んだ応答が Azure Open AI モデルから表示されることを確認します。

    応答は、カーネルに渡した Azure Open AI モデルから返されます。 Semantic Kernel SDK は、大規模言語モデル (LLM) に接続し、プロンプトを実行できます。 LLM からどれだけ迅速に応答を受け取ったかに注目してください。 Semantic Kernel SDK を使用すると、スマート アプリケーションを簡単かつ効率的に構築できます。

応答を確認したら、確認コードを削除できます。

## 演習 2:カスタム音楽ライブラリ プラグインを作成する

この演習では、音楽ライブラリ用のカスタム プラグインを作成します。 ユーザーの最近再生したものリストに曲を追加する、最近再生した曲リストを取得する、パーソナライズされた曲のレコメンデーションを提供する関数を作成します。 また、ユーザーの位置情報と最近再生した曲に基づいてコンサートを提案する関数も作成します。

**演習のおおよその所要時間**:15 分

### タスク 1:音楽ライブラリ プラグインを作成する

このタスクでは、ユーザーの最近再生したものリストに曲を追加し、最近再生した曲リストを取得できるプラグインを作成します。 わかりやすくするために、最近再生した曲はテキスト ファイルに保存されます。

1. **Plugins** フォルダーで、ファイル **MusicLibraryPlugin.cs** を開きます。

1. **Create a kernel function to get recently played songs** (最近再生された曲を取得するカーネル関数を作成する) というコメントの下に、次のカーネル関数デコレーターを追加します。


    ```c#
    // Create a kernel function to get recently played songs
    [KernelFunction("GetRecentPlays")]
    public static string GetRecentPlays()
    ```

    `KernelFunction` デコレーターでは、ネイティブ関数を宣言します。 AI が正しく呼び出せるように、わかりやすい名前を関数に付けます。 ユーザーの最近再生したものリストは "RecentlyPlayed.txt" というテキスト ファイルに保存されます。

1. **Create a kernel function to add a song to the recently played list** (最近再生された曲のリストに曲を追加するカーネル関数を作成する) というコメントの下に、次のカーネル関数デコレーターを追加します。

    ```c#
    // Create a kernel function to add a song to the recently played list
    [KernelFunction("AddToRecentPlays")]
    public static string AddToRecentlyPlayed(string artist,  string song, string genre)
    ```

    このプラグイン クラスがカーネルに追加されると、これらの関数を識別して呼び出せるようになります。

1. **Program.cs** ファイルに移動します。

1. **Import plugins to the kernel** (カーネルにプラグインをインポートする) というコメントの下に次のコードを追加します。

    ```c#
    // Import plugins to the kernel
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    ```

1. **Create prompt execution settings** (プロンプト実行の設定を作成する) というコメントの下に次のコードを追加して、関数が自動的に呼び出されるようにします。

    ```c#
    // Create prompt execution settings
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };
    ```

    この設定を使用すると、プロンプトで関数を指定しなくても、カーネルが関数を自動的に呼び出せるようになります。

1. **Get chat completion service** (チャット入力候補サービスを取得する) というコメントの下に、次のコードを追加します。

    ```c#
    // Get chat completion service.
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
    ChatHistory chatHistory = [];
    ```

1. **Create a helper function to await and output the reply from the chat completion service** (チャット入力候補サービスからの応答を待機して出力するヘルパー関数を作成する) というコメントの下に、次のコードを追加します。

    ```c#
    // Create a helper function to await and output the reply from the chat completion service
    async Task GetAssistantReply() {
        ChatMessageContent reply = await chatCompletionService.GetChatMessageContentAsync(
            chatHistory,
            kernel: kernel,
            executionSettings: openAIPromptExecutionSettings
        );
        chatHistory.AddAssistantMessage(reply.ToString());
        Console.WriteLine(reply.ToString());
    }
    ```


1. **Add system messages to the chat** (チャットにシステム メッセージを追加する) というコメントの下に、次のコードを追加します。

    ```c#
    // Add system messages to the chat
    chatHistory.AddSystemMessage("When a user has played a song, add it to their list of recent plays.");
    chatHistory.AddSystemMessage("The listener has just played the song Danse by Tiara. It falls under these genres: French pop, electropop, pop.");
    await GetAssistantReply();
    ```

1. ターミナルに「`dotnet run`」を入力してコードを実行します。

    追加したシステム メッセージ プロンプトでプラグイン関数が呼び出され、次の出力が表示されるはずです。

    ```output
    Added 'Danse' to recently played
    ```

    **Files/RecentlyPlayed.txt** を開くと、リストに新しい曲が追加されたことがわかります。

### タスク 2:パーソナライズされた曲のレコメンデーションを提供する

このタスクでは、最近再生した曲に基づいてユーザーにパーソナライズされた曲のレコメンデーションを提供するプロンプトを作成します。 プロンプトはネイティブ機能を組み合わせ、曲のレコメンデーションを生成します。 また、これを再利用できるように、プロンプトから関数を作成します。

1. **Program.cs** ファイルで、**Create a song suggester function using a prompt** (プロンプトを使って曲サジェスター関数を作成する) というコメントの下に、次のコードを追加します。

    ```c#
    // Create a song suggester function using a prompt
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next",
        functionName: "SuggestSong",
        description: "Suggest a song to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);
    ```

    このコードでは、プロンプトから関数を作成して、カーネル プラグインに追加します。

1. "Invoke the song suggester function with a prompt from the user" (ユーザーからのプロンプトで曲サジェスター関数を呼び出す) というコメントの下に、次のコードを追加します。

    ```c#
    // Invoke the song suggester function with a prompt from the user
    chatHistory.AddUserMessage("What song should I play next?");
    await GetAssistantReply();
    ```

    これで、アプリケーションはユーザーの要求に従ってプラグイン関数を自動的に呼び出すことができます。 ユーザーの情報に基づいてコンサートのレコメンデーションも提示するように、このコードを拡張しましょう。

### タスク 3:パーソナライズされたコンサートのレコメンデーションを提供する

このタスクでは、ユーザーが最近聴いた曲とユーザーの位置情報に基づいてコンサートを提案するように LLM に依頼するプラグインを作成します。

1. **Program.cs** ファイルで、**Import plugins to the kernel** (カーネルにプラグインをインポートする) というコメントの下に、コンサート プラグインをインポートします。

    ```c#
    // Import plugins to the kernel 
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    ```

1. **Create a concert suggester function using a prompt** (プロンプトを使ってコンサート サジェスター関数を作成する) というコメントの下に、次のコードを追加します。

    ```c#
    // Create a concert suggester function using a prompt
    var concertSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"This is a list of the user's recently played songs:
        {{MusicLibraryPlugin.GetRecentPlays}}

        This is a list of upcoming concert details:
        {{MusicConcertsPlugin.GetConcerts}}

        Suggest an upcoming concert based on the user's recently played songs. 
        The user lives in {{$location}}, 
        please recommend a relevant concert that is close to their location.",
        functionName: "SuggestConcert",
        description: "Suggest a concert to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestConcertPlugin", [concertSuggesterFunction]);
    ```

    この関数プロンプトは、ミュージック ライブラリと近日開催のコンサートの情報に加えて、ユーザーの位置情報を取得し、おすすめ候補を提示します。

1. 新しいプラグイン関数を呼び出す次のプロンプトを追加します。

    ```c#
    // Invoke the concert suggester function with a prompt from the user
    chatHistory.AddUserMessage("Can you recommend a concert for me? I live in Washington");
    await GetAssistantReply();
    ```

1. ターミナルに「`dotnet run`」と入力します。

    次の応答のような出力が表示されます。

    ```output
    I recommend you attend the concert of Lisa Taylor. She will be performing in Seattle, Washington, USA on 2/22/2024. Enjoy the show!
    ```
    
    LLM からの応答は、これとは異なる場合があります。 プロンプトと場所を調整してみて、他にどのような結果が生成される可能性があるかを確認します。

これで、アシスタントはユーザー入力に基づいてさまざまなアクションを自動的に実行できるようになりました。 上出来

### 確認

このラボでは、ユーザーの音楽ライブラリを管理し、カスタマイズされた曲やコンサートのレコメンデーションを提供できる AI アシスタントを作成しました。 Semantic Kernel SDK を使用して AI アシスタントを構築し、それを大規模言語モデル (LLM) サービスに接続しました。 音楽ライブラリ用のカスタム プラグインを作成し、自動関数呼び出しを有効にして、アシスタントがユーザー入力に動的に応答するようにしました。 以上でこのラボは完了です。
