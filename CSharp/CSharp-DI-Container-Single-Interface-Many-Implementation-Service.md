# 使用相同的 Interface，透過 .NET C# 程式碼，註冊多個服務，並且可以取得不同條件的實作服務物件

![](../Images/X2023-9815.png)

這篇文章主要是要探討，如何使用相同的 Interface，透過 .NET C# 程式碼，註冊多個服務，並且可以取得不同條件的實作服務物件。若要完成這樣的需求，究竟需要使用甚麼方式來進行設計與開發呢？這篇文章將會透過 .NET C# 程式碼，來說明如何完成這樣的需求。

由於 .NET8 平台有提供 [.NET 泛型主機](https://learn.microsoft.com/zh-tw/dotnet/core/extensions/generic-host?tabs=hostbuilder&WT.mc_id=DT-MVP-5002220) 的功能，在 .NET 泛型主機 服務內，將會提供 DI 相依性注入容器功能，因此，便可以直接使用 .NET 8 內建的 DI 容器來時做出這篇文章的要探討目的。

## 建立測試專案

* 打開 Visual Studio 2022 IDE 應用程式
* 從 [Visual Studio 2022] 對話窗中，點選右下方的 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗右半部
  * 切換 [所有語言 (L)] 下拉選單控制項為 [C#]
  * 切換 [所有專案類型 (T)] 下拉選單控制項為 [主控台]
* 在中間的專案範本清單中，找到並且點選 [主控台應用程式] 專案範本選項
  > 專案，用於建立可在 Windows、Linux 及 macOS 於 .NET 執行的命令列應用程式
* 點選右下角的 [下一步] 按鈕
* 在 [設定新的專案] 對話窗
* 找到 [專案名稱] 欄位，輸入 `csDISameInterface` 作為專案名稱
* 在剛剛輸入的 [專案名稱] 欄位下方，確認沒有勾選 [將解決方案與專案至於相同目錄中] 這個檢查盒控制項
* 點選右下角的 [下一步] 按鈕
* 現在將會看到 [其他資訊] 對話窗
* 在 [架構] 欄位中，請選擇最新的開發框架，這裡選擇的 [架構] 是 : `.NET 8.0 (標準字詞支援)`
* 在這個練習中，需要去勾選 [不要使用最上層陳述式(T)] 這個檢查盒控制項
  > 這裡的這個操作，可以由讀者自行決定是否要勾選這個檢查盒控制項
* 請點選右下角的 [建立] 按鈕

稍微等候一下，這個主控台專案將會建立完成

## 安裝要用到的 NuGet 開發套件

因為開發此專案時會用到這些 NuGet 套件，請依照底下說明，將需要用到的 NuGet 套件安裝起來。

### 安裝 Microsoft.Extensions.Hosting 套件

關於 Microsoft.Extensions.Hosting 套件，可以參考 [https://www.nuget.org/packages/Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) 這個網址。

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csDISameInterface] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `Microsoft.Extensions.Hosting`
* 稍待一會，將會在下方看到這個套件被搜尋出來
* 點選 [Microsoft.Extensions.Hosting] 套件名稱
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

## 開始進行程式設計

* 在此專案節點下，找到並且打開 [Program.cs] 這個檔案
* 使用底下 C# 程式碼替換掉 [Program.cs] 檔案內所有程式碼內容

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace csDISameInterface;

public interface IService
{
    void Get();
}
public class ServiceA : IService
{
    public void Get()
    {
        Console.WriteLine("ServiceA");
    }
}

public class ServiceB : IService
{
    public void Get()
    {
        Console.WriteLine("ServiceB");
    }
}

public class ServiceC : IService
{
    public void Get()
    {
        Console.WriteLine("ServiceC");
    }
}
internal class Program
{
    delegate IService ServiceResolver(string key);
    static void Main(string[] args)
    {
        IHost host = Host.CreateDefaultBuilder()
            .ConfigureServices((context, services) =>
            {
                services.AddTransient<ServiceA>();
                services.AddTransient<ServiceB>();
                services.AddTransient<ServiceC>();
                services.AddTransient<Func<string, IService>>(serviceProvider => key =>
                {
                    return key switch
                    {
                        "A" => serviceProvider.GetService<ServiceA>(),
                        "B" => serviceProvider.GetService<ServiceB>(),
                        "C" => serviceProvider.GetService<ServiceC>(),
                        _ => throw new KeyNotFoundException()
                    };
                });

                services.AddTransient<ServiceResolver>(serviceProvider => key =>
                {
                    switch (key)
                    {
                        case "A":
                            return serviceProvider.GetService<ServiceA>();
                        case "B":
                            return serviceProvider.GetService<ServiceB>();
                        case "C":
                            return serviceProvider.GetService<ServiceC>();
                        default:
                            throw new KeyNotFoundException();
                    }
                });
            }).Build();

        var serviceAccessor1 = host.Services.GetRequiredService<Func<string, IService>>();
        var serviceAccessor2 = host.Services.GetRequiredService<ServiceResolver>();
        IService service = serviceAccessor1("A");
        service.Get();
        service = serviceAccessor2("C");
        service.Get();
    }
}
```

在這裡首先會設計一個介面 [IService] ，該介面內將會宣告一個 Get 方法，並且設計三個實作類別 [ServiceA]、[ServiceB]、[ServiceC]，這三個實作類別都會實作 [IService] 這個介面。

在實作的類別內，僅僅使用 Console.WriteLine 方法來輸出訊息，以便於在執行時，可以看到是哪個實作類別被呼叫。

接著，透過 .NET 8 內建的 DI 容器，來註冊這三個實作類別，並且註冊一個 Func 委派，這個委派可以透過傳入不同的參數，來取得不同的實作類別物件。

在程式進入點的第一行，將會使用 [Host.CreateDefaultBuilder()] 方法來建立一個泛型主機物件，並且在這個泛型主機物件內，透過 [ConfigureServices()] 方法來註冊服務到 DI 容器內，這裡會註冊三個具體實作類別，並且註冊一個 Func 委派，這個委派可以透過傳入不同的參數，來取得不同的實作類別物件。

註冊這個 Func 委派的程式碼如下：

```csharp
services.AddTransient<Func<string, IService>>(serviceProvider => key =>
{
    return key switch
    {
        "A" => serviceProvider.GetService<ServiceA>(),
        "B" => serviceProvider.GetService<ServiceB>(),
        "C" => serviceProvider.GetService<ServiceC>(),
        _ => throw new KeyNotFoundException()
    };
});
```

在這裡註冊一個委派方法，一旦透過 DI 容器完成註冊之後，就可以透過相依性注入的方式來注入 `Func<string, IService>` 物件，該委派方法將會傳入一個字串，此時，將會依據傳入的字串，來決定要取得哪一個實作類別物件。

當在傳入字串之後，將會透過 switch case 來判斷傳入的字串，並且依據傳入的字串，來決定要取得哪一個實作類別物件，最後，將會透過 `serviceProvider.GetService<ServiceA>()` 這個方法來取得實作類別物件，並且將該物件回傳給呼叫端。

對於 AddTransient 方法，將會使用底下函式簽章

```csharp
public static IServiceCollection AddTransient<TService>(
    this IServiceCollection services,
    Func<IServiceProvider, TService> implementationFactory)
    where TService : class;
```

在這裡 AddTransient 方法，將會傳入一個 Func 委派，這個委派將會傳入一個 IServiceProvider 物件，並且回傳一個 TService 物件，這裡的 TService 物件，將會是 IService 介面的實作類別物件。因此，這裡的委派方法，將會得到一個 IServiceProvider 物件，並且透過 IServiceProvider 物件來取得 IService 介面的實作類別物件，最後，將該物件回傳給呼叫端。

想要取得這個委派物件，可以透過底下程式碼來取得

```csharp
var serviceAccessor1 = host.Services.GetRequiredService<Func<string, IService>>();
```

一旦取得了 serviceAccessor1 物件之後，便可以透過傳入不同的字串，來取得不同的實作類別物件，例如，透過傳入 "A" 這個字串，便可以取得 ServiceA 這個實作類別物件，透過傳入 "B" 這個字串，便可以取得 ServiceB 這個實作類別物件，透過傳入 "C" 這個字串，便可以取得 ServiceC 這個實作類別物件。

在這裡又取得了 [IService] 型別的 [service] 物件之後，便可以呼叫這個 [service] 物件的 Get 方法，來輸出訊息。

```csharp
IService service = serviceAccessor1("A");
service.Get();
```







