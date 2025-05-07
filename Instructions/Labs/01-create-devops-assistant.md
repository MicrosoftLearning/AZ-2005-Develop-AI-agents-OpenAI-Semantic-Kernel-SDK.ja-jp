---
lab:
  title: Semantic Kernel で AI アシスタントを作成する
  description: Semantic Kernel を使用して、DevOps タスクを実行できる生成 AI アシスタントを構築する方法について説明します。
---

# Semantic Kernel で AI アシスタントを作成する

このラボでは、開発業務を自動化しタスクの効率化に役立つように設計された AI アシスタントのコードを開発します。 Semantic Kernel SDK を使用して AI アシスタントを構築し、それを大規模言語モデル (LLM) サービスに接続します。 Semantic Kernel SDK を使用すると、LLM サービスとやり取りし、自然言語クエリに応答し、パーソナライズされたインサイトをユーザーに提供できるスマート アプリケーションを作成できます。 この演習では、一般的な DevOps タスクを表すモック関数が用意されています。 それでは始めましょう。

この演習は約 **30** 分かかります。

## チャット入力候補モデルをデプロイする

1. [https://portal.azure.com](https://portal.azure.com) に移動します。

1. 既定の設定を使用して、新しい Azure OpenAI リソースを作成します。

1. リソースが作成されたら、**[リソースに移動]** を選択します。

1. **[概要]** ページで、**[Go to Azure AI Foundry Portal]** を選択します。

1. **[新しいデプロイの作成]**、**[基本モデルから]** の順に選択します。

1. モデル一覧で **gpt-4o** を検索してから、それを選択して確認します。

1. デプロイの名前を入力し、既定のオプションのままにします。

1. デプロイが完了したら、Azure portal の Azure OpenAI リソースに戻ります。

1. **[リソース管理]** の下の **[キーとエンドポイント]** に移動します。

    次のタスクでこのデータを使用してカーネルを構築します。 必ずキーを秘密にして安全に保管してください。

## アプリケーション構成を準備する

1. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて Azure 資格情報を使用してサインインします。

    ウェルカム通知を閉じて、Azure portal のホーム ページを表示します。

1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成し、サブスクリプションにストレージがない ***PowerShell*** 環境を選択します。

    Azure portal の下部にあるペインに Cloud Shell のコマンド ライン インターフェイスが表示されます。 作業しやすくするために、このウィンドウのサイズを変更したり最大化したりすることができます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    **<font color="red">続行する前に、クラシック バージョンの Cloud Shell に切り替えたことを確認します。</font>**

1. Cloud Shell 画面で、次のコマンドを入力して、この演習のコード ファイルを含む GitHub リポジトリをクローンします (コマンドを入力するか、クリップボードにコピーしてから、コマンド ラインで右クリックし、プレーンテキストとして貼り付けます)。

    ```
    rm -r semantic-kernel -f
    git clone https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK semantic-kernel
    ```

    > **ヒント**: Cloudshell にコマンドを貼り付けると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. リポジトリが複製されたら、チャット アプリケーションのコード ファイルを含んだフォルダーに移動します。

    > **注**: 選択したプログラミング言語の手順に従います。

    **Python**
    ```
    cd semantic-kernel/Allfiles/Labs/Devops/python
    ```

    **C#**
    ```
    cd semantic-kernel/Allfiles/Labs/Devops/c-sharp
    ```

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    **Python**
    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    **C#**
    ```
    dotnet add package Microsoft.Extensions.Configuration
    dotnet add package Microsoft.Extensions.Configuration.Json
    dotnet add package Microsoft.SemanticKernel
    dotnet add package Microsoft.SemanticKernel.PromptTemplates.Handlebars
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    **Python**
    ```
    code .env
    ```

    **C#**
    ```
    code appsettings.json
    ```

    このファイルをコード エディターで開きます。

1. お使いの Azure OpenAI Services のモデル ID、エンドポイント、API キーで値を更新します。

    **Python**
    ```python
    MODEL_DEPLOYMENT=""
    BASE_URL=""
    API_KEY="
    ```

    **C#**
    ```json
    {
        "modelName": "",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. 値を更新したら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

## Semantic Kernel プラグインを作成する

1. 次のコマンドを入力して、提供されているコード ファイルを編集します。

    **Python**
    ```
    code devops.py
    ```

    **C#**
    ```
    code Program.cs
    ```

1. **Create a kernel builder with Azure OpenAI chat completion** (Azure OpenAI チャットによる候補を使ってカーネル ビルダーを作成する) というコメントの下に次のコードを追加します。

    **Python**
    ```python
    # Create a kernel builder with Azure OpenAI chat completion
    kernel = Kernel()
    chat_completion = AzureChatCompletion(
        deployment_name=deployment_name,
        api_key=api_key,
        base_url=base_url,
    )
    kernel.add_service(chat_completion)
    ```
    **C#**
     ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
    var kernel = builder.Build();
    ```

1. ファイルの下部付近にある **Create a kernel function to build the stage environment** (ステージング環境を構築するカーネル関数を作成する) というコメントの下に、次のコードを追加して、ステージング環境を構築するモック プラグイン関数を作成します。

    **Python**
    ```python
    # Create a kernel function to build the stage environment
    @kernel_function(name="BuildStageEnvironment")
    def build_stage_environment(self):
        return "Stage build completed."
    ```

    **C#**
    ```c#
    // Create a kernel function to build the stage environment
    [KernelFunction("BuildStageEnvironment")]
    public string BuildStageEnvironment() 
    {
        return "Stage build completed.";
    }
    ```

    `KernelFunction` デコレーターでは、ネイティブ関数を宣言します。 AI が正しく呼び出せるように、わかりやすい名前を関数に付けます。 

1. **Import plugins to the kernel** (カーネルにプラグインをインポートする) というコメントの下に、次のコードを追加します。

    **Python**
    ```python
    # Import plugins to the kernel
    kernel.add_plugin(DevopsPlugin(), plugin_name="DevopsPlugin")
    ```

    **C#**
    ```c#
    // Import plugins to the kernel
    kernel.ImportPluginFromType<DevopsPlugin>();
    ```


1. **Create prompt execution settings** (プロンプト実行の設定を作成する) というコメントの下に次のコードを追加して、関数が自動的に呼び出されるようにします。

    **Python**
    ```python
    # Create prompt execution settings
    execution_settings = AzureChatPromptExecutionSettings()
    execution_settings.function_choice_behavior = FunctionChoiceBehavior.Auto()
    ```

    **C#**
    ```c#
    // Create prompt execution settings
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };
    ```

    この設定を使用すると、プロンプトで関数を指定しなくても、カーネルが関数を自動的に呼び出せるようになります。

1. **Create chat history** (チャット履歴を作成する) というコメントの下に、次のコードを追加します。

    **Python**
    ```python
    # Create chat history
    chat_history = ChatHistory()
    ```

    **C#**
    ```c#
    // Create chat history
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
    ChatHistory chatHistory = [];
    ```

1. **User interaction logic** (ユーザーとのやり取りのロジック) というコメントの後にあるコード ブロックのコメントを解除します。

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。

## DevOps アシスタント コードを実行する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して Azure にサインインします。

    ```
    az login
    ```

    **<font color="red">Cloud Shell セッションが既に認証されている場合でも、Azure にサインインする必要があります。</font>**

    > **注**: ほとんどのシナリオでは、*[az ログイン]* を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*--tenant* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。

1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、プロンプトが表示されたら、必要な Azure AI Foundry ハブを含むサブスクリプションを選択します。

1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。


    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. メッセージが表示されたら、次のプロンプトを入力します。`Please build the stage environment`

1. 次の出力のような応答が表示されます。

    ```output
    Assistant: The stage environment has been successfully built.
    ```

1. 次に、以下のプロンプトを入力します。`Please deploy the stage environment`

1. 次の出力のような応答が表示されます。

    ```output
    Assistant: The staging site has been deployed successfully.
    ```

1. <kbd>Enter</kbd> キーを押してプログラムを終了します。

## プロンプトからカーネル関数を作成する

1. `Create a kernel function to deploy the staging environment` というコメントの下に、次のコードを追加します。

     **Python**
    ```python
    # Create a kernel function to deploy the staging environment
    deploy_stage_function = KernelFunctionFromPrompt(
        prompt="""This is the most recent build log:
        {{DevopsPlugin.ReadLogFile}}

        If there are errors, do not deploy the stage environment. Otherwise, invoke the stage deployment function""",
        function_name="DeployStageEnvironment",
        description="Deploy the staging environment"
    )

    kernel.add_function(plugin_name="DeployStageEnvironment", function=deploy_stage_function)
    ```

    **C#**
    ```c#
    // Create a kernel function to deploy the staging environment
    var deployStageFunction = kernel.CreateFunctionFromPrompt(
    promptTemplate: @"This is the most recent build log:
    {{DevopsPlugin.ReadLogFile}}

    If there are errors, do not deploy the stage environment. Otherwise, invoke the stage deployment function",
    functionName: "DeployStageEnvironment",
    description: "Deploy the staging environment"
    );

    kernel.Plugins.AddFromFunctions("DeployStageEnvironment", [deployStageFunction]);
    ```

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリケーションを実行します。

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. メッセージが表示されたら、次のプロンプトを入力します。`Please deploy the stage environment`

1. 次の出力のような応答が表示されます。

    ```output
    Assistant: The stage environment cannot be deployed because the earlier stage build failed due to unit test errors. Deploying a faulty build to stage may cause eventual issues and compromise the environment.
    ```

    LLM からの応答はこれとは異なる可能性がありますが、それでもステージ サイトをデプロイできなくなります。

## Handlebars プロンプトを作成する

1. **Create a handlebars prompt** (ハンドルバー プロンプトを作成する) というコメントの下に次のコードを追加します。

    **Python**
    ```python
    # Create a handlebars prompt
    hb_prompt = """<message role="system">Instructions: Before creating a new branch for a user, request the new branch name and base branch name/message>
        <message role="user">Can you create a new branch?</message>
        <message role="assistant">Sure, what would you like to name your branch? And which base branch would you like to use?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">"""
    ```

    **C#**
    ```c#
    // Create a handlebars prompt
    string hbprompt = """
        <message role="system">Instructions: Before creating a new branch for a user, request the new branch name and base branch name/message>
        <message role="user">Can you create a new branch?</message>
        <message role="assistant">Sure, what would you like to name your branch? And which base branch would you like to use?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">
        """;
    ```

    このコードでは、Handlebars テンプレート フォーマットを使用して、少数ショット プロンプトを作成します。 このプロンプトは、新しいブランチを作成する前にユーザーからより多くの情報を取得するようにモデルを誘導します。

1. **Create the prompt template config using handlebars format** (ハンドルバー形式を使ってプロンプト テンプレート構成を作成する) というコメントの下に次のコードを追加します。

    **Python**
    ```python
    # Create the prompt template config using handlebars format
    hb_template = HandlebarsPromptTemplate(
        prompt_template_config=PromptTemplateConfig(
            template=hb_prompt, 
            template_format="handlebars",
            name="CreateBranch", 
            description="Creates a new branch for the user",
            input_variables=[
                InputVariable(name="input", description="The user input", is_required=True)
            ]
        ),
        allow_dangerously_set_content=True,
    )
    ```

    **C#**
    ```c#
    // Create the prompt template config using handlebars format
    var templateFactory = new HandlebarsPromptTemplateFactory();
    var promptTemplateConfig = new PromptTemplateConfig()
    {
        Template = hbprompt,
        TemplateFormat = "handlebars",
        Name = "CreateBranch",
    };
    ```

    このコードでは、プロンプトから Handlebars テンプレート構成を作成します。 これを使用して、プラグイン関数を作成できます。

1. **Create a plugin function from the prompt** (プロンプトからプラグイン関数を作成する) というコメントの下に次のコードを追加します。 

    **Python**
    ```python
    # Create a plugin function from the prompt
    prompt_function = KernelFunctionFromPrompt(
        function_name="CreateBranch",
        description="Creates a branch for the user",
        template_format="handlebars",
        prompt_template=hb_template,
    )
    kernel.add_function(plugin_name="BranchPlugin", function=prompt_function)
    ```

    **C#**
    ```c#
    // Create a plugin function from the prompt
    var promptFunction = kernel.CreateFunctionFromPrompt(promptTemplateConfig, templateFactory);
    var branchPlugin = kernel.CreatePluginFromFunctions("BranchPlugin", [promptFunction]);
    kernel.Plugins.Add(branchPlugin);
    ```

    このコードでは、プロンプトのプラグイン関数を作成して、カーネルに追加します。 これで、関数を呼び出す準備が整いました。

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリケーションを実行します。

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. メッセージが表示されたら、`Please create a new branch` というテキストを入力します。

1. 次の出力のような応答が表示されます。

    ```output
    Assistant: Could you please provide the following details?

    1. The name of the new branch.
    2. The base branch from which the new branch should be created.
    ```

1. `feature-login main` というテキストを入力します。

1. 次の出力のような応答が表示されます。

    ```output
    Assistant: The new branch `feature-login` has been successfully created from `main`.
    ```

## アクションにユーザーの同意を要求する

1. ファイルの下部付近にある **Create a function filter** (関数フィルターを作成する) というコメントの下に、次のコードを追加します。

    **Python**
    ```python
    # Create a function filter
    async def permission_filter(context: FunctionInvocationContext, next: Callable[[FunctionInvocationContext], Awaitable[None]]) -> None:
        await next(context)
        result = context.result
        
        # Check the plugin and function names
    ```

    **C#**
    ```c#
    // Create a function filter
    class PermissionFilter : IFunctionInvocationFilter
    {
        public async Task OnFunctionInvocationAsync(FunctionInvocationContext context, Func<FunctionInvocationContext, Task> next)
        {
            // Check the plugin and function names
            
            await next(context);
        }
    }
    ```

1. **Check the plugin and function names** (プラグイン名と関数名を確認する) というコメントの下に次のコードを追加して、`DeployToProd` 関数が呼び出されたときにそれを検出します。

     **Python**
    ```python
    # Check the plugin and function names
    if context.function.plugin_name == "DevopsPlugin" and context.function.name == "DeployToProd":
        # Request user approval
        
        # Proceed if approved
    ```

    **C#**
    ```c#
    // Check the plugin and function names
    if ((context.Function.PluginName == "DevopsPlugin" && context.Function.Name == "DeployToProd"))
    {
        // Request user approval

        // Proceed if approved
    }
    ```

    このコードでは、`FunctionInvocationContext` オブジェクトを使用して、呼び出されたプラグインと関数を特定します。

1. 次のロジックを追加して、ユーザーがフライトを予約できるアクセス許可を要求します。

     **Python**
    ```python
    # Request user approval
    print("System Message: The assistant requires approval to complete this operation. Do you approve (Y/N)")
    should_proceed = input("User: ").strip()

    # Proceed if approved
    if should_proceed.upper() != "Y":
        context.result = FunctionResult(
            function=result.function,
            value="The operation was not approved by the user",
        )
    ```

    **C#**
    ```c#
    // Request user approval
    Console.WriteLine("System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)");
    Console.Write("User: ");
    string shouldProceed = Console.ReadLine()!;

    // Proceed if approved
    if (shouldProceed != "Y")
    {
        context.Result = new FunctionResult(context.Result, "The operation was not approved by the user");
        return;
    }
    ```

1. **Add filters to the kernel** (カーネルにフィルターを追加する) というコメントの下に、次のコードを追加します。

    **Python**
    ```python
    # Add filters to the kernel
    kernel.add_filter('function_invocation', permission_filter)
    ```

    **C#**
    ```c#
    // Add filters to the kernel
    kernel.FunctionInvocationFilters.Add(new PermissionFilter());
    ```

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリケーションを実行します。

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. ビルドを運用環境にデプロイするためのプロンプトを入力します。 次のような応答が表示されます。

    ```output
    User: Please deploy the build to prod
    System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)
    User: N
    Assistant: I'm sorry, but I am unable to proceed with the deployment.
    ```

### 確認

このラボでは、大規模言語モデル (LLM) サービスのエンドポイントを作成し、Semantic Kernel オブジェクトを構築し、Semantic Kernel SDK を使用してプロンプトを実行しました。 また、プラグインを作成し、システム メッセージを利用してモデルを誘導しました。 以上でこのラボは完了です。