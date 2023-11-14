# MongoDB 系列 - 使用 C# 來刪除存在於 Docker 容器內的 MongoDB 資料庫內的文件

![](../Images/X2023-9849.png)

對於資料庫操作的 CRUD 應用之第四個部分，也是最後一個，那就是 D Delete 這個英文字，中文翻譯過來就是刪除，因此在篇文章將會要來探討這部分的程式設計做法。

經過前篇文章 [MongoDB 系列 - 使用 C# 來刪除存在於 Docker 容器內的 MongoDB 資料庫內的文件](https://csharpkh.blogspot.com/2023/11/MongoDB-Csharp-Update-Filter-Builder-Set-Linq-one-many.html
) 介紹，學會了如何透過 Filter 或者 .NET Linq 的方式來進行查詢出資料庫內符合條件的文件。

這樣的技術將會在要刪除 MongoDB 文件的時候用到，因為，當要進行 MongoDB 文件刪除的時候，需要指定刪除條件，也就是說要指定會影響到那些文件，因此，在這裡將會需要指定一個 Filter 物件。

在這篇文章中，也會使用 Linq 作為查詢條件，讓在刪除文件的時候，可以使用 Linq 表示式來進行查詢，不過，此時需要使用 C# 強型別的方式來進行設計。

另外，在 MongoDB 的刪除 API 中，會有刪除一筆文件或者是刪除多筆文件的操作，因此，在這裡將會介紹如何透過 C# 程式碼來進行這兩種不同的刪除操作。最後將會透過查詢方式，重新取得最新的文件內容，來驗證這些文件是否已經被刪除過了。

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
* 找到 [專案名稱] 欄位，輸入 `csMongoDBDelete` 作為專案名稱
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
* 此時，將會看到 [NuGet: csMongoDBDelete] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `MongoDB.Driver`
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

## 建立要使用的程式碼

* 在 [方案總管] 內找到並且開啟 [Program.cs] 檔案這個節點
* 使用底下 C# 程式碼，將原本的程式碼取代掉

