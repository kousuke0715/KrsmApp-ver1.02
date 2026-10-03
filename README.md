# ヘアーサロンキリシマ予約アプリ

SwiftUI + SwiftDataを使用して作成した、ヘアーサロン向けの予約管理アプリです。

利用者は、

- 予約日を選択
- 予約時間を選択
- メニューを選択
- 名前を入力
- 予約を保存

できます。

店舗管理者は、

- 管理者ログイン
- 定休日の設定
- 始業時間・終業時間の設定
- 日付ごとの予約検索
- 予約内容の確認
- 予約削除

ができます。

---

# 使用技術

```text
Swift
SwiftUI
SwiftData
Calendar
Date
NavigationStack
```

主にSwiftUIで画面を構築し、SwiftDataで予約情報や店舗設定を永続化しています。

---

# アプリ全体の流れ

```text
KrsmaApp
   ↓
SelectView
   │
   ├── お客様
   │      ↓
   │   ContentView
   │      ↓
   │   DetailHourMinView
   │      ↓
   │   CutDetailSelectView
   │      ↓
   │   SwiftDataへ予約保存
   │
   └── 店舗管理者
          ↓
       MusterPassView
          ↓
       MusterView
          ↓
       SettingsEditView
```

---

# 1. アプリの開始地点

```swift
import SwiftUI
import SwiftData

@main
struct KrsmaApp: App {

    var body: some Scene {

        WindowGroup {
            SelectView()
        }

        .modelContainer(
            for: [
                Reservation.self,
                BusinessSettings.self
            ]
        )
    }
}
```

## `@main`

```swift
@main
```

は、

> このstructからアプリを起動する

という意味です。

つまり、

```swift
KrsmaApp
```

がこのアプリのスタート地点です。

---

## `WindowGroup`

```swift
WindowGroup {
    SelectView()
}
```

アプリ起動時に最初に表示する画面を指定しています。

今回は、

```swift
SelectView()
```

を最初に表示します。

---

# `.modelContainer`

```swift
.modelContainer(
    for: [
        Reservation.self,
        BusinessSettings.self
    ]
)
```

SwiftDataで保存するモデルを登録しています。

今回保存するデータは、

```text
Reservation
↓
予約情報

BusinessSettings
↓
店舗設定
```

の2種類です。

---

# 2. Reservation

予約情報を保存するモデルです。

```swift
@Model
final class Reservation {

    var dateTime: Date

    var name: String

    var service: String

    var durationMinutes: Int = 30

    init(
        dateTime: Date,
        name: String,
        service: String,
        durationMinutes: Int = 30
    ) {

        self.dateTime = dateTime

        self.name = name

        self.service = service

        self.durationMinutes = durationMinutes
    }
}
```

---

## `@Model`

```swift
@Model
```

はSwiftDataの機能です。

これを付けることで、

```swift
Reservation
```

をSwiftDataへ保存できるようになります。

例えば、

```swift
let reservation = Reservation(
    dateTime: Date(),
    name: "田中",
    service: "調髪",
    durationMinutes: 30
)
```

を作って、

```swift
modelContext.insert(reservation)
```

すると保存できます。

---

# `final class`

```swift
final class Reservation
```

の

```swift
final
```

は、

> このクラスを継承できないようにする

という意味です。

普通の、

```swift
class Reservation {
}
```

なら、

```swift
class SpecialReservation: Reservation {
}
```

のように継承できます。

しかし、

```swift
final class Reservation {
}
```

の場合、

```swift
class SpecialReservation: Reservation {
}
```

はできません。

---

## 重要

```text
@Model
↓
SwiftDataで保存するため

final
↓
継承できなくする

class
↓
クラスを作る
```

です。

つまり、

```swift
@Model
final class Reservation
```

は、

> SwiftDataに保存できるReservationクラスを作り、このクラスは継承させない

という意味です。

`final`と永続化は関係ありません。

---

# Reservationが持っているデータ

```swift
var dateTime: Date
```

