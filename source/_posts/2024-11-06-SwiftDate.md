---
title: SwiftDate
date: 2024-11-06 21:59:22
tags:
- iOS
- swift
- date
---

# SwiftDate 使用

Apple 中的**Date**完全独立于任何特定的地理位置，日历或区域设置，一个普通的**Date**对象只表示一个绝对值：实际上，它计算自 2001 年 1 月 1 日以来经过的秒数。


在开发过程中，通常需要在更具体的上下文中表示日期：世界上的特定位置，使用指定`Local`的规则输出他们

为了实现它，swiftDate 需要引入几个其他对象：`Calendar`, `TimeZone`, `Locale`


## Region & DateInRegion

为了简化特定上下文中的日期管理，SwiftDate 引入了两个简单的结构：

+ Region 是一个结构体，它定义了世界上的区域 （TimeZone）、语言 （Locale） 和参考日历 （Calendar）。
+ DateInRegion 表示特定区域中的绝对日期。使用此对象时，所有组件都将在创建对象的区域的上下文中进行评估。在 DateInRegion 中，您将具有 absoluteDate 和 region 属性。


### The Default Region

在 SwiftDate 中，您可以同时使用 DateInRegion 和 Date 实例。当您需要提取时间单位、比较日期或评估特定操作时，即使是普通的 Date 对象也会使用 Region。


但是，一个 Default Region 的特殊区域，默认情况下，它具有以下属性：

+ 时区 = GMT - 这允许与默认的日期管理保持一致，除非您更改它。

+ 日历 = 当前设备日历（自动更新）

+ Locale = 当前设备的区域设置（自动更新）


虽然始终使用 DateInRegion 是一个不错的选择，但您也可以通过更改默认区域来使用 Date，如下所示：

```swift
let rome = Region(calendar: Calendars.gregorian, zone: Zones.europeRome, locale: Locales.italian)
SwiftDate.defaultRegion = rome
```
从现在开始，所有 Date 实例都使用 rome 作为默认区域来解析和评估日期组件：

```swift
let dateInRome = "2018-01-01 00:00:00".toDate()!
print("Current year is \(dateInRome.year) and hour is \(dateInRome.hour)") // "Current year is 2018 and hour is 0\n"
```
我们仍然可以使用 convertTo（region：） 函数将此日期转换为以 UTC 为单位的默认绝对表示形式：

```swift
let dateInUTC = dateInRome.convertTo(region: Region.UTC)
print("Current year is \(dateInUTC.year) and hour is \(dateInUTC.hour)") // "Current year is 2017 and hour is 23\n"
```
设置默认区域时要小心。我们仍然建议使用 DateInRegion 实例，这样您就可以显式读取区域。


### Create Region

现在，您可以创建新的 DateInRegion。创建新日期的方法有很多种：解析字符串、设置时间组件、从另一个日期或给定的时间间隔派生它。

每个初始化方法都需要一个 region 参数，该参数定义表示日期的区域（默认值可能因 init 而异，如下所示）。

#### From String
最常见的情况是解析字符串并将其转换为日期。如您所知，DateFormatter 的创建成本很高，如果您需要解析多个字符串，则应避免在循环中创建新实例。别担心：使用 SwiftDate，该库通过重用自己的解析器（沿调用方线程共享）来帮助您。

`DateInRegion 的 init(_：format：region)`可用于从字符串初始化新日期（在 String extensions 的 toXXX 前缀下可以使用各种快捷方式）。

此对象采用三个参数：

 + 要解析的字符串 （String）
 + 字符串的格式 （String）：这表示表示字符串的格式。它是 unicode 格式（请参阅字段表）。如果跳过此参数，SwiftDate 将尝试使用 SwiftDate.autoFormats 数组中定义的内置格式之一来解析日期。如果您知道日期的格式，则应明确设置它以获得更好的性能。
 + 表示日期的区域 （Region）。默认情况下设置为 SwiftDate.defaultRegion。
