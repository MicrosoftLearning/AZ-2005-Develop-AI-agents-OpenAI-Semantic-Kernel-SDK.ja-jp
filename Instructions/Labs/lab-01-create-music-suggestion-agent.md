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
* Handlebars プランナーを使用してタスクを自動化する

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

1. `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/01/Lab-01-Starter.zip` にある ZIP ファイルをダウンロードします。

1. デスクトップ上のフォルダーなど、見つけやすく覚えやすい場所に ZIP ファイルの内容を展開します。

1. Visual Studio Code を開き、**[ファイル]** > **[フォルダーを開く]** を選択します。

1. 展開した **Starter** フォルダーに移動し、**[フォルダーの選択]** を選択します。

1. コード エディターで **Program.cs** ファイルを開きます。

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

1. カーネルを作成するには、次のコードを "Program.cs" ファイルに追加します。
    
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

1. コードを実行し、世界で最も有名なミュージシャンの上位 5 人を含む Azure Open AI モデルからの応答が表示されることを確認します。

    応答は、カーネルに渡した Azure Open AI モデルから返されます。 Semantic Kernel SDK は、大規模言語モデル (LLM) に接続し、プロンプトを実行できます。 LLM からどれだけ迅速に応答を受け取ったかに注目してください。 Semantic Kernel SDK を使用すると、スマート アプリケーションを簡単かつ効率的に構築できます。

## 演習 2:カスタム音楽ライブラリ プラグインを作成する

この演習では、音楽ライブラリ用のカスタム プラグインを作成します。 ユーザーの最近再生したものリストに曲を追加する、最近再生した曲リストを取得する、パーソナライズされた曲のレコメンデーションを提供する関数を作成します。 また、ユーザーの位置情報と最近再生した曲に基づいてコンサートを提案する関数も作成します。

**演習のおおよその所要時間**:15 分

### タスク 1:音楽ライブラリ プラグインを作成する

このタスクでは、ユーザーの最近再生したものリストに曲を追加し、最近再生した曲リストを取得できるプラグインを作成します。 わかりやすくするために、最近再生した曲はテキスト ファイルに保存されます。

1. "Lab01-Project" ディレクトリに新しいフォルダーを作成し、"Plugins" という名前を付けます。

1. "Plugins" フォルダーに、新しいファイル "MusicLibrary.cs" を作成します

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

1. 次のコードで "Program.cs" ファイルを更新します。

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

    このコードでは、`ImportPluginFromType` を使用して `MusicLibraryPlugin` をカーネルにインポートします。 その後、呼び出すプラグイン名と関数名を使用して `InvokeAsync` を呼び出します。 また、アーティスト、曲、ジャンルを引数として渡します。

1. ターミナルに「`dotnet run`」を入力してコードを実行します。

    次の出力が表示されます。

    ```output
    Added 'Danse' to recently played
    ```

    "RecentlyPlayed.txt" を開くと、リストに追加した新しい曲が表示されます。

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

1. 次のコードで "Program.cs" ファイルを更新します。

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

1. MusicConcertPlugin ファイルに、次のコードを追加します。

    ```c#
    using System.ComponentModel;
    using Microsoft.SemanticKernel;

    public class MusicConcertPlugin
    {
        [KernelFunction, Description("Get a list of upcoming concerts")]
        public static string GetTours()
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

    このプロンプトは、LLM でユーザーの入力をフィルター処理し、テキストから宛先のみを取得できるようにします。 次に、プランナーを呼び出して、ゴールを達成するためにプラグインを結合する計画を作成します。

1. "Program.cs" ファイルを開き、次のコードで更新します。

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

## 演習 3:Handlebars プランを使用して提案を自動化する

Handlebars プランナーは、タスクを実行するために必要な手順が複数ある場合に有用です。 プランナーは AI を使用し、カーネルに登録されているプラグインを選び、一連の手順に組み合わせて目標を達成します。 この演習では、Handlebars プランナーを使用してプラン テンプレートを生成し、それを使用して提案を自動化します。

**演習のおおよその所要時間**:10 分

### タスク 1:プラン テンプレートを生成する

このタスクでは、Handlebars プランナーを使用してプラン テンプレートを生成します。 プラン テンプレートは、ユーザーの入力に基づいて提案を自動化するために使用されます。

1. ターミナルで次のように入力して、Handlebars プランナーをインストールします。

    `dotnet add package Microsoft.SemanticKernel.Planners.Handlebars --version 1.2.0-preview`

    次に、SuggestConcert プロンプトを置き換え、代わりに Handlebars プランナーを使用してタスクを実行します。

1. 'Program.cs' ファイルのコードを次のように更新します。

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertPlugin>();
    kernel.ImportPluginFromPromptDirectory("Prompts");

    var planner = new HandlebarsPlanner(new HandlebarsPlannerOptions() { AllowLoops = true });

    string location = "Redmond WA USA";
    string goal = @$"Based on the user's recently played music, suggest a 
        concert for the user living in ${location}";

    var plan = await planner.CreatePlanAsync(kernel, goal);
    var result = await plan.InvokeAsync(kernel);

    Console.WriteLine($"{result}");
    ```

