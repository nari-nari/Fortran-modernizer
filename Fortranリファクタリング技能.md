**You:**

Fortranのリファクタリング・python化をcopilotで進め、さらにその過程で他のコードにも流用可能な成果物を作るには、まずどのようなスキルを用意し、どのようなプロンプトから始めれば良いでしょうか？とりあえず、superpowersをスキルとして用意し、リポジトリにレガシーコードを置いた状態で、Claude Sonnet 5のような中程度のモデルを選択した上で、プロンプトを打てば良い？

* * *

**ChatGPT:**

はい。**「Superpowersを入れる → レガシーFortranをリポジトリに置く → Claude Sonnet 5を選ぶ → 最初のプロンプトを投げる」**で開始してよいです。

ただし、今回の目的は単なる1本のFortranコードの改修ではなく、

**レガシーFortran解析 → 安全なリファクタリング → Python移植 → その過程を一般化して次のコードにも使えるツール／skillを残す**

ことなので、最初からその意図をプロンプトに明記するのが重要です。

### 最初に用意するもの

私は最初からskillを10個も20個も入れず、次の最小構成にします。

| 要素 | 最初から用意 | 役割 |
| --- | --- | --- |
| Superpowers | ○ | brainstorming、planning、TDD、debugging、subagent-driven development |
| `.github/copilot-instructions.md` | ○ | このリポジトリで常に守るルール |
| `legacy-fortran-modernization` skill | △ | 最初は薄く作るか、1回目の分析後に作る |
| Fortran解析専用skill群 | × | 実作業から抽出して後で作る |
| Python移植skill | × | Fortran側の挙動固定後に作る |
| repository-management skill | 後で | プロジェクトが育ってから追加 |

Copilotでは、常時適用したいルールはcustom instructions、特定作業でだけ必要な詳細手順はAgent Skillsに分けるのが公式にも推奨されています。 [GitHub Docs+1](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills?utm_source=chatgpt.com)

