WebApi用にTemplateを一部カスタマイズ

# ASP.NET Core WebApi 3.1 + Vue (TypeScript)構成のテンプレート

デバック実行時、ビルド時にサーバーサイドのリソースだけでなく、フロントエンドのリソースに対しても操作するように環境構築。    
TypeScriptはvue-cliからインストールしているだけなので、wwwroot配下にもTypescriptを設置したい場合は、別途dotnet側でTypescriptへのコンパイル設定が必要になる。

# 前提条件
サーバーサイドの開発に、.NET Core SDKが必要。   
クライアントサイドの開発に、Node.jsが必要。 
プロジェクトに合わせて、インストールするバージョンは選定しておく。

# Projectの作成
Projectを作成するフォルダを開く。特に理由がない限りはプロジェクトの名前が付いた空のフォルダ内で下記を実行する。

```
dotnet new webapi
dotnet add package Microsoft.AspNetCore.SpaServices.Extensions
```

一つ目のコマンドで、ターミナルのカレントディレクトリ上にWebApiのテンプレートプロジェクトが作成される。  
2つめのコマンドでは、Spa開発用のNuGetパッケージを取得している。


# vue-cliのインストール

vue開発用のコマンドラインツール。
vue-cliはプロジェクトに対してではなく、端末に対してインストールするため、
すでにインストールしている場合は実行する必要はない。
下記コマンドは省略して書いているが、`npm install --global @vue-cli`を実行するのと同じ。
```
npm i -g @vue-cli
```

# Vueセットアップ

下記コマンドを実行すると、frontendと言う名前で、vueテンプレートを含むフォルダが作成される。
vueフォルダ名はclientapp等でももちろん良い。

```
vue create frontend
```
プロジェクトに合わせてテンプレートを設定する。
Manually Selectで設定を選ぶ。

おすすめは、
Babel, TypeScript, Router, Vuexにチェックを入れること。
Babelは主にIE11や少しバージョンが古いブラウザで動作させるために入れる。
TypeScirptはプロジェクトで採用したくないというならば別であるが、Vueの開発で見通しをよくするためや、 
型推論による恩恵をうけるために採用したほうがよい。  
JavaScriptのスーパーセットなので、最悪、拡張子をTSに変えただけと言う状況で動かすこともできる。
TypeScriptによる型の指定はプロジェクトメンバーのスキルに応じて臨機応変に変えて運用することができる。    
RouterとVuexはVueの中規模以上の開発をする場合には必須である。   
(そもそもこの２つがないとSPAとして成り立たない。)
Linter/Formatterはコーディング規約を厳密にするためのモジュールなので、プロジェクトに合わせて選択するかを選ぶ。
コンポーネントをクラス記法で記述できるように、`Use Class-Syntax...`はチェックを入れる。  
`Use Babel alongside...`はTypeScriptとBabelを一緒に使うのでチェックを入れる。
TypeScriptはESNEXTに変換された後、BabelでES5に変換する。(ES5はほとんどのブラウザで動くJavaScriptのバージョン)

`Use History mode...`はチェックを入れる。
Vue Routerは、画面内の移動としてブラウザに認識させるために、コンパイル後にSPA内のRoutingパスのRootに#をつける**ハッシュモード**で動作する。    
ただし、サーバ側でfall-backルートをSPAのビルド後のパスに設定すれば、**History mode**で動作する。  
History modeの仕組み上、サーバ側でrootingマッチしなかった場合は常にSPAを返すようになるため、ルートマッチングしなかった場合の404ページはSPA側で用意する必要がある。


cliではなく、ブラウザからGUI操作でvueプロジェクトの作成/管理を行う場合には`vue ui`を実行すること。  
すでにプロジェクトをいくつか作成している場合は、作成済みのプロジェクトの管理ページが開くことがあるが、
その場合は、画面左上に表示されているプロジェクト名をクリックしてvueプロジェクトマネージャーを開くこと。

# フロントエンドライブラリの追加

ajaxでフロントエンド側とサーバのやり取りをするため、axiosをインストールしておく。
```
cd frontend
npm i axios
```
vueコンポーネントライブラリとして、Vuetifyを使用する場合は、`vue add vuetify`を実行する。   
※`vue add`で追加した場合は、テンプレートプロジェクトに含まれているコンポーネントが一部変更される。
その他フロントエンドアプリにモジュールを追加する必要がある場合は、frontendフォルダ内で`npm i`を実行すること。   
おすすめのライブラリを列挙しておく。

### vue-moment
日付処理ライブラリ  
日付の解析・検証・操作・書式設定ができる、moment.jsをvue用に派生させたもの。  
生のJavaScriptで日付操作するのはしんどいのでよっぽどのことがない限りは入れましょう。    
なおmoment.jsは日付処理ライブラリの中でもかなり昔からありますが、最近ではdays.jsのような軽量ライブラリも登場しています。

