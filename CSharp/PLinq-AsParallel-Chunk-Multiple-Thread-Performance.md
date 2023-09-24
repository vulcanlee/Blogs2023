# 在 C# 內，透過 PLinq 的 AsParallel 方法做到多執行緒與平行化處理剖析

```csharp
namespace csAsParallel
{
    internal class Program
    {
        static void Main(string[] args)
        {
            var source = Enumerable.Range(1, 10000);

            // Opt in to PLINQ with AsParallel.
            var evenNums = from num in source.AsParallel()
                           where num % 2 == 0
                           select ShowInfo(num);

            Console.WriteLine($"Total {evenNums.Count()}");
        }

        static int ShowInfo(int n)
        {
            Console.WriteLine($" {DateTime.Now:mm:ss} - {n} / Thread ID {Thread.CurrentThread.ManagedThreadId}");
            Thread.Sleep(5000);
            return n;
        }
    }  
}
```

## 在 8 個邏輯處理器下的運行結果

![](../Images/X2023-9911.png)

```
C:\Vulcan\win-x64>csAsParallel.exe
N=000001 20:48 - 1 / Thread ID 1
N=000008 20:48 - 8 / Thread ID 11
N=000006 20:48 - 6 / Thread ID 9
N=000005 20:48 - 5 / Thread ID 8
N=000007 20:48 - 7 / Thread ID 10
N=000002 20:48 - 2 / Thread ID 4
N=000004 20:48 - 4 / Thread ID 7
N=000003 20:48 - 3 / Thread ID 6
N=000009 20:53 - 9 / Thread ID 4
N=000012 20:53 - 12 / Thread ID 8
N=000010 20:53 - 10 / Thread ID 6
N=000013 20:53 - 13 / Thread ID 1
N=000011 20:53 - 11 / Thread ID 7
N=000014 20:53 - 14 / Thread ID 10
N=000015 20:53 - 15 / Thread ID 9
N=000016 20:53 - 16 / Thread ID 11
N=000017 20:58 - 17 / Thread ID 11
N=000024 20:58 - 24 / Thread ID 8
N=000019 20:58 - 19 / Thread ID 9
N=000020 20:58 - 20 / Thread ID 1
N=000021 20:58 - 21 / Thread ID 4
N=000022 20:58 - 22 / Thread ID 10
N=000023 20:58 - 23 / Thread ID 7
N=000018 20:58 - 18 / Thread ID 6
^C
C:\Vulcan\win-x64>
```

## 在 4 個邏輯處理器下的運行結果

![](../Images/X2023-9910.png)

```
C:\Vulcan\win-x64>csAsParallel.exe
N=000003 18:45 - 3 / Thread ID 4
N=000001 18:45 - 1 / Thread ID 1
N=000002 18:45 - 2 / Thread ID 7
N=000004 18:45 - 4 / Thread ID 6
N=000006 18:50 - 6 / Thread ID 4
N=000007 18:50 - 7 / Thread ID 1
N=000005 18:50 - 5 / Thread ID 7
N=000008 18:50 - 8 / Thread ID 6
N=000012 18:55 - 12 / Thread ID 7
N=000011 18:55 - 11 / Thread ID 1
N=000009 18:55 - 9 / Thread ID 6
N=000010 18:55 - 10 / Thread ID 4
N=000016 19:00 - 16 / Thread ID 7
N=000013 19:00 - 13 / Thread ID 1
N=000015 19:00 - 15 / Thread ID 4
N=000014 19:00 - 14 / Thread ID 6
N=000018 19:05 - 18 / Thread ID 4
N=000019 19:05 - 19 / Thread ID 1
N=000017 19:05 - 17 / Thread ID 6
N=000020 19:05 - 20 / Thread ID 7
^C
C:\Vulcan\win-x64>
```

## 在 2 個邏輯處理器下的運行結果

![](../Images/X2023-9909.png)

```
C:\Vulcan\win-x64>csAsParallel.exe
N=000001 33:43 - 1 / Thread ID 1
N=000002 33:43 - 2 / Thread ID 4
N=000003 33:48 - 3 / Thread ID 1
N=000004 33:48 - 4 / Thread ID 4
N=000005 33:53 - 5 / Thread ID 1
N=000006 33:53 - 6 / Thread ID 4
N=000007 33:58 - 7 / Thread ID 1
N=000008 33:58 - 8 / Thread ID 4
N=000009 34:03 - 9 / Thread ID 1
N=000010 34:03 - 10 / Thread ID 4
N=000011 34:08 - 11 / Thread ID 1
N=000012 34:08 - 12 / Thread ID 4
^C
C:\Vulcan\win-x64>
```

## 在 1 個邏輯處理器下的運行結果

![](../Images/X2023-9908.png)

```
C:\Vulcan\win-x64>csAsParallel.exe
N=000001 39:05 - 1 / Thread ID 1
N=000002 39:10 - 2 / Thread ID 1
N=000003 39:15 - 3 / Thread ID 1
N=000004 39:20 - 4 / Thread ID 1
N=000005 39:25 - 5 / Thread ID 1
N=000006 39:30 - 6 / Thread ID 1
N=000007 39:35 - 7 / Thread ID 1
^C
C:\Vulcan\win-x64>
```
