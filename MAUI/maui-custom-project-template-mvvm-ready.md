# 使用客製化 MAUI 專案範本來進行開發

![](../Images/X2023-9807.png)

當要進行 .NET MAUI 專案開發時，通常會使用 Visual Studio 提供的預設專案範本來建立專案，這裡將會有兩種做法，一種是使用 Visual Studio 2022 來建立專案，另一種是使用 .NET CLI 來建立專案。

* 使用 Visual Studio 2022 來建立專案
  * 開啟 Visual Studio 2022 程式
  * 點選右下角的 **建立新的專案**
  * 當出現 [建立新的專案] 視窗時，在該對話窗中間上方的 [所有語言 (L)] 下拉式選單中，選擇 **C#**
  * 在對話窗最上方的右側的 [所有專案類型 (T)] 下拉式選單中，選擇 **MAUI**
  * 現在可以在這個對話窗中，看到 [.NET MAUI 應用程式] 的範本，對於該範本的描述為：此專案可用於建立適用於 iOS、Android、Mac Catalyst、Tizen和 Win UI 的 .NET MAUI 應用程式
  * 如便可以建立起一個 .NET MAUI 的開發專案，畫面如下所示
  
    ![](../Images/X2023-9804.png)

    ![](../Images/X2023-9803.png)

* 若要使用 .NET CLI 來建立專案
  * 請在命令提示字元視窗下，輸入下列指令來建立專案

```
dotnet new maui -o FirstMaui
```

