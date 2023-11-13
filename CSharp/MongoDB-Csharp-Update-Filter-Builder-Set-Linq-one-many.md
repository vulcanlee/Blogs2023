# MongoDB 系列 - 使用 C# 來更新存在於 Docker 容器內的 MongoDB 資料庫內的文件

![](../Images/X2023-9850.png)

對於資料庫操作的 CRUD 應用之第三個部分，也就是 U Update 這個英文字，中文翻譯過來就是更新，因此在篇文章將會要來探討這部分的程式設計做法。

經過前篇文章 [MongoDB 系列 - 使用 C# 來查詢存在於 Docker 容器內的 MongoDB 資料庫內的文件](https://csharpkh.blogspot.com/2023/11/MongoDB-Csharp-Query-Retrive-Find-Document-Filter-Pretty.html
) 介紹，學會了如何透過 Filter 或者 .NET Linq 的方式來進行查詢出資料庫內符合條件的文件。

這樣的技術將會在要更新 MongoDB 文件的時候用到，因為，當要進行 MongoDB 文件更新的時候，需要指定更新條件，也就是說要指定會影響到那些文件，因此，在這裡將會需要指定一個 Filter 物件。

另外，在 MongoDB 的更新 API 中，會有更新一筆文件或者是更新多筆文件的操作，因此，在這裡將會介紹如何透過 C# 程式碼來進行這兩種不同的更新操作。最後將會透過查詢方式，重新取得最新的文件內容，來驗證這些文件是否已經被更新過了。

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
* 找到 [專案名稱] 欄位，輸入 `csMongoDBUpdate` 作為專案名稱
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
* 此時，將會看到 [NuGet: csMongoDBUpdate] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `MongoDB.Driver`
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

## 建立要使用的程式碼

* 在 [方案總管] 內找到並且開啟 [Program.cs] 檔案這個節點
* 使用底下 C# 程式碼，將原本的程式碼取代掉