```csharp
using MongoDB.Bson;
using MongoDB.Driver;
using System.Diagnostics;

namespace csMongoDBDelete;


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
        var collectionName = "BlogForDelete";

        // 取得 MongoDB Collection
        database = client.GetDatabase(dbName);

        #region 先行刪除這個測試用的 Collection
        await database.DropCollectionAsync(collectionName);
        #endregion

        collection = database.GetCollection<Blog>(collectionName);

        Stopwatch stopwatch = new Stopwatch();
        #endregion

        #region 建立準備要進行刪除用的測試文件
        #region 一次新增 10 筆文件
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"建立準備要進行刪除用的測試文件");
        stopwatch.Restart();
        List<Blog> blogs = new List<Blog>();
        stopwatch.Restart();
        for (int i = 0; i < 10; i++)
        {
            // 宣告一個 Blog 物件
            Blog blog = new Blog
            {
                BlogId = i,
                Title = $"Hello MongoDB{i}",
                Tag = $"C#",
                Content = $"Hello MongoDB{i%3}",
                CreateAt = DateTime.Now.AddDays(i).Date,
                UpdateAt = DateTime.Now.AddDays(i).Date
            };
            blogs.Add(blog);
        }
        // 進行批次新增 Blog 資料
        collection.InsertMany(blogs);
        stopwatch.Stop();
        // 顯示需要耗費時間
        Console.WriteLine($"一次新增 10 筆文件需要 {stopwatch.ElapsedMilliseconds} ms");
        #endregion
        #endregion

        #region 找出符合刪除條件的文件，並進行刪除一筆文件
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"找出符合刪除條件的文件，並進行刪除一筆文件");
        await Console.Out.WriteLineAsync($"Collection 內的所有文件");
        var byLinqCollectionWithClass = await collection.AsQueryable().ToListAsync();
        foreach (var item in byLinqCollectionWithClass)
        {
            Console.WriteLine($"  {item.Id} / {item.Title} / {item.Content}");
        }

        stopwatch.Restart();

        var filter1 = Builders<Blog>.Filter.Eq(r => r.Title, "Hello MongoDB5");
        var updateResult = await collection.DeleteOneAsync(filter1);

        stopwatch.Stop();
        // 顯示需要耗費時間
        Console.WriteLine($"刪除花費 {stopwatch.ElapsedMilliseconds} ms");
        await Console.Out.WriteLineAsync($"Status : {updateResult.IsAcknowledged} / {updateResult.DeletedCount}");
        await Console.Out.WriteLineAsync($"重新列出 Collection 內的所有文件");
        byLinqCollectionWithClass = await collection.AsQueryable().ToListAsync();
        foreach (var item in byLinqCollectionWithClass)
        {
            Console.WriteLine($"  {item.Id} / {item.Title} / {item.Content}");
        }
        #endregion

        #region 找出符合刪除條件的文件，並進行刪除多筆文件 使用 Builders.Filter
        Console.WriteLine();
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"找出符合刪除條件的文件，並進行刪除多筆文件 使用 Builders.Filter");
        await Console.Out.WriteLineAsync($"Collection 內的所有文件");
        byLinqCollectionWithClass = await collection.AsQueryable().ToListAsync();
        foreach (var item in byLinqCollectionWithClass)
        {
            Console.WriteLine($"  {item.Id} / {item.Title} / {item.Content}");
        }

        stopwatch.Restart();

        var filter2 = Builders<Blog>.Filter.Eq(r => r.Content, "Hello MongoDB2");
        var updateResult2 = await collection.DeleteManyAsync(filter2);
        stopwatch.Stop();
        // 顯示需要耗費時間
        Console.WriteLine($"使用 Builders.Filter 刪除花費 {stopwatch.ElapsedMilliseconds} ms");
        await Console.Out.WriteLineAsync($"Status : {updateResult2.IsAcknowledged} / {updateResult2.DeletedCount}");
        await Console.Out.WriteLineAsync($"重新列出 Collection 內的所有文件");
        byLinqCollectionWithClass = await collection.AsQueryable().ToListAsync();
        foreach (var item in byLinqCollectionWithClass)
        {
            Console.WriteLine($"  {item.Id} / {item.Title} / {item.Content}");
        }
        #endregion

        #region 找出符合刪除條件的文件，並進行刪除多筆文件 使用 LINQ
        Console.WriteLine();
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"找出符合刪除條件的文件，並進行刪除多筆文件 使用 LINQ");
        await Console.Out.WriteLineAsync($"Collection 內的所有文件");
        byLinqCollectionWithClass = await collection.AsQueryable().ToListAsync();
        foreach (var item in byLinqCollectionWithClass)
        {
            Console.WriteLine($"  {item.Id} / {item.Title} / {item.Content}");
        }

        stopwatch.Restart();

        var updateResult21 = await collection
            .DeleteManyAsync<Blog>(x => x.Content == "Hello MongoDB0" ||
            x.Title == "Hello MongoDB1");

        stopwatch.Stop();
        // 顯示需要耗費時間
        Console.WriteLine($"使用 Linq 刪除花費 {stopwatch.ElapsedMilliseconds} ms");
        //await Console.Out.WriteLineAsync($"Status : {updateResult21.IsAcknowledged} / {updateResult21.DeletedCount}");
        await Console.Out.WriteLineAsync($"重新列出 Collection 內的所有文件");
        byLinqCollectionWithClass = await collection.AsQueryable().ToListAsync();
        foreach (var item in byLinqCollectionWithClass)
        {
            Console.WriteLine($"  {item.Id} / {item.Title} / {item.Content}");
        }
        #endregion
        #endregion

    }
}
```

* 簡單說明這段範例程式碼所要做的事情
  * 先刪除測試用的 Collection
  * 新增10筆文件到剛剛刪除的 Collection
  * 取得所有的文件，列印在螢幕上
  * 使用條件 Title = "Hello MongoDB5" ，刪除一筆 Collection 內的文件
  * 取得所有的文件，列印在螢幕上，確認僅影響到一筆文件
  * 使用的條件為 Content = "Hello MongoDB2" 過濾條件，呼叫刪除多筆 API，刪除 Collection內的多筆文件
  * 取得所有的文件，列印在螢幕上，確認多筆文件是否已經刪除了
  * 最後，不使用 Builders.Filter 來建立一個過濾條件，而是直接在刪除 API 內使用 LINQ 表示式來指定出需要刪除的紀錄條件
  * 這裡使用到這樣的 `x.Content == "Hello MongoDB0" || x.Title == "Hello MongoDB1"` LINQ 表示式
  * 當然，還是要將 Collection 內的所有文件列印出來，確認是否真的有把指定條件的文件刪除了
