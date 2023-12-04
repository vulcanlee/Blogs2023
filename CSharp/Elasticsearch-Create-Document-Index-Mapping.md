# Elasticsearch 系列 - 使用 C# 來新增文件記錄到 Elasticsearch 資料庫

![](../Images/X2023-9844.png)

這一系列的文章將會是關於使用 .NET client for Elasticsearch 來進行 Elasticsearch 的相關 CRUD 新增、查詢、修改、刪除操作，這篇文章將會針對 v8 .NET Client 來進行設計，通稱為 Elasticsearch .NET Client，而 Elasticsearch v7 的 .NET Client 則是通稱為 (NEST) client。

對於 v8 版本的 Elasticsearch .NET Client，其主要的命名空間為 `Elastic.Clients.Elasticsearch`，而 v7 版本的 NEST 則是命名空間為 `Elasticsearch.Net`。

這兩者間的差異，主要在於 v8 版本的 Elasticsearch .NET Client，其主要是使用 Elasticsearch 官方所提供的 [Elasticsearch.Net] 這個套件，而 v7 版本的 NEST 則是使用 [NEST] 這個套件，而這個套件是由 Elasticsearch 官方所提供的，所以，這兩者間的差異，主要是在於命名空間的差異。

在這裡將會說明如何使用 [Elasticsearch.Net] 這個套件來存取後端的 Elasticsearch 服務，並且與 Elasticsearch 進行通訊，進而進行資料的新增操作。

以下是該檔案的主要功能：

1.	建立與 Elasticsearch 的連線：透過 ElasticsearchClientSettings 和 ElasticsearchClient 來設定並建立與 Elasticsearch 的連線。
2.	刪除索引：使用 client.Indices.DeleteAsync(indexName) 來刪除指定的索引。
3.	新增資料：這個檔案中有兩種方式來新增資料到 Elasticsearch。
•	第一種方式是透過迴圈，每次新增一筆資料，共新增 100 筆。每次新增一筆資料時，都會使用 client.IndexAsync(blog, indexName) 來將資料新增到 Elasticsearch。
•	第二種方式是一次性新增 100 筆資料。這是透過 BulkRequest 和 client.BulkAsync(bulkRequest) 來實現的。
4.	計時：使用 Stopwatch 來計算新增資料所需的時間。
這個檔案的主要目的是為了比較兩種新增資料的方式（每次新增一筆和一次性新增 100 筆）所需的時間。

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
* 找到 [專案名稱] 欄位，輸入 `csElasticsearchCreate` 作為專案名稱
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

### 安裝 Elastic.Clients.Elasticsearch 套件

對於這個 NuGet 套件 [Elastic.Clients.Elasticsearch] 其可以提供一個 .NET 平台的 Elasticsearch 客戶端庫，它允許您從 .NET 應用程序與 Elasticsearch 集群進行通信和互動。該庫為 Elasticsearch 提供了一個強類型的、易於使用的 API，可以用於執行各種操作，如索引管理、文檔檢索、搜索、數據分析等。

請依照底下說明操作步驟，將這個套件安裝到專案內

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csElasticsearchCreate] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `Elastic.Clients.Elasticsearch`
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

## 建立要使用的程式碼

* 在 [方案總管] 內找到並且開啟 [Program.cs] 檔案這個節點
* 使用底下 C# 程式碼，將原本的程式碼取代掉

```csharp
using Elastic.Clients.Elasticsearch;
using Elastic.Clients.Elasticsearch.Core.Bulk;
using Elastic.Transport;
using System.Diagnostics;

namespace csElasticsearchCreate
{
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
            var settings = new ElasticsearchClientSettings(new Uri("http://10.1.1.231:9200/"))
                .Authentication(new BasicAuthentication("elastic", "elastic"));
            var client = new ElasticsearchClient(settings);

            string indexName = "blogs".ToLower();

            // 嘗試讓 client 物件與後端 Elasticsearch 來通訊，避免第一次的延遲
            await client.Indices.DeleteAsync(indexName);

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

                var response = await client.IndexAsync(blog, indexName);

                if (response.IsValidResponse)
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
                Blog blog = new Blog()
                {
                    BlogId = i,
                    Title = $"Nice to meet your (Bulk) {i}",
                    Content = $"Hello Elasticsearch (Bulk) {i}",
                    CreateAt = DateTime.Now.AddDays(i),
                    UpdateAt = DateTime.Now.AddDays(i),
                };
                list.Add(blog);
            }

            var operations = new List<IBulkOperation>();

            foreach (var blog in list)
            {
                var indexOperation = new BulkIndexOperation<Blog>(blog) { Index = indexName };
                operations.Add(indexOperation);
            }

            var bulkRequest = new BulkRequest
            {
                Operations = operations
            };

            var response2 =  await client.BulkAsync(bulkRequest);

            if (response2.IsValidResponse)
            {
                //Console.WriteLine($"Index document with ID {response.Id} succeeded.");
            }
            else
            {
                Console.WriteLine($"Error Message : {response2.DebugInformation}");
            }

            stopwatch.Stop();
            // 顯示需要耗費時間
            Console.WriteLine($"新增 100 次文件需要 {stopwatch.ElapsedMilliseconds} ms");
            #endregion
        }
    }
}
```

