Swift 3のRange周りはけっこうややこしいので、まとめてみました。

Range周りのややこしさはこのあたりに起因していると思っています。

- Swift 3のRangeの仕様自体少し複雑
    - 現状の言語制約・型安全性など踏まえて妥当な複雑性だと思っていますが
- Swift 2.2から大きく変更
    - Swift 2.2: `Range<T>`・`ClosedInterval<T>`・`OpenInterval<T>`
    - Swift 3.0: `Range<T>`・`ClosedRange<T>`・`CountableRange<T>`・`CountableClosedRange<T>`
    - 再整理された感じなので、単純な置き換えでは無く、新たな理解が必要な部分も多いです


それでは、Swift 3.0のRange周りについて、説明していきます。

# Swift 3.0のRange周り解説

次のように、`Half-Open(..<)`/`Closed(...)` x `Comparable`/`Strideable`のマトリックスで、`Range`・`ClosedRange`・`CountableRange`・`CountableClosedRange`という4種の型があります。

| 要素 | Half-Open(..<) | Closed(...) |
| :---| :---: | :---: |
|[Comparable](https://developer.apple.com/reference/swift/comparable)のみ | [Range](https://developer.apple.com/reference/swift/range) | [ClosedRange](https://developer.apple.com/reference/swift/closedrange) |
|[Strideable](https://developer.apple.com/reference/swift/strideable)(Int刻み) | [CountableRange](https://developer.apple.com/reference/swift/countablerange) | [CountableClosedRange](https://developer.apple.com/reference/swift/countableclosedrange) |


`Half-Open(..<)`/`Closed(...)`は、次のようにendの要素を含むか否かが区別されます。こちらはSwiftを普通に書いているとお馴染みだと思います。

- Half-Open(..<): 例: `0..<3` → `[0, 1, 2]`相当
- Closed(...): 例: `0...3` → `[0, 1, 2, 3]`相当

コレクション要素が`Comparable`/`Strideable`(Int刻み)とは、次のように区別されます。

- Comparable: 比較演算のみ可能 → 非`Countable`なRange
- Strideable(Int刻み): Int刻みでの前後走査も可能 → `Countable`なRange

## `Comparable`/`Strideable`(Int刻み)について

`Comparable`/`Strideable`(Int刻み)についてはもう少し詳しく説明します。

`Countable`なRange(要素がStrideableなRange)のみ、ランダムアクセス可能なコレクションであり、[Sequence](https://developer.apple.com/reference/swift/sequence)・[Collection](https://developer.apple.com/reference/swift/collection)の有する便利なコレクション走査メソッド(`for-in`操作・`map`・`filter`)を扱えます。

```swift
for i in 0..<3 {
    print(i)
}
```

`Strideable`なものは`Int`などです。

そうではない、つまり`Countable`では無い例としては、`Float`や`Character`のRangeが挙げられます。
`Float`も`Strideable`ではあるものの、刻みが`Float`なので、`Countable`では無いです。

DoubleのRangeをシーケンス操作する次のコードを書くと、`Type 'Range<Double>' does not conform to protocol 'Sequence'`というコンパイルエラーが発生します。

```swift
for i in 0.0..<3.0 {
    print(i)
}
```

つまり、`0.0`の「次の値」が不定ということです。`0.1`かもしれないし、`1.0`かもしれないし、それは`Float`の仕様では確定されません。
ちなみに、次のように刻みを明示(この場合`0.1`)すれば、イテレート操作可能となります。

```swift
for i in stride(from: 0.0, to: 3.0, by: 0.1) {
    print(i)
}
```

ただ、これは`Sequence`型に準拠した`StrideTo`型で、Range系の型とは似て非なるものです。

また、`Character`型についても次のように書くと、同じコンパイルエラーが出ます。

```swift
for c in Character("a")...Character("z") {
    print(c)
}
```

"a"のつぎはASCIIコード的に"b"では？と思うかもしれません。

確かに、例えばRubyの場合は、次のコードを書くと、

```ruby
("a".."z").each { |c| p c }
```

このように出力されます。

```
a
b
c
...
```

Swiftの場合は、Stringが厳密な実装(ちょっとこの表現難しいですが)になっていて、単純に"a"の次は"b"とみなさないような仕様になっています。
近々String周りも解説記事書きたいと思っていますが、とりあえず今気になる方は [Strings in Swift 3 – Ole Begemann](https://oleb.net/blog/2016/08/swift-3-strings/) などご参照ください。

ちなみに次のように書くと、Swiftでも出来ます(　´･‿･｀)
これはASCIIコードと明示してイテレートしてるので、厳密で良いと思っています。

```swift
extension Character
{
    init?(asciiCode: UInt32) {
        guard let scalar = UnicodeScalar(asciiCode) else {
            return nil
        }
        self = Character(scalar)
    }
    func asciiCode() -> UInt32 {
        let characterString = String(self)
        let scalars = characterString.unicodeScalars
        
        return scalars[scalars.startIndex].value
    }
}

for code in Character("a").asciiCode()...Character("z").asciiCode() {
    print(Character(asciiCode: code)!)
}
```

以上、簡潔にまとめると、次のようになります。

- `Countable`なRange: コレクション系操作が可能 
  - `Int`などのRange
- 非`Countable`なRange: 範囲を示すことなどに、用途か限られる
  - `Double`・`Character`などのRange

## `Half-Open(..<)`/`Closed(...)`が分かれている理由

Swift 2.2では両方とも`Range`でした。
さらに詳しく書くと、`Closed(...)`と書いても、それは表記の違いだけで、`Half-Open(..<)`として扱われていました。

次の評価も、`true`となってしました。

```
(0..<2) == (0...1)
```

ただ、問題があって、0以上の整数すべてのレンジを表現したい時に`0...Int.max`が`0..<Int.maxより1多い何か`みたいな感じに解釈され、結果overflow系のコンパイルエラーが発生してしまっていました。

それでは、`switch`分内の`case`で0以上の整数すべてを受けたい時など困る、ということで`ClosedInterval`・`OpenInterval`というマッチ用ともいえる型がありました。

次のコードはSwift 3.0では動かずSwift 2.2など用のコードですが、` as ClosedInterval`を外すとコンパイルエラーになります。

```swift
let value = 5
switch value {
    case 0...Int.max as ClosedInterval:
        print("zero or positive")
    default:
        print("default")
}
```

これがイマイチなどの理由があって、レンジ系全般が`Half-Open(..<)`/`Closed(...)`とに明確に分けられました。そして、`ClosedInterval`・`OpenInterval`は不要になったので、無くなりました。


Swift 3.0についてまとめると、次の通りです。

- `Half-Open(..<)`のみ、空のRangeを表現出来る(`5..<5`など)
- `Closed(...)`のみ、その要素型のmax値を含めた書き方が出来る(`0...Int.max`など)


## Countableとそうでないものが別物の型になっている理由

本当は次のようなコードで、Range型は共通として、ジェネリクスで要素がどのプロトコルに準拠しているかなどで表現したいということみたいですが、Swift 3ではサポート外でコンパイルエラーになってしまいます。

```swift
extension Range: RandomAccessCollection
    where Bound: Strideable, Bound.Stride: SignedInteger
{    
}
```

Swift 4のメインゴールの1つである[Conditional conformances](
https://github.com/apple/swift/blob/master/docs/GenericsManifesto.md#conditional-conformances-)が実現出来れば、型を分ける必要はなくなり、次の2つに集約されシンプルになるはずです。

- `Range`
- `ClosedRange`

## Swift 3対応にあたって

今までRangeだったのが、`Half-Open(..<)`/`Closed(...)`という2つの型に分かれたのが少し厄介です。
今までRangeを引数としていたところで、この2つの型の両対応が出来なってしまいました。

スマートな解決策は無いですが、[Swift 3 migration pitfalls — Codelle](http://codelle.com/blog/2016/9/swift-3-migration-pitfalls/)にまあまあ良い対応が載っていました。

(そのサイトでは、`IterableRange.Iterator.Element: Int`だったのを汎用的に`: Stride`に変えました)

```swift
func doSomething<IterableRange>(for range: IterableRange)
    where IterableRange: Sequence, IterableRange.Iterator.Element: Strideable {
        for number in range {
            print(number)
        }
}
```

あるいは、Swift 2.2と全く同じ挙動でなるように、呼び出し側を`Half-Open(..<)`に変換することで安全に対応出来そうで、こちらの方が素直そうです。

```swift
func doSomething2<T>(for range: CountableRange<T>) {
        for number in range {
            print(number)
        }
}

// 既存コードがこの時、`Closed`か同化が非合致でコンパイルエラーになる
// doSomething2(for: 0...10)) 

// 対応1
doSomething2(for: CountableRange(0...10)) // → `0..<11`に変換されて引数に渡される
// 対応2
doSomething2(for: 0..<11))
```


## Indexが`successor()`・`predecessor()`・`advancedBy(_:)`・`advancedBy(_:limit:)`・`distanceTo(_:)`などを持たなくなった

最後に、ちょっとここまでの話と変わりますが、Range周りの変更として、もう1つ紹介します。

コレクション系の型には、そのそれぞれの要素が属する場所(インデックス)を示すIndex型があります。Index自身が走査系のメソッドを持っていましたが、それがそのIndexのオーナーであるコレクションに移りました。

例として、Swift 2.2以前のコードが色々載っている[String.Indexを使った文字列処理 - Qiita](http://qiita.com/boohbah/items/795501495e1aeab6231e) から一部コードをお借りして、書き換え例を載せておきます。

Swift 2.2:

```swift
let str = "ABC"
let idx0 = str.startIndex
let idx1 = idx0.successor()
let idx2 = idx1.successor()
str[idx0] // => 'A' Character型
str[idx1] // => 'B' Character型
str[idx2] // => 'C' Character型
```

Swift 3.0:

```swift
let str = "ABC"
let idx0 = str.startIndex
let idx1 = str.index(after: idx0)
let idx2 = str.index(after: idx1)
str[idx0] // => 'A' Character型
str[idx1] // => 'B' Character型
str[idx2] // => 'C' Character型
```
ちゃんと理解していれば大したことないですが、そうで無いと書き換え方に悩むかもしれませんね。

# 参考

- [Swift.org - Migrating to Swift 2.3 or Swift 3 from Swift 2.2](https://swift.org/migration-guide/)
- [swift-evolution/0065-collections-move-indices.md at fa75ca35911a8d20f5f38d0f19e3ab38457b4ab9 · apple/swift-evolution](https://github.com/apple/swift-evolution/blob/fa75ca35911a8d20f5f38d0f19e3ab38457b4ab9/proposals/0065-collections-move-indices.md)
- [Ranges in Swift 3 – Ole Begemann](https://oleb.net/blog/2016/09/swift-3-ranges/)
- [Swift 3 migration pitfalls — Codelle](http://codelle.com/blog/2016/9/swift-3-migration-pitfalls/)