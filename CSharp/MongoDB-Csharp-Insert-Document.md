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
* 此時，將會看到 [NuGet: csMongoDBCreate] 視窗
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
        #region 準備相關設定要進行與雲端 MongoDB 連線用的參數與物件
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
        #endregion

        #region 列出所有的資料庫名稱
        Console.WriteLine($"列出所有存在的資料庫");
        var dbs = client.ListDatabases().ToList();
        foreach (var item in dbs)
        {
            Console.WriteLine(item);
        }
        #endregion

        #region 準備新增 Document 到資料庫的 Collection 內 (執行 100 次新增文件)
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
        #endregion

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

這篇文章為透過 C# 程式碼來存取與使用 MongoDB 資料庫的第一篇文章，就算你本地端沒有 MongoDB 資料庫，也可以透過這篇文章來練習如何使用 C# 程式碼來存取與使用 MongoDB 資料庫，因為可以使用免費的 [MongoDB Atlas](https://www.mongodb.com/zh-cn/atlas)，所以，建議可以先到 MongoDB 官方網站上，註冊一個帳號與申請使用 MongoDB Atlas 來試用。

這篇文章主要會先針對如何 CRUD (Create 新增、Retrive 查詢、Update 修改、Delete 刪除) 的四個資料基本操作動作的第一個，也就是新增，來進行說明，如何使用 C# 來做到這樣的需求。

由於 MongoDB 是一個文件基礎的 NoSQL 資料庫，因此，在此，先來建立一個類別，該類別就是要準備新增到 MongoDB 文件型別。

這個類別將會出現在練習程式碼的最上方，也就是類別名稱為 Blog 這個類別，該類別內隨意建立了五個屬性，在後續的程式中，將會隨機新增這些欄位的數值。

現在要來進入到該程式進入點方法，也就是 Main 方法，在這個方法的最前面，將會有兩行，使用 `Environment.GetEnvironmentVariable` 方法的敘述，這兩行敘述的目的是要取得 MongoDB Atlas 的帳號與密碼，這兩個值將會設定在環境變數內，這樣的作法是為了避免帳號與密碼寫在程式碼內，這樣的作法是不安全的，因此，這裡使用環境變數來取得這兩個值。

當在命令提示字元視窗內輸入了 set 命令，將會看到如下窗的輸出結果

![](../Images/X2023-9856.png)

在這個輸出結果內，可以看到，有兩個環境變數，分別是 `MongoDBAccount` 與 `MongoDBPassword`，這兩個環境變數的值，就是在 MongoDB Atlas 內設定的帳號與密碼。

若沒有看到這組帳號與密碼，可以在命令提示字元視窗內，輸入下列的兩個命令，來設定這兩個環境變數

```
setx MongoDBAccount "妳的帳號" /M

setx MongoDBPassword "你的密碼" /M
```

> 請將上述的 `妳的帳號` 與 `你的密碼` 替換成你在 MongoDB Atlas 內設定的帳號與密碼

有了這樣的安全設計，就可以放心將這個專案的原始碼上傳到 GitHub 上，而不用擔心帳號與密碼的外洩。

下一行 C# 敘述 `var mongoUri = $"mongodb+srv://{MongoDBAccount}:{MongoDBPassword}@vulcanmongo.hptf95d.mongodb.net/?retryWrites=true&w=majority";` 將會建立一個 MongoDB 的連線字串，這個字串將會用來連線到 MongoDB Atlas 上。其中連線用到的身分驗證用帳號與密碼，將會使用剛剛取得的環境變數內的值。

現在準備好相關連線到 MongoDB Atlas 資料庫用的資訊，可以建立一個 MongoDB Client 物件，這個物件將會用來連線到 MongoDB Atlas 上。使用 `new MongoClient(mongoUri)` 這個敘述，就可以建立一個 MongoDB Client 物件，並且連線到 MongoDB Atlas 上，所建立的物件將會儲存到 `client` 這個變數內，該變數的型別為 IMongoClient。

不過，若有網路設定、防火牆或安全設定的問題、帳號密碼不正確、連線主機位置不正確等問題，將會造成 `new MongoClient(mongoUri)` 表示式的執行失敗，這時，將會進入到 `catch` 區塊內，並且顯示錯誤訊息，並且結束程式的執行。

若沒有發生任何的錯誤，將來簡單嘗試 MongoDB.Driver 所提供的 API，接下來，將會使用 `client.ListDatabases()` 這個方法，來列出 MongoDB Atlas 內所有的資料庫名稱，並且將這些資料庫名稱顯示在命令提示字元視窗內。

現在可以開始來寫新增文件的程式碼，準備新增 Document 到資料庫的 Collection 內 ，這裡將會執行 100 次新增動作，每次新增一筆文件，總共要新增 100 筆文件。

在這個範例中，將會使用 `Stopwatch` 這個類別，來計算新增 100 筆文件所需要的時間，這個類別位於 `System.Diagnostics` 命名空間內，因此，需要先使用 `using System.Diagnostics;` 這個敘述，來引用這個命名空間。

另外，需要先取得該文件的 Collection 物件，這裡使用 `collection = client.GetDatabase(dbName).GetCollection<Blog>(collectionName)` 這個敘述來做到，若該 Collection 不存在於資料庫內，將會自動建立該 Collection。

