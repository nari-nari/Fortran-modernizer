このリポジトリでは、既存の旧Fortran潤滑解析コードを参照しながら、将来的にFiredrake FEM、自動微分、軸心平衡、ティルティングパッド、トポロジー最適化へ接続可能な新しい潤滑解析基盤を開発したいと考えています。

このリポジトリに導入されているSuperpowers / Agent Skillsを適切に利用してください。

今回はまだ実装を開始しないでください。

まず旧Fortranコード、入力ファイル、既存ドキュメント、利用可能な参考資料を調査し、

- 現在のコードが何をしているのか
- どの数理・物理モデルが実装されているのか
- どのような解析モードが存在するのか
- 旧Fortranをどこまで再利用すべきか
- Python / Firedrakeへどのように移行するのがよいか

を整理したdraftを作成してください。

今回は「最終設計を確定する」のではなく、後で高性能モデルによるarchitecture reviewを行えるだけの事実整理と暫定提案を作ることを目的とします。


# 1. 最終目標

最終的には、このリポジトリを独立した lubrication state solver として構築したいです。

将来的には以下を扱えるようにしたいです。

- Reynolds方程式
- FiredrakeによるFEM
- Elrod–Adams系の質量保存型キャビテーション
- エネルギー方程式との熱連成
- 温度依存の粘度・密度等
- fixed-shaft解析
- 外部荷重に対する軸心位置の収束
- 将来的なtilting-pad bearing
- 各パッド傾き角のモーメント平衡
- pressure / temperature / cavitation state / film thickness / load / friction / flow等の評価
- Firedrake / pyadjoint等による自動微分
- Taylor testによる感度検証
- 別リポジトリで開発しているトポロジー最適化プログラムとの接続

ただし、これらを最初から全て実装するわけではありません。

段階的に検証しながら開発したいです。


# 2. 重要な前提

重要なのは、

「旧FortranコードをそのままPythonへ翻訳すること」

ではありません。

旧Fortranは、

- reference implementation
- 既存挙動の基準
- 数理・物理モデルの情報源
- regression testの基準

として重要です。

一方で、

- FDM固有実装
- COMMON等のglobal state
- GOTO
- legacy mode switch
- 逐次更新の実装詳細

まで必ず新コードに継承する必要はありません。

実コードを調査したうえで判断してください。


# 3. 旧Fortranの既知情報

旧Fortranには少なくとも、

- Reynolds解析
- Elrod–Adamsモデル
- エネルギー方程式
- 熱連成
- 複数の解析モード

が存在します。

入力ファイルによって、

- Elrod–Adamsを考慮するか
- エネルギー方程式を考慮するか
- その他の解析機能を使用するか

を選択できるようになっています。

また、非線形な連成問題については、Newton法による一括solveではなく、

pressure solve
→ temperature solve
→ viscosity / density等の更新
→ pressure solve
→ ...
→ convergence

のような逐次・交互反復が使われていると考えています。

これは事前情報です。

必ず実コードを確認し、異なる場合は実コードを優先してください。


# 4. トポロジー最適化との関係

トポロジー最適化は別リポジトリで開発します。

現時点では、例えば

Topology optimization repository:

rho
→ filter
→ projection
→ d_tex(x)

---------------- repository boundary ----------------

Lubrication repository:

d_tex(x)
→ film thickness
→ lubrication state
→ shaft / pad equilibrium
→ physical outputs

という責務分担を候補としています。

ここで `d_tex(x)` は物理的なテクスチャ深さfieldです。

この潤滑解析リポジトリ単独でも、将来的には `d_tex(x)` 等をcontrolとしてTaylor test可能な構造にしたいです。

ただし、このrepository boundaryが最適であるとはまだ決めていません。

実コード・AD・Firedrake構造を踏まえ、より良い案があれば提案してください。


# 5. 今回比較してほしい移行戦略

以下を比較してください。

## A
旧Fortranをリファクタリング
→ Python化
→ Firedrake化

## B
旧Fortranをreferenceとして解析
→ 数理モデル・アルゴリズム・境界条件等を抽出
→ Python / Firedrake向けにclean implementationを新規構築

## C
機能ごとにA/Bを使い分けるhybrid方式

まだどれを採用するか決めないでください。

実コードを根拠として暫定評価してください。


# 6. 今回はコードを変更しない

今回は禁止します。

