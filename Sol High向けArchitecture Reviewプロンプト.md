Lunaが旧Fortran潤滑解析コードを調査し、以下のdraft資料を作成しました。

- `LEGACY_CODE_ARCHITECTURE.md`
- `ANALYSIS_MODE_MATRIX.md`
- `PHYSICS_AND_NUMERICS_MAP.md`
- `DEPENDENCY_GRAPH.md`
- `MIGRATION_ASSESSMENT_DRAFT.md`
- `REFERENCE_AND_BENCHMARK_MAP.md`
- `VERIFICATION_STRATEGY_DRAFT.md`
- `TARGET_ARCHITECTURE_DRAFT.md`
- `DEVELOPMENT_ROADMAP_DRAFT.md`
- `REVIEW_CHECKLIST.md`

これらをarchitecture / numerical-method / AD-readinessの観点からレビューしてください。

Lunaの結論をそのまま前提にはしないでください。

ただし、旧Fortranコード全体を最初から再調査する必要もありません。

まずdraft資料を読み、重要な判断について必要な箇所だけ原Fortranコード・入力ファイル・参考資料をspot-checkしてください。

特に以下を重点的に確認してください。

1. 旧Fortranをリファクタリング・Python化する部分と、clean implementation / Firedrake新規実装にする部分の切り分け

2. Reynolds / Elrod–Adams / energy equation / thermal couplingについて、物理モデルとFDM実装が正しく分離して解釈されているか

3. pressure / temperature / viscosity / density等の逐次反復について、
   - 現在のsolver algorithm
   - 背後にあるcoupled mathematical problem
   が正しく区別されているか

4. 軸心収束を将来的に `ex, ey` と力平衡残差で表現することの妥当性

5. 将来のtilting-pad / pad-angle equilibriumへの拡張性

6. Firedrake FEMへの移行方針

7. pyadjoint等による自動微分を考慮したstate / control / outputの依存関係

8. 逐次反復をそのままtapeに載せる方法と、
   `R(state, design)=0`
   として収束状態を扱う方法について、今後どちらを優先的に検討すべきか

9. `d_tex(x)` を潤滑solver側のphysical design fieldとして、別topology-optimization repositoryとの境界に置く案の妥当性

10. 潤滑解析repository単独でのTaylor-test戦略

11. JSME資料集、旧Fortran、公開論文、解析解、Taylor testを使ったverification hierarchy

12. reference Python FDM solverを作る価値が本当にあるか

13. development roadmapの順序が適切か

重要な判断については原コードをspot-checkし、Lunaの解釈が不正確なら修正してください。

最終的に、

- `MIGRATION_ASSESSMENT.md`
- `VERIFICATION_STRATEGY.md`
- `TARGET_ARCHITECTURE.md`
- `DEVELOPMENT_ROADMAP.md`

を確定版として作成してください。

また、Lunaの調査で情報不足だった箇所については、推測せず追加調査事項として残してください。