* 在 `#region 先行刪除這個測試用的 Collection` 區塊內，將會使用 `await database.DropCollectionAsync(collectionName);` 敘述將這個 BlogForDelete Collection 集合刪除掉，目的是在於要讓這個刪除測試動作，其測試環境變得乾淨
* 不過，不用擔心這個 Collection 被刪除掉，一旦有文件要新增到這個 Collection 時候，這個 Collection 將會自動重新建立起來
* 在 `#region 建立準備要進行刪除用的測試文件` 區段內，將會建立 10 筆型別為 Blog 型別的文件到這個 BlogForDelete Collection 內，不過，對於 Content 這個屬性值，將會使用 `$"Hello MongoDB{i%3}"` 這樣的表示式來指定成為其值，也就是對於 Content 這個欄位而言，將會是 "Hello MongoDB0" or "Hello MongoDB1" or "Hello MongoDB2" 這三種字串的其中一個。
* 在 `#region 找出符合刪除條件的文件，並進行刪除一筆文件` 區塊內，將會使用 `var filter1 = Builders<Blog>.Filter.Eq(r => r.Title, "Hello MongoDB5");` 敘述，建立一個 Filter 物件，這個 Filter 物件的條件是 Title 屬性值等於 `Hello MongoDB5`；這樣的條件在這個測試用的 Collection 內，將只會查詢出僅有一筆文件
* 最後，使用 `var updateResult = await collection.DeleteOneAsync(filter1);` 敘述，呼叫 DeleteOneAsync API，將符合 Filter 條件的文件，進行刪除
* 接下來，將會把這個 Collection 內的所有文件都取出並且顯示在螢幕上，確認剛剛的刪除動作有成功
* 在 `#region 找出符合刪除條件的文件，並進行刪除多筆文件 使用 Builders.Filter` 區塊內，將會使用 `var filter2 = Builders<Blog>.Filter.Eq(r => r.Content, "Hello MongoDB2");` 敘述，建立一個 Filter 物件，這個 Filter 物件的條件是 Content 屬性值等於 `Hello MongoDB2`；透過這個 Filter 物件，將會查詢出多筆文件，因為，許多文件內的 Content 欄位內，都有這個字串值。
* 最後，使用 `var updateResult2 = await collection.DeleteManyAsync(filter2);` 敘述，呼叫 DeleteManyAsync API，將符合 Filter 條件的文件，進行刪除
* 接下來，將會把這個 Collection 內的所有文件都取出並且顯示在螢幕上，確認剛剛的刪除動作有成功
* 在 `#region 找出符合刪除條件的文件，並進行刪除多筆文件 使用 LINQ` 區塊內，將不需要使用 Builders.Filter 來建立一個過濾物件
* 最後，呼叫 DeleteManyAsync API，在第一個參數內，使用 LINQ 表示式來指定要查除文件的查詢條件，將符合條件的文件，進行刪除
```csharp
var updateResult21 = await collection
    .DeleteManyAsync<Blog>(x => x.Content == "Hello MongoDB0" ||
    x.Title == "Hello MongoDB1");
```
* 接下來，將會把這個 Collection 內的所有文件都取出並且顯示在螢幕上，確認剛剛的刪除動作有成功
* 底下將會這個程式碼執行結果