有了 Collection 物件之後，就可以開始進行新增文件的動作，這裡將會使用 `collection.InsertOne(blog)` 這個方法，透過 100 次的迴圈來逐一新增一筆文件到 MongoDB 資料庫內，這個方法的參數是一個 Blog 物件，這個物件內的屬性值，將會被新增到 MongoDB 資料庫內。

所以，可以看到 NoSQL 資料庫的優勢，那就是無需事先規劃資料庫綱要 Schema 或者對資料庫作證規劃設計，就可以直接將資料新增到資料庫內，這樣的作法，可以讓開發人員可以快速的開發應用程式，而不用花費大量的時間在資料庫的規劃上。這樣的設計是直覺的，就是把程式中的物件直接儲存到資料庫內。

接下來要來事先透過迴圈來建立100文件物件，儲存到集合物件內，最後，僅需要呼叫一次 `collection.InsertMany(blogs)` 敘述，就可以一次把 100 筆文件寫入到 MongoDB 內了

## 執行程式，觀察結果

* 按下 `F5` 鍵，開始執行這個程式
* 請觀察 Console 視窗內的內容

![](../Images/X2023-9855.png)

## 切換使用 Docker 容器內的 MongoDB 資料庫

* 首先，確認具有 MongoDB 的容器已經啟動與執行了
* 打開命令提示字元視窗，輸入 `docker ps -a` 指定，將會看到底下輸出內容

```
docker ps -a
CONTAINER ID   IMAGE          COMMAND                   CREATED      STATUS          PORTS                      NAMES
ae8e218043c1   mongo:latest   "docker-entrypoint.s…"   4 days ago   Up 13 seconds   0.0.0.0:27017->27017/tcp   mongodbmapping
```

* 在狀態欄位中看到了 [Up 13 seconds] ，這表示了這個容器已經成功啟動了
* 接下來確認 [MyCrud] 資料庫是否存在
* 在命令提示字元視窗內輸入 `docker exec -it mongodbmapping mongosh` 指令，直接進入到 MongoDB Shell 環境內，將會看到底下輸出內容

```
docker exec -it mongodbmapping mongosh
Current Mongosh Log ID: 65517ae5abf0556ffc987849
Connecting to:          mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.0.1
Using MongoDB:          7.0.2
Using Mongosh:          2.0.1

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2023-11-13T01:18:33.159+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2023-11-13T01:18:33.159+00:00: /sys/kernel/mm/transparent_hugepage/enabled is 'always'. We suggest setting it to 'never'
   2023-11-13T01:18:33.159+00:00: vm.max_map_count is too low
------
```

* 輸入 `show dbs` 指令，查看 Docker 容器內的 MongoDB 內有那些資料庫存在，將會看到底下輸出內容

```
test> show dbs
MyCrud        8.00 KiB
admin        40.00 KiB
config       84.00 KiB
from_Oracle   6.34 GiB
local        72.00 KiB
```

* 若有看到 [MyCrud] 資料庫，表示已經成功的將資料庫建立起來了
* 使用 `use MyCrud` 指令，切換到 [MyCrud] 資料庫內，將會看到底下輸出內容

```
test> use MyCrud
switched to db MyCrud
```

* 使用 `show collections` 指令，查看 [MyCrud] 資料庫內有那些集合存在，將會看到底下輸出內容

```
MyCrud> show collections
Blog
```

* 若有看到 [Blog] 集合，表示已經成功的將集合建立起來了，不過，在此希望能夠從無到有的來進行開發，因此，輸入 `db.dropDatabase()` 命令將當前資料庫刪除掉，緊接著輸入 `show dbs` 查看 MongoDB 內可用的資料庫有哪些，將會看到底下輸出內容

```
MyCrud> db.dropDatabase()
{ ok: 1, dropped: 'MyCrud' }
MyCrud> show dbs
admin         40.00 KiB
config       108.00 KiB
from_Oracle    6.34 GiB
local         72.00 KiB
```

* 這時，可以看到 [MyCrud] 資料庫已經被刪除掉了
* 在 Visual Studio 2022 內找到 [Program.cs] 檔案，並且使用滑鼠雙擊這個檔案，將會開啟這個檔案
* 找到 `var mongoUri = $"..."` 這一行敘述，使用底下敘述將其替換

```csharp
var mongoUri = $"mongodb://localhost:27017/?retryWrites=true&w=majority";
```

* 此時，已經將連線字串修正成為指向到本機上的 Docker 容器內的 MongoDB 內
* 按下 [F5] 按鍵，重新執行這個程式，最後同樣會出現底下畫面內容

  ![](../Images/X2023-9854.png)

* 回到 MongoSh 視窗內，輸入 `show dbs` ，確認 [MyCrud] 資料庫已經成功建立出來
* 確認當前資料庫還是位於 [MyCrud] ，若指向資料庫位址不正確，請先輸入 `use MyCrud` 指令
* 接著輸入 `show collections` 指令，查看這個資料庫內是否有 [Blog] 這個 Collection 建立出來
* 最終在命令提示字元視窗內，將會看到如下面截圖

  ![](../Images/X2023-9853.png)