```csharp
using System.Diagnostics;
using MongoDB.Bson;
using MongoDB.Driver;
using MongoDB.Driver.Linq;
using System.Diagnostics;
using static MongoDB.Driver.WriteConcern;

namespace csMongoDBUpdate;

// MongoDB 的 Blog 文件資料結構
public class Blog
{
    public ObjectId Id { get; set; }
    public int BlogId { get; set; }
    public string Title { get; set; } = string.Empty;
    public string Tag { get; set; } = string.Empty;
    public string Content { get; set; } = string.Empty;
    public DateTime CreateAt { get; set; } = DateTime.Now;
    public DateTime UpdateAt { get; set; } = DateTime.Now;
}

internal class Program
{
    public static async Task Main(string[] args)
    {
        #region 準備相關設定要進行與雲端 MongoDB 連線用的參數與物件
        // 使用 Environment 來抓取環境變數設定的 帳號與密碼
        string MongoDBAccount = Environment.GetEnvironmentVariable("MongoDBAccount");
        string MongoDBPassword = Environment.GetEnvironmentVariable("MongoDBPassword");

        // 使用 MongoDB Atlas 來連線
        //var mongoUri = $"mongodb+srv://{MongoDBAccount}:{MongoDBPassword}@vulcanmongo.hptf95d.mongodb.net/?retryWrites=true&w=majority";
        var mongoUri = $"mongodb://localhost:27017/?retryWrites=true&w=majority";

        // 宣告一個 MongoDB Client 變數
        IMongoClient client;

        // 宣告一個 MongoDB Database 變數
        IMongoDatabase database;

        // 宣告一個 MongoDB Collection 變數
        IMongoCollection<Blog> collection;

        // 連線到 MongoDB Atlas
        client = new MongoClient(mongoUri);
        #endregion

        #region 進行各種不同 MongoDB 資料庫的 Collection 查詢作法
        #region 建立操作 MogoDB 資料庫與Collection 物件
        // 宣告一個 Database Name 與 Collection Name
        var dbName = "MyCrud";
        var collectionName = "BlogForUpdate";

        // 取得 MongoDB Collection
        database = client.GetDatabase(dbName);

        #region 先行刪除這個測試用的 Collection
        await database.DropCollectionAsync(collectionName);
        #endregion

        collection = database.GetCollection<Blog>(collectionName);

        Stopwatch stopwatch = new Stopwatch();
        #endregion


        #region 建立準備要進行更新用的測試文件
        #region 一次新增 5 筆文件
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"建立準備要進行更新用的測試文件");
        stopwatch.Restart();
        List<Blog> blogs = new List<Blog>();
        stopwatch.Restart();
        for (int i = 0; i < 5; i++)
        {
            // 宣告一個 Blog 物件
            Blog blog = new Blog
            {
                BlogId = i,
                Title = $"Hello MongoDB{i}",
                Tag = $"C#",
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
        Console.WriteLine($"一次新增 5 筆文件需要 {stopwatch.ElapsedMilliseconds} ms");
        #endregion
        #endregion

        #region 找出符合更新條件的文件，並進行更新一筆文件
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"找出符合更新條件的文件，並進行更新一筆文件");
        await Console.Out.WriteLineAsync($"Collection 內的所有文件");
        var byLinqCollectionWithClass = await collection.AsQueryable().ToListAsync();
        foreach (var item in byLinqCollectionWithClass)
        {
            Console.WriteLine($"  {item.Id} / {item.Title} / {item.Tag}");
        }

        stopwatch.Restart();

        var filter1 = Builders<Blog>.Filter.Eq(r => r.Tag, "C#");
        var update1 = Builders<Blog>.Update.Set(x => x.Tag, "SQL");
        var updateResult = await collection.UpdateOneAsync(filter1, update1);

        stopwatch.Stop();
        // 顯示需要耗費時間
        Console.WriteLine($"更新花費 {stopwatch.ElapsedMilliseconds} ms");
        await Console.Out.WriteLineAsync($"Status : {updateResult.IsAcknowledged} / {updateResult.ModifiedCount}");
        await Console.Out.WriteLineAsync($"重新列出 Collection 內的所有文件");
        byLinqCollectionWithClass = await collection.AsQueryable().ToListAsync();
        foreach (var item in byLinqCollectionWithClass)
        {
            Console.WriteLine($"  {item.Id} / {item.Title} / {item.Tag}");
        }
        #endregion

        #region 找出符合更新條件的文件，並進行更新多筆文件
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"找出符合更新條件的文件，並進行更新多筆文件");
        await Console.Out.WriteLineAsync($"Collection 內的所有文件");
        byLinqCollectionWithClass = await collection.AsQueryable().ToListAsync();
        foreach (var item in byLinqCollectionWithClass)
        {
            Console.WriteLine($"  {item.Id} / {item.Title} / {item.Tag}");
        }

        stopwatch.Restart();

        var filter2 = Builders<Blog>.Filter.Eq(r => r.Tag, "C#");
        var update2 = Builders<Blog>.Update.Set(x => x.Tag, "XAML");
        var updateResult2 = await collection.UpdateManyAsync(filter2, update2);

        stopwatch.Stop();
        // 顯示需要耗費時間
        Console.WriteLine($"更新花費 {stopwatch.ElapsedMilliseconds} ms");
        await Console.Out.WriteLineAsync($"Status : {updateResult2.IsAcknowledged} / {updateResult2.ModifiedCount}");
        await Console.Out.WriteLineAsync($"重新列出 Collection 內的所有文件");
        byLinqCollectionWithClass = await collection.AsQueryable().ToListAsync();
        foreach (var item in byLinqCollectionWithClass)
        {
            Console.WriteLine($"  {item.Id} / {item.Title} / {item.Tag}");
        }
        #endregion
        #endregion

    }
}
```

* 簡單說明這段範例程式碼所要做的事情
  * 先刪除測試用的 Collection
  * 新增五筆文件到剛剛刪除的 Collection
  * 取得所有的文件，列印在螢幕上
  * 使用條件 Tag = "C#" ，更新一筆 Collection 內的文件
  * 取得所有的文件，列印在螢幕上，確認僅影響到一筆文件
  * 過濾條件不變，呼叫更新多筆 API，更新 Collection內的多筆文件
  * 取得所有的文件，列印在螢幕上，確認僅影響到一筆文件