予約日時です。

例：

```text
2026/10/10 10:30
```

---

```swift
var name: String
```

予約した人の名前です。

例：

```text
田中太郎
```

---

```swift
var service: String
```

選択したメニューです。

例：

```text
調髪：4400円
```

---

```swift
var durationMinutes: Int = 30
```

施術に必要な時間です。

例えば、

```text
調髪
30分
```

なら、

```swift
durationMinutes = 30
```

です。

パーマに90分必要なら、

```swift
durationMinutes = 90
```

になります。

---

# 3. BusinessSettings

店舗設定を保存します。

```swift
@Model
final class BusinessSettings {

    var holiday: Int

    var startTime: Int

    var endTime: Int

    init(
        holiday: Int,
        startTime: Int,
        endTime: Int
    ) {

        self.holiday = holiday

        self.startTime = startTime

        self.endTime = endTime
    }
}
```

保存する情報は、

```text
holiday
↓
定休日

startTime
↓
始業時間

endTime
↓
終業時間
```

です。

---

# 曜日の数字

SwiftのCalendarでは、

```text
1 = 日曜日
2 = 月曜日
3 = 火曜日
4 = 水曜日
5 = 木曜日
6 = 金曜日
7 = 土曜日
```

となっています。

そのため、

```swift
holiday = 2
```

なら、

```text
月曜日休み
```

という意味になります。

---

# 4. ServiceOption

サービス1件分を表します。

```swift
struct ServiceOption:
    Identifiable,
    Hashable {

    var id: String {
        title
    }

    let title: String

    let durationMinutes: Int
}
```

例えば、

```swift
ServiceOption(
    title: "調髪：4400円",
    durationMinutes: 30
)
```

なら、

```text
メニュー名
↓
調髪：4400円

所要時間
↓
30分
```

という1つのサービスになります。

---

# なぜStringだけではなくstructにしたのか

最初は、

```swift
let services: [String] = [
    "調髪：4400円",
    "パーマ：9400円"
]
```

でも表示できます。

しかしこれでは、

```text
名前
```

しか保存できません。

今回は、

```text
名前
+
施術時間
```

を管理したいため、

```swift
ServiceOption
```

というstructを作っています。

---

# Identifiable

```swift
Identifiable
```

はSwiftUIの、

```swift
ForEach
```

でデータを識別するために使います。

今回、

```swift
var id: String {
    title
}
```

なので、

```text
調髪：4400円
```

というtitleそのものをIDとして利用しています。

---

# Hashable

```swift
Hashable
```

を付けることで、

```swift
ServiceOption
```

同士を比較したり、SwiftUIで扱いやすくなります。

---

# 5. ServiceCatalog

店の全メニューをまとめています。

```swift
struct ServiceCatalog {

    static let services: [ServiceOption] = [

        ServiceOption(
            title: "調髪：4400円",
            durationMinutes: 30
        ),

        ServiceOption(
            title: "調髪顔剃り無し：4100円",
            durationMinutes: 30
        ),

        ServiceOption(
            title: "調髪のみ：3600円",
            durationMinutes: 30
        ),

        ServiceOption(
            title: "パーマ：9400円〜",
            durationMinutes: 30
        ),

        ServiceOption(
            title: "Cut カラー：7600円",
            durationMinutes: 30
        )
    ]
}
```

実際のコードでは全メニューをここに登録しています。

---

# `static`

```swift
static let services
```

の

```swift
static
```

によって、

```swift
let catalog = ServiceCatalog()
```

のようにインスタンスを作らなくても、

```swift
ServiceCatalog.services
```

と直接アクセスできます。

---

例えば、

```swift
private let services =
    ServiceCatalog.services
```

と書けば、

```text
ServiceCatalogの全メニュー
```

を取得できます。

---

# 6. 予約重複判定

このアプリの重要な処理です。

