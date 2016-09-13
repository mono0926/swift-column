第1回では、 2015年12月3日にSwiftがオープンソース化されたことを紹介したが、それと同じタイミングで[Swift Evolotionリポジトリ](https://github.com/apple/swift-evolution)にてSwift 3のロードマップも公開された。この第2回では、Swift 3.0リリースに向けての動向を紹介する。

# Swift 3.0のリリース予定

Swift 3.0リリース予定が[Swift EvolutionのREADME](https://github.com/apple/swift-evolution/blob/master/README.md)に明記されている。リリース予定日は「今年後半」と書かれているが、毎年9月に新しいiOS・Xcodeの正式版をリリースするのが恒例となっているので、よほどのことが無い限りSwift 3.0の正式リリースも9月となるだろう。[7月の「Endgame for Swift 3」と題したメーリングリストの投稿(英語)](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160711/024424.html)でも、Swift 3の破壊的変更は7月27日に終えるという宣言がされており、今はリリースに向けた最終調整が進んでいる模様である。

Swift 3.0開発においてフォーカスされた点は以下である。

- [APIデザインガイドライン](https://swift.org/documentation/api-design-guidelines/)をSwiftとしてあるべきものに更新
  - Swiftでコードを書く時のためのガイドラインでもあり、またSwiftライブラリ自体もこれに沿うように改めて統一をはかっている
- インポートされたObjective-C APIに、ガイドラインに沿った名前規則への自動変換の適用
  - Objective-Cでそのガイドラインに沿って書かれたAPIを、Swiftで参照した際に、Swiftらしいインターフェースに[規約ベースで自動変換](https://github.com/apple/swift-evolution/blob/master/proposals/0005-objective-c-name-translation.md)
  - 自動変換の結果が所望のもので無い場合など、[NS_SWIFT_NAME](https://developer.apple.com/reference/foundation/ns_swift_name)で明示的にSwift用の名前を指定することも可能
- Swift標準ライブラリ・コアライブラリを、Swift APIガイドラインに沿うように変更
  - 参照型のセマンティクスを持つNSプレフィックス付きのFoundationクラスはSwiftにそぐわないので、[それに対応した構造体型を導入](https://github.com/apple/swift-evolution/blob/master/proposals/0069-swift-mutability-for-foundation.md)(`NSData`クラスに対して`Data`構造体を導入、など)
  - libdispatch(マルチスレッド処理用のコアライブラリ)のインターフェースを[Swiftらしいインターフェースに変更](https://github.com/apple/swift-evolution/blob/master/proposals/0088-libdispatch-for-swift3.md)  
- インポートしたObjective-C APIの`Swiftification`(Swiftらしく加工すること)
  - [型パラメーター付きのObjective-CジェネリクスのSwiftへのインポート対応](https://github.com/apple/swift-evolution/blob/master/proposals/0057-importing-objc-generics.md)
  - [インポートされたC言語ライブラリ・フレームワークを、オブジェクト指向型のインターフェースに変換](https://github.com/apple/swift-evolution/blob/master/proposals/0044-import-as-member.md)
  - [Objective-Cで定義された定数をSwiftの列挙型に変換](https://github.com/apple/swift-evolution/blob/master/proposals/0033-import-objc-constants.md)
  - [Selectorを安全に扱える構文](https://github.com/apple/swift-evolution/blob/master/proposals/0022-objc-selectors.md)(2.2で導入済みで、古い構文は2.2で非推奨・3.0で廃止という扱い)
  - [KVOなどで必要になるkey pathを文字列リテラルではなく#keyPathで安全に扱えるように](https://github.com/apple/swift-evolution/blob/master/proposals/0062-objc-keypaths.md)
- 言語仕様の再考・見直し
  - 3.0を最後の大きな破壊的変更を含むリリースとするために全面的に見直し
  - [引数のラベルの仕様の統一化](https://github.com/apple/swift-evolution/blob/master/proposals/0046-first-label.md)など
- ツール品質向上
  - コンパイラー・IDEの品質・速度向上など

今回はそれぞれ箇条書きの簡単な紹介にとどめたが、細かいところは今後の連載で補足していく予定である。

## Swiftは3.0で安定するのか？

現最新版のSwift 2.2まではソースコードの破壊的変更に繋がる言語仕様変更が多く、それが今後どうなるのかは気になるところだと思う。これも[Swift Evolotionリポジトリ](https://github.com/apple/swift-evolution)などで公開されている情報から詳しく読み解くことが出来る。

### Swift 3.0以降もまだ破壊的な仕様変更は続く

まずSwift 3.0では、それ以降(Swift 3.xや4以降)で合理的に可能な限り互換性を保つようにする、と記載されている。言い換えると、バージョンが上がる度にコンパイルエラーが発生するようなことは少なくなるがゼロとは言えない、ということである。

Swiftは発表当初から各種言語の良いところ取りの優れた言語仕様であり、さらにその後の約2年で予想を上回る開発速度でさらに完成度を上げてきていると感じている。この開発速度は、これまで後方互換性を大胆に切り捨ててきたからこそなせる業であると思っている。例えばC#は後方互換性がかなりしっかりしている言語である(参考: [見えてきたC# 7： C#の短期リリースサイクル化 - Build Insider](http://www.buildinsider.net/column/iwanaga-nobuyuki/008))が、それはつまりそれに労力をかけている、ということである。Swiftの場合は、その労力をフルに次のバージョンへの開発に向けられてきた。

ただ、そのトレードオフとして、今までは開発者にとって負担となることが多かった。キャッチアップしていく負担が大きめであったり、書籍やインターネット上のコード例がすぐに古くなってしまい、最新版では警告やコンパイルエラーになってしまうことも多い(個人的にはこういう作業含めて楽しめているが)。これまではやはりアーリーアダプター向けの言語という感じが否めないところもあった。

まだ仕様変更が続くのか、と残念に思う人もいるかもしれないが、筆者はSwiftは3.0の段階でもまだ大きな進化の途中だと感じており、今の段階で無理に後方互換性を保つことを優先して言語仕様に妥協が生まれるよりずっと良いことだと思っている。これはトレードオフの問題であり、そういった仕様の破壊的変更が一切許容出来ない場合は、現状では枯れた言語(iOSアプリ開発の場合はObjective-C)を選択するしか無いであろう。とはいえSwiftはバージョン3.0を境に、これまでと比べるとずっと安定していくように見受けられるので、多少敬遠していた慎重な開発者にも薦められる時期ではあると思う。

### ABIの安定はSwift 4に持ち越しされた

ABI(Application Binary Interface)とは、プログラムのモジュール間のバイナリレベル(オブジェクトコードレベル)でのインタフェースのことである。Swift言語自体のバージョンが異なっても、この互換が保障されていればコンパイルされたバイナリはリンク出来る。ABIを安定させるには、実行時のデータ構造・呼び出し規約・ABIに影響を与えるであろう言語機能などを確定させる必要がある。2015年12月にSwift 3.0のマイルストーンが初めて提示された時点では、[ABIの安定対応は含まれていた](https://github.com/apple/swift-evolution/commit/06b69a6e51a71a462c268da60b51a18966dba31b)が[今年の5月に3.0での見送ると判断された](https://www.infoq.com/jp/news/2016/06/swift-3-no-stable-abi)。さらにこの後、ABIの安定化をSwift 4で最優先の達成事項として取り組むことが、[Swift 3の開発の振り返りとSwift4の計画が記されたメーリングリストの投稿(英語)](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160725/025676.html)にて発表された。Swift 4に向けた議論はSwift 3開発と並行して8月1日からスタートしており、またABI安定化とともに、ソースの後方互換性の維持も取り組む、とのことである。(筆者による日本語での紹介記事: [Swift 3の開発の振り返りとSwift 4の計画が記されたメールの紹介 - Qiita](http://qiita.com/mono0926/items/c51b74fae679e1e39d2b))

### ABI互換が無い現状の問題点

つまり、現時点の2.2でも3系でもABIが安定しないので、Swiftのバージョンが上がると、開発中のアプリケーションで以前のバージョンでビルドしたライブラリを利用している場合コンパイルエラーが発生してしまう。このコンパイルエラー解決のためには、同じバージョンのSwiftで古いライブラリをビルドし直す必要がある。OSS(Open Source Softweare, オープンソースソフトウェア)ライブラリを使っている時など、そのライブラリが所望のバージョンのSwiftに対応した版があればそのソースコードを利用してリビルド出来るが、そうでない場合自らソースコードを対応させる必要が生じる。Swiftの場合バージョンアップサイクルが早いこともあり、最新やベータ版のSwiftを使おうとすると使用ライブラリがまだ対応しておらずそのような手間がかかることが多い。とはいえ、Swiftの定番人気OSSライブラリ([Alamofire](https://github.com/Alamofire/Alamofire)・[RxSwift](https://github.com/ReactiveX/RxSwift)などGitHubスター数が数千以上程度のもの)であれば、大抵はかなり早いタイミングでSwiftの最新版用のブランチでの対応が行われているのでそれでビルドし直して解決することも多い。Swiftライブラリを使う場合、このようにアップデートが迅速であるか、あるいはそうでない場合いざとなれば自力でコードの修正を施せるかどうか、などを視野に入れて選定する必要がある。

### ABIが互換すればSwiftで書いたアプリバイナリの削減効果もある

また、ABIが安定すれば、アプリバイナリにSwiftのランタイムバイナリを含んで配布する必要が無くなる、というメリットもある。上の例ではバージョンの違うSwiftコンパイラーでビルドしたアプリケーションとライブラリのリンクの際にABI互換の必要があるということだったが、ABIはアプリケーションバイナリとその実行環境についての互換性にも同様に関係する。ABIが安定すればSwiftのランタイムバイナリをアプリに含まずOSに同梱出来るようになる。具体的には現在、同じようにコーディングしたシンプルなアプリで、SwiftはObjective-Cで記述したものと比べて10MB近くバイナリサイズが大きくなってしまうが、ABI安定によりこれが解消されるはずである。

以上をまとめると次の通りである。

- Swift 3.0まで: 積極的な仕様変更・後方互換性切り捨て
- Swift 3.x: 互換性をなるべく保つようにはする
  - バージョンアップの度に毎回コード対応が必要な場面は減りそう
  - とはいえ、ABIは変更されていくので、古いバージョンでビルドしたライブラリの扱いに手間がかかる状態は継続
- Swift 4.0: ABI安定・ソースコード後方互換性が最優先事項
  - 当初は3.0で予定されていたが延期された

初めの2年で高速に言語の完成度を高めて、その後の1年で互換性担保にフォーカス、というかなり効率の良い開発となっているように感じる。

# 前編のまとめ

前編では、Swift 3.0が間近に迫った中、いつどのような内容を含んでリリースされるかについての最新情報を伝えた。こういった情報はすべてオープンになっていて誰でも深いところまで調べられる状態であり、後編ではそれら情報がどうまとめられているのかを紹介する。