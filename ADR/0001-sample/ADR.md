# ADR-0001: サンプルデータレイヤー設計

## Status

Proposed

## Context

データプラットフォームにおいて、データレイヤーの設計は以下の観点で重要である:

- **一貫性**: チーム全体で統一された命名規約とレイヤー構造
- **可読性**: 新規メンバーがデータの流れを理解しやすい構成
- **保守性**: レイヤー間の依存関係が明確で、変更の影響範囲が予測可能

現状、以下の課題がある:

1. レイヤー名が社内独自の呼称（raw/prep/modeling/obt）で、dbt公式ベストプラクティスと乖離している
2. プレフィックスが統一されておらず、モデル名からレイヤーを判別しにくい
3. intermediate層の位置づけが曖昧で、marts直下に複雑なロジックが集中している

## Decision

dbt公式ベストプラクティスに準拠した3層構造を採用する。

### レイヤー構造

| レイヤー | プレフィックス | 役割 |
|---------|--------------|------|
| staging | `stg_` | ソースデータの型変換・リネーム |
| intermediate | `int_` | ビジネスロジックの中間変換 |
| marts | `fct_` / `dim_` | 最終的なファクト・ディメンションテーブル |

### 命名規約

```
stg_{source}__{entity}.sql
int_{domain}__{transformation}.sql
fct_{domain}__{event}.sql
dim_{domain}__{entity}.sql
```

### 既存呼称との対応表

| 既存 | 新規 | 備考 |
|------|------|------|
| raw | sources | dbt source定義 |
| prep | staging | 1:1変換のみ |
| modeling | intermediate + marts | ロジックの複雑度で分離 |
| obt | marts (fct_) | One Big Tableはfactとして扱う |

## Consequences

### Positive

- dbt公式ドキュメントやコミュニティの知見をそのまま適用できる
- 新規メンバーのオンボーディングコストが下がる
- intermediate層の導入により、marts層のSQL複雑度が低減する

### Negative

- 既存モデルのリネーム・移動が必要（移行コスト）
- 一時的にCI/CDパイプラインの修正が必要
- BI層のリファレンス更新が発生する

### Risks

- 移行期間中、新旧命名が混在することで混乱が生じる可能性
- intermediate層の粒度判断に主観が入りやすい（ガイドライン別途策定が必要）