main.ts
```ts
Vue.use(require('vue-moment'))
```

vue-chart


# フロントエンド開発サーバの設定
開発サーバはデフォルトで8080ポートを使用するが、セキュリティソフトによってはローカル環境であってもこのポートを監視しているため、負荷が上がってしまう。  
対策として、監視に使用していないポートを指定する。
frontendフォルダ直下に下記ファイルを追加する。
`vue add vuetify`を実行している場合はすでにvue.config.jsが作成されているので、
その中にdevServerの設定を追加する。

vue.config.js
```js
module.exports = {
  "devServer":{
    "port": 8010,
  },
}

```
# Project直下の設定ファイルの変更

## 本番用にcsprojを編集する

ビルドの設定を追加する。

template_core_api_vue.csproj
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
	<PropertyGroup>
		<TargetFramework>netcoreapp3.1</TargetFramework>
		<SpaRoot>frontend\</SpaRoot>
    	<DefaultItemExcludes>$(DefaultItemExcludes);$(SpaRoot)node_modules\**</DefaultItemExcludes>
    	<TypeScriptCompileBlocked>true</TypeScriptCompileBlocked>
    	<TypeScriptToolsVersion>Latest</TypeScriptToolsVersion>
    	<IsPackable>false</IsPackable>
	</PropertyGroup>
	<Target Name="PublishRunWebpack" AfterTargets="ComputeFilesToPublish">

    <Exec WorkingDirectory="$(SpaRoot)" Command="npm install" />
    <Exec WorkingDirectory="$(SpaRoot)" Command="npm run build" />


    <ItemGroup>
      <DistFiles Include="$(SpaRoot)dist\**" />
      <ResolvedFileToPublish Include="@(DistFiles->'%(FullPath)')" Exclude="@(ResolvedFileToPublish)">
        <RelativePath>%(DistFiles.Identity)</RelativePath>
        <CopyToPublishDirectory>PreserveNewest</CopyToPublishDirectory>
      </ResolvedFileToPublish>
    </ItemGroup>
  </Target>
	<ItemGroup>
		<PackageReference Include="Microsoft.AspNetCore.SpaServices.Extensions" Version="3.1.6" />
	</ItemGroup>
	<ItemGroup>

    <Content Remove="$(SpaRoot)**" />
    <None Remove="$(SpaRoot)**" />
    <None Include="$(SpaRoot)**" Exclude="$(SpaRoot)node_modules\**" />
  </ItemGroup>

  <Target Name="DebugEnsureNodeEnv" BeforeTargets="Build" Condition=" '$(Configuration)' == 'Debug' And !Exists('$(SpaRoot)node_modules') ">

    <Exec Command="node --version" ContinueOnError="true">
      <Output TaskParameter="ExitCode" PropertyName="ErrorCode" />
    </Exec>
    <Error Condition="'$(ErrorCode)' != '0'" Text="Node.js is required to build and run this project. To continue, please install Node.js from https://nodejs.org/, and then restart your command prompt or IDE." />
    <Message Importance="high" Text="Restoring dependencies using 'npm'. This may take several minutes..." />
    <Exec WorkingDirectory="$(SpaRoot)" Command="npm install" />
  </Target>
</Project>

```

## SPA起動設定
開発環境でサーバ起動時にvueのビルドが走るように細工する。

ASP.NET CoreサーバからNodeをキックするためのMiddleware  
どこに追加してもいいが、サンプルではMiddlewareフォルダをプロジェクト直下に追加し、その中に作成している。    
Portを指定しているが、フロントエンド開発サーバの設定で設定したポートを指定すること。    
言うまでもなく、namespaceはプロジェクトに合わせてかえること。

VueConnection.cs
```cs
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.SpaServices;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Net.NetworkInformation;
using System.Runtime.InteropServices;
using System.Threading.Tasks;

namespace template_core_api_vue.Middleware.VueConnection
{
    public static class VueConnection
    {
        private static int Port { get; } = 8010;
        private static Uri DevelopmentServerEndpoint { get; } = new Uri($"http://localhost:{Port}");
        private static TimeSpan Timeout { get; } = TimeSpan.FromSeconds(300);

        private static string DoneMessage { get; } = "DONE  Compiled successfully in";

