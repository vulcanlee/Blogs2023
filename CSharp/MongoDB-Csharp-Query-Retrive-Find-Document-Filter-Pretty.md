# MongoDB 系列 - 使用 C# 來查詢存在於 Docker 容器內的 MongoDB 資料庫內的文件

![FilterDefinitionBuilder<TDocument> Class](../Images/X2023-9852.png)

對於資料庫操作的 CRUD 應用之第二個部分，也就是 R Retrive 這個英文字，中文翻譯過來就是查詢，因此在篇文章將會要來探討這部分的程式設計做法。

經過前篇文章 [MongoDB 系列 - 使用 C# 來新增文件記錄到 Docker 容器內的 MongoDB 資料庫](https://csharpkh.blogspot.com/2023/11/MongoDB-Csharp-Insert-Document.html) 介紹，已經知道如何透過 MongoDB.Driver 套件來新增文件記錄到 Docker 容器內的 MongoDB 資料庫。接下來，將會介紹如何透過 MongoDB.Driver 套件來查詢存在於 Docker 容器內的 MongoDB 資料庫內的文件。

這裡將會分別使用 C# 類別最強型別 與 LINQ 來進行查詢，或使用 Builders 類別來建立一個 FilterDefinition 物件，來進行查詢。至於實際開發過程上，到底該如何選擇，則是由讀者自行決定。

除了使用 C# 強型別方式來進行查詢操作，還可以使用 MongoDB 提供的 BsonDocument 文件格式來進行查詢操作。不過，這裡將不會介紹如何使用 BsonDocument 文件格式來進行查詢操作。

[](https://mongodb.github.io/mongo-csharp-driver/2.22/html/T_MongoDB_Driver_FilterDefinitionBuilder_1.htm)

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
* 找到 [專案名稱] 欄位，輸入 `csMongoDBRetrive` 作為專案名稱
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
* 此時，將會看到 [NuGet: csMongoDBRetrive] 視窗
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
using MongoDB.Driver.Linq;
using System.Diagnostics;

namespace csMongoDBRetrive;


// MongoDB 的 Blog 文件資料結構
public class Blog
{
    public ObjectId Id { get; set; }
    public int BlogId { get; set; }
    public string Title { get; set; } = string.Empty;
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
        var collectionName = "Blog";

        // 取得 MongoDB Collection
        collection = client.GetDatabase(dbName)
           .GetCollection<Blog>(collectionName);

        Stopwatch stopwatch = new Stopwatch();
        #endregion

        #region 使用 C# 類別最強型別 與 Builders 來進行所有文件查詢
        await Console.Out.WriteLineAsync($"用 C# 類別最強型別的查詢 與 Builders 來進行所有文件查詢");
        stopwatch.Restart();
        var filter = Builders<Blog>.Filter.Empty;
        var byFilterAllResultWithClass = await collection.Find(filter).ToListAsync();

        stopwatch.Stop();
        if (byFilterAllResultWithClass.Count > 0)
        {
            Console.WriteLine($"查詢到的文件數量為：{byFilterAllResultWithClass.Count}");
            foreach (var item in byFilterAllResultWithClass)
            {
                Console.WriteLine($"查詢到的文件內容為：{item.Id} / {item.Title}");
            }
        }
        else
        {
            Console.WriteLine($"查詢不到任何文件");
        }
        // 顯示需要耗費時間
        Console.WriteLine($"查詢花費 {stopwatch.ElapsedMilliseconds} ms");
        #endregion

        #region 使用 C# 類別最強型別 與 LINQ 來進行所有文件查詢
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"用 C# 類別最強型別的查詢 與 LINQ 來進行所有文件查詢");
        stopwatch.Restart();
        var byLinqAllResultWithClass = await collection.AsQueryable()
            .ToListAsync();

        stopwatch.Stop();
        if (byLinqAllResultWithClass.Count > 0)
        {
            Console.WriteLine($"查詢到的文件數量為：{byLinqAllResultWithClass.Count}");
            foreach (var item in byLinqAllResultWithClass)
            {
                Console.WriteLine($"查詢到的文件內容為：{item.Id} / {item.Title}");
            }
        }
        else
        {
            Console.WriteLine($"查詢不到任何文件");
        }
        // 顯示需要耗費時間
        Console.WriteLine($"查詢花費 {stopwatch.ElapsedMilliseconds} ms");
        #endregion

        #region 使用 C# 類別最強型別 與 Builders 來進行條件查詢
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"用 C# 類別最強型別的查詢 與 Builders 來進行條件查詢");
        stopwatch.Restart();
        var filter5 = Builders<Blog>.Filter
            .Eq(r => r.Title, "Hello MongoDB3");
        var byFilterResultWithClass = await collection.Find(filter5).FirstOrDefaultAsync();

        stopwatch.Stop();
        if (byFilterResultWithClass != null)
        {
            Console.WriteLine($"查詢到的文件內容為：{byFilterResultWithClass.Id} / {byFilterResultWithClass.Title}");
        }
        else
        {
            Console.WriteLine($"查詢不到任何文件");
        }
        // 顯示需要耗費時間
        Console.WriteLine($"查詢花費 {stopwatch.ElapsedMilliseconds} ms");
        #endregion

        #region 使用 C# 類別最強型別 與 LINQ 來進行條件查詢
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"用 C# 類別最強型別的查詢 與 LINQ 來進行條件查詢");
        stopwatch.Restart();
        var byLinqResultWithClass = await collection.AsQueryable()
            .Where(r => r.Title == "Hello MongoDB3").FirstOrDefaultAsync();

        stopwatch.Stop();
        if (byLinqResultWithClass != null)
        {
            Console.WriteLine($"查詢到的文件數量為：{byLinqResultWithClass.Id} / {byLinqResultWithClass.Title}");
        }
        else
        {
            Console.WriteLine($"查詢不到任何文件");
        }
        // 顯示需要耗費時間
        Console.WriteLine($"查詢花費 {stopwatch.ElapsedMilliseconds} ms");
        #endregion

        #region 使用 C# 類別最強型別 與 Builders 來進行多筆文件查詢
        await Console.Out.WriteLineAsync($"用 C# 類別最強型別 與 Builders 來進行多筆文件查詢");
        stopwatch.Restart();
        var filter3 = Builders<Blog>.Filter
            .Regex(x => x.Title, new BsonRegularExpression("MongoDB1"));
        var byFilterRegexResultWithClass = await collection.Find(filter3).ToListAsync();

        stopwatch.Stop();
        if (byFilterRegexResultWithClass.Count > 0)
        {
            Console.WriteLine($"查詢到的文件內容數量為：{byFilterRegexResultWithClass.Count}");
            foreach (var item in byFilterRegexResultWithClass)
            {
                Console.WriteLine($"查詢到的文件內容為：{item.Id} / {item.Title}");
            }
        }
        else
        {
            Console.WriteLine($"查詢不到任何文件");
        }
        // 顯示需要耗費時間
        Console.WriteLine($"查詢花費 {stopwatch.ElapsedMilliseconds} ms");
        #endregion

        #region 使用 C# 類別最強型別 與 LINQ 來進行多筆文件查詢
        Console.WriteLine();
        await Console.Out.WriteLineAsync($"用 C# 類別最強型別的查詢 與 LINQ 來進行多筆文件查詢");
        stopwatch.Restart();
        var byLinqMultipleResultWithClass = await collection.AsQueryable()
            .Where(r => r.Title.Contains("MongoDB1")).ToListAsync();

        stopwatch.Stop();
        if (byLinqMultipleResultWithClass.Count > 0)
        {
            Console.WriteLine($"查詢到的文件數量為：{byLinqMultipleResultWithClass.Count}");
            foreach (var item in byLinqMultipleResultWithClass)
            {
                Console.WriteLine($"查詢到的文件內容為：{item.Id} / {item.Title}");
            }
        }
        else
        {
            Console.WriteLine($"查詢不到任何文件");
        }
        // 顯示需要耗費時間
        Console.WriteLine($"查詢花費 {stopwatch.ElapsedMilliseconds} ms");
        #endregion
        #endregion

    }
}
```

* 上面的程式碼將會分成八個區塊，如下圖

  ![](../Images/X2023-9851.png)

* 在 [準備相關設定要進行與雲端 MongoDB 連線用的參數與物件] & [建立操作 MogoDB 資料庫與Collection 物件] 這兩個區塊，已經於前篇文章 [MongoDB 系列 - 使用 C# 來新增文件記錄到 Docker 容器內的 MongoDB 資料庫](https://csharpkh.blogspot.com/2023/11/MongoDB-Csharp-Insert-Document.html) 介紹過，因此這裡就不再贅述。
* 在這裡是透過 `collection = client.GetDatabase(dbName).GetCollection<Blog>(collectionName);` 這行程式碼，來取得 MongoDB 資料庫內的 Blog 這個 Collection 物件。這裡使用的是 C# 類別最強型別的方式來進行查詢操作(透過這個程式碼 `GetCollection<Blog>` 來做到)。
* 首先來看看 #region 的 [使用 C# 類別最強型別 與 Builders 來進行所有文件查詢] 這個區塊程式碼
* 這裡先使用 `Builders<Blog>.Filter.Empty` 建立一個空白 Filter，這表示該過濾器不會套用任何篩選條件。當使用此過濾器執行查詢時，它會傳回集合中的所有文檔，而不是進行任何過濾
* 該表示式將會回傳 FilterDefinitionBuilder 型別的物件，並且儲存在 filter 這個變數內
* 現在已經擁有了 `IMongoCollection<Blog>` 與 `FilterDefinitionBuilder<Blog>` 這兩個物件，便可以來進行查詢所有文件的操作
* 在這個敘述中 `var byFilterAllResultWithClass = await collection.Find(filter).ToListAsync();` 將會把 collection 物件指向的 MongoDB 之 Collection內所有的文件，使用非同步呼叫的方式將所也有文件都取回到 [byFilterAllResultWithClass] 這個變數內
* 剩下的程式碼則會依據取得結果，顯示所查詢的結果內容，底下是執行結果

```
用 C# 類別最強型別的查詢 與 Builders 來進行所有文件查詢
查詢到的文件數量為：200
查詢到的文件內容為：65517f26f50adca945544438 / Hello MongoDB0
查詢到的文件內容為：65517f26f50adca94554443a / Hello MongoDB1
查詢到的文件內容為：65517f26f50adca94554443c / Hello MongoDB2
查詢到的文件內容為：65517f26f50adca94554443e / Hello MongoDB3
...
查詢到的文件內容為：65517f26f50adca94554455f / Hello MongoDB95
查詢到的文件內容為：65517f26f50adca945544560 / Hello MongoDB96
查詢到的文件內容為：65517f26f50adca945544561 / Hello MongoDB97
查詢到的文件內容為：65517f26f50adca945544562 / Hello MongoDB98
查詢到的文件內容為：65517f26f50adca945544563 / Hello MongoDB99
查詢花費 197 ms
```

* 接下來看看 #region 的 [使用 C# 類別最強型別 與 LINQ 來進行所有文件查詢] 這個區塊程式碼
* 為了要能夠在 C# 內使用 Linq 語法來進行 MongoDB 資料庫內文件查詢，需要在最前面加入這個 [MongoDB.Driver.Linq] 命名空間，使用 `using MongoDB.Driver.Linq` 語法
* 由於 [collection] 這個物件是個強型別，其型別為 Blog，因此，對這個物件呼叫 `AsQueryable()` 方法之後，便可以開始使用 Linq 語法了
* 這裡使用這樣的 Linq 語法 `var byLinqAllResultWithClass = await collection.AsQueryable().ToListAsync();` 來進行非同步查詢所有文件的操作
* 在這裡將會得到與使用 `FilterDefinitionBuilder` 方法所得到的內容相同
* 底下是執行結果

```
用 C# 類別最強型別的查詢 與 LINQ 來進行所有文件查詢
查詢到的文件數量為：200
查詢到的文件內容為：65517f26f50adca945544438 / Hello MongoDB0
查詢到的文件內容為：65517f26f50adca94554443a / Hello MongoDB1
查詢到的文件內容為：65517f26f50adca94554443c / Hello MongoDB2
查詢到的文件內容為：65517f26f50adca94554443e / Hello MongoDB3
...
查詢到的文件內容為：65517f26f50adca94554455f / Hello MongoDB95
查詢到的文件內容為：65517f26f50adca945544560 / Hello MongoDB96
查詢到的文件內容為：65517f26f50adca945544561 / Hello MongoDB97
查詢到的文件內容為：65517f26f50adca945544562 / Hello MongoDB98
查詢到的文件內容為：65517f26f50adca945544563 / Hello MongoDB99
查詢花費 74 ms
```

* 接下來看看 #region 的 [用 C# 類別最強型別的查詢 與 Builders 來進行條件查詢] 這個區塊程式碼
* 在此使用 `var filter5 = Builders<Blog>.Filter.Eq(r => r.Title, "Hello MongoDB3");` 這個敘述，建立一個過濾條件，要找出 Title 這個欄位的內容值為 "Hello MongoDB3" 的所有文件出來。
* 有了上述過濾條件，接下來使用 `var byFilterResultWithClass = await collection.Find(filter5).FirstOrDefaultAsync();` 敘述取得查詢結果文件
* 根據得到的文件內容，顯示查詢結果在螢幕上
* 底下是執行後的輸出內容

```
用 C# 類別最強型別的查詢 與 Builders 來進行條件查詢
查詢到的文件內容為：65517f26f50adca94554443e / Hello MongoDB3
查詢花費 49 ms
```

* 接下來看看 #region 的 [使用 C# 類別最強型別 與 LINQ 來進行條件查詢] 這個區塊程式碼
* 這裡使用這樣 Linq 敘述 `var byLinqResultWithClass = await collection.AsQueryable().Where(r => r.Title == "Hello MongoDB3").FirstOrDefaultAsync();` 來取得查詢結果，並且根據得到的物件，將結果顯示在螢幕上
* 底下是執行後的輸出內容

```
用 C# 類別最強型別的查詢 與 LINQ 來進行條件查詢
查詢到的文件數量為：65517f26f50adca94554443e / Hello MongoDB3
查詢花費 243 ms
```

* 接下來看看 #region 的 [使用 C# 類別最強型別 與 Builders 來進行多筆文件查詢] 這個區塊程式碼
* 這裡想要展示的是類似 SQL 命令中的 Like 功能，也就是說要找出所有文件中，在某個欄位內，有包含指定文字的所有文件出來
* 由於 Builders.Filter 內並沒有類似直接的方法，可以用於 SQL 命令中的 Like 功能，因此，這裡需要使用正規表示式來做到這樣需求
* 在此使用了 `var filter3 = Builders<Blog>.Filter.Regex(x => x.Title, new BsonRegularExpression("MongoDB1"));` 敘述來做到實踐這樣功能，這裡使用了 BsonRegularExpression 這個物件，建立出一個 BSON Regex 物件出來，若所指定的字串可以被找到，將會回傳為 true
* 最後，將剛剛建立的 Filter3 物件，傳遞到這個敘述 `var byFilterRegexResultWithClass = await collection.Find(filter3).ToListAsync();` 中的 Find 方法內，如此便可以找到所有文件中，Title 欄位內有包含 MongoDB1 這樣文字的文件
* 底下是執行後的輸出內容

```
用 C# 類別最強型別 與 Builders 來進行多筆文件查詢
查詢到的文件內容數量為：22
查詢到的文件內容為：65517f26f50adca94554443a / Hello MongoDB1
查詢到的文件內容為：65517f26f50adca94554444c / Hello MongoDB10
查詢到的文件內容為：65517f26f50adca94554444e / Hello MongoDB11
查詢到的文件內容為：65517f26f50adca945544450 / Hello MongoDB12
查詢到的文件內容為：65517f26f50adca945544452 / Hello MongoDB13
查詢到的文件內容為：65517f26f50adca945544454 / Hello MongoDB14
查詢到的文件內容為：65517f26f50adca945544456 / Hello MongoDB15
查詢到的文件內容為：65517f26f50adca945544458 / Hello MongoDB16
查詢到的文件內容為：65517f26f50adca94554445a / Hello MongoDB17
查詢到的文件內容為：65517f26f50adca94554445c / Hello MongoDB18
查詢到的文件內容為：65517f26f50adca94554445e / Hello MongoDB19
查詢到的文件內容為：65517f26f50adca945544501 / Hello MongoDB1
查詢到的文件內容為：65517f26f50adca94554450a / Hello MongoDB10
查詢到的文件內容為：65517f26f50adca94554450b / Hello MongoDB11
查詢到的文件內容為：65517f26f50adca94554450c / Hello MongoDB12
查詢到的文件內容為：65517f26f50adca94554450d / Hello MongoDB13
查詢到的文件內容為：65517f26f50adca94554450e / Hello MongoDB14
查詢到的文件內容為：65517f26f50adca94554450f / Hello MongoDB15
查詢到的文件內容為：65517f26f50adca945544510 / Hello MongoDB16
查詢到的文件內容為：65517f26f50adca945544511 / Hello MongoDB17
查詢到的文件內容為：65517f26f50adca945544512 / Hello MongoDB18
查詢到的文件內容為：65517f26f50adca945544513 / Hello MongoDB19
查詢花費 8 ms
```

* 最後來看看 #region 的 [使用 C# 類別最強型別 與 LINQ 來進行多筆文件查詢] 這個區塊程式碼
* 對於已經相當孰悉 Linq 語法的 C# 開發者而言，這裡的程式碼就顯得相當簡單多了
* 使用 `collection.AsQueryable()` 表示式，將這個 Collection 物件轉換成為可查詢的物件，如此，便可以在這個表示式之後，開始使用 Linq 語法來下達查詢條件了
* 如此便可以找到所有文件中，Title 欄位內有包含 MongoDB1 這樣文字的文件
* 底下是執行後的輸出內容

```
用 C# 類別最強型別的查詢 與 LINQ 來進行多筆文件查詢
查詢到的文件數量為：22
查詢到的文件內容為：65517f26f50adca94554443a / Hello MongoDB1
查詢到的文件內容為：65517f26f50adca94554444c / Hello MongoDB10
查詢到的文件內容為：65517f26f50adca94554444e / Hello MongoDB11
查詢到的文件內容為：65517f26f50adca945544450 / Hello MongoDB12
查詢到的文件內容為：65517f26f50adca945544452 / Hello MongoDB13
查詢到的文件內容為：65517f26f50adca945544454 / Hello MongoDB14
查詢到的文件內容為：65517f26f50adca945544456 / Hello MongoDB15
查詢到的文件內容為：65517f26f50adca945544458 / Hello MongoDB16
查詢到的文件內容為：65517f26f50adca94554445a / Hello MongoDB17
查詢到的文件內容為：65517f26f50adca94554445c / Hello MongoDB18
查詢到的文件內容為：65517f26f50adca94554445e / Hello MongoDB19
查詢到的文件內容為：65517f26f50adca945544501 / Hello MongoDB1
查詢到的文件內容為：65517f26f50adca94554450a / Hello MongoDB10
查詢到的文件內容為：65517f26f50adca94554450b / Hello MongoDB11
查詢到的文件內容為：65517f26f50adca94554450c / Hello MongoDB12
查詢到的文件內容為：65517f26f50adca94554450d / Hello MongoDB13
查詢到的文件內容為：65517f26f50adca94554450e / Hello MongoDB14
查詢到的文件內容為：65517f26f50adca94554450f / Hello MongoDB15
查詢到的文件內容為：65517f26f50adca945544510 / Hello MongoDB16
查詢到的文件內容為：65517f26f50adca945544511 / Hello MongoDB17
查詢到的文件內容為：65517f26f50adca945544512 / Hello MongoDB18
查詢到的文件內容為：65517f26f50adca945544513 / Hello MongoDB19
查詢花費 8 ms
```