不論是透過 GUI 或者 CLI 的方式來建立起來的專案，這些預設範本並不會包含 MVVM 的相關套件與相關的資料夾等等，因此當每次要建立一個 .NET MAUI 專案之後，就需要開始進行一下重複性的工作，現在，可以透過 [Vulcan.Maui.Template](https://www.nuget.org/packages/Vulcan.Maui.Template) 這個客製化的專案範本來建立 .NET MAUI 專案，這樣就可以省去後續繁複的開發時間。

現在來檢視使用預設 .NET MAUI 專案範本所建立起來的專案樣貌

![](../Images/X2023-9802.png)

這裡將會使用 Visual Studio 2022 來開啟剛剛建立的專案，從方案總管中可以看到，這裡僅有兩個預設的 [Platforms] 與 [Resources] 資料夾，這兩個資料夾是 .NET MAUI 專案的必要資料夾。

對於 .NET MAUI 的程式進入點程式碼，將會用到 [MauiProgram.cs] / [App.xaml] / [AppShell.xaml] / [MainPage.xaml] 這四個檔案，這些檔案都是預設的範本檔案，這裡僅列出 [MauiProgram.cs] 的程式碼，如下所示

```csharp
using Microsoft.Extensions.Logging;

namespace FirstMaui
{
    public static class MauiProgram
    {
        public static MauiApp CreateMauiApp()
        {
            var builder = MauiApp.CreateBuilder();
            builder
                .UseMauiApp<App>()
                .ConfigureFonts(fonts =>
                {
                    fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
                    fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
                });

#if DEBUG
    		builder.Logging.AddDebug();
#endif

            return builder.Build();
        }
    }
}
```

## 安裝 Vulcan.Maui.Template 專案範本

* 首先，需要開啟命令提示字元視窗
* 接著輸入下列指令來查詢目前已經安裝的專案範本

```
dotnet new list
```

![](../Images/X2023-9806.png)

* 若發現到已經有安裝 [Vulcan.Maui.Template] 專案範本，可以著輸入下列指令來移除 Vulcan.Maui.Template 專案範本

```
dotnet new uninstall Vulcan.Maui.Template
```

* 接著輸入下列指令來安裝 [Vulcan.Maui.Template](https://www.nuget.org/packages/Vulcan.Maui.Template) 專案範本到這台開發機上

```
dotnet new install Vulcan.Maui.Template
```

底下是安裝完成的畫面

![](../Images/X2023-9805.png)

## 開始使用 Vulcan.Maui.Template 專案範本

* 想要使用 [Vulcan.Maui.Template] 專案範本來建立 .NET MAUI 專案，可以透過 CLI 操作方式，輸入下列指令來建立專案

```
c:
cd c:\temp
dotnet new Vulcan-Maui -o c:\Temp\MA12
```

* 底下為操作過程的畫面

  ![](../Images/X2023-9801.png)

* 對於使用 CLI 方式建立的專案，其檔案結構與內容如下所示

  ![](../Images/X2023-9800.png)

* 若想要使用 GUI 方式來建立專案，可以開啟 Visual Studio 2022 應用程式
* 在 Visual Studio 2022 對話視窗中，點選右下角的 **建立新的專案**
* 當出現 [建立新的專案] 對話視窗
* 在該對話窗中間上方的 [所有語言 (L)] 下拉式選單中，選擇 **C#**
* 在對話窗最上方的右側的 [所有專案類型 (T)] 下拉式選單中，選擇 **MAUI**
* 現在可以在這個對話窗中，看到 [Vulcan Template .NET MAUI (Vulcan)] 的範本，對於該範本的描述為：此專案可用於建立適用於 iOS、Android、Mac Catalyst、Tizen和 Win UI 的 .NET MAUI 應用程式，如下圖所示

   ![](../Images/X2023-9799.png)
* 點選 [Vulcan Template .NET MAUI (Vulcan)] 的範本，接著點選 [下一步(N)] 按鈕
* 現在將會看到 [設定新的專案] 對話視窗
* 在 [專案名稱] 文字方塊中，輸入 **MA12**
* 點選螢幕右下方的 [建立] 按鈕
  ![建立新的專案](../Images/X2023-9798.png)
* 稍微等候一下，專案就會建立完成

## 檢視 Vulcan.Maui.Template 專案範本

現在使用 Visual Studio 2022 來打開剛剛建立的專案，從方案總管中可以看到，這裡有許多的資料夾，這些資料夾都是 .NET MAUI 專案的必要資料夾，這些資料夾的用途如下所示

![](../Images/X2023-9797.png)

* [Views] 資料夾：用來放置所有的頁面，每一個頁面都是一個 .xaml 檔案，例如：[MainPage.xaml] / [AboutPage.xaml] / [SettingsPage.xaml] 等等
* [ViewModels] 資料夾：用來放置所有的 ViewModel，每一個 ViewModel 都是一個 .cs 檔案，例如：[MainPageViewModel.cs] / [AboutPageViewModel.cs] / [SettingsPageViewModel.cs] 等等
* [Services] 資料夾：用來放置所有的服務類別，每一個服務類別都是一個 .cs 檔案，例如：[IAppInfoService.cs] / [AppInfoService.cs] 等等
* [Models] 資料夾：用來放置所有的資料模型類別，每一個資料模型類別都是一個 .cs 檔案，例如：[MessageModel.cs] / [ProductModel.cs] 等等
* [Helpers] 資料夾：用來放置所有的輔助類別，每一個輔助類別都是一個 .cs 檔案，例如：[MagicValueHelper.cs] / [ApiHelper.cs] 等等
* [Events] 資料夾：用來放置所有的事件類別，每一個事件類別都是一個 .cs 檔案，例如：[MyValueChangedMessage.cs] 等等

使用滑鼠雙擊這個 .NET MAUI 專案節點，就可以看到專案的內容，如下圖所示

![](../Images/X2023-9796.png)

這表示了這個專案除了原先 .NET MAUI 專案範本會用到的專案之外，還多了一些額外的套件，這些額外的套件都是為了讓開發者可以更快速的進行開發，這些額外的套件如下所示

* [CommunityToolkit.Mvvm] 套件：這是一個社群開發的 MVVM 套件，透過原始碼產生器來生成在 MVVM 設計模式下會用到的額外程式碼，如此，讓整體 ViewModel 的原始碼看起來更加簡潔，並且可以讓開發者專注在 ViewModel 的商業邏輯上，而不是在繁瑣的程式碼上；另外一個好處將會是，可以大幅提升整體開發的速度與品質。
* [CommunityToolkit.Maui] 套件：這是一個社群開發的 .NET MAUI 套件，這個套件提供了許多 .NET MAUI 專案開發時會用到的額外功能，這些功能都是在 .NET MAUI 專案開發時會用到的功能，透過這個套件，可以讓開發者更加快速的進行開發。
* [Newtonsoft.Json] 套件：這是一個 JSON 處理的套件，這個套件可以讓開發者更加快速的進行 JSON 資料的序列化與反序列化，這個套件在 .NET MAUI 專案開發時會用到的功能。






