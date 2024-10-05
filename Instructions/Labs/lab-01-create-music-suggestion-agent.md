---
lab:
  title: 'ラボ:Azure OpenAI と Semantic Kernel SDK を使って AI エージェントを開発する'
  module: 'Module 01: Build your kernel'
---

# ラボ:AI 音楽レコメンデーション エージェントを作成する
# 受講生用ラボ マニュアル

このラボでは、ユーザーの音楽ライブラリを管理し、カスタマイズされた曲やコンサートのレコメンデーションを提供できる AI エージェントのコードを作成します。 Semantic Kernel SDK を使用して AI エージェントを構築し、それを大規模言語モデル (LLM) サービスに接続します。 Semantic Kernel SDK を使用すると、LLM サービスと対話し、カスタマイズされたレコメンデーションをユーザーに提供できるスマート アプリケーションを作成できます。

## 課題シナリオ

あなたは国際的なオーディオ ストリーミング サービスの開発者です。 サービスを AI と統合して、よりパーソナライズされたエクスペリエンスをユーザーに提供する職務を担っています。 ユーザーの視聴履歴や好みに基づいて、曲や今後のアーティストのツアーを AI で推奨できるようにする必要があります。 あなたは Semantic Kernel SDK を使用して、大規模言語モデル (LLM) サービスと対話できる AI エージェントを構築することにしました。

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

1. **[概要]** ページで、**[Azure OpenAI Studio に移動]** を選択します。

:::image type="content" source="../media/model-deployments.png" alt-text="Azure OpenAI デプロイ ページのスクリーンショット。":::

1. **[新しいデプロイの作成]**、**[モデルのデプロイ]** の順に選択します。

1. **[モデルの選択]** で **[gpt-35-turbo-16k]** を選択します。

    既定のモデル バージョンを使用します

1. デプロイの名前を入力します

1. デプロイが完了したら、Azure OpenAI リソースに戻ります。

1. **[リソース管理]** の下の **[キーとエンドポイント]** に移動します。

    次のタスクでこのデータを使用してカーネルを構築します。 必ずキーを秘密にして安全に保管してください。

### タスク 2:カーネルを構築する

この演習では、最初の Semantic Kernel SDK プロジェクトをビルドする方法について学習します。 学習するのは、新しいプロジェクトを作成し、Semantic Kernel SDK の NuGet パッケージを追加して、Semantic Kernel SDK への参照を追加する方法です。 それでは始めましょう。

1. Visual Studio Code プロジェクトに戻ります。

1. **[ターミナル]** > **[新しいターミナル]** を選択してターミナルを開きます。

1. ターミナルで、次のコマンドを実行して Semantic Kernel SDK をインストールします。

    `dotnet add package Microsoft.SemanticKernel --version 1.2.0`

1. カーネルを作成するには、次のコードを **Program.cs** ファイルに追加します。
    
    ```c#
    using Microsoft.SemanticKernel;

    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    ```

    プレースホルダーは必ず Azure リソースの値に置き換えてください。