```swift
func reservationOverlaps(
    start: Date,
    durationMinutes: Int,
    reservation: Reservation
) -> Bool {

    let calendar = Calendar.current

    guard let newReservationEnd =
        calendar.date(
            byAdding: .minute,
            value: durationMinutes,
            to: start
        )
    else {
        return false
    }

    guard let existingReservationEnd =
        calendar.date(
            byAdding: .minute,
            value: reservation.durationMinutes,
            to: reservation.dateTime
        )
    else {
        return false
    }

    return start < existingReservationEnd
        &&
        reservation.dateTime < newReservationEnd
}
```

この関数は、

> 新しく入れようとしている予約と、既存予約が重なっているか

を判定します。

---

# 引数

```swift
start: Date
```

新規予約の開始時間。

---

```swift
durationMinutes: Int
```

新規予約の施術時間。

---

```swift
reservation: Reservation
```

既に保存されている予約です。

---

# 新規予約の終了時間

```swift
calendar.date(
    byAdding: .minute,
    value: durationMinutes,
    to: start
)
```

例えば、

```text
start
10:30

durationMinutes
30
```

なら、

```text
10:30 + 30分
↓
11:00
```

になります。

---

# 既存予約の終了時間

```swift
calendar.date(
    byAdding: .minute,
    value: reservation.durationMinutes,
    to: reservation.dateTime
)
```

例えば、

```text
既存予約開始
10:00

施術時間
60分
```

なら、

```text
既存予約終了
11:00
```

です。

---

# 重複判定の式

```swift
return start < existingReservationEnd
    &&
    reservation.dateTime < newReservationEnd
```

日本語にすると、

```text
新規予約開始 < 既存予約終了

かつ

既存予約開始 < 新規予約終了
```

です。

両方成立すると予約が重なっています。

---

## 重複する例

```text
既存予約
10:00 -------- 11:00

新規予約
       10:30 -------- 11:30
```

これは、

```swift
true
```

になります。

---

## 重複しない例

```text
既存予約
10:00 -------- 11:00

新規予約
                  11:00 -------- 11:30
```

11時ちょうどから次の予約なら、

```swift
false
```

です。

---

# 7. SelectView

最初の画面です。

```swift
struct SelectView: View {

    @Query
    private var settings:
        [BusinessSettings]

    @Environment(\.modelContext)
    private var modelContext

    var body: some View {

        NavigationStack {

            VStack(spacing: 30) {

                Text(
                    "ヘアーサロンキリシマ"
                )
                .font(.largeTitle)
                .bold()

                NavigationLink {
                    ContentView()
                } label: {
                    Text(
                        "お客様として利用"
                    )
                }

                NavigationLink {
                    MusterPassView()
                } label: {
                    Text(
                        "店舗管理者としてログイン"
                    )
                }
            }
            .padding()
        }
    }
}
```

利用者と店舗管理者の画面へ分岐します。

---

# `NavigationStack`

```swift
NavigationStack
```

は画面遷移を管理します。

---

# NavigationLink

例えば、

```swift
NavigationLink {
    ContentView()
} label: {
    Text("お客様として利用")
}
```

を押すと、

```swift
ContentView()
```

へ移動します。

---

# 8. `@Query`

```swift
@Query
private var settings:
    [BusinessSettings]
```

SwiftDataから、

```swift
BusinessSettings
```

を読み込みます。

つまり、

```text
SwiftData
↓
BusinessSettings取得
↓
settingsへ入る
```

という流れです。

---

# 9. `modelContext`

```swift
@Environment(\.modelContext)
private var modelContext
```

SwiftDataへ、

```text
追加
削除
```

するときに使います。

例えば、

```swift
modelContext.insert(setting)
```

で追加。

```swift
modelContext.delete(reservation)
```

で削除できます。

---

# 10. 店舗設定の初期作成

```swift
.task {

    if settings.isEmpty {

        let setting =
            BusinessSettings(
                holiday: 2,
                startTime: 9,
                endTime: 18
            )

        modelContext.insert(
            setting
        )
    }
}
```

店舗設定がまだ存在しない場合、

