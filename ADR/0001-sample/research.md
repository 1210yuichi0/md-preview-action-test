# リサーチ: データレイヤー設計

## 調査対象

- dbt公式ベストプラクティス
- 他社事例（GitLab, dbt Labs内部プロジェクト）
- 社内現状の課題

## dbt公式ベストプラクティス

### How we structure our dbt projects (dbt Labs, 2024)

dbt Labsが推奨する標準的なプロジェクト構造:

1. **staging**: ソースごとに1:1でモデルを作成。型変換、リネーム、基本的なクレンジングのみ
2. **intermediate**: staging層のモデルを結合・変換する中間層。直接BIツールから参照されない
3. **marts**: ビジネスユーザーが直接利用するファクト・ディメンションテーブル

> staging models are the atomic unit of data modeling. Each model bears a one-to-one relationship with the source data table it represents.
> — dbt Labs Best Practices

### 命名規約の根拠

プレフィックスによりDAG上でモデルの役割が即座に判別できる:

- `stg_`: 「このモデルはソースデータの整形のみ」
- `int_`: 「このモデルは中間ロジックで、直接参照すべきでない」
- `fct_`/`dim_`: 「このモデルがビジネスの最終成果物」

## 他社事例

### GitLab (gitlab-data/analytics)

- staging / intermediate / marts の3層構造を採用
- intermediate層を積極的に活用し、marts層のSQLを簡潔に保つ
- 500+モデル規模でもプレフィックスにより管理可能

### dbt Labs内部 (jaffle_shop_advanced)

- 公式デモプロジェクトとして3層構造のリファレンス実装
- semantic layer (MetricFlow) との統合を前提とした設計

## 比較検討

| 選択肢 | メリット | デメリット |
|--------|---------|-----------|
| A. 3層 (staging/int/marts) | 業界標準、ドキュメント豊富 | intermediate の粒度判断が必要 |
| B. 4層 (+raw/base) | ソース変更への耐性が高い | 層が多く複雑、小規模では過剰 |
| C. 2層 (staging/marts) | シンプル | marts が肥大化しやすい |
| D. 現状維持 (raw/prep/modeling/obt) | 移行不要 | dbtコミュニティとの乖離が拡大 |

## 結論

選択肢Aを推奨。規模感（100-300モデル想定）に対して適切な複雑度であり、dbtコミュニティの知見を最大限活用できる。
