タイトル(仮): Swift 3.1 のリリースプロセスおよびそれに含まれる変更内容の紹介(後編)

---

# Swift 3.1 の主な変更一覧

[SwiftリポジトリのCHANGELOG](https://github.com/apple/swift/blob/master/CHANGELOG.md#swift-31)・[Swift EvolutionリポジトリのProposal Status](https://apple.github.io/swift-evolution/)を見ると、開発状況が分かる。以下に12月13日時点で最新の `DEVELOPMENT-SNAPSHOT-2016-12-14-a` ツールチェインでの実装状況を記す。

## 実装済み

- [SE-0045: `Sequence` プロトコルに `prefix(while:) and drop(while:)` が追加](https://github.com/apple/swift-evolution/blob/master/proposals/0045-scan-takewhile-dropwhile.md)
- [SE-0141: `@available`構文においてのSwift バージョン分岐](https://github.com/apple/swift-evolution/blob/master/proposals/0141-available-by-swift-version.md)
- [SR-1009: 具体的な型を用いたジェネリクス制約](https://bugs.swift.org/browse/SR-1009)
- [SR-1446: ジェネリクスの入れ子](https://bugs.swift.org/browse/SR-1446)
- [SR-1882: 文字列補間にOptional型を直接用いると警告](https://bugs.swift.org/browse/SR-1882)

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

# Swift 3.1 のToolchains版を使えるようにする

TODO消すかも

Xcode の設定で Toolchain を変更して Playground で試しても良いですが、せっかくなので前編でとりあげた [kylef/swiftenv](https://github.com/kylef/swiftenv) を使ってみましょう。

以下では、元々コマンドライン上の Swift が Xcode 8.1 付属の Swift 3.1 だったのを `swiftenv local DEVELOPMENT-SNAPSHOT-2016-12-14-a` を実行することで、Toolchains版を使えるように変更しています。バージョイン表記が `3.1-dev` などではなく `3.0-dev` となっていますが、今後の版では変わるかもしれません。

```sh
$ sample git:(swift-3.1) ✗ swiftenv local DEVELOPMENT-SNAPSHOT-2016-12-14-a
$ sample git:(swift-3.1) ✗ swift --version      
→ Apple Swift version 3.0-dev (LLVM a00a468bc7, Clang d6d5b695de, Swift ca165e5c4a)
Target: x86_64-apple-macosx10.9
```

というわけで、以下 `DEVELOPMENT-SNAPSHOT-2016-12-14-a` の Swift を使って、Swift 3.1 の変更内容をなぞっていきますが、これ以降のSnopshots版であれば動くはずです。

また、[IBM Swift Sandbox](https://swiftlang.ng.bluemix.net/)でも、すでに同等のバージョンである`Dev Build (Dec 9, 2016)`を選択出来るので、よりお手軽にWeb上で試すことが出来ます。この IBM Swift Sandbox に関しては、[Swift 3.1 を先取り👀 - Qiita](http://qiita.com/mono0926/items/0667f6b99adeaec1e231) にて詳しく触れています。

# Swift 3.1 の変更を先取り

## [SE-0045: `Sequence` プロトコルに `prefix(while:) and drop(while:)` が追加](https://github.com/apple/swift-evolution/blob/master/proposals/0045-scan-takewhile-dropwhile.md)

5月に作られた Proposal で、番号も古めです。


[`Sequence` プロトコル](https://developer.apple.com/reference/swift/sequence)にはすでに

- `dropFirst(_:)`
- `filter(_:)`

forの方が効率良いので、それベタ書き