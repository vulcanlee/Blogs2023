# 使用 Entity Framework 卻發生了連線池區 Connection Pool 已經超過最大大小的限制，導致系統噴出例外異常

這是一個滿有趣的問題，也許是我寫的專案並沒有很大量的人在使用，所以，我幾乎沒有機會遇到如題目上的問題，也就是說，當取得一個 DbContext 物件的時候，當需要進行資料庫存取的時候，所使用 資料庫連線 Database Connection 資訊，其實是透過一個 連線集區 Connection Pool 來管理，因此，將會透過此集區來取的一個資料庫連線物件來使用，透過此資料庫連線物件，便可以對遠端資料庫系統進行資料存取操作，一旦對資料庫的存取完成之後，將會把這個資料庫連線物件歸還給連線集區內，如此可以達到重複使用的目的。

在一個偶然機會，聽到同事遇到一個狀況，那就是系統回報資料庫連線集區耗盡，因為時間逾期，最終結果將會造成了底下的例外異常的錯誤訊息；在還沒有寫這篇文章之前，我還真的是百思不得其解，基於好奇心又或者求知慾，讓我想要來做些實驗，究竟在什麼情況下，也就是做了甚麼樣的操作，會造成有這樣的 [The timeout period elapsed prior to obtaining a connection from the pool] 錯誤訊息發生，一旦有了這樣的知識之後，我在日後寫有關於資料庫存取的程式碼時候，將會積極、努力的來避免這樣的問題發生。

```
System.InvalidOperationException
  HResult=0x80131509
  Message=Timeout expired.  The timeout period elapsed prior to obtaining a connection from the pool.  This may have occurred because all pooled connections were in use and max pool size was reached.
  Source=Microsoft.Data.SqlClient
  StackTrace: 
   於 Microsoft.Data.ProviderBase.DbConnectionFactory.TryGetConnection(DbConnection owningConnection, TaskCompletionSource`1 retry, DbConnectionOptions userOptions, DbConnectionInternal oldConnection, DbConnectionInternal& connection)
   於 Microsoft.Data.ProviderBase.DbConnectionInternal.TryOpenConnectionInternal(DbConnection outerConnection, DbConnectionFactory connectionFactory, TaskCompletionSource`1 retry, DbConnectionOptions userOptions)
   於 Microsoft.Data.ProviderBase.DbConnectionClosed.TryOpenConnection(DbConnection outerConnection, DbConnectionFactory connectionFactory, TaskCompletionSource`1 retry, DbConnectionOptions userOptions)
   於 Microsoft.Data.SqlClient.SqlConnection.TryOpen(TaskCompletionSource`1 retry, SqlConnectionOverrides overrides)
   於 Microsoft.Data.SqlClient.SqlConnection.Open(SqlConnectionOverrides overrides)
   於 Microsoft.Data.SqlClient.SqlConnection.Open()
   於 EFCoreUsingDbContextPoolSize.Program.<>c__DisplayClass0_1.<<Main>b__0>d.MoveNext() 在 C:\Vulcan\Github\CSharp2023\EFCoreUsingDbContextPoolSize\EFCoreUsingDbContextPoolSize\Program.cs:行 83 中
   於 EFCoreUsingDbContextPoolSize.Program.<Main>d__0.MoveNext() 在 C:\Vulcan\Github\CSharp2023\EFCoreUsingDbContextPoolSize\EFCoreUsingDbContextPoolSize\Program.cs:行 112 中
```

## 建立測試用的專案

為了簡化測試用專案的複雜度，因此，在這裡將會建立一個 Console 主控台應用類型的專案。

* 打開 Visual Studio 2022 IDE 應用程式
* 從 [Visual Studio 2022] 對話窗中，點選右下方的 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗右半部
  * 切換 [所有語言 (L)] 下拉選單控制項為 [C#]
  * 切換 [所有專案類型 (T)] 下拉選單控制項為 [主控台]
* 在中間的專案範本清單中，找到並且點選 [主控台應用程式] 專案範本選項
  > 專案，用於建立可在 Windows、Linux 及 macOS 於 .NET 執行的命令列應用程式
* 點選右下角的 [下一步] 按鈕
* 在 [設定新的專案] 對話窗
* 找到 [專案名稱] 欄位，輸入 `EFCoreUsingDbContextPoolSize` 作為專案名稱
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

### 安裝 Microsoft.EntityFrameworkCore 套件

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: EFCoreUsingDbContextPoolSize] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `Microsoft.EntityFrameworkCore`
* 稍待一會，將會在下方看到這個套件被搜尋出來
* 點選 [Microsoft.EntityFrameworkCore] 套件名稱
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

### 安裝 Microsoft.EntityFrameworkCore.SqlServer 套件

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: EFCoreUsingDbContextPoolSize] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `Microsoft.EntityFrameworkCore.SqlServer`
* 稍待一會，將會在下方看到這個套件被搜尋出來
* 點選 [Microsoft.EntityFrameworkCore.SqlServer] 套件名稱
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

