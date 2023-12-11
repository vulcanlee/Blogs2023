# Elasticsearch 系列 - 使用 C# / NEST 來刪除 Elasticsearch 資料庫文件記錄

![](../Images/X2023-9817.png)

在上一篇文章中，寫了關於 [Elasticsearch 系列 - 使用 C# / NEST 來查詢 Elasticsearch 資料庫文件記錄](https://csharpkh.blogspot.com/2023/12/Elasticsearch-NEST-Retrive-Query-Term-Size-ElasticsearchType-IdProperty.html) 文章，這是透過 C# / NEST (Elasticsearch V7) 用戶端類別庫，進行文件查詢的作法。

接下來就是要進行 CRUD 新增、查詢、修改、刪除程式設計作法的 Retrive 修改的程式設計方式說明，在這裡，請將 [Elasticsearch 系列 - 使用 C# / NEST 來新增文件記錄到 Elasticsearch 資料庫](https://csharpkh.blogspot.com/2023/12/Elasticsearch-NEST-Create-Document-Index-Mapping.html) 文章的新增專案再度跑一次，其目的是要確保在後端的 Elasticsearch 資料庫內的 [blogs] 索引內，有一致性的文件存在，這樣，在這篇文章中的操作，才會有可預測的結果。

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
* 找到 [專案名稱] 欄位，輸入 `csElasticsearchNestDelete` 作為專案名稱
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
* 此時，將會看到 [NuGet: csElasticsearchNestDelete] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `Elasticsearch.Net`
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

### 安裝 NEST 套件

這個 NEST 套件是 Elasticsearch 的官方 .NET 高階客戶端。它是一個強大的、易於使用的 .NET 庫，旨在與 Elasticsearch 交互。NEST 提供了一個豐富的 .NET 接口，使得在 .NET 應用中與 Elasticsearch 進行通信變得容易和直觀。

請依照底下說明操作步驟，將這個套件安裝到專案內

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csElasticsearchNestDelete] 視窗
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

namespace csElasticsearchNestDelete;

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

        Stopwatch stopwatch = new Stopwatch();

        #region 取得需要刪除的文件
        await Console.Out.WriteLineAsync(); await Console.Out.WriteLineAsync();
        await Console.Out.WriteLineAsync($"取得需要刪除的文件");
        int needDeleteBlogId = 10006;
        stopwatch.Restart();
        var searchResponse3 = await client
            .SearchAsync<Blog>(s =>
              s.Index(indexName)
               .Size(200)
               .Query(q => q
                      .Term(t => t.BlogId, needDeleteBlogId)
                 )
            );
        stopwatch.Stop();

        Blog needDeleteBlog = null;
        if (searchResponse3.IsValid)
        {
            var documents = searchResponse3.Documents;
            needDeleteBlog = documents.FirstOrDefault();
            if (needDeleteBlog != null)
            {
                Console.WriteLine($"查詢到的文件數量為：{documents.Count} / Title : {needDeleteBlog.Title}， 花費 {stopwatch.ElapsedMilliseconds} ms");
            }
            else
            {
                Console.WriteLine($"沒有搜尋到文件， 花費 {stopwatch.ElapsedMilliseconds} ms");
            }
        }
        #endregion

        #region 進行刪除文件
        if (needDeleteBlog != null)
        {
            await Console.Out.WriteLineAsync(); await Console.Out.WriteLineAsync();
            await Console.Out.WriteLineAsync($"進行刪除文件");
            stopwatch.Restart();
            var searchResponse4 = await client
                .DeleteAsync<Blog>(needDeleteBlog.BlogId,
                  d => d.Index(indexName)
            );
            if (searchResponse4.IsValid)
            {
                Console.WriteLine($"成功刪除文件 Title : {needDeleteBlog.Title}， 花費 {stopwatch.ElapsedMilliseconds} ms");
            }
        }
        #endregion

        #region 再次查詢已經刪除的文件
        await Console.Out.WriteLineAsync(); await Console.Out.WriteLineAsync();
        await Console.Out.WriteLineAsync($"稍後3秒鐘");
        await Task.Delay(3000);
        await Console.Out.WriteLineAsync($"查詢已經刪除的文件");
        stopwatch.Restart();
        var searchResponse5 = await client
            .SearchAsync<Blog>(s =>
              s.Index(indexName)
               .Size(200)
               .Query(q => q
                      .Term(t => t.BlogId, needDeleteBlogId)
                 )
            );
        stopwatch.Stop();

        if (searchResponse5.IsValid)
        {
            var documents = searchResponse5.Documents;
            needDeleteBlog = documents.FirstOrDefault();
            if (needDeleteBlog != null)
            {
                Console.WriteLine($"查詢到的文件數量為：{documents.Count} / Title : {needDeleteBlog.Title}， 花費 {stopwatch.ElapsedMilliseconds} ms");
            }
            else
            {
                Console.WriteLine($"沒有搜尋到文件，該文件已經被刪除了， 花費 {stopwatch.ElapsedMilliseconds} ms");
            }
        }
        #endregion

    }
}
```

對於這個 Index 的模型，將會定義在 [Blog] 這個類別內，由於 NEST 預設將會使用 [Id] 這樣的欄位作為唯一識別碼，不過，在這個 [Blog] 類別內，卻沒有這樣的屬性存在，而是有個 [BlogId] 這樣的屬性，所以，我們需要在這個類別內使用 `ElasticsearchType` 這個屬性來指定這個欄位為唯一識別碼。

在此，將會在該類別之外，使用 `[ElasticsearchType(IdProperty = nameof(BlogId))]` 這樣的屬性來指定這個欄位為唯一識別碼。

對於這個 [Blog] 類別內的屬性，將會定義如下
* [BlogId] : 這個屬性將會作為唯一識別碼
* [Title] : 這個屬性將會作為標題
* [Content] : 這個屬性將會作為內容
* [CreateAt] : 這個屬性將會作為建立時間
* [UpdateAt] : 這個屬性將會作為更新時間

接下來就是這個程式進入點內的程式碼，首先，將會建立一個 [ConnectionSettings] 物件，用來宣告與 Elasticsearch 進行通訊的相關設定，這個物件將會傳入一個 Uri 物件，這個 Uri 物件將會指定 Elasticsearch 的伺服器位址。

由於這台 Elasticsearch 的伺服器，有設定帳號與密碼，所以，這裡將會使用 `BasicAuthentication` 這個方法，來指定帳號與密碼。

一旦得到 [ConnectionSettings] 物件，接下來的工作將會是要建立一個 [ElasticClient] 這個類別，這個類別將會是與 Elasticsearch 進行通訊的主要類別，這個類別的建構式，將會需要傳入一個 [ConnectionSettings] 類別的物件，這個物件將會是用來設定與 Elasticsearch 進行通訊的相關設定。

接下來，因為底下的許多操作都需要指定在 Elasticsearch 內的索引名稱，因此，將會建立一個字串變數 [indexName]，這個變數將會是用來指定要操作的 Index 名稱，這裡將會指定為 `blogs`。這裡有使用 `ToLower()` 方法，將這個字串轉換為小寫字串，這是因為在 Elasticsearch 內，對於所有的 Index 名稱，都是使用小寫字母來表示。

接下來，將會建立一個 [Stopwatch] 類別的物件，這個物件將會用來計算執行時間。

由於這裡要說明如何透過 C# 與 NEST 來做到特定文件更新的需求，所以，在這裡需要先針對要更新的文件把他搜尋出來，透過底下的程式碼，便可以做到找出 [BlogId] 欄位為 10006 的文件。

```csharp
int needDeleteBlogId = 10006;
var searchResponse3 = await client
    .SearchAsync<Blog>(s =>
        s.Index(indexName)
        .Size(200)
        .Query(q => q
                .Term(t => t.BlogId, needDeleteBlogId)
            )
    );
