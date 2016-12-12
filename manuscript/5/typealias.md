# メモ


伏せるか、書くとしてももう少し柔らかく:
[inaka / Function Naming In Swift 3](http://inaka.net/blog/2016/09/16/function-naming-in-swift-3/)の`typealias`の使い方を見て、アンチパターンだと思った。
ただ、もうちょっと調べてから、デメリットの多い使い方と述べたい。
良い例: [Go言語におけるエイリアス型を使ったパターン - Qiita](http://qiita.com/tenntenn/items/c3afc87a20d9f50998bb)
 これはnamedかどうかなのか, [Why can I type alias functions and use them without casting? - Stack Overflow](http://stackoverflow.com/questions/19334542/why-can-i-type-alias-functions-and-use-them-without-casting)

どっちが良いかということではなく、仕様に合わせて適切に使う


// 別解でリテラル代入のやつも

---

# 基本型にtypealiasで別名付ける？

## 可読性向上？

```swift
typealias Path = String      // To the rescue!

func request(for path: Path) -> URLRequest {  }
let request = request(for: "local:80/users")
```

- `path`であることは引き数名に表現されているので、Swift 3で避けられている冗長な表現になっててむしろマイナス
- `Path`型って何？→定義を見る→ただのStringじゃん


## 堅牢性アップ？