* 首先，在 Blog 類別內，新增加了一個 Tag 屬性
* 在 `#region 進行各種不同 MongoDB 資料庫的 Collection 查詢作法` 前的程式碼，將會與前面文章的做法相同，若還有不太了解的，可以參考前面文章的說明
* 在 `#region 先行刪除這個測試用的 Collection` 區塊內，將會使用 `await database.DropCollectionAsync(collectionName);` 敘述將這個 BlogForUpdate Collection 集合刪除掉，目的是在於要讓這個更新測試動作，其測試環境變得乾淨
* 不過，不用擔心這個 Collection 被刪除掉，一旦有文件要新增到這個 Collection 時候，這個 Collection 將會自動重新建立起來
* 接下來，使用 `collection.InsertMany(blogs);` 敘述，將五個新建立的 Blog 物件，一次新增到這個 BlogForUpdate Collection 內；而每個 Blog 物件的 Tag 屬性值都是 `C#`
* 在 `#region 找出符合更新條件的文件，並進行更新一筆文件` 區塊內，將會使用 `var filter1 = Builders<Blog>.Filter.Eq(r => r.Tag, "C#");` 敘述，建立一個 Filter 物件，這個 Filter 物件的條件是 Tag 屬性值等於 `C#`
* 接著，使用 `var update1 = Builders<Blog>.Update.Set(x => x.Tag, "SQL");` 敘述，建立一個 Update 物件，這個 Update 物件的作用是將 Tag 屬性值改為 `SQL`
* 最後，使用 `var updateResult = await collection.UpdateOneAsync(filter1, update1);` 敘述，呼叫 UpdateOneAsync API，將符合 Filter 條件的文件，進行更新，更新的內容是 Update 物件所定義的內容
* 接下來，將會把這個 Collection 內的所有文件都取出並且顯示在螢幕上，確認剛剛的更新動作有成功
* 在 `#region 找出符合更新條件的文件，並進行更新多筆文件` 區塊內，將會使用 `var filter2 = Builders<Blog>.Filter.Eq(r => r.Tag, "C#");` 敘述，建立一個 Filter 物件，這個 Filter 物件的條件是 Tag 屬性值等於 `C#`
* 接著，使用 `var update2 = Builders<Blog>.Update.Set(x => x.Tag, "XAML");` 敘述，建立一個 Update 物件，這個 Update 物件的作用是將 Tag 屬性值改為 `XAML`
* 最後，使用 `var updateResult2 = await collection.UpdateManyAsync(filter2, update2);` 敘述，呼叫 UpdateManyAsync API，將符合 Filter 條件的文件，進行更新，更新的內容是 Update 物件所定義的內容
* 接下來，將會把這個 Collection 內的所有文件都取出並且顯示在螢幕上，確認剛剛的更新動作有成功
* 底下將會這個程式碼執行結果

```
建立準備要進行更新用的測試文件
一次新增 5 筆文件需要 177 ms

找出符合更新條件的文件，並進行更新一筆文件
Collection 內的所有文件
  6551cc2516242b9436f5f8c9 / Hello MongoDB0 / C#
  6551cc2516242b9436f5f8ca / Hello MongoDB1 / C#
  6551cc2516242b9436f5f8cb / Hello MongoDB2 / C#
  6551cc2516242b9436f5f8cc / Hello MongoDB3 / C#
  6551cc2516242b9436f5f8cd / Hello MongoDB4 / C#
更新花費 66 ms
Status : True / 1

重新列出 Collection 內的所有文件
  6551cc2516242b9436f5f8c9 / Hello MongoDB0 / SQL
  6551cc2516242b9436f5f8ca / Hello MongoDB1 / C#
  6551cc2516242b9436f5f8cb / Hello MongoDB2 / C#
  6551cc2516242b9436f5f8cc / Hello MongoDB3 / C#
  6551cc2516242b9436f5f8cd / Hello MongoDB4 / C#

找出符合更新條件的文件，並進行更新多筆文件
Collection 內的所有文件
  6551cc2516242b9436f5f8c9 / Hello MongoDB0 / SQL
  6551cc2516242b9436f5f8ca / Hello MongoDB1 / C#
  6551cc2516242b9436f5f8cb / Hello MongoDB2 / C#
  6551cc2516242b9436f5f8cc / Hello MongoDB3 / C#
  6551cc2516242b9436f5f8cd / Hello MongoDB4 / C#
更新花費 9 ms
Status : True / 4

重新列出 Collection 內的所有文件
  6551cc2516242b9436f5f8c9 / Hello MongoDB0 / SQL
  6551cc2516242b9436f5f8ca / Hello MongoDB1 / XAML
  6551cc2516242b9436f5f8cb / Hello MongoDB2 / XAML
  6551cc2516242b9436f5f8cc / Hello MongoDB3 / XAML
  6551cc2516242b9436f5f8cd / Hello MongoDB4 / XAML
  ```