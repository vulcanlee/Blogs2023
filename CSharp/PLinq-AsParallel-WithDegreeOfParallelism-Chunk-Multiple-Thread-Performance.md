# 在 C# 內，透過 PLinq 的 AsParallel 方法並且使用 WithDegreeOfParallelism 指定平行度做到多執行緒與平行化處理剖析

![](../Images/X2023-9900.png)

延續上一篇 [在 C# 內，透過 PLinq 的 AsParallel 方法做到多執行緒與平行化處理剖析](https://csharpkh.blogspot.com/2023/09/PLinq-AsParallel-Chunk-Multiple-Thread-Performance.html) 文章，在這篇文章提到了想要使用 PLinq 的 AsParallel 語法，讓原先的 Linq 查詢由同步查詢，頓時轉化成為平行查詢；可是，往往事與願違，似乎沒有達到如預期的效果，因此，開發人員便化身成為 柯南 ，自己假設、想像各種理由，論述出一個讓自己與他人相信的講法，可是，許多情況卻不是這樣。

若開發者的電腦有八個邏輯處理器，並且使用 PLinq 語法做到平行查詢，此時，理論上這個查詢的速度應該可以達到八倍左右，因為，PLinq 會將原先的列舉所有項目，切割成為八個 Chunk，接著會由八個執行緒來分別做這八個 Chunk 項目的查詢工作。

這裡要注意到的是：
* 不是所有的 Linq 敘述改成平行查詢，執行速度都會大幅提升到你想像到的
* Linq 是屬於 CPU Bound 密集的計算作業，因此，若查詢過程用到大量 I/O Bound 的計算作業，建議可以採用其他的 [TPL 工作平行程式庫](https://learn.microsoft.com/zh-tw/dotnet/standard/parallel-programming/task-parallel-library-tpl?WT.mc_id=DT-MVP-5002220) 所提供的作法。
* PLinq 可以提供的最大平行度，預設將會取決於當時執行程式碼的這台電腦上的硬體架構。
* 經過平行查詢處理之後，所得到的集合項目順序，可能會有所不同。

在這篇文章中，將會探討 PLinq 平行度的問題，所謂平行計算中的平行度，將指的是可以同時執行的運算數量。平行計算的目標是利用多個處理器或核心來加快計算速度，因此平行度越高，計算速度就越快。

平行度可以分為以下幾個方面：

* 指令層級平行度：指的是在一個程式運行中，可以同時執行的指令數量。指令層級平行度可以通過流水線、超標量等技術來提高。
* 資料平行度：指的是可以同時處理的資料數量。資料平行度可以通過將資料分割成小塊，並分配給不同的處理器或核心來提高。
* 任務平行度：指的是可以同時執行的任務數量。任務平行度可以通過將任務分割成小塊，並分配給不同的處理器或核心來提高。

所以，接下來將會透過 PLinq 所提供的 [WithDegreeOfParallelism](https://learn.microsoft.com/zh-tw/dotnet/api/system.linq.parallelenumerable.withdegreeofparallelism?view=net-7.0&WT.mc_id=DT-MVP-5002220) 方法，觀察與了解在特定硬體架構下，選擇使用不同平行度將會有甚麼問題。在微軟官方文件中，對於 WithDegreeOfParallelism 的解釋為：設定於查詢中使用的平行處理原則程度。 平行處理原則的程度，就是可在處理查詢時同步執行的最大作業數目

## 建立 PLinq 與使用 WithDegreeOfParallelism 方法 測試專案

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
* 找到 [專案名稱] 欄位，輸入 `csWithDegreeOfParallelism` 作為專案名稱
* 在剛剛輸入的 [專案名稱] 欄位下方，確認沒有勾選 [將解決方案與專案至於相同目錄中] 這個檢查盒控制項
* 點選右下角的 [下一步] 按鈕
* 現在將會看到 [其他資訊] 對話窗
* 在 [架構] 欄位中，請選擇最新的開發框架，這裡選擇的 [架構] 是 : `.NET 7.0 (標準字詞支援)`
* 在這個練習中，需要去勾選 [不要使用最上層陳述式(T)] 這個檢查盒控制項
  > 這裡的這個操作，可以由讀者自行決定是否要勾選這個檢查盒控制項
* 請點選右下角的 [建立] 按鈕

稍微等候一下，這個主控台專案將會建立完成

## 撰寫測試用的程式碼

* 在此專案節點下，找到並且打開 [Program.cs] 這個檔案
* 使用底下 C# 程式碼替換掉 [Program.cs] 檔案內所有程式碼內容

```csharp
namespace csWithDegreeOfParallelism;

internal class Program
{
    static void Main(string[] args)
    {
        //ThreadPool.SetMinThreads(20, 20);
        var source = Enumerable.Range(1, 20);

        var evenNums = source.AsParallel()
            .WithDegreeOfParallelism(4)
            .Select(ShowInfo);

        Console.WriteLine($"Total {evenNums.Count()}");
    }

    static int ShowInfo(int n)
    {
        Console.WriteLine($"N={n:d6} {DateTime.Now:mm:ss} / Thread ID {Thread.CurrentThread.ManagedThreadId}");
        Thread.Sleep(5000);
        return n;
    }
}
```

在這裡將會使用 [Enumerable.Range(1, 20)] 方法，產生出 20 個整數數值列舉物件， source ，接著對這個列著列舉物件下達 AsParallel() 方法呼叫，讓這個 Linq 敘述，轉換成為要使用 PLinq 平行處理的計算方式。

緊接著使用了 WithDegreeOfParallelism(4) 方法，宣告這個 PLinq 查詢的平行度為 4

最後，使用了 Select(ShowInfo) 方法，產生出新的列舉值，不過，這裡所生成的列舉值，將會使用平行處理方式來處理。

對於 [ShowInfo] 這個方法，將會使用 Thread.Sleep(5000) 方法，模擬要產生 PDF 或者 HTML 檔案所要花費的時間，而在這個方法之前，將會列出所接收到的列舉項目的數值(整數值)，當時的時間，當前所用的執行緒 ID。

現在開始來觀察各種不同平行度下的查詢結果

## 在 4 個邏輯處理器下 與 WithDegreeOfParallelism(4) 的運行結果

在這裡將會準備了一台具有四個邏輯處理器的電腦，所有的實驗都會在這台電腦上來運行，這台電腦上僅安裝了最基本的 Windows 10 最新更新的作業系統，透過 Visual Studio 所開發出來的測試程式專案，將會透過發佈過程，取得所產生的獨立執行檔案，並且將這個檔案複製到這台四個邏輯處理器的電腦來執行。

將上述的專案採用底下的模式來發布

* 採用發佈到資料夾模式
* 部署模式設定為 獨立式
* 目標執行階段設定為 win-x64
* 檔案發行選項內要勾選 產生單一檔案 與 修剪未使用的程式碼
* 將發布後的兩個檔案複製到這台主機上

透過命令提示字元視窗來執行這個 csWithDegreeOfParallelism.exe 程式，將會看到底下輸出結果

```
C:\Vulcan\win-x64>csWithDegreeOfParallelism.exe
N=000003 52:42 / Thread ID 7
N=000004 52:42 / Thread ID 6
N=000001 52:42 / Thread ID 1
N=000002 52:42 / Thread ID 4
N=000005 52:47 / Thread ID 7
N=000007 52:47 / Thread ID 1
N=000006 52:47 / Thread ID 4
N=000008 52:47 / Thread ID 6
N=000009 52:52 / Thread ID 7
N=000010 52:52 / Thread ID 6
N=000012 52:52 / Thread ID 1
N=000011 52:52 / Thread ID 4
N=000013 52:57 / Thread ID 7
N=000014 52:57 / Thread ID 4
N=000016 52:57 / Thread ID 6
N=000015 52:57 / Thread ID 1
N=000017 53:02 / Thread ID 7
N=000018 53:02 / Thread ID 4
N=000020 53:02 / Thread ID 1
N=000019 53:02 / Thread ID 6
Total 20
```

從執行結果得到底下結論
* 因為在 PLinq 語法中，有使用 WithDegreeOfParallelism 方法來指定平行度為 4，因此，PLinq 在執行的時候將會把 20 個項目切割成為 4 個 Chunk
* 將會有 4 個執行緒分別來處理不同 Chunk 內的項目物件
* 由於這台主機具備 4 個邏輯處理器，若沒有使用 WithDegreeOfParallelism 方法，所得到的執行結果也是相同的
* 在這個列舉物件內的 20 個整數項目中，分別由四個執行緒，在 52:42 , 52:47 , 52:52 , 52:57 , 53:02 五個階段，進行查詢作業
* 由於透過 4 個執行緒平行執行，可以讓原先需要耗費 20 * 5 (秒) = 100 秒的工作，改善成為 5 * 5 (秒) = 25 秒，使得整體執行速度提升了四倍

## 在 4 個邏輯處理器下 與 WithDegreeOfParallelism(2) 的運行結果

將此專案原始碼的 WithDegreeOfParallelism 方法修改成為 WithDegreeOfParallelism(2)

請重新發佈此專案，並將生成最終執行檔案複製到測試用的主機上

透過命令提示字元視窗來執行這個 csWithDegreeOfParallelism.exe 程式，將會看到底下輸出結果

```
C:\Vulcan\win-x64>csWithDegreeOfParallelism.exe
N=000002 56:08 / Thread ID 4
N=000001 56:08 / Thread ID 1
N=000003 56:13 / Thread ID 1
N=000004 56:13 / Thread ID 4
N=000005 56:18 / Thread ID 4
N=000006 56:18 / Thread ID 1
N=000007 56:23 / Thread ID 4
N=000008 56:23 / Thread ID 1
N=000009 56:28 / Thread ID 4
N=000010 56:28 / Thread ID 1
N=000011 56:33 / Thread ID 4
N=000012 56:33 / Thread ID 1
N=000013 56:38 / Thread ID 4
N=000014 56:38 / Thread ID 1
N=000015 56:43 / Thread ID 4
N=000016 56:43 / Thread ID 1
N=000017 56:48 / Thread ID 4
N=000019 56:48 / Thread ID 1
N=000018 56:53 / Thread ID 4
N=000020 56:53 / Thread ID 1
Total 20
```

從執行結果得到底下結論
* 此時，這個 PLinq 將會採用平行度為 2 的方式來執行，也就是說，將會使用兩個執行緒來執行這次的查詢作業
* 由於這台執行的主機具備有 4 個邏輯處理器，因此，當平行度低於這台主機的所有邏輯處理器數量的時候，沒有太多問題發生，這是因為每個要進行查詢的後，將會使用所指定平行度數量的執行緒來平行運算。
* 從底下執行結果可以看出，在這個列舉物件內的 20 個整數項目中，分別由 2 個執行緒，在 56:08 , 56:13 , 56:18 , 56:23 , 56:28 , 56:33 , 56:38 , 56:43 , 56:48 , 56:53 10 個階段，進行查詢作業
* 由於透過 2 個執行緒平行執行，可以讓原先需要耗費 20 * 5 (秒) = 100 秒的工作，改善成為 10 * 5 (秒) = 50 秒，使得整體執行速度提升了 2 倍

## 在 4 個邏輯處理器下 與 WithDegreeOfParallelism(8) 的運行結果

將此專案原始碼的 WithDegreeOfParallelism 方法修改成為 WithDegreeOfParallelism(8)

此時，這個 PLinq 將會採用平行度為 8 的方式來執行，也就是說，將會使用 8 個執行緒來執行這次的查詢作業(問題真的是這樣嗎?和你想像的相同的嗎?)

請重新發佈此專案，並將生成最終執行檔案複製到測試用的主機上

透過命令提示字元視窗來執行這個 csWithDegreeOfParallelism.exe 程式，將會看到底下輸出結果

```
C:\Vulcan\win-x64>csWithDegreeOfParallelism.exe
N=000003 58:20 / Thread ID 8
N=000002 58:20 / Thread ID 4
N=000004 58:20 / Thread ID 7
N=000005 58:20 / Thread ID 6
N=000001 58:20 / Thread ID 1
N=000006 58:20 / Thread ID 9
N=000007 58:21 / Thread ID 10
N=000008 58:22 / Thread ID 11
N=000012 58:25 / Thread ID 8
N=000010 58:25 / Thread ID 7
N=000011 58:25 / Thread ID 6
N=000013 58:25 / Thread ID 4
N=000009 58:25 / Thread ID 1
N=000014 58:25 / Thread ID 9
N=000015 58:26 / Thread ID 10
N=000016 58:27 / Thread ID 11
N=000018 58:30 / Thread ID 6
N=000019 58:30 / Thread ID 4
N=000020 58:30 / Thread ID 7
N=000017 58:30 / Thread ID 1
Total 20
```

從執行結果得到底下結論
* 此時，這個 PLinq 將會採用平行度為 8 的方式來執行，也就是說，將會使用 8 個執行緒來執行這次的查詢作業
* 當在使用 PLinq 進行平行查詢的時候，需要用到的額外執行緒，將會透過 [執行緒集區](https://learn.microsoft.com/zh-tw/dotnet/standard/threading/the-managed-thread-pool?WT.mc_id=DT-MVP-5002220) 來取得
* 由於這台執行的主機具備有 4 個邏輯處理器，因此，當有需要透過執行緒集區取得執行緒的時候，只要在 4 個執行緒內，執行緒集區會立刻提供給應用程式來使用，反之，若需要的執行緒已經超過 4 個以上，此時，執行緒集區將會採用執行緒注入的方式來提供可用的執行緒物件
* 執行緒集區注入執行緒的時間，在每個 .NET Runtime 皆會有可能有些差異，原則上，將會落在 0.5 秒到 1 秒之間才會注入一個執行緒到執行緒集區內
* 當平行度低於這台主機的所有邏輯處理器數量的時候，沒有太多問題發生，在這裡的測試情況是，指定的平行度將會大於台主機的所有邏輯處理器數量，從執行結果輸出文字，可以看到執行緒注入的情況發生
* 從底下執行結果可以看出，在 58:20 這個時間點已經有 6 個執行緒用於平行查詢之用(為什麼一開始不是 4 個執行緒呢？關於這點，留給讀者來自行思考?)
* 在 58:21 , 58:22 這兩個時間點，又分別注入了兩個執行緒到執行緒集區內
* 在第一次啟動這個平行查詢之後的五秒後，這些平行查詢已經都完成了，所以，可以看到在 58:25 這個時間點又有六個執行緒(之前完成查詢的執行緒，因為用完了，就會歸還給執行緒集區，而此時又觸發需要執行緒，所以，就又從執行緒集區取得這六個執行緒來作為查詢計算之用)
* 同理，在 58:26 , 58:27 這兩個時間點，因為之前的執行緒執行完成了(已經過了五秒)，這兩個執行緒用完後就會歸還給執行緒集區，因此，當又需要透過執行緒集區取得執行緒的時後，就不再需要透過注入的方式取得，而是直接使用在執行緒集區內快取的執行緒來使用
* 原則上，執行緒集區內的快取執行緒，大約會保留約 20~30 秒左右
* 由於透過平行度為 8 來平行執行查詢，原本期望可以在 20 / 8 = 2.5 ，約 5(秒) * 3 = 15 秒內完成所有的查詢工作，可是，事實上好像不是這樣

## 在 4 個邏輯處理器下 與 WithDegreeOfParallelism(20) 是否有機會在 5 秒內完成查詢計算呢?

將此專案原始碼的 WithDegreeOfParallelism 方法修改成為 WithDegreeOfParallelism(20)

找到 ThreadPool.SetMinThreads(20, 20); 敘述，將其解除註解

請重新發佈此專案，並將生成最終執行檔案複製到測試用的主機上

透過命令提示字元視窗來執行這個 csWithDegreeOfParallelism.exe 程式，將會看到底下輸出結果

```
C:\Vulcan\win-x64>csWithDegreeOfParallelism.exe
N=000016 01:11 / Thread ID 19
N=000020 01:11 / Thread ID 23
N=000008 01:11 / Thread ID 11
N=000005 01:11 / Thread ID 8
N=000011 01:11 / Thread ID 14
N=000010 01:11 / Thread ID 13
N=000002 01:11 / Thread ID 4
N=000017 01:11 / Thread ID 20
N=000006 01:11 / Thread ID 9
N=000001 01:11 / Thread ID 1
N=000003 01:11 / Thread ID 6
N=000007 01:11 / Thread ID 10
N=000018 01:11 / Thread ID 21
N=000009 01:11 / Thread ID 12
N=000019 01:11 / Thread ID 22
N=000012 01:11 / Thread ID 15
N=000015 01:11 / Thread ID 18
N=000013 01:11 / Thread ID 16
N=000004 01:11 / Thread ID 7
N=000014 01:11 / Thread ID 17
Total 20
```

從執行結果得到底下結論
* 首先 ThreadPool.SetMinThreads(20, 20) 將會宣告執行緒集區對於前 20 個執行緒使用請求，可以直接提供，不需要使用爬山理論的方式來注入執行緒。
* 此時，這個 PLinq 將會採用平行度為 20 的方式來執行，也就是說，將會使用 20 個執行緒來執行這次的查詢作業，不管當時的硬體 CPU 邏輯處理器核心數量有多少
* 當然，這樣強制修改執行緒集區預設最小提供執行緒數量的作法，有好處也會其他的副作用，這個時候需要由當前開發者自己決定
* 透過實際執行結果可以看到，在次這 PLinq 查詢，因為平行度為 20，執行緒集區最小可用執行緒數量為 20，並且這個列舉長度也為 20
* 所以，在執行 Linq 查詢的時候，就會有 20 個執行緒出現，全部執行時間就是 5 秒鐘


