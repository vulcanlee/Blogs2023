# .NET 8 MAUI 查看當前 BindingContext 使用的物件型別

![](../Images/X2023-9810.png)

在 .NET MAUI 開發環境中，透過 MVVM 設計模式與 XAML 資料綁定功能，可以將 ViewModel 與 View 進行綁定，這樣就可以在 ViewModel 內透過屬性與命令，來控制 View 的顯示與行為。不過，在使用 XAML 來進行資料綁定宣告與使用此功能時，有一個重要的觀念，就是 BindingContext，這個屬性可以讓 View 與 ViewModel 進行綁定，而且，BindingContext 屬性的型別，必須要與 View 的型別相同，否則，就無法進行綁定。

也就是說，當在 UI 控制項的屬性中，使用了標記延伸的資料綁定語法，例如：`{Binding Name}`，這個語法中的 `Name` 這個屬性，必須要存在於 BindingContext 的物件型別中，否則，就無法進行綁定。所以，了解到當前的 BindingContext 使用的物件型別，是件很重要的事情，因為這需要決定可以使用哪些屬性來進行資料綁定。

若沒有使用 [編譯的系結](https://learn.microsoft.com/zh-tw/dotnet/maui/fundamentals/data-binding/compiled-bindings?view=net-maui-8.0&WT.mc_id=DT-MVP-5002220) ，例如 `x:DataType` 這樣的語法，對於 `{Binding Name}` 這樣的資料綁定的宣告，需要等到執行時期的時候，才能夠知道資料綁定是否有成功，因為當前的 BindingContext 物件內，若沒有 Name 這個屬性，則會造成資料綁定失敗，但是， App 應用程式是不會閃退，但是會在螢幕上無法看到資料綁定對象的實際資料內容。

因此，若能夠在 .NET MAUI 專案設計過程中，使用 `x:DataType` 語法來指定編譯時期的資料綁定功能，在編譯時期就可以進行所宣告的資料綁定用法是否正確，指向的是否為所規劃的要綁定的對象，就變得相當重要，因為，這樣可以大幅提整個應用程式的開發速度與品質，不用甚麼事情都要等到執行時期才知道資料綁定的宣告用法是否正確。

對於 .NET MAUI 中的 XAML 內之每個頁面 Page 、版面配置 Layout、控制項/項目 Control / Element，都有個 BindingContext 屬性，而當在設計一個 XAML 文件的時候，其實就是在描述整體畫面的各個視覺控制項 VisualElement 之間的關係，這些關係將會使用呈現為一個樹狀結構的關係，也就是說，每個 頁面 Page 、版面配置 Layout、控制項/項目 Control / Element 都會有個父項目。

對於 BindingContext 這個屬性而言，它具有一個可以繼承屬性質的特性，也就是說，當在設計 XAML 文件的時候，可以在某個父項目上，設定 BindingContext 屬性，這樣，該父項目下的所有子項目，都會繼承該父項目的 BindingContext 屬性值，這樣，就可以讓所有子項目，都可以使用 BindingContext 屬性值所指定的物件型別，來進行資料綁定。當然，若在某個子項目想要強制覆蓋 BindingContext 屬性值，也是可以的。這樣的話，從這個子項目開始，就會使用新的 BindingContext 屬性值所指定的物件型別，來進行資料綁定。這樣的特性將會在 ListView 或者 CollectionView 這樣的控制項上，使用到 BindingContext 屬性時，變得相當重要。因為，對於要顯示的每個項目，其每個明細項目的 BindingContext 屬性值將會切換成為這個當時要顯示的明細物件所指定的物件型別，而不是這類集合物件當時所用的 BindingContext 物件。

在這裡將會透過一個範例來說明，當前的 BindingContext 使用的物件型別，是件很重要的事情，因為這需要決定可以使用哪些屬性來進行資料綁定。這裡將會使用編譯的綁定語法來進行說明。

## 建立 .NET 8 MAUI 專案

* 打開 Visual Studio 2022 IDE 應用程式
* 從 [Visual Studio 2022] 對話窗中，點選右下方的 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗右半部
  * 切換 [所有語言 (L)] 下拉選單控制項為 [C#]
  * 切換 [所有專案類型 (T)] 下拉選單控制項為 [MAUI]
* 在中間的專案範本清單中，找到並且點選 [.NET MAUI 應用程式] 專案範本選項
  > 此專案可用於建立適用於 iOS、Android、Mac Catalyst、Tizen 和 WinUI 的 .NET MAUI 應用程式。
* 點選右下角的 [下一步] 按鈕
* 在 [設定新的專案] 對話窗
* 找到 [專案名稱] 欄位，輸入 `MA10` 作為專案名稱
* 在剛剛輸入的 [專案名稱] 欄位下方，確認沒有勾選 [將解決方案與專案至於相同目錄中] 這個檢查盒控制項
* 點選右下角的 [下一步] 按鈕
* 現在將會看到 [其他資訊] 對話窗
* 在 [架構] 欄位中，請選擇最新的開發框架，這裡選擇的 [架構] 是 : `.NET 8.0 (長期支援)`
* 請點選右下角的 [建立] 按鈕

稍微等候一下，這個主控台專案將會建立完成

## 安裝要用到的 NuGet 開發套件

因為開發此專案時會用到這些 NuGet 套件，請依照底下說明，將需要用到的 NuGet 套件安裝起來。

### 安裝 CommunityToolkit.Mvvm 套件

CommunityToolkit.Mvvm 是微軟官方提供的 MVVM 套件，提供了一些 MVVM 開發常用的功能，例如：ObservableObject、ObservableProperty、RelayCommand 等等，這些功能在 WPF、UWP、Xamarin.Forms 都可以使用，而且在 .NET 8 MAUI 也可以使用。

請依照底下說明操作步驟，將這個套件安裝到專案內

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: MA10] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `CommunityToolkit.Mvvm`
* 稍待一會，將會在下方看到這個套件被搜尋出來
* 點選 [CommunityToolkit.Mvvm] 套件名稱
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內