        public static void UseVueDevelopmentServer(this ISpaBuilder spa)
        {
            spa.UseProxyToSpaDevelopmentServer(async () =>
            {
                var loggerFactory = spa.ApplicationBuilder.ApplicationServices.GetService<ILoggerFactory>();
                var logger = loggerFactory.CreateLogger("Vue");

                if (IsRunning())
                {
                    return DevelopmentServerEndpoint;
                }


                var isWindows = RuntimeInformation.IsOSPlatform(OSPlatform.Windows);
                var processInfo = new ProcessStartInfo
                {
                    FileName = isWindows ? "cmd" : "npm",
                    Arguments = $"{(isWindows ? "/c npm " : "")}run serve",
                    WorkingDirectory = "frontend",
                    RedirectStandardError = true,
                    RedirectStandardInput = true,
                    RedirectStandardOutput = true,
                    UseShellExecute = false,
                };
                var process = Process.Start(processInfo);
                var tcs = new TaskCompletionSource<int>();
                _ = Task.Run(() =>
                {
                    try
                    {
                        string line;
                        while ((line = process.StandardOutput.ReadLine()) != null)
                        {
                            logger.LogInformation(line);
                            if (!tcs.Task.IsCompleted && line.Contains(DoneMessage))
                            {
                                tcs.SetResult(1);
                            }
                        }
                    }
                    catch (EndOfStreamException ex)
                    {
                        logger.LogError(ex.ToString());
                        tcs.SetException(new InvalidOperationException("'npm run serve' failed.", ex));
                    }
                });
                _ = Task.Run(() =>
                {
                    try
                    {
                        string line;
                        while ((line = process.StandardError.ReadLine()) != null)
                        {
                            logger.LogError(line);
                        }
                    }
                    catch (EndOfStreamException ex)
                    {
                        logger.LogError(ex.ToString());
                        tcs.SetException(new InvalidOperationException("'npm run serve' failed.", ex));
                    }
                });

                var timeout = Task.Delay(Timeout);
                if (await Task.WhenAny(timeout, tcs.Task) == timeout)
                {
                    throw new TimeoutException();
                }

                return DevelopmentServerEndpoint;
            });

        }

        private static bool IsRunning() => IPGlobalProperties.GetIPGlobalProperties()
                .GetActiveTcpListeners()
                .Select(x => x.Port)
                .Contains(Port);
    }
}
```

サーバ起動プログラムの内容を変える。    
Spa接続用に処理を追加する。

Startup.cs
```diff
using Microsoft.Extensions.Logging;
+ using template_core_api_vue.Middleware.VueConnection;

namespace template_core_api_vue
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();
+            services.AddSpaStaticFiles(configuration =>
+            {
+                configuration.RootPath = "frontend/dist";
+            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseHttpsRedirection();
+            app.UseSpaStaticFiles();
            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
+            app.UseSpa(spa =>
+            {
+                spa.Options.SourcePath = "frontend";
+                if (env.IsDevelopment())
+                {
+                    spa.UseVueDevelopmentServer();
+                }
+            });
        }
    }
}

```

# 開発証明書の信頼
開発環境用の証明書を信頼する。  
端末上で一度でも実行していれば不要。
```
dotnet dev-certs https --trust
```

# パッケージの復元
主に途中からプロジェクトに参画した場合に、行う。    
一般にgitのようなバージョン管理ツールを使用して開発を行う場合は、外部ライブラリ本体はgitの管理対象外にして、
ライブラリの名前とバージョンのみを設定ファイルに残す。設定ファイルの記載は直接いじらず、パッケージ管理ツールで管理する。  
NuGetパッケージは.csproj内に、node_moduleはpackage.json内に記載される。 

NuGetパッケージの復元    
```
dotnet restore
```
失敗した場合は、一度プロジェクト直下にあるbinフォルダとobjフォルダを削除し、TERMINALを再起動する。
その後もう一度コマンドを実行する。

node_moduleの再インストール
frontendフォルダ内で以下のコマンドを実行する。
```
npm ci
```
package-lock.jsonを元に特定のバージョンのパッケージが再インストールされる。


# 開発実行
F5
サーバー と フロントエンドの開発サーバの2つが立ち上がる。   
ただし、フロントエンドの起動にタイムアウトが設定されているため、適宜時間を書き換える。  
タイムアウトの秒数内で開発サーバが立ち上がらないばあいは、以下をコメントアウトする。コメントアウトする場合は、自動起動するブラウザは一旦終了させ、コンソールで起動完了のログメッセージが表示されたのを確認してから、ブラウザからサーバに接続する。

VueConnection.cs
```cs
var timeout = Task.Delay(Timeout);
if (await Task.WhenAny(timeout, tcs.Task) == timeout)
{
    throw new TimeoutException();
}
```

また、フロントエンド単体で起動する場合はTerminalでfrontend内に移動し、
`npm run serve`を実行する。


apiサーバのみ起動したい場合は、下記をコメントアウトしてF5起動する。

Startup.cs
```cs
app.UseSpa(spa =>
{
    spa.Options.SourcePath = "frontend";
    if (env.IsDevelopment())
    {
        spa.UseVueDevelopmentServer();
    }
});
```

# ビルド
```
dotnet Publish
```
bin\Debug\netcoreapp3.1\publish\
に出力される。

パス指定する場合は -o オプションをつけること。

# その他

ソース管理ツールを利用する場合は、  
.editorconfigや.gitignoreを追加した方がいいと思われる。