1. ターミナルに「`dotnet run`」と入力します。

    次の出力のような応答が表示されます。

    ```output
    Based on the user's recently played songs, the artist "Mademoiselle" has an upcoming concert in Seattle WA, USA on February 22, 2024, which is close to Redmond WA. Therefore, the recommended concert for the user would be Mademoiselle's concert in Seattle.
    ```

    次に、Handlebars プラン テンプレートを出力するようにコードを変更します。

1. 'Program.cs' ファイルのコードを次のように更新します。

    ```c#
    var plan = await planner.CreatePlanAsync(kernel, goal);
    Console.WriteLine("Plan:");
    Console.WriteLine(plan);
    ```

    これで、生成されたプランを確認できます。 次に、曲の提案を含めたり、ユーザーの最近再生したものリストに曲を追加したりするようにプランを変更します。

1. 次のスニペットを使用してコードを拡張します。

    ```c#
    var plan = await planner.CreatePlanAsync(kernel, 
        @$"If add song:
        Add a song to the user's recently played list.
        
        If concert recommendation:
        Based on the user's recently played music, suggest a concert for 
        the user living in a given location.

        If song recommendation:
        Suggest a song from the music library to the user based on their 
        recently played songs.");

    Console.WriteLine("Plan:");
    Console.WriteLine(plan);
    ```

1. ターミナルに「`dotnet run`」と入力すると、作成したプランの出力が表示されます。

    次の出力のようなテンプレートが表示されます。

    ```output
    Plan:
    {{!-- Step 1: Identify Key Values --}}
    {{set "location" location}}
    {{set "addSong" addSong}}
    {{set "concertRecommendation" concertRecommendation}}
    {{set "songRecommendation" concertRecommendation}}

    {{!-- Step 2: Use the Right Helpers --}}
    {{#if addSong}}
        {{set "song" song}}
        {{set "artist" artist}}
        {{set "genre" genre}}
        {{set "songAdded" (MusicLibraryPlugin-AddToRecentlyPlayed artist=artist song=song genre=genre)}}  
        {{json songAdded}}
    {{/if}}

    {{#if concertRecommendation}}
        {{set "concertSuggested" (Prompts-SuggestConcert location=location recentlyPlayedSongs=recentlyPlayedSongs musicLibrary=musicLibrary)}}
        {{json concertSuggested}}
    {{/if}}

    {{#if songRecommendation}}
        {{set "songSuggested" (SuggestSongPlugin-SuggestSong recentlyPlayedSongs=recentlyPlayedSongs musicLibrary=musicLibrary)}}
        {{json songSuggested}}
    {{/if}}

    {{!-- Step 3: Output the Result --}}
    {{json "Goal achieved"}}
    ```

     `{{#if ...}}` 構文に注目してください。 この構文は、C# の従来の `if`-`else` ブロックと同様に、Handlebars プランナーが使用できる条件付きステートメントとして機能します。 `if` ステートメントは `{{/if}}` で閉じる必要があります。

    次に、この生成されたテンプレートを使用して、独自の Handlebars プランを作成します。 

