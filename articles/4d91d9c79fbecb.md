---
title: "ABC253 C - Max - Min Query Python解答例"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["競技プログラミング", "競プロ", "cpp", "AtCoder"]
published: true
---

AtCoder Beginner Contest 253 C - Max - Min QueryをPythonで解きます。

## 問題

問題文を[AtCoderのページ](https://atcoder.jp/contests/abc253/tasks/abc253_c)より引用します。

### 問題文

> 整数の多重集合$S$があります。はじめ$S$は空です。
>
> $Q$個のクエリが与えられるので順に処理してください。
> クエリは次の$3$種類のいずれかです。
>
> - `1 x` : $S$に$x$を$1$個追加する。
> - `2 x c` :$S$から$x$を$\min(c,$ ($S$に含まれる$x$の個数$))$個削除する。
> - `3` : ($S$の最大値)−($S$の最小値)を出力する。このクエリを処理するとき、$S$が空でないことが保証される。

### 制約

> - $1 \leq Q \leq 2\times 10^5$
> - $0 \leq x \leq 10^9$
> - $1 \leq c \leq Q$
> - `3`のクエリを処理するとき、$S$は空でない。
> - 入力は全て整数

## 解答例

### 座標圧縮 + セグメント木

この問題の想定解は順序付き多重集合(C++では[std::multiset](https://cpprefjp.github.io/reference/set/multiset.html))を使ったものになります。
が、Pythonには順序付き多重集合が標準ライブラリで用意されていないので、代替のデータ構造を使う必要があります。

[ユーザー解説](https://atcoder.jp/contests/abc253/editorial/4040)では、平方分割によるSortedSetを用いるもの、ヒープを用いるもの、BITを用いるものなどが紹介されています。

ここでは座標圧縮 + セグメント木を用いて解いてみます。

### 使用するデータ構造

セグメント木では多重集合$S$の元の最大値と最小値を管理することとします。
$S$の元のそれぞれの個数は辞書(`defaultdict`)で管理することにします。

従って、この解法では以下の3つのデータ構造を使用することになります。

1. $S$の元の最小値を管理するセグメント木
1. $S$の元の最大値を管理するセグメント木
1. $S$の元の個数を管理する`defaultdict`

### 下準備とクエリ処理

セグメント木に乗せる要素を定義します。
まず、クエリによって与えられる全ての$x$を座標圧縮し、「全体の中で何番目に小さいか」を求めておきます(これを$i$と表します)。
こうした後、もし集合$S$に$x$が存在するならば、セグメント木の$i$番目の葉に$x$を乗せます。
このようにすることで、セグメント木全体の積を取ったとき、集合$S$の中の最小値(あるいは最大値)を取得できるようになります。

集合$S$に$x$が存在するかどうかは、`defaultdict`で管理します。この辞書を$D$とおきます。
クエリ1が来たときは$D[x]$を1つ増やし、クエリ2が来たときは$\min(c, D[x])$を$D[x]$から減算します。
もし$D[x]$が$0$になったときは、集合$S$から$x$が無くなったことになるため、対応するセグメント木の葉の値を単位元に戻しておきます。

このようにすることで、この問題を$\mathcal{O}(Q\log{Q})$で解けるようになりました。

### 実装

以上のアルゴリズムを実装したのが以下のコードです。
セグメント木の実装部分は長くなってしまったので、[提出したコード](https://atcoder.jp/contests/abc253/submissions/32085418)を参照してください。

```python:c.py
from typing import List, Tuple, Deque, Set, Dict, TypeVar, Callable, Generic
import sys
import collections
import itertools
import bisect
import math


sys.setrecursionlimit(1000000)
input = sys.stdin.readline


### セグメント木の実装(省略)


def compress(A: List[int]) -> List[int]:
    """座標圧縮"""
    X: List[int] = sorted(set(A))
    D: Dict[int, int] = {x: i for i, x in enumerate(X)}
    return list(map(lambda x: D[x], A))


def main():
    Q: int = int(input())
    query: List[List[int]] = [list(map(int, input().split())) for i in range(Q)]

    # クエリ先読み
    X: List[int] = []
    for q in query:
        if q[0] == 1 or q[0] == 2:
            X.append(q[1])

    # 座標圧縮
    Y: List[int] = compress(X)
    # xからiへの変換のための辞書
    x_to_i: Dict[int, int] = {x: y for x, y in zip(X, Y)}

    # 集合Sの元の最大値を管理するセグメント木
    seg_max = SegmentTree[int](lambda a, b: max(a, b), lambda: 0, [0] * len(Y))
    # 集合Sの元の最小値を管理するセグメント木
    seg_min = SegmentTree[int](
        lambda a, b: min(a, b), lambda: 1 << 60, [1 << 60] * len(Y)
    )
    # 多重集合Sの元の個数を管理するための辞書
    counter = collections.defaultdict(int)

    # クエリ処理
    for q in query:
        # クエリ1
        if q[0] == 1:
            t, x = q
            # xが全体で何番目に小さいかのインデックスの取得
            i: int = x_to_i[x]
            # 集合Sにxが追加されたので、セグメント木を更新する
            seg_max.set(i, x)
            seg_min.set(i, x)
            # 元の数を1増やす
            counter[x] += 1
        # クエリ2
        elif q[0] == 2:
            t, x, c = q
            # xが全体で何番目に小さいかのインデックスの取得
            i: int = x_to_i[x]
            # 指定された数だけ元の数を減らす
            counter[x] -= min(c, counter[x])

            # 集合Sからxが無くなったら、セグメント木の葉の値を単位元に戻す
            if counter[x] == 0:
                seg_max.set(i, 0)
                seg_min.set(i, 1 << 60)
        # クエリ3
        else:
            # 各セグメント木の全体の積が最大値と最小値を表す
            print(seg_max.prod(0, len(Y)) - seg_min.prod(0, len(Y)))


if __name__ == "__main__":
    main()

```

実際の提出は[こちら](https://atcoder.jp/contests/abc253/submissions/32085418)。