1. カーネルとエンドポイントが動作していることを確認するために、次のコードを入力します。

    ```c#
    var result = await kernel.InvokePromptAsync(
        "Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. 「`dotnet run`」と入力してコードを実行し、世界で最も有名なミュージシャンの上位 5 人を含んだ応答が Azure Open AI モデルから表示されることを確認します。

    応答は、カーネルに渡した Azure Open AI モデルから返されます。 Semantic Kernel SDK は、大規模言語モデル (LLM) に接続し、プロンプトを実行できます。 LLM からどれだけ迅速に応答を受け取ったかに注目してください。 Semantic Kernel SDK を使用すると、スマート アプリケーションを簡単かつ効率的に構築できます。

## 演習 2:カスタム音楽ライブラリ プラグインを作成する

この演習では、音楽ライブラリ用のカスタム プラグインを作成します。 ユーザーの最近再生したものリストに曲を追加する、最近再生した曲リストを取得する、パーソナライズされた曲のレコメンデーションを提供する関数を作成します。 また、ユーザーの位置情報と最近再生した曲に基づいてコンサートを提案する関数も作成します。

**演習のおおよその所要時間**:15 分

### タスク 1:音楽ライブラリ プラグインを作成する

このタスクでは、ユーザーの最近再生したものリストに曲を追加し、最近再生した曲リストを取得できるプラグインを作成します。 わかりやすくするために、最近再生した曲はテキスト ファイルに保存されます。

1. "Lab01-Project" ディレクトリに新しいフォルダーを作成し、"Plugins" という名前を付けます。

1. "Plugins" フォルダーに、新しいファイル "MusicLibraryPlugin.cs" を作成します。

    まず、いくつかのクイック関数を作成し、曲を取得してユーザーの "最近再生したもの" リストに追加します。

1. 次のコードを入力します。

    ```c#
    using System.ComponentModel;
    using System.Text.Json;
    using System.Text.Json.Nodes;
    using Microsoft.SemanticKernel;

    public class MusicLibraryPlugin
    {
        [KernelFunction, 
        Description("Get a list of music recently played by the user")]
        public static string GetRecentPlays()
        {
            string content = File.ReadAllText($"Files/RecentlyPlayed.txt");
            return content;
        }
    }
    ```

    このコードでは、`KernelFunction` デコレーターを使用してネイティブ関数を宣言します。 `Description` デコレーターも使用して、関数の動作の説明を追加します。 ユーザーの最近再生したものリストは "RecentlyPlayed.txt" というテキスト ファイルに保存されます。 次に、リストに曲を加えるコードを追加できます。

1. `MusicLibraryPlugin` クラスに次のコードを追加します。

    ```c#
    [KernelFunction, Description("Add a song to the recently played list")]
    public static string AddToRecentlyPlayed(
        [Description("The name of the artist")] string artist, 
        [Description("The title of the song")] string song, 
        [Description("The song genre")] string genre)
    {
        // Read the existing content from the file
        string filePath = "Files/RecentlyPlayed.txt";
        string jsonContent = File.ReadAllText(filePath);
        var recentlyPlayed = (JsonArray) JsonNode.Parse(jsonContent);

        var newSong = new JsonObject
        {
            ["title"] = song,
            ["artist"] = artist,
            ["genre"] = genre
        };

        recentlyPlayed.Insert(0, newSong);
        File.WriteAllText(filePath, JsonSerializer.Serialize(recentlyPlayed,
            new JsonSerializerOptions { WriteIndented = true }));

        return $"Added '{song}' to recently played";
    }
    ```

    このコードで、アーティスト、曲、ジャンルを文字列として受け入れる関数を作成します。 関数の `Description` に加えて、入力変数の説明を追加することもできます。 "RecentlyPlayed.txt" ファイルには、ユーザーが最近再生した曲の JSON 形式のリストが含まれています。 このコードは、ファイルから既存のコンテンツを読み取り、解析して、新しい曲をリストに追加します。 その後、更新されたリストがファイルに書き戻されます。

1. **Program.cs** ファイルを次のコードで更新します。

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();

    var result = await kernel.InvokeAsync(
        "MusicLibraryPlugin", 
        "AddToRecentlyPlayed", 
        new() {
            ["artist"] = "Tiara", 
            ["song"] = "Danse", 
            ["genre"] = "French pop, electropop, pop"
        }
    );
    
    Console.WriteLine(result);
    ```

    このコードでは、ImportPluginFromType を使用して MusicLibraryPlugin をカーネルにインポートします。 その後、呼び出すプラグイン名と関数名を使用して InvokeAsync を呼び出します。 また、アーティスト、曲、ジャンルを引数として渡します。

1. ターミナルに「`dotnet run`」を入力してコードを実行します。

    次の出力が表示されます。

    ```output
    Added 'Danse' to recently played
    ```

    "Files/RecentlyPlayed.txt" を開くと、リストに追加した新しい曲が表示されます。

> [!NOTE]
> ターミナルに null 値の警告が表示されても、結果に影響を与えないので無視できます。

### タスク 2:パーソナライズされた曲のレコメンデーションを提供する

このタスクでは、最近再生した曲に基づいてユーザーにパーソナライズされた曲のレコメンデーションを提供するプロンプトを作成します。 プロンプトはネイティブ機能を組み合わせ、曲のレコメンデーションを生成します。 また、これを再利用できるように、プロンプトから関数を作成します。

1. `MusicLibraryPlugin.cs` ファイルに次の関数を追加します。

    ```c#
    [KernelFunction, Description("Get a list of music available to the user")]
    public static string GetMusicLibrary()
    {
        string dir = Directory.GetCurrentDirectory();
        string content = File.ReadAllText($"Files/MusicLibrary.txt");
        return content;
    }
    ```

    この関数を使って、"MusicLibrary.txt" というファイルから使用できる音楽のリストを読み取ります。 ファイルには、JSON 形式のユーザーが使用できる曲リストが含まれています。