1. 次のテキストを含む 'handlebarsTemplate.txt' という新しいファイルを作成します。

    ```output
    {{set "addSong" addSong}}
    {{set "concertRecommendation" concertRecommendation}}
    {{set "songRecommendation" songRecommendation}}

    {{#if addSong}}
        {{set "song" song}}
        {{set "artist" artist}}
        {{set "genre" genre}}
        {{set addedSong (MusicLibraryPlugin-AddToRecentlyPlayed artist song genre)}}  
        Output The following content, do not make any modifications:
        {{json addedSong}}     
    {{/if}}

    {{#if concertRecommendation}}
        {{set "location" location}}
        {{set "concert" (Prompts-SuggestConcert location)}}
        Output The following content, do not make any modifications:
        {{json concert}}
    {{/if}}

    {{#if songRecommendation}}
        {{set "recentlyPlayedSongs" (MusicLibraryPlugin-GetRecentPlays)}}
        {{set "musicLibrary" (MusicLibraryPlugin-GetMusicLibrary)}}
        {{set "song" (SuggestSongPlugin-SuggestSong recentlyPlayedSongs musicLibrary)}}
        Output The following content, do not make any modifications:
        {{json song}}
    {{/if}}
    ```

    このテンプレートでは、出力をプラグインで厳密に管理するために、テキスト生成を実行しないように LLM への命令を追加します。 それでは、テンプレートを試してみましょう。

### タスク 2:Handlebars プランナーを使用して提案を自動化する

このタスクでは、Handlebars プラン テンプレートから関数を作成し、それを使用してユーザーの入力に基づいて提案を自動化します。

1. 既存のコードを変更して、Handlebars プランを削除します。

    ```c#
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
    
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"Based on the user's recently played music:
        {{$recentlyPlayedSongs}}
        recommend a song to the user from the music library:
        {{$musicLibrary}}",
        functionName: "SuggestSong",
        description: "Suggest a song to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);
    ```

1. テンプレート ファイルを読み取るコードを追加して関数を作成します。

    ```c#
    string template = File.ReadAllText($"handlebarsTemplate.txt");

    var handlebarsPromptFunction = kernel.CreateFunctionFromPrompt(
        new() {
            Template = template,
            TemplateFormat = "handlebars"
        }, new HandlebarsPromptTemplateFactory()
    );
    ```

    このコードでは、`Template` オブジェクトを `TemplateFormat` と共にカーネル メソッド `CreateFunctionFromPrompt` に渡します。 `CreateFunctionFromPrompt` は、特定のテンプレートの解析方法をカーネルに指示する `IPromptTemplateFactory` 型も受け入れます。 Handlebars テンプレートを使っているため、`HandlebarsPromptTemplateFactory` 型を使います。

    次に、いくつかの引数を指定して関数を実行し、結果を確認してみましょう。

1. 次のコードを `Program.cs` ファイルに追加します。

    ```c#
    string location = "Redmond WA USA";
    var templateResult = await kernel.InvokeAsync(handlebarsPromptFunction,
        new() {
            { "location", location },
            { "concertRecommendation", true },
            { "songRecommendation", false },
            { "addSong", false },
            { "artist", "" },
            { "song", "" },
            { "genre", "" }
        });

    Console.WriteLine(templateResult);
    ```

1. ターミナルに「`dotnet run`」と入力すると、プランナー テンプレートの出力が表示されます。

    次の出力のような応答が表示されます。

    ```output
    Based on the user's recently played songs, Ly Hoa seems to be a relevant artist. The closest concert to Redmond WA, USA would be in Portland OR, USA on April 16th, 2024.  
    ```

    プロンプトは、最近再生された音楽リストとその位置情報に基づいてユーザーにコンサートを提案できました。 他の変数を true に設定して、何が起こるか試してみることもできます。

これで、ユーザーの入力に基づいてさまざまなアクションをコードで実行できるようになりました。 上出来

### 確認

このラボでは、ユーザーの音楽ライブラリを管理し、パーソナライズされた曲とコンサートのレコメンデーションを提供できる AI エージェントを作成しました。 Semantic Kernel SDK を使用して AI エージェントを構築し、それを大規模言語モデル (LLM) サービスに接続しました。 音楽ライブラリのカスタム プラグインを作成し、Handlebars プランナーを使用して提案を自動化し、Handlebars プラン テンプレートから関数を作成してユーザーの入力に基づいて提案を自動化しました。 以上でこのラボは完了です。