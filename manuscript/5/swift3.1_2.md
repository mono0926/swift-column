タイトル(仮): Swift 3.1 のリリースプロセスおよびそれに含まれる変更内容の紹介(後編)

---

# Swift 3.1 の主な変更一覧

[SwiftリポジトリのCHANGELOG](https://github.com/apple/swift/blob/master/CHANGELOG.md#swift-31)・[Swift EvolutionリポジトリのProposal Status](https://apple.github.io/swift-evolution/)を見ると、開発状況が分かる。以下に12月13日時点で最新の `DEVELOPMENT-SNAPSHOT-2016-12-15-a` ツールチェイン時点での実装状況を記す。

## 実装済み

- [SE-0045: `Sequence` プロトコルに `prefix(while:) and drop(while:)` が追加](https://github.com/apple/swift-evolution/blob/master/proposals/0045-scan-takewhile-dropwhile.md)
- [SE-0141: `@available`構文においてのSwift バージョン分岐](https://github.com/apple/swift-evolution/blob/master/proposals/0141-available-by-swift-version.md)
- [SR-1009: 具体的な型を用いたジェネリクス制約](https://bugs.swift.org/browse/SR-1009)
- [SR-1446: ジェネリクスの入れ子](https://bugs.swift.org/browse/SR-1446)
- [SR-1882: 文字列補間(‘String interpolation‘)にOptional型を直接用いると警告](https://bugs.swift.org/browse/SR-1882)

## 実装中

現時点のSnapshots版では試せないが実装中であり、基本的に Swift 3.1 に入る予定のものは以下である。

- [SE-0042: 未適用のメソッド参照の平坦化](https://github.com/apple/swift-evolution/blob/master/proposals/0042-flatten-method-types.md)
- [SE-0075: インポート可能なモジュールによって分岐可能なビルド設定](https://github.com/apple/swift-evolution/blob/master/proposals/0075-import-test.md)
- [SE-0080: 数値変換に失敗したらnilを返すイニシャライザー](https://github.com/apple/swift-evolution/blob/master/proposals/0080-failable-numeric-initializers.md)
  - TODO: 大体弄れるので紹介
- [SE-0104: Protocol-oriented なInt型](https://github.com/apple/swift-evolution/blob/master/proposals/0104-improved-integers.md)

## 実装着手前

Swift に組み込むことは決定しているが、まだ実装着手に至っていないものもある。Swift 3.1 に含められるように目指しているが、前編で述べたタイムリミットである2016年1月16日に間に合わなければ時期バージョン以降に持ち越しとなる。

- [SE-0068: `Self`による`class`メンバーへのアクセス](https://github.com/apple/swift-evolution/blob/master/proposals/0068-universal-self.md)
- [SE-0082: Swift Package Managerの編集](https://github.com/apple/swift-evolution/blob/master/proposals/0082-swiftpm-package-edit.md)
  - TODO試せる？
- [SE-0110: 関数引数が単一タプルか複数引数によって、型を区別](https://github.com/apple/swift-evolution/blob/master/proposals/0110-distingish-single-tuple-arg.md)
- [SE-0142: Protocolの`associatedtype`を`where`句で制約可能に](https://github.com/apple/swift-evolution/blob/master/proposals/0142-associated-types-constraints.md)
- [SE-0143: 条件付きのジェネリクス制約](https://github.com/apple/swift-evolution/blob/master/proposals/0143-conditional-conformances.md)
- [SE-0145: Swift Package Managerで依存解決したバージョンを固定(Pin)できる仕組み](https://github.com/apple/swift-evolution/blob/master/proposals/0145-package-manager-version-pinning.md)
  - TODO試せる
- [SE-0146: Swift Package Managerで生成するTargetの定義](https://github.com/apple/swift-evolution/blob/master/proposals/0146-package-manager-product-definitions.md)
  - TODO試せる？

## レビュー中

- [SE-0147: ](https://github.com/apple/swift-evolution/blob/master/proposals/0147-move-unsafe-initialize-from.md)


# Swift 3.1 の変更を先取り

それでは、今実際に試せる Swift 3.1 に含まれる予定の変更を見ていこう。コード例は以下の環境によるものだが、基本的にはこれ以降のバージョンであれば同様に動くはずだ。

- Xcode 8.2.1
- Swift Toolchain バージョン: `DEVELOPMENT-SNAPSHOT-2016-12-15-a`

## [SE-0045: `Sequence` プロトコルに `prefix(while:)`・`drop(while:)` が追加](https://github.com/apple/swift-evolution/blob/master/proposals/0045-scan-takewhile-dropwhile.md)

`Swift` には、 `map(_:)`・`filter(_:)`など、多くのコレクション操作メソッドが存在するが、そこに欠けていた `prefix(while:)`・`drop(while:)` が追加された。

まず、`prefix(while:)` の簡単な使い方としては、次のようにすると、条件に一致するまでの要素を抽出出来る。

```swift
let x = [1, 2, 3, 4, 5]
x.prefix { $0 < 3 } // [1, 2]
// `filter(_:)` と同じ結果
x.filter { $0 < 3 } // [1, 2]
```

この時、結果は `filter(_:)` と同じになるが、`prefix(while:)` は条件に一致するとそれ以降の走査をしないため評価は3回だけ走るが、`filter(_:)`は全要素を走査するため評価が5回走る違いがある。このように「ある条件までの要素を抽出したい」という用途であれば結果が同じだとしても`prefix(while:)`を用いるのが良い。

ただ、上の例だと偶々ソート済みだったので、結果が同じになったが、例えば逆順にしてから同じ処理を行うと、`filter(_:)`の結果は同じだが、`prefix(while:)`の場合、1つ目の要素5が条件を満たさないため、そこで打ち切られて結果は空になる。

```swift
let y = x.reversed() // [5, 4, 3, 2, 1]
y.prefix { $0 < 3 } // []
// `filter(_:)` と結果が異なる
y.filter { $0 < 3 } // [1, 2]
```

このように一見似ているがまったく別物なので、適宜最適なコレクション操作メソッドを使い分ける必要がある。

`drop(while:)` は、`prefix(while:)` とは逆に「ある条件までの要素を除去したい」ときに用い、例えば次のように使える。

```swift
let x = [1, 2, 3, 4, 5]
x.drop { $0 < 3 } // [3, 4, 5]
x.reversed().drop { $0 < 3 } // [5, 4, 3, 2, 1]
```

この `prefix(while:)`・`drop(while:)` の追加は、5月に作られた SE-0045 という古めの Proposalというのを見ても分かる通り、Swift 3.1 の目玉の変更ではなく、単に Swift 3.0 に間に合わなかったのが、ようやく Swift 3.1 のタイミングで入ることになったということである。単なるメソッドの追加なので、 Swift 3.0 に詰め込みたかった破壊的変更の絡む変更と比べて優先度が低かったためである。

## [SE-0141: `@available`構文においてのSwift バージョン分岐](https://github.com/apple/swift-evolution/blob/master/proposals/0141-available-by-swift-version.md)

Swift 2.0 で、`@available`・`#available` を用いてプラットフォームとそのバージョン指定を行えるようになった。

```swift
@available(iOS, introduced: 10)
func f1() { // iOS 10 以上でないと呼べない
}

func f2() {
    if #available (iOS 10, *) {
        // iOS 10以上の時のみ処理される
    }
}
```

Swift 3.1 では、そのうち `@available` で Swift バージョンも指定出来るようになる。ただし、`#available` 指定は対応しておらず、`Swift language version checks not allowed in #available(...)` というコンパイルエラーになってしまう。

```swift
func ng() {
    if #available(swift 3, *) {
    }
}
```

基本的な使用例は次の通りであり、使えるバージョン・非推奨バージョン・廃止バージョンを表現でき、対応外のバージョンの時は警告やコンパイルエラーを発生してくれる。`f5`メソッドに付けた`renamed`はオプション引数で、これを付けることで廃止になった代わりに新設されたメソッドを示せる。

```swift
@available(swift, introduced: 4)
func f3() { // Swift 4以降で使える
}
@available(swift, deprecated: 4)
func f4() { // Swift 4で非推奨(警告表示)
}
@available(swift, obsoleted: 3, renamed: "f3")
func f5() { // Swift 4で廃止(代わりにf3メソッドを使う)
}
```

第一引数に `iOS` などを指定した場合は、`unavailable`によって、そのプラットフォームではバージョン問わず使えないということも表現できるが、`swift`を指定した場合は `unavailable` 指定はできないことが `Unknown platform 'swift' for attribute 'available'` という警告によって示される。

```swift
@available(swift, unavailable)
func f6() {
}
```

元々、`#if swift(>=N)` マクロによって、次のように Swift バージョンによってソースコードを切り替えることは出来たが、これはコンパイル時の指定でどちらかの処理が切り捨てられてしまう。

```swift
#if swift(>=4)
    // Swift 4以降用のコード
#else
    // Swift 4より前用のコード
#endif
```


この Proposal は、 前編で取り上げた `-swift-version N` フラグによって過去の Swift バージョンの互換モードをサポート出来るようにする対応( [SR-2582: Add -swift-version command line flag](https://bugs.swift.org/browse/SR-2582) )に関連している。

以下の2つの選択肢があったが、後者の方が標準ライブラリ配布のやりやすさなどの理由で後者に決まって実装された。

- `#if swift(>=N)`マクロを利用して、バージョンごと複数バイナリを生成
- `@available` で Swift のバージョンも指定できるようにして、複数の Swift バージョンに対応した単一バイナリを生成

経緯としては Swift 言語自体の開発を円滑にするためのものだったが、通常のアプリ開発においても例えば「Swift 3 の制約でどうしてもこの対処が必要だが、 Swift 4に含まれるあの対応が入ればもっとシンプルに書ける。」という時などに、予め `@available(swift, deprecated: 4)` を仕込んで Swift 4 でビルドした際にコンパイルエラーにする、という活用などもできるであろう。

## [SR-1009: 具体的な型を用いたジェネリクス制約](https://bugs.swift.org/browse/SR-1009)

まず、 Swift 3.0 まで「具体的な型を用いたジェネリクス制約」に対応しておらず不便だったことを、 `Optional<String>` 型に新しいメソッドを足す例にとって説明する。

まず、 `Optional` 型は次のように `Wrapped` というジェネリクス型で含む値の方を表現するようになっている。

```swift
public enum Optional<Wrapped> : ExpressibleByNilLiteral { ... }
```

`Optional<String>` 型に新しいメソッドを足したい時、素直に考えれば次のような書き方になるであろう。

```swift
// コードA
extension Optional where Wrapped == String {
    /** nilや空文字の場合はtrue、それ以外の時はfalseを返す */
    var isNilOrEmpty: Bool {
        return self?.isEmpty ?? true
    }
}
```

ただ、この書き方では Swift 3.0 では次のコンパイルエラーとなってしまう。

```
Same-type requirement makes generic parameter 'Wrapped' non-generic
```

「具体的な型を用いたジェネリクス制約」に対応していない、ということである。

対処としては次のように、 `String` を直接使わずにそれを包む `Protocol` (`StringProtocol`)を用意して、型制約としては `Wrapped` が `StringProtocol` に準拠している(`= String` ではなく `: StringProtocol`)という記述にすれば、制限を回避出来る。

```swift
protocol StringProtocol {
    var value: String { get }
}
extension String: StringProtocol {
    var value: String { return self }
}

extension Optional where Wrapped: StringProtocol {
    /** 値があればそれを返して無ければ空文字を返す */
    var getOrDefault: String {
        return self?.value ?? ""
    }
}
```

Swift 3.1 では、この対処コードを書かずとも、「コードA」の書き方でコンパイルが通るようになった。様々なライブラリやアプリコードでこの対処がなされている現状なので、この書き方が可能となる恩恵は大きい。


## [SR-1446: ジェネリクス型の入れ子](https://bugs.swift.org/browse/SR-1446)

クラスの所属・役割を明確にしたり、名前の重複を回避するために、型を入れ子で表現することがある。特に Swift の場合、名前空間がモジュール単位であるため細かい粒度で名前空間を用意しにくいこともあり、型の入れ子定義は多用されていると感じている。

ただ、これもジェネリクス型の場合、入れ子にできないという制限があったため、Swift 3.0 までは仕方なくジェネリクスの入れ子を諦めるという、設計の妥協をせざるを得なかった。

Swift 3.1 では、以下のいずれも書けるようになった。

```swift
// [SwiftリポジトリのCHANGELOG](https://github.com/apple/swift/blob/master/CHANGELOG.md#swift-31)のコード例をそのまま掲載
struct OuterNonGeneric {
    struct InnerGeneric<T> {}
}

struct OuterGeneric<T> {
    struct InnerNonGeneric {}
    struct InnerGeneric<T> {}
}

extension OuterNonGeneric.InnerGeneric {}
extension OuterGeneric.InnerNonGeneric {}
extension OuterGeneric.InnerGeneric {}
```

ジェネリクスについては、2016年5月に、[[swift-evolution] [Manifesto] Completing Generics](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160229/011666.html)という声明がメーリングリストに投稿されていて、今の大きな課題の1つであった。

[SR-1009: 具体的な型を用いたジェネリクス制約](https://bugs.swift.org/browse/SR-1009)・[SR-1446: ジェネリクスの入れ子](https://bugs.swift.org/browse/SR-1446)の2件でその一部が解消して、これらの対応は Swift 3.1 の目玉の改善の1つと言えるだろう。まだ着手前であるが、[SE-0143: 条件付きのジェネリクス制約](https://github.com/apple/swift-evolution/blob/master/proposals/0143-conditional-conformances.md)などまだ残件はあるが、今後もジェネリクスの制限は少しずつ改善されていくであろう。

## [SR-1882: 文字列補間(‘String interpolation‘)にOptional型を直接用いると警告](https://bugs.swift.org/browse/SR-1882)


以下のコードでは `hello Optional("world")` という文字列が出力されます。

```swift
let s: String? = "world"
print("hello \(s)")
```

このコードだけ見れば妥当ですが、通常のアプリでは意図せず `Optional` の変数を用いて文字列を組み立ててしまうミス(`hello world`とするつもりが`hello Optional("world")`になるなど)の発生に繋がります。

Swift 3.1 では以下の警告が発生するようになる。

```
String interpolation produces a debug description for an optional value; did you mean to make this explicit?
```

本当に `Optional()` で包まれた文字列で良いのであれば、次のようにすれば警告を解消できる。

```swift
print("hello \(s as String?)")
print("hello \(String(describing: s))")
```

`Optional()`を外したい場合は普通に予めアンラップしてから使うか、次のように `?? ""` でデフォルト値を設定するなどすれば良い。

```swift
print("hello \(s ?? "")")
```


特に、Swift 3.0 で [SE-0054: `ImplicitlyUnwrappedOptional` 型の廃止](https://github.com/apple/swift-evolution/blob/master/proposals/0054-abolish-iuo.md) によって以前 `ImplicitlyUnwrappedOptional` 型だったものが `Optional` 型に変わった箇所が多く発生したので、警告で追従漏れに気付けるのは地味ながら嬉しい( Swift 3.0 と同時にこの対応がなされていればベストだったが…)。