# MongoDB 系列 - 使用 C# 來新增文件記錄到 Docker 容器內的 MongoDB 資料庫

![](../Images/X2023-9859.png)

經過前面兩篇文章 [MongoDB 系列 - 在 Windows 作業系統上安裝 Docker](https://csharpkh.blogspot.com/2023/11/MongoDb-Installation-Windows-Ducker-Desktop.html) & [MongoDB 系列 - 使用 Docker Hub 拉取 MongoDB Image 並且啟動該容器](https://csharpkh.blogspot.com/2023/11/MongoDB-Pull-Image-From-Docker-Hub-Start-Container.html) 說明如何架設 Docker 系統，並且拉取一個 MongoDB Image 下來，並且建立這樣的容器，使得可以在本機電腦上來跑 MongoDB 資料庫。

接下來，將會說明如何使用 C# 來新增文件記錄到 MongoDB 資料庫。

不過，這裡將會嘗試使用 [MongoDB Atlas](https://www.mongodb.com/zh-cn/atlas) 這個服務來建立一個 MongoDB 資料庫，並且使用 C# 來新增文件記錄到這個 MongoDB 資料庫。也就是說，雖然這個 MongoDB 資料庫是建立在雲端上，不過，我們仍然可以使用 C# 來新增文件記錄到這個 MongoDB 資料庫，重要的是，不論是觀念或者程式碼的使用，都是相同的。

在這篇文章中，將會建立一個迴圈，試圖建立 100 筆文件到 MongoDB 內，每次迴圈都會新增一筆文件到 MongoDB 資料庫內，另外一種作法則是，在迴圈內，先建立 100 筆文件的物件，當迴圈結束後，再一次新增 100 筆文件到 MongoDB 資料庫內。在此，可以來觀察這兩種作法在效能上的差異。

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
* 找到 [專案名稱] 欄位，輸入 `csMongoDBCreate` 作為專案名稱
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

### 安裝 MongoDB.Driver 套件

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csLog02] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `MongoDB.Driver`
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

## 建立要使用的程式碼

* 在 [方案總管] 內找到並且開啟 [Program.cs] 檔案這個節點
* 使用底下 C# 程式碼，將原本的程式碼取代掉

```csharp
using MongoDB.Driver;
using System.Diagnostics;

namespace csMongoDBCreate;

// MongoDB 的 Blog 文件資料結構
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
    public static void Main(string[] args)
    {
        // 使用 Environment 來抓取環境變數設定的 帳號與密碼
        string MongoDBAccount = Environment.GetEnvironmentVariable("MongoDBAccount");
        string MongoDBPassword = Environment.GetEnvironmentVariable("MongoDBPassword");

        // 使用 MongoDB Atlas 來連線
        var mongoUri = $"mongodb+srv://{MongoDBAccount}:{MongoDBPassword}@vulcanmongo.hptf95d.mongodb.net/?retryWrites=true&w=majority";

        // 宣告一個 MongoDB Client 變數
        IMongoClient client;

        // 宣告一個 MongoDB Database 變數
        IMongoDatabase database;

        // 宣告一個 MongoDB Collection 變數
        IMongoCollection<Blog> collection;

        try
        {
            // 連線到 MongoDB Atlas
            client = new MongoClient(mongoUri);
        }
        catch (Exception e)
        {
            Console.WriteLine("{e.Message}");
            Console.WriteLine(e);
            Console.WriteLine();
            return;
        }

        // 宣告一個 Database Name 與 Collection Name
        var dbName = "MyCrud";
        var collectionName = "Blog";

        // 取得 MongoDB Collection
        collection = client.GetDatabase(dbName)
           .GetCollection<Blog>(collectionName);

        Stopwatch stopwatch = new Stopwatch();
        stopwatch.Restart();

        #region 每次新增一筆文件
        for (int i = 0; i < 100; i++)
        {
            // 宣告一個 Blog 物件
            Blog blog = new Blog
            {
                BlogId = i,
                Title = $"Hello MongoDB{i}",
                Content = $"Hello MongoDB{i}",
                CreateAt = DateTime.Now.AddDays(i),
                UpdateAt = DateTime.Now.AddDays(i)
            };

            // 新增一筆 Blog 資料
            collection.InsertOne(blog);
        }
        #endregion

        stopwatch.Stop();
        // 顯示需要耗費時間
        Console.WriteLine($"新增 100 次文件需要 {stopwatch.ElapsedMilliseconds} ms");

        #region 一次新增100筆文件
        List<Blog> blogs = new List<Blog>();
        stopwatch.Restart();
        for (int i = 0; i < 100; i++)
        {
            // 宣告一個 Blog 物件
            Blog blog = new Blog
            {
                BlogId = i,
                Title = $"Hello MongoDB{i}",
                Content = $"Hello MongoDB{i}",
                CreateAt = DateTime.Now.AddDays(i),
                UpdateAt = DateTime.Now.AddDays(i)
            };
            blogs.Add(blog);
            // 新增一筆 Blog 資料
        }
        collection.InsertMany(blogs);
        stopwatch.Stop();
        // 顯示需要耗費時間
        Console.WriteLine($"一次新增 100 筆文件需要 {stopwatch.ElapsedMilliseconds} ms");
        #endregion
    }
}
```







