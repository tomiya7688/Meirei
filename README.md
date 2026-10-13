# Meirei

**Meirei（命令）** は、プログラムを「命令」を中心に記述することを目指す、命令志向プログラミング言語（Command-Oriented Programming Language）のプロジェクトです。

> **開発状況：設計初期段階**  
> この README の構文例と変換例は構想を示すものです。仕様は未確定であり、現時点で動作するコンパイラがあることを意味しません。

## 基本思想

Meirei では、変数・定数・関数・クラス・制御構造などを、できるだけ一貫した**命令名と引数**で表現します。

重視するのは、プログラムの「ひとかたまり」が分かりやすいことです。開始や終了を意味のある命令名で表すことで、構造を読み取りやすくします。

- **命令が基本単位**：構造の定義も処理の実行も命令として記述する。
- **構造が明確**：`Class_start` や `Variable_start` など、何を開始したかが命令名から分かる。
- **記述は簡潔に**：省略できる情報や終了命令は、プリプロセッサが必要に応じて補完する。
- **オブジェクト指向も命令で構成**：クラスなどを、別の根本原理ではなく命令体系の上で表現する。

## 構文のイメージ（暫定）

例えば、`player` クラスに `hp` を定義します。

```text
{Class_start [name=player]}
{Variable_start [name=hp type=int]}
{Class_end}
```

定数には、意図を読み取りやすい `Const_start` のような専用命令を用意する方針です。

```text
{Const_start [name=max_hp type=int value=100]}
```

必要な場合には終了を明示できます。

```text
{Function_start [name=example]}
{Variable_start [name=temp type=int]}
{Variable_end [name=temp]}
{Function_end}
```

### `start` / `end` とプリプロセッサ

`*_start` で始まった要素に明示的な `*_end` がない場合は、**所属する上位の要素が終了するときにプリプロセッサが補完する**ことを想定しています。

先ほどのクラスの例は、概念的には次のように正規化できます。

```text
{Cmd=Class_start, name=player}
{Cmd=Variable_start, name=hp, type=int}
{Cmd=Variable_end, name=hp}
{Cmd=Class_end, name=player}
```

`Variable_end` による明示的な終了・破棄も設計対象です。ただし、**クラスのフィールド定義の終了**と**実行時の変数の寿命・破棄**をどう区別するかなど、詳細な意味論は今後定義します。

公開範囲や状態などの属性を省略してプリプロセッサで補うことも想定していますが、デフォルト値はまだ確定していません。

## 二つの命令表記

現在、次の二つの近い表記を検討しています。

**読み書きしやすい表記**

```text
{Variable_start [name=hp type=int]}
```

**命令名をデータとして明示する表記**

```text
{Cmd=Variable_start, name=hp, type=int}
```

この二つを別の言語として扱うか、ソース表記と正規表記として扱うかを含め、実装方式は検討中です。いずれの場合も、コンパイラへ渡す前に命令を正規化することを基本方針とします。

## コンパイル方針

最初の実装では **Cへの変換**を目指します。

```text
Meirei source
    ↓
Preprocessor（省略の補完・正規化）
    ↓
Normalized commands
    ↓
Analysis / Internal representation
    ↓
C code generator
    ↓
C compiler
    ↓
Executable
```

例えば、先ほどのクラス定義は C では概念的に次のように表せます。

```c
typedef struct {
    int hp;
} player;
```

将来的には **Bitlang への変換**も追加したいと考えています。

```text
Meirei
  ├── C       （初期目標）
  └── Bitlang （将来の目標）
```

[Bitlang](https://github.com/tomiya7688/Bitlang) は別の独立した言語プロジェクトです。Meirei の完成に Bitlang の完成を必須とせず、Meirei 自体を Bitlang と同じ厳格さにすることも目的ではありません。Bitlang 固有の詳細は、必要に応じて将来の変換処理で扱います。

## 当面の開発目標

1. 命令の種類、引数、開始・終了・所属関係を定義する。
2. 人間向けの記述を読み取り、省略を補完するプリプロセッサを作る。
3. 正規命令形式と内部表現を定める。
4. Cコード生成器を実装し、簡単なプログラムを動かす。
5. 言語仕様と実装が安定した後、Bitlangへの変換を検討する。

---

**Meirei の中心にある考え方はシンプルです。**  
**「プログラムの基本単位を命令として扱い、何が始まり、何が終わるのかを分かりやすくする」。**
