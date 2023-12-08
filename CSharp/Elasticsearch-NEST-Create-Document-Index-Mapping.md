# Elasticsearch 系列 - 使用 C# / NEST 來新增文件記錄到 Elasticsearch 資料庫

![](../Images/X2023-9821.png)

之前寫了一篇關於 [Elasticsearch 系列 - 使用 C# 來新增文件記錄到 Elasticsearch 資料庫](https://csharpkh.blogspot.com/2023/12/Elasticsearch-Create-Document-Index-Mapping.html) 文章，原本是想要撰寫一些關於使用 .NET client for Elasticsearch (v8 .NET Client) 來進行 Elasticsearch 的相關 CRUD 新增、查詢、修改、刪除操作；不過，由於在官方網站與網路資源上，可以查看到的參考文件與說明內容，真的少得可憐，迫於事實情況，我決定還是要採用 Elasticsearch v7 的 .NET Client 則是通稱為 (NEST) client 來寫出相關文件的  CRUD 新增、查詢、修改、刪除程式設計作法。

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
* 找到 [專案名稱] 欄位，輸入 `csElasticsearchNestCreate` 作為專案名稱
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

### 安裝 Elasticsearch.Net 套件

這個 Elasticsearch.Net 套件是 Elasticsearch 的低階 .NET 客戶端，與 NEST 不同，它提供了更多的彈性和直接控制，但也意味著需要手動處理較多的細節。

請依照底下說明操作步驟，將這個套件安裝到專案內

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csElasticsearchNestCreate] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `Elasticsearch.Net`
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

### 安裝 NEST 套件

這個 NEST 套件是 Elasticsearch 的官方 .NET 高階客戶端。它是一個強大的、易於使用的 .NET 庫，旨在與 Elasticsearch 交互。NEST 提供了一個豐富的 .NET 接口，使得在 .NET 應用中與 Elasticsearch 進行通信變得容易和直觀。

請依照底下說明操作步驟，將這個套件安裝到專案內

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csElasticsearchNestCreate] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `NEST`
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

## 建立要使用的程式碼

* 在 [方案總管] 內找到並且開啟 [Program.cs] 檔案這個節點
* 使用底下 C# 程式碼，將原本的程式碼取代掉

```csharp
using Nest;
using System.Diagnostics;

namespace csElasticsearchNestCreate;

[ElasticsearchType(IdProperty = nameof(BlogId))]
public class Blog
{
    public int BlogId { get; set; }
    public string Title { get; set; } = string.Empty;
    public string Content { get; set; } = string.Empty;
    public DateTime CreateAt { get; set; } = DateTime.Now;
    public DateTime UpdateAt { get; set; } = DateTime.Now;
}

internal class Program
{
    static async Task Main(string[] args)
    {
        var settings = new ConnectionSettings(new Uri("http://10.1.1.231:9200/"))
            .DisableDirectStreaming()
            .BasicAuthentication("elastic", "elastic");

        var client = new ElasticClient(settings);

        string indexName = "blogs".ToLower();

        // 嘗試讓 client 物件與後端 Elasticsearch 來通訊，避免第一次的延遲
        await client.Indices.DeleteAsync(indexName);

        // 建立 index
        await client.IndexAsync<Blog>(new Blog()
        {
            BlogId = 999,
            Title = $"Nice to meet your 999",
            Content = $"Hello Elasticsearch 999",
            CreateAt = DateTime.Now.AddDays(999),
            UpdateAt = DateTime.Now.AddDays(999),
        }, idx=>idx.Index(indexName));

        Stopwatch stopwatch = new Stopwatch();

        #region 每次新增一筆文件，共 100 次
        stopwatch.Restart();
        for (int i = 0; i < 100; i++)
        {
            Blog blog = new Blog()
            {
                BlogId = i,
                Title = $"Nice to meet your {i}",
                Content = $"Hello Elasticsearch {i}",
                CreateAt = DateTime.Now.AddDays(i),
                UpdateAt = DateTime.Now.AddDays(i),
            };

            var response = await client.IndexAsync(blog,
                idx => idx.Index(indexName));

            if (response.IsValid)
            {
                //Console.WriteLine($"Index document with ID {response.Id} succeeded.");
            }
            else
            {
                Console.WriteLine($"Error Message : {response.DebugInformation}");
            }
        }

        stopwatch.Stop();
        // 顯示需要耗費時間
        Console.WriteLine($"新增 100 次文件需要 {stopwatch.ElapsedMilliseconds} ms");
        #endregion

        #region 一次新增 100 筆文件
        stopwatch.Restart();
        Console.WriteLine();
        List<Blog> list = new List<Blog>();
        for (int i = 0; i < 100; i++)
        {
            int foo = i + 10000;
            Blog blog = new Blog()
            {
                BlogId = foo,
                Title = $"Nice to meet your (Bulk) {foo}",
                Content = $"Hello Elasticsearch (Bulk) {foo}",
                CreateAt = DateTime.Now.AddDays(i),
                UpdateAt = DateTime.Now.AddDays(i),
            };
            list.Add(blog);
        }

        var response2 = await client
            .BulkAsync(b => b.Index(indexName).IndexMany(list));

        if (response2.IsValid)
        {
            //Console.WriteLine($"Index document with ID {response.Id} succeeded.");
        }
        else
        {
            //Console.WriteLine($"Error Message : {response2.DebugInformation}");
        }

        stopwatch.Stop();
        // 顯示需要耗費時間
        Console.WriteLine($"新增 100 次文件需要 {stopwatch.ElapsedMilliseconds} ms");
        #endregion
    }
}
```

## 執行程式碼

* 按下 `F5` 鍵，開始執行這個程式
* 請觀察 Console 視窗內的內容
* 對於 [每次新增一筆文件，共 100 次] 這樣動作，將會耗時 3284 ms
* 對於 [一次新增 100 筆文件] 這樣動作，將會耗時 120 ms

```
新增 100 次文件需要 3284 ms

新增 100 次文件需要 120 ms
```