## 建立需要用到的資料項目模型

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 節點
* 從彈出視窗中，依序點選 [加入] > [類別] 功能表選項
* 此時，[新增項目] 對話窗將會出現
* 在此對話窗的最下方 [名稱] 欄位內輸入 `Student.cs` 這個類別名稱
  > 請確認該對話窗中間的清單，已經確實選擇了 [類別] 這個項目
* 點選此對話窗右下方的 [新增] 按鈕
* 確認 [Student.cs] 這個視窗出現在 Visual Studio 2022 中
* 使用底下 C# 程式碼替換掉剛剛產生的類別程式碼內容

```csharp
namespace EFCoreUsingDbContextPoolSize;

public class Student
{
    public int StudentId { get; set; }
    public string Name { get; set; }
}
```

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 節點
* 從彈出視窗中，依序點選 [加入] > [類別] 功能表選項
* 此時，[新增項目] 對話窗將會出現
* 在此對話窗的最下方 [名稱] 欄位內輸入 `Course.cs` 這個類別名稱
  > 請確認該對話窗中間的清單，已經確實選擇了 [類別] 這個項目
* 點選此對話窗右下方的 [新增] 按鈕
* 確認 [Course.cs] 這個視窗出現在 Visual Studio 2022 中
* 使用底下 C# 程式碼替換掉剛剛產生的類別程式碼內容

```csharp
namespace EFCoreUsingDbContextPoolSize;

public class Course
{
    public int CourseId { get; set; }
    public string CourseName { get; set; }
}
```

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 節點
* 從彈出視窗中，依序點選 [加入] > [類別] 功能表選項
* 此時，[新增項目] 對話窗將會出現
* 在此對話窗的最下方 [名稱] 欄位內輸入 `CouSchoolContextrse.cs` 這個類別名稱
  > 請確認該對話窗中間的清單，已經確實選擇了 [類別] 這個項目
* 點選此對話窗右下方的 [新增] 按鈕
* 確認 [SchoolContext.cs] 這個視窗出現在 Visual Studio 2022 中
* 使用底下 C# 程式碼替換掉剛剛產生的類別程式碼內容

```csharp
using Microsoft.EntityFrameworkCore;

namespace EFCoreUsingDbContextPoolSize;

public class SchoolContext : DbContext
{
    public SchoolContext(DbContextOptions<SchoolContext> options)
    : base(options)
    {
    }

    public DbSet<Student> Students { get; set; }
    public DbSet<Course> Courses { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        //optionsBuilder.UseSqlServer("Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=TestPool;MultipleActiveResultSets=True");
    }
}
```

## 修正主程序的程式碼

* 在此專案節點下，找到並且打開 [Program.cs] 這個檔案
* 使用底下 C# 程式碼替換掉 [Program.cs] 檔案內所有程式碼內容

```csharp
using Microsoft.EntityFrameworkCore;

namespace EFCoreUsingDbContextPoolSize;

/// <summary>
/// 模擬 DbContext 的 Connection Pool 超出使用量，造成例外異常的情況
/// </summary>
internal class Program
{
    static async Task Main(string[] args)
    {
        // 設定資料庫連線的最大連線數量
        int MaxPoolSize = 4;
        // 設定資料庫連線的最大等待時間
        int ConnectTimeout = 5;
        bool UsingEFCoreQuery = true;

        #region 準備進行資料庫連線或者建立資料庫
        // 連線字串
        string connectionString = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=TestPool;" +
            $"Max Pool Size={MaxPoolSize};Connect Timeout={ConnectTimeout};MultipleActiveResultSets=True";
        // 建立資料庫連線
        DbContextOptions<SchoolContext> options =
            new DbContextOptionsBuilder<SchoolContext>()
            .UseSqlServer(connectionString)
            .Options;

        using (var context = new SchoolContext(options))
        {
            // 檢查資料庫是否存在
            if (context.Database.CanConnect() == false)
            {
                // 建立資料庫
                await context.Database.EnsureCreatedAsync();

                context.Database.GetDbConnection();
                var connection = context.Database.GetDbConnection();
            }
        }
        #endregion

        #region 模擬建立多個 DbContext 來進行資料庫存取，故意造成 Connection Pool 被耗盡的情況

        List<Task> tasks = new List<Task>();

        for (int i = 0; i < 10; i++)
        {
            // 建立 10 個 DbContext 來進行資料庫存取
            int idx = i;
            Task task = Task.Run(async () =>
            {
                using (var context = new SchoolContext(options))
                {
                    if (UsingEFCoreQuery)
                    {
                        #region 使用 EF Core 的 SqlQuery 來進行查詢
                        Console.WriteLine($"第{idx}工作已經開始 " + DateTime.Now.ToString("mm:ss"));

                        var result = context.Database
                        .SqlQuery<string>($"SELECT GETDATE() AS D");
                        Console.WriteLine($"第{idx}工作準備休息 " + DateTime.Now.ToString("mm:ss"));
                        // 這裡休息多久，都不會有例外異常發生，因為，執行完 SQL 命令，連線就立即關閉了
                        Thread.Sleep(5000);
                        Console.WriteLine($"第{idx}工作正要結束 " + DateTime.Now.ToString("mm:ss"));

                        //var std = new Student()
                        //{
                        //    Name = "Bill"
                        //};
                        //context.Students.Add(std);
                        //context.SaveChanges();
                        #endregion
                    }
                    else
                    {
                        #region 使用 DbConnection 來存取資料庫，但是故意延遲關閉連線時間
                        using (var cn = context.Database.GetDbConnection())
                        {
                            Console.WriteLine($"第{idx}工作開啟連線 " + DateTime.Now.ToString("mm:ss"));
                            cn.Open();
                            // 關閉前休息3秒，並註解底下四行，同樣會造成例外異常
                            var cmd = cn.CreateCommand();
                            cmd.CommandText = "SELECT GETDATE() AS D";
                            var dr = cmd.ExecuteReader();
                            dr.Read();
                            Console.WriteLine($"第{idx}工作準備休息 " + DateTime.Now.ToString("mm:ss"));

                            // 這裡若將休息時間改為 3000 (3秒)，將會造成底下錯誤
                            // -----------------------------------------------------
                            // System.InvalidOperationException: 'Timeout expired.
                            // The timeout period elapsed prior to obtaining a connection
                            // from the pool.  This may have occurred because
                            // all pooled connections were in use and max pool size was reached.'
                            Thread.Sleep(3000);
                            // 這裡換成 await Task.Delay 同樣無效，會造成例外異常
                            //await Task.Delay(3000);

                            Console.WriteLine($"第{idx}工作正要結束 " + DateTime.Now.ToString("mm:ss"));
                            cn.Close();
                        }
                        #endregion
                    }
                }
            });
            tasks.Add(task);
        }
        #endregion

        await Task.WhenAll(tasks);

        Console.WriteLine("Press any key for continuing...");
        Console.ReadKey();
    }
}
```