```text
月曜休み
9時開店
18時閉店
```

という初期設定を作ります。

---

# 11. MusterPassView

店舗管理者ログイン画面です。

```swift
struct MusterPassView: View {

    private let musterPassword =
        "CHANGE_ME"

    @State private var pass = ""

    @State
    private var loginSuccess = false

    @State
    private var loginFailed = false
}
```

---

# `@State`

```swift
@State
```

は、

> 画面の状態を保持する

ために使います。

例えば、

```swift
@State private var pass = ""
```

には入力したパスワードが入ります。

---

# SecureField

```swift
SecureField(
    "パスワードを入力",
    text: $pass
)
```

通常のTextFieldとは違い、

```text
●●●●●●
```

のように入力内容を隠します。

---

# ログイン判定

```swift
Button("ログイン") {

    if pass == musterPassword {

        loginFailed = false

        loginSuccess = true

    } else {

        loginFailed = true
    }
}
```

パスワードが一致すると、

```swift
loginSuccess = true
```

になります。

---

# navigationDestination

```swift
.navigationDestination(
    isPresented: $loginSuccess
) {

    MusterView()
}
```

`loginSuccess` が `true` になると、

```swift
MusterView()
```

へ移動します。

---

# 注意

コード内にパスワードを直接書く方式は、開発中の簡易実装です。

本番アプリなら、

```text
Firebase Authentication
サーバー認証
Keychain
```

などを検討する必要があります。

---

# 12. MusterView

店舗管理画面です。

```swift
struct MusterView: View {

    @Query
    private var records:
        [Reservation]

    @Query
    private var settings:
        [BusinessSettings]

    @Environment(\.modelContext)
    private var modelContext

    @State
    private var searchDate = Date()

    @State
    private var showResult = false
}
```

---

# records

```swift
@Query
private var records:
    [Reservation]
```

SwiftDataに保存されている、

```text
全予約
```

を取得します。

---

# filteredRecords

```swift
var filteredRecords:
    [Reservation] {

    records
        .filter {

            Calendar.current.isDate(
                $0.dateTime,
                inSameDayAs: searchDate
            )
        }

        .sorted {
            $0.dateTime < $1.dateTime
        }
}
```

選択された日付と同じ日の予約だけ抽出します。

さらに、

```swift
.sorted
```

で時間順に並べます。

---

例えば、

```text
11:00
9:30
15:00
```

という予約が保存されていても、

```text
9:30
11:00
15:00
```

の順番で表示されます。

---

# 予約削除

```swift
.swipeActions {

    Button(
        role: .destructive
    ) {

        modelContext.delete(
            reservation
        )

    } label: {

        Label(
            "予約削除",
            systemImage: "trash"
        )
    }
}
```

予約をスワイプすると削除できます。

---

# 13. SettingsEditView

店舗設定画面です。

```swift
struct SettingsEditView: View {

    @Bindable
    var setting:
        BusinessSettings
}
```

---

# `@Bindable`

ここがSwiftDataで重要です。

```swift
@Bindable
```

は、

> SwiftDataから取得した@Modelの値を、画面から直接編集する

ために使います。

---

例えば、

```swift
$setting.holiday
```

や、

```swift
$setting.startTime
```

をPickerやStepperへ渡せます。

---

## @Queryとの違い

```text
@Query
↓
SwiftDataから取得

@Bindable
↓
取得したデータを画面から編集

modelContext
↓
追加・削除
```

です。

---

# 定休日選択

```swift
Picker(
    "定休日",
    selection: $setting.holiday
) {

    ForEach(
        weekdays,
        id: \.0
    ) { weekday in

        Text(weekday.1)
            .tag(weekday.0)
    }
}
```

曜日一覧から定休日を選択できます。

---

# 営業時間設定

```swift
Stepper(
    "始業時間：\(setting.startTime):00",
    value: $setting.startTime,
    in: 0...22
)
```

例えば、

```text
9
```

なら、

```text
9:00
```