- Fortranコードの修正
- Pythonへの自動変換
- Firedrake実装
- solver変更
- 大規模refactoring
- ファイル構成変更

今回は、

- investigation
- documentation
- dependency analysis
- migration assessment
- verification planning
- draft architecture
- draft roadmap

だけを行ってください。


# 7. コード全体の構造調査

以下を調査してください。

- source files
- main program
- subroutines
- functions
- COMMON
- INCLUDE
- MODULE
- call graph
- global/shared state
- input/output
- major loops
- GOTO等のcontrol flow

各主要procedureについて可能な範囲で、

- file
- procedure name
- inputs
- outputs
- shared variables
- called procedures
- physical/numerical role

を整理してください。


# 8. 入力ファイル・解析モードの調査

入力ファイルの、

- mode
- flag
- switch
- option
- model-selection parameter

を抽出してください。

最低限、以下の表を作ってください。

| Input option | Meaning | Allowed values | Read location | Affected physics/procedure |
|---|---|---|---|---|

さらに、

input option
→ active model
→ active equations
→ state variables
→ solver sequence
→ outputs

の対応を整理してください。

特に、

- basic Reynolds
- Elrod–Adams
- isothermal / thermal
- shaft equilibrium

等を、単なるmode番号ではなく独立したモデル選択として整理できるか調べてください。


# 9. 物理モデルと数値実装を分けて調査する

以下について、

「物理・数理モデル」

と

「現在のFDM実装」

を分けて整理してください。

- Reynolds equation
- Elrod–Adams model
- energy equation
- viscosity model
- density model
- film-thickness model
- boundary conditions
- cavitation treatment
- load integration
- friction
- flow rate
- minimum film thickness
- dynamic coefficients（存在する場合）

FDMの更新式だけでなく、

「元になっている連続方程式は何か」

を可能な範囲で特定してください。

コードだけから確定できない場合は、推測せず、

`requires literature verification`

としてください。


# 10. 非線形連成・収束処理

特に、

- pressure
- temperature
- viscosity
- density
- cavitation variable
- film thickness
- shaft position

について、

どの順序で更新され、何を基準に収束判定されているか調査してください。

以下を整理してください。

- outer iteration
- inner iteration
- convergence criteria
- relaxation
- initialization
- previous-iteration values
- restart
- mode-dependent convergence logic

重要なのは、

「現在の反復手順」

と

「その反復が最終的に解こうとしている連成数理問題」

を分けることです。

可能であれば、

R_pressure(...) = 0
R_energy(...) = 0
R_cavitation(...) = 0

のような残差形式として解釈できるか、暫定的に整理してください。


# 11. 軸心収束

軸心位置をどのように求めているか調査してください。

以下に相当するものを探してください。

- eccentricity
- eccentricity ratio
- eccentricity angle
- ex / ey相当量
- hydrodynamic force
- external load
- force residual
- shaft-center update
- relaxation
- convergence criterion

将来的に、

R_x(state, ex, ey, ...) = 0
R_y(state, ex, ey, ...) = 0

という力平衡問題として整理できそうか評価してください。

現在の更新アルゴリズムと物理的な力平衡条件を混同しないでください。


# 12. 将来のtilting-pad拡張

現コードにtilting-pad機能がなくても構いません。

将来的には、

- lubrication PDE
- energy equation
- x/y force equilibrium
- each pad moment equilibrium

を含むstate problemへ拡張したいです。

現在のコードから再利用可能そうな部分と、将来一般化が必要そうな部分を整理してください。


# 13. Dependency graph

call graphとは別に、主要物理量のdependency graphを作ってください。

例えば、

d_tex
→ film thickness
→ pressure / cavitation / temperature
→ viscosity / density
→ hydrodynamic force
→ shaft equilibrium
→ converged state
→ outputs

のような関係です。

主要変数を可能な範囲で、

- parameter
- physical design variable
- geometry variable
- PDE state
- equilibrium state
- derived quantity
- output

へ分類してください。


# 14. Firedrake / ADへの移行上の注意点

将来Firedrake / pyadjoint等を使用することを想定し、

- COMMON
- global state
- hidden dependency
- in-place update
- GOTO
- iteration-history dependency
- if-based switching
- max/min
- clipping
- cavitation switching
- mode-dependent branch

等を調査してください。

ただし、今回はAD設計を確定しないでください。

特に、

A.
現在の逐次反復をそのまま微分対象にする

B.
収束解を

R(state, design) = 0

として扱う

