前編ではSwift 3.0のリリース予定内容について紹介したが、この後編ではそれらの情報をどのように得るのかを紹介する。

# Swift 3の開発がどのように進められているか

以前はSwiftの開発が内部的にどのように進められているかはAppleのコアメンバー以外にはほぼブラックボックスとなっていた。オープンソース化以降は開発に関するあらゆる情報が公開され、体系的にまとめられ、オープンかつ統制の取れた開発が行われている。といっても、どこをどう見れば良いのかという疑問が生じると思うが、基本的には[swift.org](https://swift.org)と[Swift Evolutionリポジトリ]((https://github.com/apple/swift-evolution)の2つに目を通せば良い。

## [swift.org](https://swift.org)とは

[swift.org](https://swift.org)は、2015年12月3日のSwiftオープンソース化と同時に公開され、Swiftに関する情報のハブ的役割を担っている。あらゆる情報が網羅されているが、以下に主要なものを挙げる。それぞれの項目のページ上にその概要の記載があり、さらに詳しい内容はそれぞれ別ページへのリンクがある、という構造である。

- [Swiftとは](https://swift.org/about/)
	- 「Swiftプロジェクトの目標は、システムプログラミング・モバイル/デスクトップアプリ・クラウドサービスなどあらゆる用途に使えるベストな言語を作ることである。」と明記されている
	- 安全・高速・表現豊か、という3つの特徴を挙げている
	- ソースコードは全て[GitHubリポジトリ](https://github.com/apple)で管理されている
- [Swiftブログ](https://swift.org/blog/)
	- [Swiftが発表された2014年6月からあった別ブログ](https://developer.apple.com/swift/blog)はアップデート情報(新機能紹介など)が中心だが、こちらはSwiftの開発状況の展開など、よりコアな内容となっている
- [Swiftでの開発を始めるにあたって](https://swift.org/getting-started/)
	- 特に、Xcodeを使わずコマンドライン経由でビルド・実行する方法が詳しく記載されている
- [ドキュメント](https://swift.org/documentation/)
	- [各ドキュメントがまとめられているページ](https://developer.apple.com/swift/resources/)へのリンクが掲載されている
	- 特に、言語仕様を体系的・網羅的に学びたい時は、[iBooks Store経由で配布されている電子書籍](https://itunes.apple.com/us/book-series/swift-programming-series/id888896989?mt=11)に目を通すことをお勧めする
	- [APIガイドライン](https://swift.org/documentation/api-design-guidelines/)
		- `Swifty`(Appleもセッションや文書で使用おり、Swiftらしいという意味)なコードを書くためのポイントが記載されている
		- 前編で述べた通り、現在Swift 3に向けて改めてSwift自身やコアライブラリがこれに遵守するように刷新中であり、重要なガイドラインである
- [コミュニティガイドライン](https://swift.org/community/)
	- [メーリングリスト](https://swift.org/community/#mailing-lists)
		- Swiftに関するコミュニケーションは主に、カテゴリごとに分けられたメーリングリストを通して行うようになっている
	- [コミュニティの構造](https://swift.org/community/#community-structure)
	  - Apple社がプロジェクトリードかつ裁定者であり、プロジェクトの指揮を執る
	  - Swiftの開発を始めたChris Lattner氏(clattner@apple.com)がプロジェクトリードの代表である
	  - コアチームメンバーが開発指揮・コードオーナーがコード品質の維持など責任を持ち高い権限がある一方、プルリクエスト・レビューなどはSwift開発に関心を持つすべての開発者から受け入れていることなど記載されている
	    - コアチームメンバーは現在6名全員Apple社員であるが、コミュニティの貢献によってApple社外の開発者がメンバーに加わることもあり得る、とのこと
	    - コードオーナーとその担当領域は各レポジトリ直下の`CODE_OWNERS.TXT`に記載されている(例: [Swiftレポジトリのコードオーナー](https://github.com/apple/swift/blob/master/CODE_OWNERS.TXT))
- [Swiftへのコントリビューションガイド](https://swift.org/contributing/)
	- Swiftを知るだけで無く開発に貢献したい場合は、こちらのガイドに従って行う
	- [コードによる貢献](https://swift.org/contributing/#contributing-code)
	  - GitHub上でプルリクエストを送る際の、細かい手順・コーディング規約などが記載してあり、Swift関係無くソフトウェア開発全般に通じる内容も多く参考になる
	- [バグレポート](https://swift.org/contributing/#reporting-bugs)
		- [バグレポートはJiraによって管理](https://bugs.swift.org/)されており、アカウント登録すれば、バグ報告・報告されたバグの閲覧など出来る
	- [Swift発展の活動への参加](https://swift.org/contributing/#participating-in-the-swift-evolution-process)
		- [メーリングリスト](https://lists.swift.org/mailman/listinfo/swift-evolution)で議論を行い、それが収束すると[Swift EvolutionリポジトリのProposal](https://github.com/apple/swift-evolution/tree/master/proposals)としてまとめられる(さらにここからレビューがあり、実際にリリースに含めるかなど決定される)

## Swiftの開発体制

次に、[swift.org](https://swift.org)から読み取れるSwiftの開発体制を見ていこう。

Swift開発の場合、GitHub上で管理されているのは、ソースコード・ドキュメント・各種議論のまとめ、などまとまりのある情報のみである。途中経過の議論などはGitHub上ではなされていない。オープンソースのライブラリ開発は、GitHubのIssue上で機能追加・仕様変更などの議論が進められていることが多いが、[Appleのswift関連のリポジトリ](https://github.com/apple)のほとんどで、[Issueが無効化](https://help.github.com/articles/disabling-issues/)されている。これはおそらく、Swift開発においてやりとりする情報量が多すぎて、シンプルな作りのGitHub Issueでは収集が付かなくなる、という判断がなされたからであろうと思っている。

代わりに、上でも触れた以下の手段が使われている。

- [Jiraによって管理されたバグレポート](https://bugs.swift.org/)
- [目的ごとに分かれたメーリングリスト](https://lists.swift.org)

次期Swiftに向けた提案やそれについての議論などはメーリングリストでなされるが、バグ報告や明らかに対応した方がベターな既存挙動についての改善提案などはJiraのバグレポートでなされている。

[Swift Evolution用のメーリングリスト](https://lists.swift.org/mailman/listinfo/swift-evolution)をチェックすることで、次期Swiftの最新動向の把握が出来る。また、メーリングリストの購読の際、[Hirundo](https://stylemac.com/hirundo/)というmacOS用アプリを使うと、閲覧・検索などしやすく、お勧めである。
		
## Swift Evolutionとは

Swiftのロードマップ・現在の開発ステータスなどの管理の場として、[Swift Evolutionリポジトリ](https://github.com/apple/swift-evolution)がある。

次期Swiftについてのこみ入った議論はメーリングリスト上で行われると説明したがそれが収束すると、Proposalとしてこのリポジトリに反映される。Proposalはソース管理されており、プルリクエストなどを通じて(コアチームメンバーは直接`master`へコミット)、追加・更新されている。さらに、Proposalのステータスが[Swiftの変更提案のレビューステータス追跡サイト](https://apple.github.io/swift-evolution/)上でまとめられており、どれがSwift 3にすでに実装済みか、どれが今レビュー中か、など一目で確認出来る。

### Swift Evolutionに自ら提案していくには？

そして最後に、自ら提案していくにはどのようにすれば良いか、について説明しよう。　
リポジトリ内に、[運用フロー・ルールなどが記載された文書(英語)](https://github.com/apple/swift-evolution/blob/master/process.md)がある。ざっとルールを抜粋すると以下の通りである。

1. これまでされた提案で同様のものが無いかチェック(現在進行中・リジェクトされたものなども含む)
2. メーリングリストにて、提案の概要を説明・展開
3. 議論を経て、提案を洗練させていき、[テンプレート](https://github.com/apple/swift-evolution/blob/master/0000-template.md)を使ってまとめる
4. コアチームメンバーにレビュー対象としてもらえるように、Swift EvolutionリポジトリにPull Requestを投げる
5. Pull Requestが通ったら、定められた期間にレビューを受け、採用・不採用の判断がなされる(採用の場合、優先度に応じて順次実装されていく)

このように提案が採用されるまでは長い道のりであるが、今後ともSwiftをオープンソースプロジェクトで一貫性のある優れた言語として円滑に進化させていくにあたって、こういった明確なフローが定められているのは重要である。また、例えばメーリングリストを読んでいて気になった点について返信するなどもフィードバックという大事な貢献であるので、そういう小さいところからSwiftプロジェクトに参加していくのも良いであろう。

初めにチェックするべきものである、過去にリジェクトされた提案だが、[その代表例がまとめられたページもある](https://github.com/apple/swift-evolution/blob/master/commonly_proposed.md)。例えば、[Python](https://www.python.org)のように[{}をタブで表現する提案](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151214/003656.html)や[`&&`・`||`を`and`・`or`にする提案](https://lists.swift.org/pipermail/swift-evolution/2015-December/000032.html)などあり、その不採用理由を見ることによって、Swift開発コミュニティの考えなども読み取れて面白い。基本的に、「今からSwift言語にその仕様変更を加えた場合、そのメリットがデメリットを充分上回るか」という一貫した基準を満たすかどうかで判断されているように見える。あくまでここに載っているのはサマリーであるが、メーリングリストを検索するなどして、過去の実際の議論をすべて見ることも出来る。

# まとめ

後編では、前編で述べたような情報が[swift.org](https://swift.org)・[Swift Evolutionリポジトリ]((https://github.com/apple/swift-evolution)などで事細かにまとめられていることや開発事情などを紹介した。この第2回まではほとんどSwiftコードも出さずに全体的な動向の説明だったが、次回以降の連載でSwift 3でなされる変更から注目すべき点を取り上げて具体的に紹介していく。