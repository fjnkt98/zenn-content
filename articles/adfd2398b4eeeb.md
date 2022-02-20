---
title: "ABC239 B - Integer Division C++解答例"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["競技プログラミング", "競プロ", "cpp", "AtCoder"]
published: true
---

AtCoder Beginner Contest 239 B - Integer Division を C++で解きます。

## 問題

問題分を[AtCoder のページ](https://atcoder.jp/contests/abc239/tasks/abc239_b)より引用します。

### 問題文

> $-10^{18}$以上$10^{18}$以下の整数$X$が与えられるので、$\lfloor \frac{X}{10} \rfloor$を出力してください。

### 制約

> - $-10^{18} \leq X \leq 10^{18}$
> - 入力はすべて整数である。

## 解答例

### 床関数と天井関数の性質を利用する

C++11 以降では、整数の割り算は「商の小数部がゼロ方向に切り捨てられた結果となること」と規定されています。[^1]
そのため、被除数$X$が正ならば単純に割り算をすることで床関数と同じ挙動を示しますが、$X$が負の場合は単純な割り算は天井関数と同じ挙動を示すことになります。

そこで、$X$の正負で場合分けをすることで解答を試みます。
床関数および天井関数の性質として、以下があります。[^2]

$$
\begin{align}
\lceil x \rceil &= - \lfloor -x \rfloor \\
\lfloor x \rfloor &= - \lceil -x \rceil \\
\end{align}
$$

$X$の床関数の値$\lfloor X \rfloor$は、$X$の符号を反転させたものに天井関数を適用したあと、結果の符号を反転させたものと等しくなることがわかります(その逆も然り)。
これをプログラムで実装すればよいです。

### プログラム実装例

C++のプログラム実装例を以下に示します。
なお、床関数と天井関数は標準ライブラリ`<cmath>`に用意されていますが、これらの関数は浮動小数点型の変数を引数に取るため、およそ$10^{15}$までの値しか正確に扱うことができません。
正の整数に対する床関数は、単純に割り算を行うだけで実現することができますが、天井関数は少し工夫する必要があります。
天井関数を整数同士の演算のみで行う方法として、以下のイディオムがよく使われます。

```cpp
int64_t a, b, x;

x = (a + b - 1) / b;
```

$x = \lfloor \frac{a}{b} \rfloor$を表現するために、被除数`a`に除数`b`を足して$1$を引いてから`b`で割ります。これにより、天井関数を適用したときと同じ結果を得ることができます。[^3]

```cpp: b.cpp
#include <iostream>

int main() {
  int64_t X;
  std::cin >> X;

  // Xの符号で場合分け
  if (X < 0) {
    // Xの符号を反転させた値Yを用意する
    int64_t Y = -X;
    // Yに対して天井関数を適用し、その結果の符号を負にした値を出力する
    // 天井関数のイディオム
    std::cout << -((Y + 10 - 1) / 10) << std::endl;
  } else {
    std::cout << X / 10 << std::endl;
  }

  return 0;
}
```

実際に提出したコードは[こちら](https://atcoder.jp/contests/abc239/submissions/29490730)。

[^1]: [整数に対する除算と剰余算の丸め結果を規定 cpprefjp](https://cpprefjp.github.io/lang/cpp11/result_of_integer_division_and_modulo.html)
[^2]: [床関数と天井関数 Wikipedia](https://ja.wikipedia.org/wiki/%E5%BA%8A%E9%96%A2%E6%95%B0%E3%81%A8%E5%A4%A9%E4%BA%95%E9%96%A2%E6%95%B0)
[^3]: [【競プロ】切り捨てと切り上げ なかけんの数学ノート](https://math.nakaken88.com/textbook/cp-round-down-and-round-up/)