1. **Program.cs** ファイルを次のコードで更新します。

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    
    string prompt = @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next";

    var result = await kernel.InvokePromptAsync(prompt);
    Console.WriteLine(result);
    ```

このコードで、ネイティブ関数とセマンティック プロンプトを組み合わせます。 ネイティブ関数は、大規模言語モデル (LLM) 単独ではアクセスできなかったユーザー データを取得でき、LLM はテキスト入力に基づいて曲のレコメンデーションを生成できます。

1. コードをテストするには、ターミナルに入力 `dotnet run` します。

    次の出力のような応答が表示されます。

    ```output 
    Based on the user's recently played music, a suggested song to play next could be "Sabry Aalil" by Yasemin since the user seems to enjoy pop and Egyptian pop music.
    ```

    >[!NOTE] お勧めの曲は、ここに示されている曲とは異なる場合があります。

1. コードを変更してプロンプトから関数を作成します。

    ```c#
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

    var result = await kernel.InvokeAsync(songSuggesterFunction);
    Console.WriteLine(result);
    ```

    このコードでは、プロンプトからユーザーに曲を提案する関数を作成します。 次に、それをカーネル プラグインに追加します。 最後に、関数を実行するようにカーネルに指示します。

### タスク 3:パーソナライズされたコンサートのレコメンデーションを提供する

このタスクでは、今後のコンサートの詳細を取得するプラグインを作成します。 また、ユーザーの最近再生した曲と位置情報に基づいてコンサートを提案するように LLM に依頼するプラグインも作成します。

1. "Plugins" フォルダーに、"MusicConcertPlugin.cs" という名前の新しいファイルを作成します

1. "MusicConcertPlugin" ファイルに、次のコードを追加します。

    ```c#
    using System.ComponentModel;
    using Microsoft.SemanticKernel;

    public class MusicConcertsPlugin
    {
        [KernelFunction, Description("Get a list of upcoming concerts")]
        public static string GetConcerts()
        {
            string content = File.ReadAllText($"Files/ConcertDates.txt");
            return content;
        }
    }
    ```

    このコードでは、"ConcertDates.txt" というファイルから今後のコンサート リストを読み取る関数を作成します。 このファイルには、JSON 形式の今後のコンサート リストが含まれています。 次に、LLM にコンサートの提案を求めるプロンプトを作成する必要があります。

1. "プロンプト" フォルダーに、"SuggestConcert" という名前の新しいフォルダーを作成します

1. 次の内容を含む "config.json" ファイルを "SuggestConcert" フォルダーに作成します。

    ```json
    {
        "schema": 1,
        "type": "completion",
        "description": "Suggest a concert to the user",
        "execution_settings": {
            "default": {
                "max_tokens": 4000,
                "temperature": 0
            }
        },
        "input_variables": [
            {
                "name": "location",
                "description": "The user's location",
                "required": true
            }
        ]
    }
    ```

1. 次の内容を含む "skprompt.txt" ファイルを "SuggestConcert" フォルダーに作成します。

    ```output
    This is a list of the user's recently played songs:
    {{MusicLibraryPlugin.GetRecentPlays}}

    This is a list of upcoming concert details:
    {{MusicConcertsPlugin.GetConcerts}}

    Suggest an upcoming concert based on the user's recently played songs. 
    The user lives in {{$location}}, 
    please recommend a relevant concert that is close to their location.
    ```

    このプロンプトは、LLM でユーザーの入力をフィルター処理し、テキストから宛先のみを取得できるようにします。 次に、プラグインをテストして出力を確認します。

1. **Program.cs** ファイルを開き、次のコードで更新します。

    ```c#
    var kernel = builder.Build();    
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
    // code omitted for brevity
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);

    string location = "Redmond WA USA";
    var result = await kernel.InvokeAsync<string>(prompts["SuggestConcert"],
        new() {
            { "location", location }
        }
    );
    Console.WriteLine(result);
    ```

1. ターミナルに「`dotnet run`」と入力します。

    次の応答のような出力が表示されます。

    ```output
    Based on the user's recently played songs and their location in Redmond WA USA, a relevant concert recommendation would be the upcoming concert of Lisa Taylor in Seattle WA, USA on February 22, 2024. Lisa Taylor is an indie-folk artist, and her music genre aligns with the user's recently played songs, such as "Loanh Quanh" by Ly Hoa. Additionally, Seattle is close to Redmond, making it a convenient location for the user to attend the concert.
    ```

    プロンプトと場所を調整してみて、他にどのような結果が生成される可能性があるかを確認します。

## 演習 3: ユーザー入力に基づいて提案を自動化する

代わりに自動関数呼び出しを使用して、手動によるプラグイン関数の呼び出しを回避できます。 LLM は、カーネルに登録されているプラグインを自動的に選択して結合し、目標を達成します。 この演習では、自動関数呼び出しを有効にして推奨事項を自動化します。

**演習のおおよその所要時間**:10 分

### タスク 1: ユーザー入力に基づいて提案を自動化する

このタスクでは、自動関数呼び出しを有効にして、ユーザーの入力に基づいて提案を生成します。

1. **Program.cs** ファイルのコードを次のように更新します。

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.Connectors.OpenAI;
    
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    kernel.ImportPluginFromPromptDirectory("Prompts");

    OpenAIPromptExecutionSettings settings = new()
    {
        ToolCallBehavior = ToolCallBehavior.AutoInvokeKernelFunctions
    };
    
    string prompt = @$"Based on the user's recently played music, suggest a 
        concert for the user living in ${location}";

    var autoInvokeResult = await kernel.InvokePromptAsync(prompt, new(settings));
    Console.WriteLine(autoInvokeResult);
    ```