です。

---

終業時間：

```swift
Stepper(
    "終業時間：\(setting.endTime):00",
    value: $setting.endTime,
    in: (setting.startTime + 1)...23
)
```

始業時間より後の時間だけ選べます。

---

# 14. ContentView

利用者が予約する日付を選びます。

```swift
@State
private var selectedDay =
    Calendar.current.startOfDay(
        for: Date()
    )

@State
private var selectedDateTime:
    Date?
```

ここでは、

```text
selectedDay
↓
日付

selectedDateTime
↓
日付 + 時間
```

に分けています。

---

## なぜ分けるのか

以前の設計では、

```swift
selectedDate
```

1つに、

```text
日付
時間
```

を両方持たせていました。

しかし予約アプリでは、

```text
日付選択
↓
時間選択
```

という流れなので分けた方が管理しやすくなります。

---

# 日付変更時

```swift
.onChange(
    of: selectedDay
) { _, _ in

    selectedDateTime = nil
}
```

例えば、

```text
10月10日 10:30
```

を選択したあと、

```text
10月11日
```

へ変更した場合、

```text
10:30
```

という以前の時間選択をリセットします。

---

# 15. DetailHourMinView

予約時間を選択します。

```swift
struct DetailHourMinView: View {

    @Query
    private var settings:
        [BusinessSettings]

    @Query
    private var records:
        [Reservation]

    let selectedDay: Date

    @Binding
    var selectedDateTime:
        Date?
}
```

---

# `@Binding`

```swift
@Binding
```

は、

> 前の画面のデータをこの画面から変更する

ために使います。

ContentView側：

```swift
@State
private var selectedDateTime:
    Date?
```

DetailHourMinView側：

```swift
@Binding
var selectedDateTime:
    Date?
```

という関係です。

---

# イメージ

```text
ContentView

selectedDateTime
      ↑
      │ @Binding
      │
DetailHourMinView
```

時間選択画面で、

```swift
selectedDateTime = slot
```

するとContentView側の値も変わります。

---

# 曜日の取得

```swift
var weekday: Int {

    Calendar.current.component(
        .weekday,
        from: selectedDay
    )
}
```

例えば月曜日なら、

```text
2
```

を取得します。

---

# 定休日判定

```swift
var holiday: Int {

    setting?.holiday ?? 2
}
```

店舗設定が取得できれば、

```swift
setting.holiday
```

を利用します。

設定がなければ、

```text
2 = 月曜日
```

を仮の値として利用します。

---

# 開始時間をDateへ変換

店舗設定では、

```swift
startTime = 9
```

のようにIntで保存しています。

しかし予約時間として扱うには、

```text
2026/10/10 09:00
```

というDateが必要です。

そのため、

```swift
Calendar.current.date(
    bySettingHour:
        setting.startTime,

    minute: 0,

    second: 0,

    of: selectedDay
)
```

でDateへ変換します。

---

# 予約枠生成

```swift
var timeSlots: [Date] {

    var result: [Date] = []

    var currentTime =
        startTime

    while currentTime < endTime {

        result.append(
            currentTime
        )

        currentTime =
            currentTime + 30分
    }

    return result
}
```

実際にはCalendarを使っています。

例えば営業時間が、

```text
9:00 ～ 12:00
```

なら、

```text
9:00
9:30
10:00
10:30
11:00
11:30
```

という予約枠を生成します。

---

# 過去の時間を表示しない

```swift
let now = Date()

if currentTime >= now {

    result.append(
        currentTime
    )
}
```

例えば現在時刻が、

```text
15:20
```

なら、

```text
9:00
9:30
10:00
...
15:00
```

は表示しません。

---

# 予約済み判定

```swift
func isSlotBooked(
    _ slot: Date
) -> Bool {

    records.contains {
        reservation in

        reservationOverlaps(
            start: slot,
            durationMinutes: 30,
            reservation: reservation
        )
    }
}
```

既存予約と重なる時間は、

```text
予約済み
```