```swift
let date1 = DateInRegion("2016-01-05", format: "yyyy-MM-dd", region: regionNY)
let date2 = DateInRegion("2015-09-24T13:20:55", region: regionNY)
```


## Parse Custom Format

将输入字符串转换为有效日期的最简单方法是使用可用作 String 实例扩展的 .toDate（） 函数之一。这些方法的目的是获得可以表示输入字符串的最佳格式，并使用它来生成有效的 DateInRegion。

与 moment.js 等其他库一样，SwiftDate 有一个内置格式列表，它可以用来获得有效的结果。您可以通过调用 SwiftDate.autoFormats 来获取这些格式的列表。此数组的顺序很重要，因为 SwiftDate 会循环访问此列表，直到返回有效日期 （顺序本身允许库减少误报列表） 。

您可以通过添加/删除或替换此数组的内容来更改此列表。

```swift
.toDate(_ format: String?, region: Region?)
.toDate(_ formats: [String]?, region: Region?)
```
functions takes as input two arguments:
functions 将两个参数作为输入：

+ format （String|Array）：它是可选的，允许您显式设置 SwiftDate 必须用于解析日期的格式（或格式的有序列表）。允许的值列在 Unicode DateTime Table 中，您可以在此处找到。如果省略，则 SwiftDates 尝试解析遍历 SwiftDate.autoFormats 中列出的 auto 模式列表的字符串。
+ region （Region）：描述表示日期的区域 （locale/calendar/timezone）。如果省略，则使用 iinstead 的默认区域 （SwiftDate.defaultRegion）。
这些函数的结果是一个可选的 DateInRegion 实例（如果解析失败，则返回 nil）。

Some examples: 一些例子：

```swift
let _ = "2018-01-01 15:00".toDate()
let _ = "15:40:50".toDate("HH:mm:ss")
let _ = "2015-01-01 at 14".toDate("yyyy-MM-dd 'at' HH", region: rome)

// Support for locale
let itRegion = Region(calendar: Calendars.gregorian, zone: Zones.europeRome, locale: Locales.italian)
let enRegion = Region(calendar: Calendars.gregorian, zone: Zones.europeRome, locale: Locales.english)

let srcString = "July 15 - 15:30"
// it returns nil because itRegion has Locales.italian
let _ = srcString.toDate(["yyyy-MM-dd","MMM dd '-' HH:mm"], region: itRegion)
// it's okay because enRegion has locale set to english
let _ = srcString.toDate(["yyyy-MM-dd","MMM dd '-' HH:mm"], region: enRegion)
```
> 性能为了保持性能，如果您知道 input 格式，则应传递 format 参数。
> LOCALE 参数如果您使用可读的单位名称（例如表示月份的 MMM），请确保在 region 参数中选择正确的区域设置，以获得有效的结果。


## Date Formatting
日期格式设置非常简单，您可以指定自己的格式、区域设置或使用提供的任何格式、区域设置。

```swift
// Date Formatting
let london = Region(calendar: .gregorian, zone: .europeLondon, locale: .english)
let date = ... // 2017-07-22T18:27:02+02:00 in london region
let _ = date.toDotNET() // /Date(1500740822000+0200)/
let _ = date.toISODate() // 2017-07-22T18:27:02+02:00
let _ = date.toFormat("dd MMM yyyy 'at' HH:mm") // "22 July 2017 at 18:27"

// You can also easily change locale when formatting a region
let _ = date.toFormat("dd MMM", locale: .italian) // "22 Luglio"

// Time Interval Formatting as Countdown
let interval: TimeInterval = (2.hours.timeInterval) + (34.minutes.timeInterval) + (5.seconds.timeInterval)
let _ = interval.toClock() // "2:34:05"

// Time Interval Formatting by Components
let _ = interval.toString {
	$0.maximumUnitCount = 4
	$0.allowedUnits = [.day, .hour, .minute]
	$0.collapsesLargestUnit = true
	$0.unitsStyle = .abbreviated
} // "2h 34m"
```