* 在這個程式碼中，有一個 Blog 類別，用來代表要新增到 Elasticsearch 的文件內容，Blog 類別內有幾個屬性，分別是 `BlogId`、`Title`、`Content`、`CreateAt`、`UpdateAt`，這些屬性都是用來代表文件的內容，而這個類別的內容，可以依照需求進行調整。
* 接下來將會建立一個 [ElasticsearchClientSettings] 物件，這個物件是用來設定 Elasticsearch 的連線資訊，這裡設定的是 Elasticsearch 的連線位址，以及使用者名稱和密碼，這裡的使用者名稱和密碼，是在 Elasticsearch 的安裝時，所設定的使用者名稱和密碼。
* 在 [ElasticsearchClientSettings] 建構式內，將會把後端的 Elasticsearch 連線位址傳入到建構式內，接著使用流暢的 API 語法，來設定使用者名稱和密碼，這裡的使用者名稱和密碼，是在 Elasticsearch 的安裝時，所設定的使用者名稱和密碼。
* 接下來將會建立一個 [ElasticsearchClient] 物件，這個物件是用來與 Elasticsearch 進行通訊，這裡的 ElasticsearchClient 物件，是使用剛剛建立的 ElasticsearchClientSettings 物件來建立的。
* 有了 [ElasticsearchClient] 物件之後，就可以使用這個物件來與 Elasticsearch 進行通訊，這裡的通訊方式，是使用 Elasticsearch 的 RESTful API 來進行通訊。
* 在這個範例中，將會進行量測呼叫100次 API，而每一次文件新增，或者一次新增 100 筆文件，所需要的時間，這裡使用了 Stopwatch 來進行計時，並且在每次呼叫新增文件的時候，都會將 Stopwatch 重新啟動，這樣就可以計算出每次呼叫新增文件所需要的時間。此時，為了確保量測時間準確，所以在這裡會先把存放文件的索引刪除掉，這樣就可以避免第一次呼叫新增文件時，因為索引不存在，所以會花費比較多的時間。
* 要刪除 Elasticsearch 內的所有，將會呼叫 `await client.Indices.DeleteAsync(indexName);` 這個敘述，其中， indexName 是索引的名稱，這裡的索引名稱是 `blogs`。
* 接下來就是使用了 `client.IndexAsync` 這個方法來進行文件的新增，這個方法的第一個參數是要新增的文件內容，這裡是使用了 Blog 類別來代表文件內容，第二個參數是索引的名稱，這裡的索引名稱是 `blogs`。在這裡呼叫這個 API 的目的是因為剛剛已經把 Elasticsearch 內的索引刪除掉了，所以這裡需要重新建立索引，這樣才能夠新增文件。所以，這裡透過新建一筆文件，來讓不存在的索引自動生成出來。
* 接下來就是要做到每次新增一筆文件，共 100 次，在這段程式碼中，將會有個迴圈，在迴圈內，將會建立一個 [Blog] 型別的物件，接著呼叫 `client.IndexAsync` 這個方法，做到新增 100 次文件的目的
* 當呼叫完成 `client.IndexAsync` 這個方法之後，可以得到一個 [IndexResponse] 物件，這個物件可以用來判斷是否呼叫 API 成功，如果成功，則可以透過這個物件來取得新增文件的 ID，這個 ID 是由 Elasticsearch 自動產生的。若發生了錯誤，則可以透過這個物件來取得錯誤訊息，這裡將會透過 DebugInformation 這個屬性來取得錯誤訊息。
* 對於 [一次新增 100 筆文件] 這樣的需求，將會先建立 `List<Blog> list = new List<Blog>()` 這個集合物件，接著透過迴圈來產生 100 個 Blog 物件到 list 集合物件內
* 接下來將會建立一個 `List<IBulkOperation> operations = new List<IBulkOperation>()` 這個集合物件，這個集合物件是用來存放要新增的文件內容，這裡的 `IBulkOperation` 是一個介面，這個介面是用來定義要新增文件的操作，這裡的 `IBulkOperation` 介面有幾個實作類別，分別是 `BulkIndexOperation`、`BulkCreateOperation`、`BulkDeleteOperation`、`BulkUpdateOperation`，這些實作類別都是用來定義要新增文件的操作，這裡將會使用 `BulkIndexOperation` 這個實作類別來定義要新增文件的操作。
* 現在可以呼叫 `await client.BulkAsync(bulkRequest)` 這個方法來進行一次性新增 100 筆文件的操作，這個方法的參數是一個 `BulkRequest` 物件，這個物件是用來定義要新增文件的操作，這裡的 `BulkRequest` 物件內有一個 `Operations` 屬性，這個屬性是用來定義要新增文件的操作，這裡將會把剛剛定義的 `List<IBulkOperation> operations` 集合物件，指定給 `BulkRequest` 物件的 `Operations` 屬性，這樣就可以定義要新增文件的操作。


## 執行程式碼

* 按下 `F5` 鍵，開始執行這個程式
* 請觀察 Console 視窗內的內容
* 對於 [每次新增一筆文件，共 100 次] 這樣動作，將會耗時 3475 ms
* 對於 [一次新增 100 筆文件] 這樣動作，將會耗時 137 ms

```
新增 100 次文件需要 3475 ms

新增 100 次文件需要 137 ms
```