になります。

---

# 時間選択UI

```swift
Button {

    selectedDateTime = slot

    dismiss()

} label: {

    HStack {

        Text(
            slot,
            format:
                .dateTime
                .hour()
                .minute()
        )

        Spacer()

        if booked {

            Text("予約済み")

        } else {

            Text("空き")
        }
    }
}
.disabled(booked)
```

予約済みならボタンを押せません。

---

# 16. CutDetailSelectView

予約者情報とメニューを選びます。

```swift
struct CutDetailSelectView: View {

    let selectedDate: Date

    @Query
    private var records:
        [Reservation]

    @Environment(\.modelContext)
    private var modelContext

    @State
    private var name = ""

    @State
    private var selectedService:
        ServiceOption?
}
```

---

# メニュー一覧

```swift
private let services =
    ServiceCatalog.services
```

先ほど作った、

```swift
ServiceCatalog
```

から全メニューを取得しています。

---

# 名前から空白を削除

```swift
var trimmedName: String {

    name.trimmingCharacters(
        in:
            .whitespacesAndNewlines
    )
}
```

例えば、

```text
"    "
```

だけ入力して予約できてしまうのを防ぎます。

---

# メニュー選択

```swift
ForEach(services) {
    service in

    Button {

        selectedService =
            service

    } label: {

        HStack {

            Text(
                service.title
            )

            Spacer()

            if selectedService?.id
                == service.id {

                Image(
                    systemName:
                        "checkmark"
                )
            }
        }
    }
}
```

選択したメニューには、

```text
✓
```

が付きます。

---

# 最終的な重複確認

```swift
var hasReservationConflict:
    Bool {

    guard let selectedService
    else {

        return false
    }

    return records.contains {
        reservation in

        reservationOverlaps(
            start: selectedDate,

            durationMinutes:
                selectedService
                .durationMinutes,

            reservation:
                reservation
        )
    }
}
```

ここでは、

```text
選んだメニューの施術時間
```

も含めて重複チェックします。

---

例えば、

```text
既存予約
10:00 ～ 11:00

新規予約
10:30 ～ 12:00
```

なら予約できません。

---

# 17. 予約保存

```swift
Button("予約完了") {

    guard let selectedService
    else {
        return
    }

    let record =
        Reservation(

            dateTime:
                selectedDate,

            name:
                trimmedName,

            service:
                selectedService.title,

            durationMinutes:
                selectedService
                .durationMinutes
        )

    modelContext.insert(
        record
    )

    dismiss()
}
```

まず、

```swift
Reservation
```

を作ります。

その後、

```swift
modelContext.insert(record)
```

でSwiftDataへ保存します。

---

# SwiftData全体の流れ

```text
@Model
↓
保存するデータの設計

@Query
↓
保存したデータを取得

@Bindable
↓
取得した@Modelを編集

modelContext.insert()
↓
追加

modelContext.delete()
↓
削除
```

---

# SwiftUIの状態管理

このアプリでは主に、

```text
@State
@Binding
@Query
@Bindable
@Environment
```

を使用しています。

---

## @State

```swift
@State
```

その画面自身が管理するデータ。

例：

```swift
@State
private var name = ""
```

---

## @Binding

```swift
@Binding
```

親画面のStateを子画面から変更します。

例：

```swift
@Binding
var selectedDateTime:
    Date?
```

---

## @Query

```swift
@Query
```

SwiftDataからデータ取得。

---

## @Bindable

```swift
@Bindable
```

SwiftDataのモデルを画面から編集。

---

## @Environment

```swift
@Environment(
    \.modelContext
)
```

SwiftDataの追加・削除。

また、

```swift
@Environment(
    \.dismiss
)
```

なら画面を閉じるために使用します。

---

# classとstructの使い分け

このアプリでは、

```swift
@Model
final class Reservation
```

と、

```swift
struct ServiceOption
```

があります。

理由は役割の違いです。

---

## Reservation

```text
SwiftDataに保存したい
↓
@Model class
```

