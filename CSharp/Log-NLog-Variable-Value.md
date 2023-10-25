# .NET C# 使用 NLog 紀錄狀態物件值

![](../Images/X2023-9899.png)

當使用日誌套件提供的寫入日誌訊息的時候，通常會將指定的訊息寫入到 日誌系統指定的輸出目的地裝置上，例如：螢幕、檔案等等。可是有些時候，期望能夠將當時的 .NET 物件值寫入到日誌內，方便日後的系統除厝與維護之用。

因此，在這篇文章中，將會介紹如何使用 [NLog] 這個 .NET 日誌套件，來實現這樣的需求。

## 建立測試專案

請依照底下的操作，建立起這篇文章需要用到的練習專案

* 打開 Visual Studio 2022 IDE 應用程式
* 從 [Visual Studio 2022] 對話窗中，點選右下方的 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗右半部
  * 切換 [所有語言 (L)] 下拉選單控制項為 [C#]
  * 切換 [所有專案類型 (T)] 下拉選單控制項為 [主控台]
* 在中間的專案範本清單中，找到並且點選 [主控台應用程式] 專案範本選項
  > 專案，用於建立可在 Windows、Linux 及 macOS 於 .NET 執行的命令列應用程式
* 點選右下角的 [下一步] 按鈕
* 在 [設定新的專案] 對話窗
* 找到 [專案名稱] 欄位，輸入 `csLog04` 作為專案名稱
* 在剛剛輸入的 [專案名稱] 欄位下方，確認沒有勾選 [將解決方案與專案至於相同目錄中] 這個檢查盒控制項
* 點選右下角的 [下一步] 按鈕
* 現在將會看到 [其他資訊] 對話窗
* 在 [架構] 欄位中，請選擇最新的開發框架，這裡選擇的 [架構] 是 : `.NET 7.0 (標準字詞支援)`
* 在這個練習中，需要去勾選 [不要使用最上層陳述式(T)] 這個檢查盒控制項
  > 這裡的這個操作，可以由讀者自行決定是否要勾選這個檢查盒控制項
* 請點選右下角的 [建立] 按鈕

稍微等候一下，這個主控台專案將會建立完成

## 安裝要用到的 NuGet 開發套件

因為開發此專案時會用到這些 NuGet 套件，請依照底下說明，將需要用到的 NuGet 套件安裝起來。

### 安裝 Newtonsoft.Json 套件

這個套件將會是 Newtonsoft.Json 日誌架構的核心套件，它提供 JSON 物件之序列與反序列化操作。

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csLog04] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `Newtonsoft.Json`
* 點選 [Newtonsoft.Json] 套件名稱
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

### 安裝 NLog 套件

這個套件將會是 NLog 日誌架構的核心套件，它提供 NLog 日誌架構的核心功能。

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csLog04] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `NLog`
* 點選 [NLog] 套件名稱，請選擇作者為 [Jarek Kowalski,Kim Christensen,Julian Verdurmen] 的套件
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

### 安裝 NLog.Schema 套件

NLog.Schema 是一個 .NET NuGet 套件，它提供 NLog 日誌架構的 XML 定義。此定義可用於在 NLog 日誌配置中使用 XML 語法。

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csLog04] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `NLog.Schema`
* 點選 [NLog.Schema] 套件名稱，請選擇作者為 [Jarek Kowalski,Kim Christensen,Julian Verdurmen] 的套件
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內
* 此時，從方案總管視窗內，將會看到有個 [NLog.xsd] 檔案，這個檔案是 NLog.Schema 套件安裝後，自動產生的檔案
* 在 [方案總管] 內找到並且點選 [NLog.xsd] 檔案這個節點
* 從 [屬性] 視窗中，將 [複製到輸出目錄] 屬性值改為 [有更新時才複製]，這樣才能讓 [NLog.xsd] 檔案在執行時，能夠被複製到執行目錄內

  >若沒有發現到 [屬性] 視窗，請在 [Visual Studio] 功能表中，點選 [檢視] > [屬性視窗] 功能選項