という2つの考え方があることを認識したうえで、どの部分が今後の詳細検討を必要とするか整理してください。


# 15. 参考文献・benchmark

旧Fortranは、日本機械学会の
「すべり軸受の静特性及び動特性資料集」
を比較対象の一つとして開発されたと聞いています。

利用可能であれば、基礎的なstatic / dynamic characteristicsのreference候補として整理してください。

ただし、JSME資料集だけを唯一のreferenceにはしないでください。

特に、

- Elrod–Adams
- thermal / THD
- temperature-dependent properties
- textured bearing
- modern cavitation treatment
- dynamic coefficients

等については、必要に応じて公開論文やbenchmarkを調査・提案してください。

今回は文献を大量に探すこと自体を目的にはしません。

「この機能にはどの種類のreferenceが必要か」
を整理することを優先してください。


# 16. 文献provenance

文献を実際に使用した場合は、必ず出典を記録してください。

最低限、

- title
- authors
- year
- journal
- DOI or stable URL
- equation / figure / table / page（可能な場合）
- 何の理解・検証に利用したか

を記録してください。

出典が確認できない情報は、

`reference not yet verified`

としてください。


# 17. 検証戦略

少なくとも以下を区別してください。

## Legacy regression
旧Fortranとの比較。

## Mathematical / numerical verification
解析解、benchmark、mesh refinement、conservation等。

## Literature / experimental validation
JSME資料集や公開論文との比較。

## AD verification
将来のTaylor test。

旧Fortranとの一致だけを新solverの正しさの唯一の根拠にしないでください。


# 18. Taylor testの将来計画

将来的には潤滑解析リポジトリ単独で、

1. fixed shaft + basic Reynolds
2. + Elrod–Adams
3. + thermal coupling
4. + shaft equilibrium
5. + tilting-pad equilibrium

という順でTaylor testを拡張できる構造を検討したいです。

`d_tex(x)` をfield-valued controlの候補としています。

今回はTaylor testを実装せず、実現するためにどの依存関係を明確にする必要があるかだけ整理してください。


# 19. 機能別migration assessment

主要機能について暫定的に以下のどれが適切そうか評価してください。

A.
ほぼそのままPythonへ移植

B.
最低限Fortranを整理してからPythonへ移植

C.
Fortranから仕様を抽出しclean Pythonとして再実装

D.
Python FDMへの忠実移植を経由せずFiredrakeで新規実装

最低限、

- geometry
- film thickness
- Reynolds
- Elrod–Adams
- energy equation
- viscosity / density
- load integration
- friction
- flow
- shaft equilibrium
- nonlinear coupling
- input/output

を評価してください。

ただし、このA/B/C/D判断は今回は「draft」です。

確定判断は後続のarchitecture reviewで行います。


# 20. Reference Python FDM

旧Fortranを比較的忠実にPython化したreference FDM solverを作る価値があるか評価してください。

作る価値がある場合は、

- 何のために作るのか
- どこまで再現するか
- 最終Firedrake版とどう使い分けるか

を整理してください。

これも暫定案として扱ってください。


# 21. 成果物

今回は最低限、以下をdraftとして作成してください。

1. `LEGACY_CODE_ARCHITECTURE.md`
2. `ANALYSIS_MODE_MATRIX.md`
3. `PHYSICS_AND_NUMERICS_MAP.md`
4. `DEPENDENCY_GRAPH.md`
5. `MIGRATION_ASSESSMENT_DRAFT.md`
6. `REFERENCE_AND_BENCHMARK_MAP.md`
7. `VERIFICATION_STRATEGY_DRAFT.md`
8. `TARGET_ARCHITECTURE_DRAFT.md`
9. `DEVELOPMENT_ROADMAP_DRAFT.md`

必要に応じてMermaidを使用してください。


# 22. 特に重要なルール

今回は「立派な最終案を書く」ことより、

「後続レビューで確認できる根拠を集める」

ことを優先してください。

調査結果を必ず、

- confirmed from code
- confirmed from reference
- inferred
- requires literature verification
- requires human confirmation

に区別してください。

重要な判断には、可能な限り、

- file
- procedure
- variable
- input option

等の根拠を示してください。

分からないことを推測で埋めないでください。

最後に、

「Sol等の高性能モデルによるreviewで重点的に再確認すべき事項」

を `REVIEW_CHECKLIST.md` としてまとめてください。

今回はコードを変更しないでください。