---

## ServiceOption

```text
メニュー情報として使うだけ
↓
struct
```

です。

---

# データの関係

```text
ServiceCatalog

   ↓

ServiceOption

   ↓

利用者が選択

   ↓

Reservation

   ↓

SwiftDataへ保存
```

例えば、

```text
ServiceOption

パーマ
90分
```

を選択すると、

```text
Reservation

名前：田中
日時：10:00
メニュー：パーマ
施術時間：90分
```

として保存されます。

---

# 現在の予約システムの特徴

このアプリでは、

```text
日付選択
↓
30分単位の予約枠生成
↓
既存予約との重複確認
↓
サービス選択
↓
サービスの施術時間を考慮
↓
最終重複確認
↓
SwiftData保存
```

という流れになっています。

---

# 今後改善できるポイント

現在は学習用・試作用として十分ですが、実際に店舗で使用するなら次の改善が考えられます。

## 1. 管理者認証

現在：

```text
アプリ内にパスワード
```

将来：

```text
Firebase Authentication

または

サーバー認証
```

---

## 2. 複数の定休日

現在：

```text
holiday
↓
1曜日のみ
```

将来：

```text
月曜
第3日曜
祝日
臨時休業
```

などにも対応できます。

---

## 3. メニュー管理

現在は、

```swift
ServiceCatalog
```

へ直接書いています。

将来的には、

```text
メニュー名
価格
施術時間
```

もSwiftDataへ保存して、

管理者画面から変更できるようにできます。

---

## 4. 予約キャンセル

利用者自身が予約を確認し、

```text
予約変更
予約キャンセル
```

できるようにする。

---

## 5. 電話番号

現在：

```text
名前のみ
```

ですが、

```text
名前
電話番号
```

にすると店舗側が連絡できます。

---

## 6. 通知

予約前日に、

```text
明日10:00から予約があります
```

という通知を出すこともできます。

Swiftでは、

```text
UserNotifications
```

を使えます。

---

## 7. CloudKit

現在のSwiftDataだけでは、

```text
店舗端末
利用者端末
```

で予約情報を共有できません。

実際に複数端末で使うなら、

```text
CloudKit
Firebase
独自サーバー
```

などが必要になります。

---

# 学習したSwiftの技術

このアプリを通して、

```text
SwiftUI

SwiftData

@Model

@Query

@Bindable

@State

@Binding

@Environment

NavigationStack

NavigationLink

DatePicker

Calendar

Date

Optional

guard let

computed property

struct

class

final

Identifiable

Hashable

ForEach

filter

sorted

contains
```

などを使用しています。

---

# このアプリで特に重要な部分

個人的に、このアプリで特に重要なのは次の3点です。

## 1

```swift
@Model
```

によるSwiftDataの永続化。

## 2

```swift
@State
```

と、

```swift
@Binding
```

による画面間のデータ共有。

## 3

```swift
return start
    < existingReservationEnd
    &&
    reservation.dateTime
    < newReservationEnd
```

による予約時間の重複判定。

この3つを理解すると、このアプリ全体の構造がかなり理解しやすくなります。

---

# 最終的なデータフロー

```text
利用者

↓ 日付選択

selectedDay

↓ 時間選択

selectedDateTime

↓ メニュー選択

ServiceOption

↓

Reservation作成

↓

modelContext.insert()

↓

SwiftData

↓

管理者画面

↓

@Query

↓

予約一覧表示
```

---

# まとめ

このアプリは、

```text
SwiftUI
+
SwiftData
+
日時処理
+
予約重複判定
```

を組み合わせた予約管理アプリです。

単純に画面を作るだけではなく、

```text
データ保存
予約検索
営業時間
定休日
予約枠
重複判定
管理者画面
```

まで扱っています。

今後、

```text
CloudKit
Firebase
通知
ユーザー認証
ネットワーク通信
```

などを追加すれば、より実際の店舗運用に近い予約アプリへ発展させることができます。