```
建立準備要進行刪除用的測試文件
一次新增 10 筆文件需要 138 ms

找出符合刪除條件的文件，並進行刪除一筆文件
Collection 內的所有文件
  6552da9f782cea2d1a7089a3 / Hello MongoDB0 / Hello MongoDB0
  6552da9f782cea2d1a7089a4 / Hello MongoDB1 / Hello MongoDB1
  6552da9f782cea2d1a7089a5 / Hello MongoDB2 / Hello MongoDB2
  6552da9f782cea2d1a7089a6 / Hello MongoDB3 / Hello MongoDB0
  6552da9f782cea2d1a7089a7 / Hello MongoDB4 / Hello MongoDB1
  6552da9f782cea2d1a7089a8 / Hello MongoDB5 / Hello MongoDB2
  6552da9f782cea2d1a7089a9 / Hello MongoDB6 / Hello MongoDB0
  6552da9f782cea2d1a7089aa / Hello MongoDB7 / Hello MongoDB1
  6552da9f782cea2d1a7089ab / Hello MongoDB8 / Hello MongoDB2
  6552da9f782cea2d1a7089ac / Hello MongoDB9 / Hello MongoDB0
刪除花費 27 ms
Status : True / 1
重新列出 Collection 內的所有文件
  6552da9f782cea2d1a7089a3 / Hello MongoDB0 / Hello MongoDB0
  6552da9f782cea2d1a7089a4 / Hello MongoDB1 / Hello MongoDB1
  6552da9f782cea2d1a7089a5 / Hello MongoDB2 / Hello MongoDB2
  6552da9f782cea2d1a7089a6 / Hello MongoDB3 / Hello MongoDB0
  6552da9f782cea2d1a7089a7 / Hello MongoDB4 / Hello MongoDB1
  6552da9f782cea2d1a7089a9 / Hello MongoDB6 / Hello MongoDB0
  6552da9f782cea2d1a7089aa / Hello MongoDB7 / Hello MongoDB1
  6552da9f782cea2d1a7089ab / Hello MongoDB8 / Hello MongoDB2
  6552da9f782cea2d1a7089ac / Hello MongoDB9 / Hello MongoDB0


找出符合刪除條件的文件，並進行刪除多筆文件 使用 Builders.Filter
Collection 內的所有文件
  6552da9f782cea2d1a7089a3 / Hello MongoDB0 / Hello MongoDB0
  6552da9f782cea2d1a7089a4 / Hello MongoDB1 / Hello MongoDB1
  6552da9f782cea2d1a7089a5 / Hello MongoDB2 / Hello MongoDB2
  6552da9f782cea2d1a7089a6 / Hello MongoDB3 / Hello MongoDB0
  6552da9f782cea2d1a7089a7 / Hello MongoDB4 / Hello MongoDB1
  6552da9f782cea2d1a7089a9 / Hello MongoDB6 / Hello MongoDB0
  6552da9f782cea2d1a7089aa / Hello MongoDB7 / Hello MongoDB1
  6552da9f782cea2d1a7089ab / Hello MongoDB8 / Hello MongoDB2
  6552da9f782cea2d1a7089ac / Hello MongoDB9 / Hello MongoDB0
使用 Builders.Filter 刪除花費 5 ms
Status : True / 2
重新列出 Collection 內的所有文件
  6552da9f782cea2d1a7089a3 / Hello MongoDB0 / Hello MongoDB0
  6552da9f782cea2d1a7089a4 / Hello MongoDB1 / Hello MongoDB1
  6552da9f782cea2d1a7089a6 / Hello MongoDB3 / Hello MongoDB0
  6552da9f782cea2d1a7089a7 / Hello MongoDB4 / Hello MongoDB1
  6552da9f782cea2d1a7089a9 / Hello MongoDB6 / Hello MongoDB0
  6552da9f782cea2d1a7089aa / Hello MongoDB7 / Hello MongoDB1
  6552da9f782cea2d1a7089ac / Hello MongoDB9 / Hello MongoDB0


找出符合刪除條件的文件，並進行刪除多筆文件 使用 LINQ
Collection 內的所有文件
  6552da9f782cea2d1a7089a3 / Hello MongoDB0 / Hello MongoDB0
  6552da9f782cea2d1a7089a4 / Hello MongoDB1 / Hello MongoDB1
  6552da9f782cea2d1a7089a6 / Hello MongoDB3 / Hello MongoDB0
  6552da9f782cea2d1a7089a7 / Hello MongoDB4 / Hello MongoDB1
  6552da9f782cea2d1a7089a9 / Hello MongoDB6 / Hello MongoDB0
  6552da9f782cea2d1a7089aa / Hello MongoDB7 / Hello MongoDB1
  6552da9f782cea2d1a7089ac / Hello MongoDB9 / Hello MongoDB0
使用 Linq 刪除花費 28 ms
重新列出 Collection 內的所有文件
  6552da9f782cea2d1a7089a7 / Hello MongoDB4 / Hello MongoDB1
  6552da9f782cea2d1a7089aa / Hello MongoDB7 / Hello MongoDB1
```