# Elasticsearch 系列 - 使用 C# / NEST 來查詢  Elasticsearch 資料庫文件記錄

![](../Images/X2023-9819.png)

在上一篇文章中，寫了關於 [Elasticsearch 系列 - 使用 C# / NEST 來新增文件記錄到 Elasticsearch 資料庫](https://csharpkh.blogspot.com/2023/12/Elasticsearch-NEST-Create-Document-Index-Mapping.html) 文章，這是透過 C# / NEST (Elasticsearch V7) 用戶端類別庫，進行新增文件到 Elasticsearch 所有內的作法。

接下來就是要進行 CRUD 新增、查詢、修改、刪除程式設計作法的 Retrive 查詢的程式設計方式說明，在這裡，請將 [Elasticsearch 系列 - 使用 C# / NEST 來新增文件記錄到 Elasticsearch 資料庫](https://csharpkh.blogspot.com/2023/12/Elasticsearch-NEST-Create-Document-Index-Mapping.html) 文章的新增專案再度跑一次，其目的是要確保在後端的 Elasticsearch 資料庫內的 [blogs] 索引內，有一致性的文件存在，這樣，在這篇文章中的操作，才會有可預測的結果。

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
* 找到 [專案名稱] 欄位，輸入 `csElasticsearchNestRetrive` 作為專案名稱
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
* 此時，將會看到 [NuGet: csElasticsearchNestRetrive] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `Elasticsearch.Net`
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

### 安裝 NEST 套件

這個 NEST 套件是 Elasticsearch 的官方 .NET 高階客戶端。它是一個強大的、易於使用的 .NET 庫，旨在與 Elasticsearch 交互。NEST 提供了一個豐富的 .NET 接口，使得在 .NET 應用中與 Elasticsearch 進行通信變得容易和直觀。

請依照底下說明操作步驟，將這個套件安裝到專案內

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csElasticsearchNestRetrive] 視窗
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

namespace csElasticsearchNestRetrive;

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

        #region 取得所有文件 (沒有指定頁面大小)
        await Console.Out.WriteLineAsync($"取得所有文件 (沒有指定頁面大小)");
        stopwatch.Restart();
        var searchResponse1 = await client
            .SearchAsync<Blog>(s =>
              s.Index(indexName)
            );
        stopwatch.Stop();
        if (searchResponse1.IsValid)
        {
            var document1 = searchResponse1.Documents;
            foreach (var item in document1)
            {
                Console.WriteLine($"    BlogId:{item.BlogId} , Title:{item.Title}");
            }
            Console.WriteLine($"查詢到的文件數量為：{document1.Count} ， 花費 {stopwatch.ElapsedMilliseconds} ms");
        }
        #endregion

        #region 取得所有文件 (每頁20筆文件，指定第三頁)
        await Console.Out.WriteLineAsync(); await Console.Out.WriteLineAsync();
        await Console.Out.WriteLineAsync($"取得所有文件 (沒有指定頁面大小)");
        stopwatch.Restart();
        var searchResponse2 = await client
            .SearchAsync<Blog>(s =>
              s.Index(indexName)
               .From(20 * 3)
               .Size(20)
            );
        stopwatch.Stop();
        if (searchResponse2.IsValid)
        {
            var documents = searchResponse2.Documents;
            foreach (var item in documents)
            {
                Console.WriteLine($"    BlogId:{item.BlogId} , Title:{item.Title}");
            }
            Console.WriteLine($"查詢到的文件數量為：{documents.Count} ， 花費 {stopwatch.ElapsedMilliseconds} ms");
        }
        #endregion

        #region 取得 Title 有 3 文字之所有文件
        await Console.Out.WriteLineAsync(); await Console.Out.WriteLineAsync();
        await Console.Out.WriteLineAsync($"取得 Title 有 1003 文字之所有文件");
        stopwatch.Restart();
        var searchResponse3 = await client
            .SearchAsync<Blog>(s =>
              s.Index(indexName)
               .Size(200)
               .Query(q => q
                      .Match(m => m
                            .Field(f => f.Title)
                            .Query("33")
                      )
                 )
            );
        stopwatch.Stop();
        if (searchResponse3.IsValid)
        {
            var documents = searchResponse3.Documents;
            foreach (var item in documents)
            {
                Console.WriteLine($"    BlogId:{item.BlogId} , Title:{item.Title}");
            }
            Console.WriteLine($"查詢到的文件數量為：{documents.Count} ， 花費 {stopwatch.ElapsedMilliseconds} ms");
        }
        #endregion

        #region 取得 Title 有包含任何 3 文字之所有文件
        await Console.Out.WriteLineAsync(); await Console.Out.WriteLineAsync();
        await Console.Out.WriteLineAsync($"取得 Title 有包含任何 3 文字之所有文件");
        stopwatch.Restart();
        var searchResponse4 = await client
            .SearchAsync<Blog>(s =>
              s.Index(indexName)
               .Size(200)
               .Query(q => q
                        .Wildcard(m => m.Value("*33*").Field(f => f.Title))
                     )
            );
        stopwatch.Stop();
        if (searchResponse4.IsValid)
        {
            var documents = searchResponse4.Documents;
            foreach (var item in documents)
            {
                Console.WriteLine($"    BlogId:{item.BlogId} , Title:{item.Title}");
            }
            Console.WriteLine($"查詢到的文件數量為：{documents.Count} ， 花費 {stopwatch.ElapsedMilliseconds} ms");
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

### 取得所有文件 (沒有指定頁面大小)

想要對 Elasticsearch 資料庫進行搜尋，可以透過 [ElasticClient] 物件的 [SearchAsync] 方法來做到這樣需求，在這裡將會使用 C# 強型別的方式來進行，使用到 ` var searchResponse1 = await client.SearchAsync<Blog>(s => s.Index(indexName));` 這樣的敘述，對 Elasticsearch Server 發出搜尋請求指令。

當上面的敘述執行完成之後，將會得到一個 [ISearchResponse<Blog>] 這應型別的物件，該物件的 [IsValid] 可以判斷此次搜尋的呼叫是否正常無誤；另外，透過 [DebugInformation] 這個屬性，可以觀察到此次對 Elasticsearch Server 的呼叫，的最底階實際語法。

對於 [SearchAsync] 方法，將會使用 HTTP POST 方法，使用 [/blogs/_search?typed_keys=true] 這個 URI ，請求服務，而在 [Body] 部分，則會有底下的請求 Payload

```json
{}
```

這是一個 JSON 空物件，表示要取得所有該指定索引內的文件，所以，當完成這個方法呼叫之後，在底層中，將會得到底下的 JSON 物件

```json
{
   "took":3,
   "timed_out":false,
   "_shards":{
      "total":3,
      "successful":3,
      "skipped":0,
      "failed":0
   },
   "hits":{
      "total":{
         "value":201,
         "relation":"eq"
      },
      "max_score":1.0,
      "hits":[
         {
            "_index":"blogs",
            "_id":"5",
            "_score":1.0,
            "_source":{
               "blogId":5,
               "title":"Nice to meet your 5",
               "content":"Hello Elasticsearch 5",
               "createAt":"2023-12-16T11:40:11.6723768+08:00",
               "updateAt":"2023-12-16T11:40:11.6723778+08:00"
            }
         },
         {
            "_index":"blogs",
            "_id":"7",
            "_score":1.0,
            "_source":{
               "blogId":7,
               "title":"Nice to meet your 7",
               "content":"Hello Elasticsearch 7",
               "createAt":"2023-12-18T11:40:11.7417595+08:00",
               "updateAt":"2023-12-18T11:40:11.7417607+08:00"
            }
         },

        ...

        {
            "_index":"blogs",
            "_id":"44",
            "_score":1.0,
            "_source":{
               "blogId":44,
               "title":"Nice to meet your 44",
               "content":"Hello Elasticsearch 44",
               "createAt":"2024-01-24T11:40:12.9413472+08:00",
               "updateAt":"2024-01-24T11:40:12.9413475+08:00"
            }
         }
      ]
   }
}
```

這是一個 Elasticsearch 查詢指令的結果。以下是這個 JSON 格式化資料的概要：

* 執行時間 : 查詢耗時3毫秒 `"took":3`。
* 超時 : 沒有超時 `"timed_out":false`。
* 分片信息 : `"_shards"`
  * 總共3個分片 `"total":3`。
  * 3個分片成功執行 `"successful":3`。
  * 沒有跳過的分片 `"skipped":0`。
  * 沒有失敗的分片 `"failed":0`。
* 擊中信息 : `"hits"`
* 總共找到201個匹配的文件 `"total.value" : 201`。
* 最高分數為1.0 `"max_score" = 1.0`。

返回的是「blogs」索引中的一些文件。每個文件包括以下信息：

* _index: 文件所在的索引。
* _id: 文件的ID。
* _score: 文件的匹配分數。
* _source: 文件的原始內容，包括 blogId（博客ID）、title（標題）、content（內容）、createAt（創建時間）和 updateAt（更新時間）。

例如，第一個文件的 JSON 是：

```json
{
    "_index":"blogs",
    "_id":"5",
    "_score":1.0,
    "_source":{
        "blogId":5,
        "title":"Nice to meet your 5",
        "content":"Hello Elasticsearch 5",
        "createAt":"2023-12-16T11:40:11.6723768+08:00",
        "updateAt":"2023-12-16T11:40:11.6723778+08:00"
    }
}
```

一旦執行結果得到成功的狀態，程式將會 [searchResponse1.Documents] 這個屬性得到這次指定查詢結果的所有文件，不過，由於在這裡沒有指定查詢出多少文件，因此，將會採用預設值 10。

底下將會是這次查詢後的內容

```
    BlogId:5 , Title:Nice to meet your 5
    BlogId:7 , Title:Nice to meet your 7
    BlogId:13 , Title:Nice to meet your 13
    BlogId:22 , Title:Nice to meet your 22
    BlogId:24 , Title:Nice to meet your 24
    BlogId:26 , Title:Nice to meet your 26
    BlogId:41 , Title:Nice to meet your 41
    BlogId:42 , Title:Nice to meet your 42
    BlogId:43 , Title:Nice to meet your 43
    BlogId:44 , Title:Nice to meet your 44
```

### 取得所有文件 (每頁20筆文件，指定第三頁)

在這裡將會說明如何對查詢結果做到分頁查詢與指定每頁的文件數量的程式設計作法

這裡將會使用到底下 C# 敘述

```csharp
var searchResponse2 = await client
    .SearchAsync<Blog>(s =>
        s.Index(indexName)
        .From(20 * 3)
        .Size(20)
    );
```

觀察低層 API 的呼叫，同樣是使用 HTTP POST: /blogs/_search?typed_keys=true 的方法，不過，對於 Body 內的 Payload 文字部分，將會使用到這個 JSON 物件

```json
{
   "from":60,
   "size":20
}
```

這裡同樣的使用到 [SearchAsync] 這個方法，不過，這裡使用到 [From] 方法來指定跳頁與使用 [Size] 來指定每頁文件數量大小

底下將會是接收到的低層 API 回傳的 JSON 物件

```json
{
   "took":3,
   "timed_out":false,
   "_shards":{
      "total":3,
      "successful":3,
      "skipped":0,
      "failed":0
   },
   "hits":{
      "total":{
         "value":201,
         "relation":"eq"
      },
      "max_score":1.0,
      "hits":[
         {
            "_index":"blogs",
            "_id":"10091",
            "_score":1.0,
            "_source":{
               "blogId":10091,
               "title":"Nice to meet your (Bulk) 10091",
               "content":"Hello Elasticsearch (Bulk) 10091",
               "createAt":"2024-03-11T11:40:14.4140614+08:00",
               "updateAt":"2024-03-11T11:40:14.4140614+08:00"
            }
         }, ...
      ]
   }
}
```

此時，程式會將取得的 20 筆文件輸出在螢幕上

```
    BlogId:10091 , Title:Nice to meet your (Bulk) 10091
    BlogId:10092 , Title:Nice to meet your (Bulk) 10092
    BlogId:10096 , Title:Nice to meet your (Bulk) 10096
    BlogId:0 , Title:Nice to meet your 0
    BlogId:2 , Title:Nice to meet your 2
    BlogId:3 , Title:Nice to meet your 3
    BlogId:4 , Title:Nice to meet your 4
    BlogId:10 , Title:Nice to meet your 10
    BlogId:12 , Title:Nice to meet your 12
    BlogId:14 , Title:Nice to meet your 14
    BlogId:15 , Title:Nice to meet your 15
    BlogId:19 , Title:Nice to meet your 19
    BlogId:20 , Title:Nice to meet your 20
    BlogId:21 , Title:Nice to meet your 21
    BlogId:23 , Title:Nice to meet your 23
    BlogId:28 , Title:Nice to meet your 28
    BlogId:30 , Title:Nice to meet your 30
    BlogId:34 , Title:Nice to meet your 34
    BlogId:39 , Title:Nice to meet your 39
    BlogId:40 , Title:Nice to meet your 40
```

### 取得 Title 有 3 文字之所有文件

現在，要來指定搜尋條件，這裡將會使用取得 Title 有 33 文字之所有文件這樣需求

為了要能夠做到這樣需求，使用底下的 C# 程式碼

```csharp
var searchResponse3 = await client
    .SearchAsync<Blog>(s =>
        s.Index(indexName)
        .Size(200)
        .Query(q => q
                .Match(m => m
                    .Field(f => f.Title)
                    .Query("33")
                )
            )
    );
```

`var searchResponse3 = await client`： 這行代碼初始化一個非同步操作，用於執行搜索查詢。 'searchResponse3' 是存儲查詢結果的變數。`. SearchAsync<Blog>（s =>： 這部分代碼調用 'SearchAsync' 方法來執行一個非同步搜索操作。 '<Blog>' 指定了搜索的類型，這意味著查詢將返回 'Blog' 類型的結果。

`s.Index（indexName）`： 這指定了要搜索的 Elasticsearch 索引的名稱。`. Size（200）`這設置了返回的文件數量上限為200。`.Query（q => q`： 這開始構建查詢的條件。`.Match（m => m`： 這指定了一個匹配查詢，它是 Elasticsearch 中最基本的全文搜索查詢之一。`.Field（f => f.Title）`： 這指定了要在 'Blog' 類型中搜索的字段，這裡是 [Title]。`.Query（"33"）`： 這是匹配查詢的實際文本，表示搜索 'Title' 字段中包含“33”的文件。

所以，上述代碼的功能是在指定的索引中搜索 [Title] 欄位包含 [33] 的 [Blog] 類型文件，並且最多返回200個結果。 

在低層 API 呼叫中，使用到 HTTP POST: /blogs/_search?typed_keys=true 方法

使用請求 Payload 為

```json
{
   "query":{
      "match":{
         "title":{
            "query":"33"
         }
      }
   },
   "size":200
}
```

而得到的執行結果 JSON 內容，則會與上一個需求得到的 JSON 物件類似

底下將會是執行完成之後，所顯示查詢到的文件資訊

```
    BlogId:33 , Title:Nice to meet your 33
```

### 取得 Title 有包含任何 33 文字之所有文件

接下來的設計，是要找出文件中的 Title 欄位內，只要有出現過 `33` 文字的文件，就要把他找出來，這裡將會使用到底下的 C# 程式碼

```csharp
var searchResponse4 = await client
    .SearchAsync<Blog>(s =>
        s.Index(indexName)
        .Size(200)
        .Query(q => q
                .Wildcard(m => m.Value("*33*").Field(f => f.Title))
                )
    );
```

在低層 API 呼叫中，使用到 HTTP POST: /blogs/_search?typed_keys=true 方法

使用請求 Payload 為

```json
{
   "query":{
      "wildcard":{
         "title":{
            "value":"*33*"
         }
      }
   },
   "size":200
}
```

而得到的執行結果 JSON 內容，則會與上一個需求得到的 JSON 物件類似

底下將會是執行完成之後，所顯示查詢到的文件資訊

```
    BlogId:33 , Title:Nice to meet your 33
    BlogId:10033 , Title:Nice to meet your (Bulk) 10033
```

## 執行程式碼

* 按下 `F5` 鍵，開始執行這個程式
* 請觀察 Console 視窗內的內容

```
取得所有文件 (沒有指定頁面大小)
    BlogId:5 , Title:Nice to meet your 5
    BlogId:7 , Title:Nice to meet your 7
    BlogId:13 , Title:Nice to meet your 13
    BlogId:22 , Title:Nice to meet your 22
    BlogId:24 , Title:Nice to meet your 24
    BlogId:26 , Title:Nice to meet your 26
    BlogId:41 , Title:Nice to meet your 41
    BlogId:42 , Title:Nice to meet your 42
    BlogId:43 , Title:Nice to meet your 43
    BlogId:44 , Title:Nice to meet your 44
查詢到的文件數量為：10 ， 花費 549 ms


取得所有文件 (沒有指定頁面大小)
    BlogId:10091 , Title:Nice to meet your (Bulk) 10091
    BlogId:10092 , Title:Nice to meet your (Bulk) 10092
    BlogId:10096 , Title:Nice to meet your (Bulk) 10096
    BlogId:0 , Title:Nice to meet your 0
    BlogId:2 , Title:Nice to meet your 2
    BlogId:3 , Title:Nice to meet your 3
    BlogId:4 , Title:Nice to meet your 4
    BlogId:10 , Title:Nice to meet your 10
    BlogId:12 , Title:Nice to meet your 12
    BlogId:14 , Title:Nice to meet your 14
    BlogId:15 , Title:Nice to meet your 15
    BlogId:19 , Title:Nice to meet your 19
    BlogId:20 , Title:Nice to meet your 20
    BlogId:21 , Title:Nice to meet your 21
    BlogId:23 , Title:Nice to meet your 23
    BlogId:28 , Title:Nice to meet your 28
    BlogId:30 , Title:Nice to meet your 30
    BlogId:34 , Title:Nice to meet your 34
    BlogId:39 , Title:Nice to meet your 39
    BlogId:40 , Title:Nice to meet your 40
查詢到的文件數量為：20 ， 花費 43 ms


取得 Title 有 33 文字之所有文件
    BlogId:33 , Title:Nice to meet your 33
查詢到的文件數量為：1 ， 花費 53 ms


取得 Title 有包含任何 33 文字之所有文件
    BlogId:33 , Title:Nice to meet your 33
    BlogId:10033 , Title:Nice to meet your (Bulk) 10033
查詢到的文件數量為：2 ， 花費 36 ms
```