1. ターミナルに「`dotnet run`」と入力します。

    次の出力のような応答が表示されます。

    ```output
    Based on the user's recently played songs, the artist "Mademoiselle" has an upcoming concert in Seattle WA, USA on February 22, 2024, which is close to Redmond WA. Therefore, the recommended concert for the user would be Mademoiselle's concert in Seattle.
    ```

    セマンティック カーネルは、適切なパラメーターを使用して `SuggestConcert` 関数を自動的に呼び出すことが可能です。 エージェントは、最近再生された音楽リストとその位置情報に基づいてユーザーにコンサートを提案できました。 次に、音楽のレコメンデーションのサポートを追加できます。

1. **Program.cs** ファイルを次のコードで変更します。

    ```c#
    OpenAIPromptExecutionSettings settings = new()
    {
        ToolCallBehavior = ToolCallBehavior.AutoInvokeKernelFunctions
    };
    
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"Based on the user's recently played music:
        {{$recentlyPlayedSongs}}
        recommend a song to the user from the music library:
        {{$musicLibrary}}",
        functionName: "SuggestSong",
        description: "Recommend a song from the music library"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);

    string prompt = "Can you recommend a song from the music library?";

    var autoInvokeResult = await kernel.InvokePromptAsync(prompt, new(settings));
    Console.WriteLine(autoInvokeResult);
    ```

    このコードでは、LLM に曲の提案方法を指示する KernelFunction をプロンプト テンプレートから作成します。 その後、カーネルに登録し、自動関数呼び出し設定を有効にしてプロンプトを呼び出します。 カーネルは、関数を実行し、プロンプトを完了するための正しいパラメーターを指定できます。

1. ターミナルで、「`dotnet run`」と入力してコードを実行します。

    出力結果には、ユーザーが最近聴いた曲に基づいて、おすすめの曲を提案するようにします。 応答は次の出力のようになります。
    
    ```
    Based on your recently played music, I recommend you listen to the song "Luv(sic)". It falls under the genres of hiphop and rap, which aligns with some of your recently played songs. Enjoy!  
    ```

    次に、最近再生した曲の一覧を更新するプロンプトを試してみましょう。

1. **Program.cs** ファイルを次のコードで更新します。

    ```c#
    string prompt = @"Add this song to the recently played songs list:  title: 'Touch', artist: 'Cat's Eye', genre: 'Pop'";

    var result = await kernel.InvokePromptAsync(prompt, new(settings));

    Console.WriteLine(result);
    ```

1. ターミナルに「`dotnet run`」と入力します

    出力は次のようになります。

    ```
    I have added the song 'Touch' by Cat's Eye to the recently played songs list.
    ```

    recentlyplayed.txt ファイルを開くと、新しい曲がリストの一番上に追加されていることがわかります。
    

`AutoInvokeKernelFunctions` 設定を使用すると、ユーザーのニーズに合わせてプラグインを構築することに集中できます。 これで、エージェントはユーザー入力に基づいてさまざまなアクションを自動的に実行できるようになりました。 上出来

### 確認

このラボでは、ユーザーの音楽ライブラリを管理し、パーソナライズされた曲とコンサートのレコメンデーションを提供できる AI エージェントを作成しました。 Semantic Kernel SDK を使用して AI エージェントを構築し、それを大規模言語モデル (LLM) サービスに接続しました。 音楽ライブラリ用のカスタム プラグインを作成し、自動関数呼び出しを有効にして、エージェントがユーザー入力に動的に応答するようにしました。 以上でこのラボは完了です。