## 建立 NLog.config 設定檔

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點]
* 從彈出功能表清單中，點選 [新增項目] 這個功能選項清單
* 此時，將會看到 [新增項目 - csLog04] 視窗
* 在左方的清單選項中，點選 [已安裝] > [C# 項目] > [資料] 節點
* 在該對話窗的中間區域，找到並點選 [XML 檔案]
* 在下方 [名稱] 欄位內，輸入 `NLog.config` 作為檔案名稱
* 點選右下方 [新增] 按鈕，將這個檔案加入到專案內
* 在 [方案總管] 內找到並且點選 [NLog.config] 檔案這個節點
* 從 [屬性] 視窗中，將 [複製到輸出目錄] 屬性值改為 [有更新時才複製]，這樣才能讓 [NLog.config] 檔案在執行時，能夠被複製到執行目錄內

  >若沒有發現到 [屬性] 視窗，請在 [Visual Studio] 功能表中，點選 [檢視] > [屬性視窗] 功能選項

* 使用底下的 XML 內容來替換掉這個檔案內的內容

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	  autoReload="true"
	  throwConfigExceptions="true"
	  internalLogLevel="Info"
	  internalLogFile="c:\temp\Sample-internal-nlog.txt"
	  >

	<!-- 在這裡宣告要用到的 Target 項目 -->
	<targets>
		<!-- 宣告 File 目標，將會把 Log 內容寫入到檔案內 -->
		<!--更多 Layout Renderer 變數，可以參考 https://nlog-project.org/config/?tab=layout-renderers-->
		<!--https://github.com/NLog/NLog/wiki/Layouts-->
		<target xsi:type="File" name="allfile"
				fileName="c:\temp\AllLog.log"
				layout="${longdate}|${uppercase:${level}}|${logger}|[${threadname:whenEmpty=${threadid}}]|${message} ${exception:format=type,message,method:maxInnerExceptionLevel=5:innerFormat=shortType,message,method}"
				/>

		<!--宣告 Console 目標，將會把 Log 內容寫入到 命令提示字元視窗內 -->
		<target xsi:type="Console" name="lifetimeConsole"
				layout="${level:truncate=4:lowercase=true}: ${logger} [${threadname:whenEmpty=${threadid}}]${newline}      ${message}${exception:format=tostring}|${all-event-properties}|${all-event-properties} "
		        />
	</targets>

	<rules>

		<!--紀錄剩下其他的日誌項目 (視為黑洞)-->
		<logger name="*" minlevel="Trace"
				writeTo="lifetimeConsole,allfile" />
	</rules>
</nlog>
```

## 建立要使用 NLog 套件的程式碼

* 在 [方案總管] 內找到並且開啟 [Program.cs] 檔案這個節點
* 使用底下 C# 程式碼，將原本的程式碼取代掉

```csharp
using Newtonsoft.Json;
using NLog;

namespace csLog04;

public class SomeClass
{
    public int Value { get; set; }
    public string Title { get; set; }
}

public class Program
{
    // 取得當前執行這個方法的類別對應的 Logger 物件
    public static Logger logger =
        LogManager.GetCurrentClassLogger();
    static void Main(string[] args)
    {
        // 請觀察 Console & Log File 所寫入的內容為何?
        Console.WriteLine($"寫入各種不同層級的 日誌項目");

        var someObject = new SomeClass() { Title = "外部物件", Value = 168 };
        Thread.Sleep(1000);
        logger.Trace("我是追蹤:Trace {someObject}", JsonConvert.SerializeObject(someObject));
        logger.Debug("我是偵錯:Debug {someObject}", someObject);
        logger.Info("我是資訊:Info {someObject}", someObject);
        logger.Warn("我是警告:Warn {someObject}", someObject);
        logger.Error("我是錯誤:error {someObject}", someObject);
        logger.Fatal("我是致命錯誤:Fatal {someObject}", someObject);


        Console.WriteLine("Press any key for continuing...");
        Console.ReadKey();
    }
}
```

* 在這個程式碼中，首先建立一個型別為 [Logger] 的靜態變數 [logger]，這個變數是用來記錄 NLog 系統的 Logger 物件
* 在 Main 程式進入點的程式碼中，首先建立一個型別為 [SomeClass] 的類別，這個類別內將會有兩個屬性，分別為整數的 Value 與字串的 Title 
* 接著，將這個類別的物件實體化，並且將這個物件的屬性值設定為 168 與 "外部物件"
* 所使用的程式碼如下

```csharp
var someObject = new SomeClass() { Title = "外部物件", Value = 168 };
```

* 使用 Thread.Sleep(1000) 使當前執行緒休息一秒鐘
* 而在 [Main] 程式進入點方法內，將會依序呼叫 [logger] 這個物件的 [Trace] , [Debug] , [Info] , [Warn] , [Error] , [Fatal] 這六個方法，來寫入不同級別的日誌內容
* 其中在呼叫 [Trace] 方法的時候，使用了這樣的語法

```csharp
logger.Trace("我是追蹤:Trace {someObject}", JsonConvert.SerializeObject(someObject));
```

* 這個敘述將會把一個 JSON 物件的文字描述寫入到層級為 Trace 的日誌系統內
* 底下為輸出結果

```
trac: csLog04.Program [1]
      我是追蹤:Trace "{"Value":168,"Title":"外部物件"}"|someObject={"Value":168,"Title":"外部物件"}
```


* 而對於 `logger.Warn("我是警告:Warn {someObject}", someObject);` 這個敘述，將會輸出下列的結果

```
2023-10-25 15:12:29.7908|WARN|csLog04.Program|[1]|我是警告:Warn csLog04.SomeClass 
```

* 對於複雜的.NET物件或嵌套的物件，最好使用JSON序列化的方法，因為它可以更清晰地顯示物件的層次結構

## 執行程式，觀察結果

請先確認這台電腦上在 C:\ 根目錄下，有一個 temp 資料夾，以便可以讓 NLog 系統寫入相關 Log 資訊。

* 按下 `F5` 鍵，開始執行這個程式

```
寫入各種不同層級的 日誌項目
trac: csLog04.Program [1]
      我是追蹤:Trace "{"Value":168,"Title":"外部物件"}"|someObject={"Value":168,"Title":"外部物件"}|someObject={"Value":168,"Title":"外部物件"}
debu: csLog04.Program [1]
      我是偵錯:Debug csLog04.SomeClass|someObject=csLog04.SomeClass|someObject=csLog04.SomeClass
info: csLog04.Program [1]
      我是資訊:Info csLog04.SomeClass|someObject=csLog04.SomeClass|someObject=csLog04.SomeClass
warn: csLog04.Program [1]
      我是警告:Warn csLog04.SomeClass|someObject=csLog04.SomeClass|someObject=csLog04.SomeClass
erro: csLog04.Program [1]
      我是錯誤:error csLog04.SomeClass|someObject=csLog04.SomeClass|someObject=csLog04.SomeClass
fata: csLog04.Program [1]
      我是致命錯誤:Fatal csLog04.SomeClass|someObject=csLog04.SomeClass|someObject=csLog04.SomeClass
```