特にSuperpowersには現在、**skillそのものを作成・テストするための `writing-skills`** が含まれています。「skillなしで失敗するケースを確認→skillを作る→改善を確認」という考え方なので、今回の「作業しながら汎用skillを育てる」という目的と非常に相性がよいです。 [GitHub](https://github.com/obra/superpowers/blob/main/skills/writing-skills/SKILL.md?plain=1&utm_source=chatgpt.com)

### 今回は「最初から完成したFortran skill」を作らない方がよい

ここは結構重要です。

例えば最初から、

`fortran-refactoring`  
`common-block-converter`  
`implicit-none`  
`fortran-to-python`  
`fixed-to-free`  
`fortran-testing`

などを一生懸命設計してしまうと、実コードに当てたときに「想定していた問題と違う」ということが起こります。

今回のコードなら、実際には以前出てきたような

COMMONブロックの宣言不整合、オーバーレイ、CALL引数のkind不一致、IMPLICIT依存、固定形式特有の構文、include、fparserで解析できない箇所

などが重要です。こうした**実際に遭遇した問題への解法を、成功後に一般化する**方が良いskillになります。

したがって、流れはこうするのがおすすめです。

1. Superpowersだけでコードベースを分析する。
2. 「このコード固有の問題」と「他Fortranにも使える問題」を分離する。
3. 汎用部分についてスクリプト・テスト・手順を作る。
4. 2回以上使えそうなものをAgent Skillへ昇格させる。
5. 最終的に別のレガシーコードに対する再現テストをする。

これなら「今回だけ動く魔改造スクリプト」ではなく、実際に資産になります。

---

## 最初のプロンプト

Sonnet 5を選択して、リポジトリのルートからAgent modeで、まずこれを投げるのをおすすめします。

Use the Superpowers workflow for this project.

This repository contains a legacy Fortran codebase that I ultimately want to:

1. understand and characterize,
2. lock down its current numerical behavior with tests,
3. safely refactor and modernize,
4. convert from fixed-form / legacy Fortran toward modern Fortran where appropriate,
5. migrate the validated implementation to Python,
6. verify numerical equivalence between the original Fortran, modernized Fortran, and Python implementations.

There is an additional project goal:

Do not treat this only as a one-off migration.

As we work, identify analyses, scripts, tests, transformation procedures, validation methods, templates, and workflows that are general enough to reuse on other legacy Fortran repositories.

Reusable artifacts should be separated from code that is specific to this application.

When a workflow has been demonstrated to be useful and repeatable, consider turning it into a reusable Agent Skill using the Superpowers skill-authoring methodology.

For now, do NOT begin large-scale rewriting.

Start by exploring the entire repository and building an evidence-based understanding of the codebase.

Specifically:

* identify programs, subroutines, functions, modules, INCLUDE files, COMMON blocks, EQUIVALENCE usage, implicit typing, global state, file I/O, external dependencies, and call relationships;
* identify constructs that make refactoring risky;
* identify compiler warnings or errors that can expose latent inconsistencies;
* determine how the current program can be built and executed reproducibly;
* identify representative input/output cases that can serve as behavioral regression tests;
* distinguish application-specific logic from reusable modernization tooling;
* propose a staged modernization and Python-migration architecture;
* identify candidate reusable tools or Agent Skills, but do not create speculative skills until their usefulness has been demonstrated.

Preserving numerical behavior is the highest priority.

Prefer small, testable transformations over large rewrites.

The original implementation should remain available as a reference implementation until numerical equivalence has been demonstrated.

Please use the Superpowers brainstorming and planning workflow.

Before implementation, produce:

* a repository architecture / dependency assessment,
* a risk register,
* a proposed behavioral-lock testing strategy,
* a staged migration plan,
* and a proposed directory structure for application-specific versus reusable artifacts.

Then proceed through the Superpowers workflow.

これが「最初の一手」としてかなり良いと思います。

### Sonnet 5でいいのか

**はい。最初の親エージェントとしてSonnet 5はかなり適しています。**

現在のCopilotではClaude Sonnet 5が正式にサポートされており、推論レベルも変更できます。 [GitHub Docs+1](https://docs.github.com/en/copilot/reference/ai-models/supported-models?library=true&utm_source=chatgpt.com)

今回なら私は、

**最初のコードベース解析・設計：Sonnet 5 + 標準〜高推論**

から始めます。

10,000行程度のFortranなら、いきなり最大コンテキストを使う必要性もそれほど高くありません。まず通常コンテキストでリポジトリ検索やファイル読み取りをさせる方がよいです。Copilotの公式ドキュメントも、通常は標準コンテキスト・標準推論を使い、複雑な問題でだけ拡張することを推奨しています。 [GitHub Docs](https://docs.github.com/en/copilot/reference/ai-models/supported-models?authuser=0&utm_source=chatgpt.com)

逆に、**一番最初の設計だけを安価な小モデルに任せるのはおすすめしません**。ここでCOMMONやデータフローを誤認すると、その後の数十タスクが全部間違った方向に進むからです。

一旦設計が固まった後なら、

`COMMON一覧を生成`  
→ 軽量モデル

`80サブルーチンにIMPLICIT NONEを段階導入`  
→ 中程度モデル

`COMMON→module化の依存関係設計`  
→ Sonnet 5 / Sol

`FortranとPythonの数値差異の原因解析`  
→ Sonnet 5 / Sol

のように切り替える方が合理的です。

---

## 最終的にはこういうskill群が自然に生まれる

最初から作るのではなく、今回の開発の**成果物として**、例えばこのあたりが育ってくるのが理想です。

| Skill | 内容 |
| --- | --- |
| `fortran-codebase-audit` | legacy構文、COMMON、implicit typing、call graph等を診断 |
| `fortran-behavior-lock` | 既存Fortranの入出力をgolden test化 |
| `fortran-safe-modernization` | 小さな変更→compile→test→比較の反復 |
| `fortran-common-modernization` | COMMONの不整合検出・module移行 |
| `fortran-interface-hardening` | explicit interface、kind、引数整合性 |
| `fortran-fixed-to-free` | fixed → free form変換＋検証 |
| `fortran-python-parity` | Fortran/Pythonの数値比較・許容誤差・差異解析 |
| `legacy-migration-report` | 移行前後の証跡・変更内容・検証結果を自動生成 |

その下に、skillから呼び出される

`scan_common.py`  
`scan_calls.py`  
`scan_implicit.py`  
`compare_outputs.py`  
`generate_golden_cases.py`

のような**普通のPythonツール**が存在する、という構造がよいです。

つまり、

> **skill = 判断とワークフロー**  
> **Python script = 決定論的な解析・変換・検査**

と分けます。

ここが非常に重要で、何でもLLMにやらせるのではなく、**一度確立した処理をPythonツールへ落とし、skillはそれを適切な順番で使わせる**ようにします。

---

なので、現在の段階では私は、

**Superpowers + Sonnet 5 + レガシーコード + 上のkickoff prompt**

だけでまず始めます。

そして最初の成果物として**「レガシーFortran近代化パイプライン自体」を作る**、という位置付けにします。今回の10,000行程度のコードは、そのパイプラインを作るための「第1号実証コード」にすると非常にきれいです。 [GitHub](https://github.com/obra/superpowers?utm_source=chatgpt.com)

次に重要なのは、実際のリポジトリを **`legacy/`, `src/`, `tools/`, `tests/`, `.agents/skills/`, `docs/` のどこに何を置くか**です。ここまで先に決めておくと、Copilotが作った成果物がごちゃ混ぜにならず、そのまま第2、第3のFortranコードへ流用できる構造にできます。

* * *

**You:**

最初にまずAbsoft Profortranで開発していたコードがgfortranで回るようにしなければならないのですが、ここは出力を固定する前に地道に修正した方が良いでしょうか？他のコードも同様の問題を抱えていることを考えると、ここも含めてsuperpowersでやってもらい、あとでツール・skill化した方が良い？

* * *

**ChatGPT:**

はい。**そこもSuperpowersの対象に含めた方がよい**です。むしろ「Absoft Pro Fortran → gfortran移行」は、今回だけでなく他のレガシーFortranにも再利用しやすい最初の成果物になります。

ただし、順序は少し重要です。

### まず理想は「Absoft版の出力を固定」

もし現在もAbsoft Pro Fortranで元コードを実行できるなら、

**Absoftでビルド → 代表ケースを実行 → 入出力・中間結果を保存 → その後gfortran対応**

が最善です。

つまり、

```
Absoft original
    ↓
Golden reference
    ↓
gfortran compatibility port
    ↓
同じ出力か確認
    ↓
modernization
    ↓
Python migration
```

です。

これは「gfortran対応のための修正」でうっかり計算ロジックまで変えてしまうリスクをかなり下げます。

一方、**Absoft環境がもう使えない**のであれば、出力固定より先にgfortran対応をするしかありません。ただし、その場合も普通のリファクタリングとは完全に分離して、

> **Phase 0: Compiler portability / bootstrap**

として扱うのがよいです。

この段階では「きれいにする」のではなく、

> **gfortranで元コードを可能な限り意味を変えずに動かす**

だけを目的にします。

---

## 「地道に自分で修正」よりSuperpowersにやらせる方がよい

今回の目的なら私はそうします。

例えばAbsoft→gfortran移行では、

* 非標準構文
* Hollerith
* fixed-form特有の書式
* `COMMON`
* `EQUIVALENCE`
* implicit typing
* 引数型不一致
* integer / real kind不一致
* `DO` 文などの旧構文
* Absoft固有拡張
* ファイルI/O差
* unformatted binaryの差
* 配列境界
* 初期化されていない変数
* compiler依存のメモリ配置

などが出てきます。

これは別の古いFortranコードでもかなり繰り返します。

したがって、

```
1件目
AIが調査しながら修正
↓
修正パターンを記録
↓
決定論的に検出できるものをPythonツール化
↓
判断が必要なものをskill化
↓
2件目で再利用
```

という流れが理想です。

以前考えていたCOMMON不整合検出などともそのまま繋がります。

---

## ただし最初から「自動修正ツール」を作らせない

ここも重要です。

最初のコードでいきなり、

> 「Absoft→gfortran変換ツールを作って」

とすると、かなり危険です。

まずAI自身に実コードを移植させ、

**どんな問題が実際に発生したか**

を収集します。

例えば、

```
issue 001
CALL FOO(A)

A = REAL*8
FOO dummy = REAL*4
```

という問題が50箇所出たとします。

そこで初めて、

```
scan_argument_mismatch.py
```

のようなツールを作る価値が出てきます。

つまり、

> **実コードで発見 → 一般化 → ツール化**

です。

---

# Phase 0はこのくらい厳密に分離する

私はリポジトリ上でも、

```
legacy/
    original_absoft/

src/
    gfortran_compatible/

tests/
    golden/

tools/
    portability/

docs/
    migration/
```

くらい分けます。

`legacy/original_absoft/` は**原則変更禁止**です。

gfortran対応コードだけ別に置きます。

さらに変更を、

### A. Build-only

Makefile / compiler option / link設定だけ。

### B. Syntax compatibility

gfortranが読めない構文だけ修正。

### C. Semantic-risk

型、COMMON、EQUIVALENCE、初期値など計算結果に影響し得る変更。

の3つくらいに分類させると良いです。

特にCはCopilotに勝手に直させず、

> 発見 → 原因説明 → 修正案 → テスト → 修正

の順にします。

---

## gfortranの「動かすためのオプション」と「直すべき問題」も分離する

例えば古いコードだと、一時的にgfortranの互換オプションで通ることがあります。

ただし、

> コンパイラオプションで黙らせる

ことと、

> ソースコード上の問題を解決する

ことは別です。

最初は「bootstrap configuration」として多少寛容にしてもよいですが、最終的には厳しい診断へ移行します。

イメージは、

```
Stage 0
とにかく元コードに近い状態でcompile

Stage 1
warningsの棚卸し

Stage 2
危険なwarningを解消

Stage 3
strict build

Stage 4
modernization
```

です。

これは後にかなり良いskillになります。

---

# 最初のSuperpowersへの依頼も変えた方がよい

先ほどの「modernizationしてください」より前に、まずこのPhase 0を入れることをおすすめします。

Use the Superpowers workflow for this task.

This repository contains a legacy Fortran codebase originally developed and built with Absoft Pro Fortran.

The immediate goal is NOT modernization and NOT Python migration yet.

The first goal is to establish a reproducible gfortran build while preserving the behavior of the original program as closely as possible.

Treat this as a separate compiler-portability phase.

Do not perform broad refactoring or cleanup unless it is strictly necessary for gfortran compatibility.

First:

1. inspect the repository and determine the existing build structure;
2. identify Absoft-specific compiler options, language extensions, runtime assumptions, and library dependencies;
3. attempt to compile the code with gfortran;
4. collect and classify all compiler and linker diagnostics;
5. classify each required change as:
    * build/configuration only,
    * syntax/compiler compatibility,
    * potentially behavior-changing;
6. prioritize the smallest possible changes required to obtain a working executable.

If an original Absoft executable or Absoft build environment is available, create representative baseline outputs before modifying source behavior.

Preserve the original source as an immutable reference.

For every source-level change, record:

* the original code,
* the modified code,
* why gfortran requires the change,
* whether the change could affect numerical behavior,
* how the change will be validated.

Do not hide serious incompatibilities using permissive compiler flags without documenting them.

Compatibility flags may be used temporarily to bootstrap the build, but distinguish temporary compiler workarounds from issues that should eventually be corrected in the source.

As problems are encountered, identify patterns that may recur in other legacy Fortran projects.

Do not immediately build generalized tooling.

After each recurring pattern has been observed and understood, propose whether it should become:

* a deterministic analysis script,
* an automated test,
* a reusable migration procedure,
* or an Agent Skill.

The deliverables for this phase are:

* a reproducible gfortran build,
* a compiler-portability issue inventory,
* a migration log,
* baseline/regression tests where possible,
* and a list of demonstrated candidates for reusable tooling.

Use small, testable changes and verify the build after each logical change.

これを**第1プロンプト**にしてよいと思います。

---

## そしてPhase 0終了時に最初のskillを作る

おそらく最初に生まれるskillは、

```
legacy-fortran-compiler-port
```

あたりになります。

中身は例えば、

```
1. 元コンパイラを特定
2. 元build optionを取得
3. gfortranでdiagnostic compile
4. error分類
5. compiler extension判定
6. minimal fix
7. regression comparison
8. strict build
9. portability report
```

です。

そのskillから、

```
scan_common.py
scan_calls.py
scan_extensions.py
scan_implicit.py
build_matrix.py
compare_outputs.py
```

などを呼ぶ構造になっていくはずです。

---

### 今回の全体工程

最終的には、かなりきれいに

```
Phase 0
Absoft → gfortran
        ↓
Phase 1
挙動固定・golden test
        ↓
Phase 2
legacy Fortran安全化
        ↓
Phase 3
COMMON / interface / implicit none
        ↓
Phase 4
modern Fortran化
        ↓
Phase 5
Python移植
        ↓
Phase 6
Fortran ↔ Python parity
        ↓
Phase 7
高速化・配布
```

とできます。

そして**各Phaseから汎用tool/skillを抽出する**。

この進め方なら、今回のFortranコードを直すこと自体が、「社内の別のレガシーFortranをCopilotで安全に近代化するための基盤開発」になります。

特にPhase 0のAbsoft→gfortran対応は捨て作業ではなく、**最初に汎用化価値がかなり高い工程**だと思います。

* * *

**You:**

挙動固定に関して、リファクタリングの過程で例えばCOMMONの不整合や暗黙の型宣言によるタイポを発見し、元のコードが間違っている可能性があるかと思うのですが、その場合にも元の挙動に固定されてしまったりしませんか？

* * *

**ChatGPT:**

はい。その懸念は正しくて、**挙動固定を「正しい挙動の固定」と扱うと危険**です。

レガシーコードでは、COMMON不整合や暗黙型によるタイポ、未初期化変数などの結果、**現在の出力そのものがバグを含んでいる可能性があります**。なので、挙動固定は「正解を保存する」のではなく、**現状を観測して記録する characterization test** と考えるのがよいです。

つまり、基準を2種類に分けます。

* **Current-behavior baseline**：今のコードが実際に何を出しているか
* **Correctness baseline**：理論解、実験値、仕様、物理法則などから期待される正しい挙動

この2つは一致するとは限りません。

たとえばCOMMON不整合を見つけた場合、いきなりgolden testを通すために元の挙動へ戻すのではなく、

```
元コード
  ↓
現状出力を保存
  ↓
COMMON不整合を発見
  ↓
「これはバグか？」を評価
  ↓
バグと判断
  ↓
修正
  ↓
出力が変化
  ↓
変更理由と検証結果を記録
  ↓
新しい正しいbaselineへ更新
```

とします。

重要なのは、**baseline変更を失敗扱いにしないこと**です。

むしろ、

> 意図していない出力変化はNG  
> 根拠のあるバグ修正による出力変化はOK

というルールにします。

### 典型例

例えば、

```fortran
COMMON /PARAM/ ALPHA, BETA
```

があるサブルーチンでは

```fortran
REAL*8 ALPHA, BETA
```

なのに、別のサブルーチンでは

```fortran
REAL ALPHA, BETA
```

だったとします。

gfortran化や解析ツールでこれを発見したら、単純に「元の結果と違うから修正禁止」ではありません。

まず、

1. どちらの型が設計意図か
2. メモリレイアウトが実際にどう崩れていたか
3. その変数が結果にどの程度影響しているか
4. 他の計算式、資料、入力データ、理論値と整合するのはどちらか

を調べます。

そして倍精度が正しいと判断できたなら、出力が変わっても**それは望ましい変更**です。

---

### テストも3層に分けるとかなり安全です

私は今回ならこうします。

**1. Characterization tests**

「元コードはこの入力でこの出力だった」という記録。

バグを含んでいても構いません。

**2. Invariant / physics tests**

例えば、

* 圧力や温度が物理的にあり得る範囲
* 対称条件なら対称になる
* 荷重0なら反力0
* 極限条件で既知解に近づく
* 保存則
* 単位整合性

のようなもの。

これは元コードが間違っていれば失敗してよいです。

**3. Verification tests**

理論解、既知benchmark、実験値、別ソルバーとの比較。

最終的にこちらを「正解」に近い基準にします。

---

### つまりgolden testは「永久固定」ではない

ここはCopilotにも明示しておいた方がよいです。

例えば、

> Golden outputs represent observed legacy behavior, not necessarily correct behavior.  
> Do not preserve a legacy output when strong evidence indicates that it is caused by a defect.  
> Any intentional behavioral change must be documented, justified, and accompanied by a new verification test.

というルールです。

かなり重要なので、これはskillではなく最初から`copilot-instructions.md`に置いてもよいくらいです。

---

### バグ候補を見つけたときの扱いも決めておく

私はCopilotに次の4分類をさせます。

| 分類 | 例 | 扱い |
| --- | --- | --- |
| Behavior-preserving | fixed→free form | golden維持 |
| Suspected defect | COMMON型不一致 | 調査 |
| Confirmed defect | 明らかな変数名タイポ | 修正＋baseline更新 |
| Uncertain semantic change | アルゴリズム変更 | 保留または別branch |

特にCOMMONやimplicit typingは、**自動修正候補ではなく「suspected defect」としてまず報告**させるのが安全です。

---

### 暗黙型のタイポは典型的

例えばFortran 77で、

```fortran
REAL PRESSURE
...
PRESURE = ...
```

と書いてしまうと、`PRESURE`が別変数として暗黙に宣言されてしまいます。

`IMPLICIT NONE`化すると初めてエラーになります。

この場合、

```
元コードを再現する
```

ことより、

```
PRESSUREのタイプミスだったことを確認する
```

ことの方が重要です。

なので、IMPLICIT NONE化で発見された問題は、

**modernization failureではなく、latent defect discovery**

として記録します。

これはむしろリファクタリングの成果です。

---

### 今回のプロジェクトでは「二重の履歴」を残すのがおすすめです

例えば、

```
tests/
  characterization/
      legacy_absoft/
      legacy_gfortran/

  verification/
      analytical/
      benchmark/
      regression/

docs/
  defect_log.md
  behavior_changes.md
```

として、

`characterization`には昔の挙動を残す。

`verification`には正しいと判断した挙動を置く。

そしてバグ修正ごとに、

```
BUG-014
COMMON /STATE/ variable T was REAL*4 in routine A
but REAL*8 in all other routines.

Legacy behavior:
...

Evidence:
...

Decision:
REAL*8 is intended.

Behavioral impact:
maximum output changed by 0.37%.

Verification:
analytical case error improved from ...
```

のように残します。

これなら「古い結果を失う」ことも、「古いバグに縛られる」こともありません。

---

なので、今回の全体思想は

**Preserve behavior by default, but preserve correctness over accidental legacy behavior.**

くらいがちょうどよいです。

そして私は、Phase 1を単に「挙動固定」と呼ぶより、**「Characterization & Verification」**と呼ぶ方がよいと思います。そうするとCopilotも「元コードの出力＝正解」と誤解しにくくなります。

* * *

**You:**

初めのプロンプトはどのようなものがよい？
日本語で教えて

* * *

**ChatGPT:**

最初のプロンプトは、**いきなりリファクタリングやPython化を始めさせず、「Absoft→gfortran移行」「現状把握」「再利用可能な知見の抽出」までを最初のフェーズとして明確化**するのがよいです。

そのままCopilot ChatのAgent modeに貼れる形なら、以下をおすすめします。

このプロジェクトでは、Superpowersのワークフローを使用してください。

このリポジトリには、Absoft Pro Fortranで長年開発されてきたレガシーFortranコードがあります。

最終的な目的は以下です。

1. 現在のFortranコードの構造と挙動を理解する
2. Absoft Pro Fortran依存を解消し、gfortranで再現可能にビルド・実行できるようにする
3. 潜在的なバグを発見しながら、安全にFortranコードをリファクタリング・近代化する
4. COMMON、暗黙型宣言、古いFortran構文などへの依存を段階的に解消する
5. 十分に検証されたFortranコードをPythonへ移植する
6. 元Fortran、近代化Fortran、Python版の数値結果を比較・検証する
7. この作業で得られた知見、解析ツール、テスト方法、変換手順を、他のレガシーFortranコードにも流用可能な成果物として残す

ただし、現時点では大規模なリファクタリングやPython化を開始しないでください。

まずは「Phase 0: Absoft Pro Fortranからgfortranへの移行とコードベース調査」として進めてください。

## 最重要方針

元コードの挙動は重要な参考情報ですが、「元コードの出力＝正しい結果」とは仮定しないでください。

COMMONブロックの型・順序不整合、暗黙型宣言による変数名のタイプミス、未初期化変数、引数型不一致などによって、元コード自体に潜在的なバグが存在する可能性があります。

したがって、以下を区別してください。

* Characterization:  
    現在のレガシーコードが実際にどのような挙動をしているかを記録するもの
* Verification:  
    理論解、物理法則、仕様、既知のベンチマーク、実験値などから正しいと考えられる挙動を確認するもの

Characterization testは、元コードの挙動を記録するためのものであり、その挙動を永久に維持することを意味しません。

強い根拠によって元コードのバグと判断できる場合には、元の挙動を維持するためにバグを残さないでください。

ただし、結果を変更する修正については、必ず

* 何が問題だったか
* なぜバグと判断したか
* どのように修正したか
* 修正によって結果がどの程度変化したか
* 修正後の結果をどのように検証したか

を記録してください。

意図しない結果変化は避け、意図した結果変化は根拠と検証を残してください。

## Phase 0で最初に行うこと

まずリポジトリ全体を調査してください。

以下を確認してください。

* メインプログラム
* サブルーチン
* 関数
* INCLUDEファイル
* COMMONブロック
* EQUIVALENCE
* BLOCK DATA
* 暗黙型宣言
* 固定形式Fortran特有の記述
* Absoft固有の構文や拡張
* 外部ライブラリ
* ファイル入出力
* バイナリ入出力
* コンパイラオプション
* リンカオプション
* グローバル状態
* サブルーチン間の依存関係
* ビルド方法
* 実行方法

その後、gfortranでビルドを試み、コンパイラエラー、警告、リンカエラーを収集・分類してください。

問題を少なくとも以下に分類してください。

1. ビルド設定だけで解決できるもの
2. Absoftとgfortranの構文・拡張差によるもの
3. 数値結果に影響しないと考えられるソース変更
4. 数値結果に影響する可能性がある変更
5. 潜在的なバグの可能性があるもの
6. 現時点では意図を判断できないもの

## gfortran移行の方針

この段階ではコードをきれいにすることを目的にしないでください。

目的は、

「元コードの意味を可能な限り維持したまま、gfortranで再現可能にビルド・実行できる状態を作ること」

です。

必要最小限の変更を優先してください。

一時的にgfortranの互換オプションを使用しても構いません。

ただし、

* 一時的な互換オプション
* 将来的にソースコード側で修正すべき問題

を必ず区別してください。

警告を単純に無効化して問題を隠さないでください。

## ソースコードを変更するときのルール

元のAbsoft用ソースコードは、可能であれば変更せず参照用として保存してください。

ソースコードを変更する場合には、小さな変更単位で進めてください。

各変更について、

1. 変更前
2. 変更後
3. 変更理由
4. gfortran対応のために必要か
5. 数値挙動への影響可能性
6. 検証方法

を記録してください。

変更後は可能な限り、

* コンパイル
* 実行
* 代表的な出力比較

を行ってください。

## 潜在バグの扱い

以下のようなものを発見した場合、機械的に修正しないでください。

* COMMONの変数型不一致
* COMMONの変数順序不一致
* 引数型の不一致
* 引数数の不一致
* 暗黙型宣言による未宣言変数
* 変数名のタイプミスの疑い
* 未初期化変数
* 配列範囲外アクセス
* EQUIVALENCEやCOMMONによる危険なメモリ共有

これらは「潜在バグ候補」として記録し、

* 他の使用箇所
* 宣言
* 呼び出し関係
* 計算式
* コメント
* 入出力
* 周辺コード

を調査した上で、修正すべきか判断してください。

## 再利用可能な成果物について

このプロジェクトは、一つのコードだけを直して終わるものではありません。

作業中に、他のレガシーFortranにも使えそうなものを識別してください。

例えば、

* COMMON不整合の検出
* サブルーチン呼び出し関係の解析
* 引数型不一致の検出
* implicit typing依存の検出
* Absoft固有構文の検出
* fixed-form構文の解析
* コンパイラ警告の分類
* Fortran出力の比較
* regression test生成

などです。

ただし、最初から大量の汎用ツールやAgent Skillを作らないでください。

まず実際のコードで問題を発見し、同じ種類の作業が繰り返されることを確認してください。

その後、

* 決定論的に処理できるもの → Pythonなどの解析ツール
* 判断や手順が必要なもの → Agent Skill
* 検証方法 → 自動テスト
* プロジェクト固有の情報 → このリポジトリ内のドキュメント

という形で整理してください。

再利用可能な成果物と、このコード固有の実装は明確に分離してください。

## 最初に作成してほしい成果物

まだ大規模な実装修正には入らず、まず以下を作成してください。

1. リポジトリ全体の構造調査結果
2. ビルド・実行方法の現状整理
3. Absoft依存箇所の一覧
4. gfortranでの初回ビルド結果
5. コンパイラ・リンカ問題の分類
6. 潜在的なバグ候補一覧
7. リファクタリング上のリスク一覧
8. CharacterizationとVerificationのテスト方針
9. Absoft → gfortran移行計画
10. その後のFortran近代化・Python移植まで含めた大まかなフェーズ計画
11. 将来的に汎用ツールまたはAgent Skillになりそうな候補一覧

ここまでをSuperpowersのbrainstorming / planningの流れで整理してください。

計画を立てる前にコードベースを十分に調査し、実際のコードから得られた証拠に基づいて判断してください。

大規模な変更はまだ開始しないでください。

このプロンプトなら、**最初にSonnet 5級のモデルでリポジトリ全体を読ませる**使い方にかなり合っています。

特に重要なのは、最初から「COMMONを直せ」「IMPLICIT NONE化しろ」と指示していない点です。まずCopilot自身に**実コードを調査→問題を分類→計画を作らせる**ことで、Superpowersの良さを活かせます。

その計画を見てから、次のプロンプトで「Phase 0の実装を開始してください」と進めるのがよいです。最初の1回はSonnet 5の標準～高めの推論を使い、具体的な機械的修正に入った後は軽量モデルへ振り分ける構成が合うと思います。

* * *

**You:**

fparserなどのパッケージをあらかじめuv環境に入れておいた方が良い？

* * *

**ChatGPT:**

はい。**fparserは最初からuv環境に入れておいてよい**です。ただし、最初からFortran解析用パッケージを大量に入れるより、**最小限の「解析・テスト環境」だけ作る**のがおすすめです。

今回なら、まずはこの程度で十分です。

```Bash
uv init
uv add --dev fparser pytest numpy
```

用途は次のイメージです。

* `fparser`：Fortranソースの構文解析、COMMONや宣言、CALLなどを調べるための基盤
* `pytest`：後でcharacterization testや解析ツール自身のテストに使う
* `numpy`：Fortran/Python間の数値出力比較などでほぼ確実に使う

さらに、比較ツールを作る段階で必要になれば追加します。

```Bash
uv add --dev scipy
```

ただし、**fparserを最初から「これで全部解析できる」とは考えない方がよい**です。

古いAbsoft向けFortranだと、

* vendor固有構文
* fixed-formの特殊な記述
* 古い拡張構文
* プリプロセッサ
* 壊れ気味だがAbsoftでは通っていたコード

などでfparserが解析に失敗する可能性があります。

なのでCopilotには、

> fparserを利用可能な解析手段の一つとして使う。ただし、fparserで解析できないコードを「不正なコード」と即断せず、テキスト解析・gfortranの診断・手動コード調査も併用する

と指示するのがよいです。

特に今回のPhase 0では、**最も信頼できる解析器はある意味gfortran自身**です。

例えば、

```Bash
gfortran -Wall -Wextra ...
```

などで出る診断と、fparserによる静的解析を組み合わせます。

流れとしては、

```
                 legacy Fortran
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
      gfortran                    fparser
   compiler diagnostics       static analysis
          │                         │
          └────────────┬────────────┘
                       ↓
                 Python tools
                       ↓
            structured issue report
```

がかなり良いです。

### 私なら最初はこの構成にします

```
repository/
├─ legacy/
│  └─ ...
│
├─ tools/
│  └─ fortran_audit/
│
├─ tests/
│
├─ docs/
│
├─ pyproject.toml
└─ uv.lock
```

そして、

```Bash
uv add --dev fparser pytest numpy
```

だけ入れておきます。

Copilotが作る解析ツールは、

```
tools/fortran_audit/
    scan_common.py
    scan_calls.py
    scan_declarations.py
    scan_implicit.py
```

のように徐々に増やします。

---

一方で、**最初からfparserベースの巨大な解析フレームワークを作らせる必要はありません**。

例えばCOMMONの不整合なら、最初はCopilotにコードを調査させ、

```fortran
COMMON /AAA/ A, B, C
```

の宣言を全ファイルから集めるだけの小さなツールを作らせます。

それが安定したら、

```Python
fparser
→ AST
→ COMMONブロック抽出
→ 型・順序比較
```

へ発展させればよいです。

この**小さなツールを積み重ねる方向**の方が、将来別のFortranコードにも流用しやすいです。

### さらに重要な点

`fparser`自体もプロジェクトの実行依存ではなく、**開発・解析用依存**にしておくのがおすすめです。

つまり、

```Bash
uv add --dev fparser
```

です。

最終的なPython化コードの実行環境にfparserを要求する必要はありません。

最終的には、

```
[本番コード]
Python化した解析プログラム
        ↑
fparser不要

[開発環境]
fparser
pytest
解析・変換tool群
```

と分離できます。

なので、**今の段階では `fparser + pytest + numpy` をuvのdev dependencyとして先に入れておく**、というのがちょうどよいと思います。特にfparserは、以前考えていたCOMMON不整合検出のような汎用ツールの中核候補になります。

* * *

**You:**

今回単純に対象fortranコードをリファクタリング・python化するだけではなく、共通して使える部分をツール・スキル化するわけですが、その場合もfparserをdev dependency扱いで良いの？

* * *

**ChatGPT:**

今回のように、**fparserを使う解析ツール自体を成果物として他プロジェクトでも再利用する**なら、`fparser`を単純な`dev dependency`に置くのは少し違います。

ポイントは、**「何の依存関係か」を分けること**です。

* `pytest`, `ruff` など → 開発・テストにしか使わないので `dev`
* `fparser` → Fortran解析ツールを実行するために必要なので、そのツールにとっては**実行時依存**
* Python化した解析本体 → Fortran解析ツールを使わないなら `fparser` は不要

uvでも、通常の実行依存は`project.dependencies`、公開可能な任意機能は`project.optional-dependencies`、開発専用は`dependency-groups`に分ける考え方です。 [Astral Docs+1](https://docs.astral.sh/uv/concepts/projects/dependencies/?utm_source=chatgpt.com)

今回なら、私は**fparserをoptional dependencyの`fortran-tools`として分離**するのが一番きれいだと思います。

たとえば、

```TOML
[project]
name = "legacy-fortran-modernizer"
version = "0.1.0"
dependencies = []

[project.optional-dependencies]
fortran-tools = [
    "fparser",
    "numpy",
]

[dependency-groups]
dev = [
    "pytest",
    "ruff",
]
```

という形です。

すると、

```Bash
uv sync
```

は最低限の環境、

```Bash
uv sync --extra fortran-tools
```

ならFortran解析ツール込み、

という運用にできます。uvはextrasを`[project.optional-dependencies]`で管理し、`--extra`で必要なものだけ有効化できます。 [Astral Docs+1](https://docs.astral.sh/uv/concepts/projects/dependencies/?utm_source=chatgpt.com)

### 今回はさらに「ツール」と「移植先コード」を分けるとよい

最終的には、こんな構造を狙うとかなり再利用しやすいです。

```
repository/
├── legacy/
│   └── original_fortran/
│
├── src/
│   └── application/
│       └── ...               # Python化した本体
│
├── tools/
│   └── fortran_migration/
│       ├── scan_common.py
│       ├── scan_calls.py
│       ├── scan_implicit.py
│       ├── compare_outputs.py
│       └── ...
│
├── tests/
│   ├── application/
│   └── fortran_migration/
│
├── .agents/
│   └── skills/
│       └── legacy-fortran-modernization/
│
└── pyproject.toml
```

ここで、

```
Python解析本体
    ↓
fparser不要

Fortran migration tools
    ↓
fparser必要

Agent Skill
    ↓
migration toolsを必要に応じて呼ぶ
```

という依存関係にします。

これが今回の狙いにかなり合っています。

### さらに本格化したら別パッケージ化もあり

最初は1リポジトリ内のoptional dependencyで十分です。

しかし今回のツール群が育って、

> 別のFortranコードにも普通に使える

ところまで来たら、

```
legacy-fortran-toolkit/
    pyproject.toml
    src/
        legacy_fortran_toolkit/
    tests/
    skills/
```

として**独立パッケージ／独立リポジトリに昇格**させるのが理想です。

そうなれば、

```TOML
dependencies = [
    "fparser",
]
```

とし、`fparser`はそのツールキットの正式なruntime dependencyです。

一方、それを利用する各解析プロジェクトでは、

```
bearing-code/
gear-code/
thermal-code/
```

などから共通の`legacy-fortran-toolkit`を使えるようになります。

---

したがって、前の回答の

```Bash
uv add --dev fparser
```

は、**「今回だけ解析補助として使う」なら正しい**ですが、今決めた目的を考えると少し修正した方がよいです。

今回なら私は最初から、

```Bash
uv add fparser --optional fortran-tools
uv add numpy --optional fortran-tools

uv add --dev pytest
uv add --dev ruff
```

くらいにします。

そしてCopilotにも、

> `tools/fortran_migration`以下は今回のコード専用ではなく、将来的に独立した再利用可能ツールキットへ切り出せる設計にする。ただし、現段階では過度な汎用化はしない。

と伝えておくのが良いです。

つまり、**fparserは「開発ツール」ではなく「Fortran近代化ツール群の実行依存」と考える**のが今回には合っています。

* * *

**You:**

python化に必要最低限なリファクタリングをさせようと思っているのですが、具体的にどこまでリファクタリングさせるのが良いのでしょうか？そこもsuperpowers+copilotに判断させれば良い？

* * *

**ChatGPT:**

はい。ただし、**「どこまでリファクタリングするか」をCopilotに完全に丸投げするのはおすすめしません**。Superpowers + Copilotには「必要性を評価して提案する」ところまでやらせ、こちらで**Python移植のための終了条件**を先に与えるのがよいです。

今回の目的なら、Modern Fortranとして美しくすることは目指さず、

> **Pythonへ機械的・段階的に写せるほど、意味・依存関係・状態が明示された時点で止める**

のがちょうどよいです。

私なら次の基準にします。

| 項目 | Python化前 | 理由 |
| --- | --- | --- |
| gfortranで安定実行 | **必須** | 移植元の実行基準 |
| Characterization / Verification | **必須** | Python版との比較基準 |
| COMMON不整合の解消 | **必須** | 変数の意味が曖昧なまま移植できない |
| 暗黙型の解消 | **ほぼ必須** | Pythonで変数を誤対応しやすい |
| 引数型・個数不整合 | **必須** | 関数インターフェース確定 |
| 未初期化変数 | **必須** | コンパイラ依存挙動を除去 |
| 配列境界問題 | **必須** | Python結果との一致を阻害 |
| EQUIVALENCE | 使用中なら**解消推奨** | Pythonへの対応が危険 |
| COMMON→module | **部分的でよい** | 完全Modern Fortran化までは不要 |
| explicit interface | 主要routineは**推奨** | 入出力を明確化 |
| `intent(in/out/inout)` | 主要routineは**推奨** | Python関数設計に非常に役立つ |
| fixed→free form | **推奨だが必須ではない** | 解析・保守が容易 |
| GOTOの全面除去 | **不要** | Python移植時に整理可能 |
| OO化 | **不要** | むしろ余計な変更 |
| 高度なmodule設計 | **不要** | Python化前には過剰 |
| 性能最適化 | **不要** | parity確認後 |
| アルゴリズム改善 | **しない** | 移植との切り分け不能になる |

特に重要なのが、**COMMONを全部module化する必要はない**という点です。

たとえば、

```fortran
COMMON /FLUID/ RHO, MU
```

が80個のルーチンに出てくるなら、Python化する前に最低限、

```
FLUID
 ├─ RHO : real64
 └─ MU  : real64
```

という**唯一の正しい定義が確定している状態**にはしたいです。

そのうえで、

```fortran
module fluid_state
```

まで移すか、Python化の際に

```Python
@dataclass
class FluidState:
    rho: float
    mu: float
```

へ直接移すかはケースバイケースです。

つまり、

**COMMON完全廃止が目的なのではなく、グローバル状態の意味を一意にすることが目的**

です。

---

### IMPLICIT NONEはかなり優先度が高いです

これはPython化前にやっておく価値があります。

例えば、

```fortran
PRESSURE = ...
...
PRESURE = ...
```

のようなタイプミスがあった場合、Pythonへそのまま移植すると、AIが「これは別の変数なのか、タイプミスなのか」を判断する必要が出ます。

Fortran側で、

```fortran
IMPLICIT NONE
```

を通しておけば、

> **Fortranコンパイラによって変数集合が確定したコード**

をPythonへ移せます。

これはかなり大きいです。

ただし、一括で全ファイルをIMPLICIT NONE化して大量の修正をするのではなく、

```
routine単位
↓
implicit none
↓
undeclared variable検出
↓
意図を確認
↓
compile
↓
test
```

と段階的に進めるべきです。

---

## Python化前に一番やっておきたいのは「計算核とI/Oの分離」

これはModern Fortran化より重要かもしれません。

レガシーコードが、

```
入力ファイル読み込み
↓
COMMON更新
↓
計算
↓
途中でファイル出力
↓
COMMON更新
↓
別計算
↓
結果出力
```

となっているとPython化が非常に難しくなります。

理想はPython化前に、

```
driver
 ├─ input
 ├─ calculation
 └─ output
```

程度まで分離することです。

例えば、

```fortran
CALL READ_INPUT(...)
CALL SOLVE(...)
CALL WRITE_RESULT(...)
```

くらいになれば十分です。

さらに計算routineも、

```fortran
CALL REYNOLDS_SOLVE(H, MU, U, P)
```

のように、

**何を入力して何を変更するのか**

が分かれば、Pythonへ非常に移しやすくなります。

完璧なModern Fortran設計にする必要はありません。

---

## 「ここまで来たらPython化開始」というゲートを作る

私はCopilotに次の終了条件を与えます。

```
Fortran pre-migration gate

[ ] gfortranでwarningを把握した上で安定実行できる
[ ] 主要なcharacterization testが存在する
[ ] verification caseが少なくともいくつか存在する
[ ] COMMONの型・順序不整合がない
[ ] 暗黙宣言による未確認変数がない
[ ] routine間の引数型・個数不整合がない
[ ] 未初期化変数・範囲外アクセスの既知問題が解決されている
[ ] 主要routineの入力・出力・副作用が分かっている
[ ] グローバル状態の一覧がある
[ ] 計算部分と主要なI/Oが識別できる
[ ] Python移植対象routineの依存順序が分かっている
```

**全部Modern Fortranになったか**は条件にしません。

これを満たしたらFortran側の大規模リファクタリングを止めて、Python化へ移ります。

---

### Superpowers + Copilotにはこう判断させるのがよいです

完全に

> 「Python化に必要なところまで適当にリファクタリングして」

ではなく、

> 「上記のpre-migration gateを満たすために必要な最小限のリファクタリングを提案し、各変更について『Python移植に必要』『安全性確保に必要』『単なるコード美化』のどれかに分類してください。単なるコード美化は実施しないでください。」

と指示します。

これはかなり効くと思います。

さらに、

> **Python移植に必要か不明なリファクタリングについては、実装前に必要性を説明する**

というルールも入れます。

---

今回のプロジェクトなら、私なら概ね

**Absoft→gfortran → 潜在バグ除去 → IMPLICIT NONE → COMMON整合性確保 → interface明確化 → I/Oと計算核を軽く分離 → Python化**

くらいで止めます。

**COMMONを全部きれいなmoduleにする、全コードをOO化する、全GOTOを消す、完全なModern Fortranにする**ところまではやりません。それらは「Fortranを今後も主言語として長期保守する」場合には価値がありますが、**Python移植が目的なら投資過多**になりやすいです。

むしろ、この「どこまでやったらPython化へ進むか」という判定基準そのものも、後々 `fortran-python-migration-readiness` のような再利用可能なskillにすると価値が高いと思います。

* * *

**You:**

もし複数のfortranコード(それぞれ独立したコード)がある場合、それら全てをリポジトリ内においておいた方がよい？それともまずはメインのリファクタリング対象だけでよい？

* * *

**ChatGPT:**

まずは**メインのリファクタリング対象だけをリポジトリに置く**のがおすすめです。

最初から独立したFortranコードを全部入れると、Copilotが「どれが主対象か」「どこまで共通化すべきか」を誤認しやすくなり、Superpowersの探索フェーズも無駄に広がります。特に最初の段階では、1つのコードで実際の問題を十分に経験してから共通化した方がよいです。

理想的には、

```
Phase 1
Fortran A を対象に作業
    ↓
COMMON解析、引数検査、出力比較などの
再利用可能ツールを抽出
    ↓
Phase 2
Fortran B に適用
    ↓
「本当に汎用的か」を検証
    ↓
必要ならtool / skillを修正
    ↓
Phase 3
Fortran C に適用
```

という進め方です。

これはかなり重要で、**1コード目は「開発対象兼、ツールキット開発用の実証コード」**、2コード目は**「汎用性を検証するテストケース」**として扱えます。

たとえば最初のリポジトリは、

```
bearing-fortran-migration/
├── legacy/
│   └── original/
├── src/
├── tools/
│   └── fortran_migration/
├── tests/
├── docs/
└── .agents/
    └── skills/
```

くらいで十分です。

1コード目で、

```
scan_common
scan_implicit
scan_call_interfaces
compare_outputs
migration_readiness
```

などが育ったら、それらを独立した

```
legacy-fortran-toolkit/
```

へ切り出します。

その後、2コード目では例えば、

```
fortran-code-B/
```

からそのtoolkitを利用する形にします。

### 複数コードを同じリポジトリに入れる場合

例外として、

> 「この複数のFortranコード群をまとめて移行する一つの社内プロジェクト」

として管理したいなら、monorepoもありです。

その場合でも、

```
legacy-modernization/
├── applications/
│   ├── bearing/
│   ├── gear/
│   └── thermal/
│
├── toolkit/
│   └── ...
│
└── skills/
```

のように、**各アプリと共通toolkitを明確に分離**した方がよいです。

ただ、今の段階では私はこれをやりません。

理由は、最初から

```
Aにも使える
Bにも使える
Cにも使える
```

ように設計しようとすると、Copilotが過剰に抽象化しやすいからです。

例えばCOMMON解析一つでも、

* あるコードでは単純なCOMMON
* 別コードではEQUIVALENCE併用
* 別コードではINCLUDE経由
* 別コードではプリプロセッサ付き

かもしれません。

1コード目しか見ていないのに全部対応しようとすると、ツールが複雑になりすぎます。

---

したがって今回なら、

**① メイン対象だけを置く  
→ ② Superpowersで移行を進める  
→ ③ 実際に繰り返された処理だけtool/skill化  
→ ④ 2本目のFortranを追加して汎用性検証  
→ ⑤ 本当に共通なら独立toolkit化**

が最もよいと思います。

特に**2本目を早めに「汎用ツールの受け入れ試験」として使う**のがポイントです。1本目だけで完成扱いにしないことで、「このコード専用なのに汎用ツールっぽく見えるもの」をかなり排除できます。

* * *

**You:**

では改めて、初めに送るプロンプトを教えて

* * *

**ChatGPT:**

最初の一発目なら、今までの方針をまとめて、**「まず1本のメインFortranを対象に、Absoft→gfortran移行と調査から始める。Python化に必要な最小限のリファクタリングを目指し、共通化できるものは後からtool/skill化する」**と明示するのがよいです。

そのままCopilot Agent modeに貼るなら、私は以下にします。

このプロジェクトでは、Superpowersのワークフローを使用して進めてください。

このリポジトリには、Absoft Pro Fortranで開発されてきたレガシーFortranコードがあります。

現時点では、複数あるFortranコードのうち、まずこのリポジトリに置かれている1つのメインコードだけを対象にしてください。

最終的な目的は、このコードを安全にPythonへ移植することです。

ただし、このプロジェクトは単なる一回限りのコード移植ではありません。

作業の過程で得られた、

* 静的解析方法
* コンパイラ移行方法
* バグ検出方法
* テスト方法
* 出力比較方法
* リファクタリング手順
* Python移植手順
* 自動化スクリプト
* Agent Skillとして再利用可能なワークフロー

のうち、他のレガシーFortranコードにも再利用できるものを、将来的に共通ツール・共通Skillとして切り出せるようにしてください。

ただし、最初から過度に汎用化しないでください。

まずこの1本の実コードで問題を経験し、実際に繰り返し必要になった処理だけを共通化してください。

---

# 最終目標

最終的には以下の状態を目指します。

1. Absoft Pro Fortran依存を解消し、gfortranで再現可能にビルド・実行できる
2. レガシーコード内の潜在的な不整合・バグを把握する
3. Python移植に必要な範囲だけ安全にFortranコードをリファクタリングする
4. Pythonへ段階的に移植する
5. Fortran版とPython版の数値結果を比較・検証する
6. 他のレガシーFortranにも流用可能なtool / test / Skillを抽出する

重要なのは、「完全なModern Fortranへ書き直すこと」ではありません。

Python移植に必要な程度まで、

* 型
* インターフェース
* グローバル状態
* 入出力
* 計算依存関係
* 副作用

を明示的にし、安全にPythonへ移せる状態にすることを目標としてください。

---

# 現在のフェーズ

まずは

「Phase 0: コードベース調査とAbsoft Pro Fortran → gfortran移行」

から始めてください。

現時点では、

* 大規模なリファクタリング
* COMMONの全面module化
* fixed-formからfree-formへの全面変換
* GOTOの全面除去
* OO化
* Python化

にはまだ進まないでください。

まず現状を把握し、gfortranで動作可能な状態を作ることを優先してください。

---

# 最重要方針：元コードの挙動を「正解」と仮定しない

元コードの現在の出力は重要な基準ですが、

「元コードの出力 = 正しい結果」

とは仮定しないでください。

レガシーコードには、

* COMMONブロックの型不一致
* COMMONブロックの変数順序不一致
* 暗黙型宣言によるタイプミス
* 未初期化変数
* 引数型不一致
* 引数数不一致
* 配列範囲外アクセス
* EQUIVALENCEによる危険なメモリ共有
* コンパイラ依存挙動

などの潜在バグが存在する可能性があります。

したがって、次の2つを明確に区別してください。

## Characterization

現在のレガシーコードが実際にどのように動作しているかを記録するもの。

これは「現在の挙動の記録」であり、「正しい挙動の保証」ではありません。

## Verification

理論解、物理法則、仕様、既知のベンチマーク、入力条件、実験値などから、正しいと考えられる挙動を確認するもの。

Characterization testを通すためだけに、明らかなレガシーバグを残してはいけません。

潜在バグが発見され、十分な根拠から修正すべきと判断した場合には、出力が変化しても構いません。

ただしその場合は、

* 何が問題だったか
* なぜバグと判断したか
* どのように修正したか
* 修正によって出力がどの程度変わったか
* 修正後の結果をどのように検証したか

を記録してください。

原則として、

「意図しない挙動変化は防ぐ」  
「根拠のある挙動変化は許容し、記録・検証する」

という方針にしてください。

---

# 最初に調査してほしいこと

まずリポジトリ全体を探索し、このコードの構造を把握してください。

最低限、以下を確認してください。

* メインプログラム
* SUBROUTINE
* FUNCTION
* INCLUDEファイル
* COMMONブロック
* EQUIVALENCE
* BLOCK DATA
* implicit typing
* 固定形式Fortran
* 古いFortran構文
* Absoft固有構文・拡張
* 外部ライブラリ
* ファイル入出力
* unformatted / binary I/O
* コンパイラオプション
* リンカオプション
* グローバル状態
* 配列
* サブルーチン間の呼び出し関係
* 入力ファイル
* 出力ファイル
* 現在のビルド方法
* 現在の実行方法

可能であれば、call graphや依存関係も整理してください。

---

# gfortran移行

その後、gfortranで可能な限り元コードに近い状態のままビルドを試してください。

コンパイル・リンク時に発生する、

* error
* warning
* linker error
* runtime error

を収集してください。

各問題を少なくとも以下に分類してください。

1. ビルド設定だけで解決可能
2. Absoft固有機能・コンパイラ差
3. 数値挙動への影響が小さい互換修正
4. 数値挙動に影響する可能性がある修正
5. 潜在バグの可能性がある問題
6. 現時点では意図を判断できない問題

このフェーズの目的はコードを美しくすることではありません。

「元コードの意味を可能な限り保ちつつ、gfortranで再現可能にビルド・実行できるようにすること」

を優先してください。

必要最小限の修正を行ってください。

gfortranの互換オプションを一時的に使うことは許可します。

ただし、

* 移行用の一時的な互換オプション
* 将来的にソースコード側で修正すべき問題

は明確に区別してください。

警告を無効化して問題を隠すことは避けてください。

---

# 潜在バグの扱い

以下を発見した場合、機械的に自動修正しないでください。

* COMMONの型不一致
* COMMONの変数順序不一致
* 引数型不一致
* 引数数不一致
* implicit typingによる未宣言変数
* 変数名のタイプミスの可能性
* 未初期化変数
* 配列範囲外アクセス
* EQUIVALENCE関連
* コンパイラ依存のメモリレイアウト

これらはまず「潜在バグ候補」として記録してください。

その上で、

* 宣言箇所
* 他の使用箇所
* CALL関係
* 計算式
* コメント
* 入出力
* 周辺コード
* 他の関連ルーチン

を調査し、意図を推定してください。

十分な根拠が得られた場合のみ修正してください。

---

# ソースコード変更の方針

可能であれば、元のAbsoft用コードは参照用として変更せず保存してください。

変更は小さい単位で行ってください。

各重要な変更について、

* 変更前
* 変更後
* 変更理由
* gfortran対応のために必要か
* Python移植のために必要か
* 潜在バグ修正か
* 数値結果への影響可能性
* 検証方法

を記録してください。

変更後は可能な限り、

compile  
→ run  
→ test  
→ output comparison

を行ってください。

---

# Python移植前に必要なリファクタリングについて

gfortran移行後にFortranリファクタリングへ進む際は、

「完全なModern Fortran化」

を目指さないでください。

Python移植のために必要な最小限のリファクタリングを提案してください。

特に以下を重視してください。

* COMMONの型・順序不整合の解消
* implicit typing依存の解消
* 主要な引数型・引数数不整合の解消
* 未初期化変数の解消
* 配列範囲外アクセスの解消
* 主要routineの入力・出力・副作用の明確化
* グローバル状態の整理
* 計算部分とI/Oの分離
* Pythonへ移植するroutineの依存順序の整理

一方で、Python移植に不要な、

* 全COMMONの完全module化
* 全コードのOO化
* 全GOTOの除去
* 全fixed-formコードの全面的書き換え
* 過度な抽象化
* 性能最適化
* アルゴリズム変更

は原則として行わないでください。

リファクタリング案を、

* Python移植に必須
* 安全性・検証のために必要
* あれば望ましい
* 単なるコード美化

に分類してください。

「単なるコード美化」は実施しないでください。

---

# Python移植開始条件

Fortran側のリファクタリングを無期限に続けないため、Python移植開始のgateを設けてください。

少なくとも以下を確認してください。

* gfortranで安定してビルド・実行できる
* Characterization testが存在する
* Verificationの方法が存在する
* COMMONの既知の重大な不整合が解消されている
* implicit typingによる未確認変数がない
* 主要routine間の引数不整合がない
* 既知の未初期化変数問題が解消されている
* 既知の配列範囲外アクセスが解消されている
* 主要routineの入力・出力・副作用が把握できている
* グローバル状態が把握できている
* I/Oと計算部分が識別できている
* Python移植対象の依存順序が分かっている

この条件を満たせば、Fortran側をさらに美しくすることよりもPython移植へ進むことを優先してください。

---

# 共通ツール・Skill化について

このコードで作業する過程で、他のFortranコードにも再利用できそうな作業を識別してください。

例えば、

* COMMON解析
* COMMON不整合検出
* implicit typing依存検出
* CALL関係解析
* 引数型不一致検出
* Absoft固有構文検出
* fixed-form解析
* compiler diagnostics整理
* Fortran出力比較
* Characterization test生成
* Fortran→Python parity test
* migration readiness判定

などです。

ただし、最初からこれらをすべて実装しないでください。

実際に必要になり、繰り返し使用された処理についてのみ、

* 決定論的に処理できるもの → Python等のtool
* 判断・手順・ワークフロー → Agent Skill
* 検証ロジック → test
* このコード固有の知識 → docs

として整理してください。

fparserなど利用可能な解析ライブラリは必要に応じて使用して構いません。

ただし、特定の解析ライブラリだけに依存して判断せず、

* fparser等による静的解析
* gfortranのcompiler diagnostics
* テキスト解析
* コードベースの直接調査

を適切に組み合わせてください。

fparserで解析できないことだけを理由に、ソースコードが誤っているとは判断しないでください。

---

# 最初の成果物

現時点では、大規模な実装修正に入る前に以下を作成してください。

1. リポジトリ構造・コード構造の調査結果
2. 現在のビルド・実行フロー
3. Absoft依存箇所
4. gfortranでの初回ビルド結果
5. compiler / linker diagnosticsの分類
6. 潜在バグ候補
7. リファクタリング上のリスク
8. Characterization / Verification方針
9. Absoft → gfortran移行計画
10. Python移植開始までに必要な最小限のFortranリファクタリング計画
11. Python移植のおおまかな戦略
12. 将来的に共通tool / Skillになりそうな候補

Superpowersのbrainstorming / planningを使い、実際のコードを十分に調査した上で、根拠に基づいて計画を作ってください。

まだ大規模なコード変更は開始しないでください。

このくらいまで書いておけば、**Sonnet 5級のモデルに最初の全体調査を任せるプロンプトとして十分**です。

一点だけ重要なのは、この最初のプロンプトでは**「計画まで」**に止めていることです。出てきた計画を一度確認して、問題なければ次にかなり短く、

> 「計画に従ってPhase 0を実装してください。Superpowersの手順に従い、小さい単位で変更・検証してください。」

と進めればよいです。

私はこの2段階方式をおすすめします。最初から「最後まで全部やって」とすると、CopilotがAbsoft→gfortran移行、バグ修正、Modern Fortran化、Python化を一気に混ぜてしまうリスクが上がるためです。