在這個進入點程式碼內，將會宣告三個物件： MaxPoolSize, ConnectTimeout, UsingEFCoreQuery

* MaxPoolSize

  MaxPoolSize是連線集區的一個屬性，用於指定連線集區中可以容納的最大連接數。連線集區是一種性能優化技術，用於重用已建立的資料庫連接，以減少建立新連接所需的時間和資源。 

  當應用程式需要與資料庫建立連接時，它會首先檢查連線集區中是否有可用的連接。如果有可用連接，應用程式將重用該連接，而不是建立新的連接。如果連線集區已滿（即達到 MaxPoolSize），則應用程式必須等待，直到連線集區中有連接可用。 

  MaxPoolSize的預設值通常為 100，但可以根據應用程式的需求進行調整。將 MaxPoolSize 設置得太小可能會導致應用程式在高負載情況下性能下降，因為需要等待連接可用；將其設置得太大可能會導致資源浪費，因為可能無法充分利用所有連接。

* ConnectTimeout

  設定資料庫連線的最大等待時間，這個物件值將會宣告在連線字串內，其中，連線字串中的Connect Timeout單位是秒（seconds）

  在C#中，如果在連線字串中將 ConnectTimeout 設置為5，則表示連接操作將在5秒後超時。如果在5秒內無法建立連接，操作將被終止，並且通常會拋出一個異常，例如SqlException或其他相應的異常，具體取決於您使用的資料庫類型。這樣可以防止程式無限期地等待無法建立的連接。

  若在連線字串內沒有指定 ConnectTimeout ，且使用 MSSQL Server 時，若在連線字串中沒有指定Connect Timeout，預設值為30秒。這表示如果在30秒內無法建立連接，操作將被終止。

* UsingEFCoreQuery

  這個布林物件將會用於控制要採用 Entity Framework 的方式來存取資料庫紀錄，還是要使用 DbConnection 來存取資料庫，這裡會需要手動 Open 與 Close 這次的資料庫連線，用來模擬與分析不同的作法差別在哪裡。
  
接下來就是要宣告連線字串文字到 connectionString 物件內，這裡透過 MaxPoolSize & ConnectTimeout 兩個數值，宣告這個連線字串的連線物件執行特性。


        // 設定資料庫連線的最大連線數量
        int MaxPoolSize = 4;
        // 設定資料庫連線的最大等待時間
        int ConnectTimeout = 5;
        bool UsingEFCoreQuery = true;

        #region 準備進行資料庫連線或者建立資料庫
        // 連線字串
        string connectionString = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=TestPool;" +
            $"Max Pool Size={MaxPoolSize};Connect Timeout={ConnectTimeout};MultipleActiveResultSets=True";
        // 建立資料庫連線
        DbContextOptions<SchoolContext> options =
            new DbContextOptionsBuilder<SchoolContext>()
            .UseSqlServer(connectionString)
            .Options;