```

然後，根據所找出來的文件，透過底下敘述，將這份文件從 

```csharp
var searchResponse4 = await client
    .DeleteAsync<Blog>(needDeleteBlog.BlogId,
        d => d.Index(indexName)
);
```

從回應物件的 [ApiCall.DebugInformation] 屬性，可以看到這樣的需求是透過了 HTTP DELETE: /blogs/_doc/10006 請求來完成，並且在裡沒有存在任何 Payload

一旦更新文件完成之後，就會得到底下的 JSON 物件

```json
{
   "_index":"blogs",
   "_id":"10006",
   "_version":2,
   "result":"deleted",
   "_shards":{
      "total":2,
      "successful":2,
      "failed":0
   },
   "_seq_no":63,
   "_primary_term":1
}
```

* **"_index"**: `"blogs"` - 這表示刪除操作發生在 `"blogs"` 索引上。

* **"_id"**: `"10006"` - 這是被刪除文檔的唯一標識符。

* **"_version"**: `2` - 這表示文檔在被刪除前的版本號。 在 Elasticsearch 中，每次文檔被更新或修改時，版本號會遞增。

* **"result"**: `"deleted"` - 這表明操作的結果。 在這個例子中，它表明文檔已經成功被刪除。

* **"_shards"**: 
  * `"total": 2` - 表示刪除操作影響了2個分片。
  * `"successful": 2` - 其中2個分片在刪除操作中成功執行。
  * `"failed": 0` - 沒有分片在刪除過程中失敗。

* **"_seq_no"**: `63` - 序列號（Sequence Number），是一個內部用於保持版本控制的數字。 每當文檔被更新或刪除時，序列號會增加。

* **"_primary_term"**: `1` - 主術語号（Primary Term），用於標識分片的當前主副本的術語號。 它與序列號一起幫助維持 Elasticsearch 集群中的數據一致性。

總結來說，這個 JSON 結構表明一個文檔在 `"blogs"` 索引中成功被刪除，並且在刪除過程中沒有遇到任何分片失敗的情況。 此外，它還提供了文檔被刪除時的狀態信息，包括版本號和序列號。

## 執行程式碼

* 按下 `F5` 鍵，開始執行這個程式
* 請觀察 Console 視窗內的內容

```
取得需要刪除的文件
查詢到的文件數量為：1 / Title : Nice to meet your (Bulk) 10006， 花費 744 ms


進行刪除文件
成功刪除文件 Title : Nice to meet your (Bulk) 10006， 花費 57 ms


稍後3秒鐘
查詢已經刪除的文件
沒有搜尋到文件，該文件已經被刪除了， 花費 23 ms
```
