# Elasticsearch 系列 - 使用 C# 來新增文件記錄到 Elasticsearch 資料庫

![](../Images/X2023-9844.png)

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




