# データモデル設計

## R1で必要なテーブル

### User / Account / Session

Auth.js(NextAuth) + Prisma Adapterの標準テーブルをそのまま使う。独自カラムを追加する場合はここに追記する。

### Exercise(種目)

| カラム       | 型                                            | 備考                                                     |
| ------------ | ---------------------------------------------- | --------------------------------------------------------- |
| id           | string                                          |                                                             |
| name         | string                                          |                                                             |
| bodyPart     | enum(胸/背中/脚/肩/腕/腹筋/その他)              | 部位。タグ付けに使用                                       |
| equipment    | enum(barbell/dumbbell/machine/bodyweight/cable/other) | 器具タイプ                                          |
| recordType   | enum(weight_reps/bodyweight/time)               | 記録形式。ボリューム計算・PR判定に必要                      |
| userId       | string \| null                                  | null = 共通マスタ、値あり = そのユーザーの独自種目           |
| deletedAt    | datetime \| null                                | 論理削除。過去の記録が参照しているため物理削除しない         |
| createdAt    | datetime                                        |                                                             |

### Workout(ワークアウト実績)

| カラム      | 型       | 備考 |
| ----------- | -------- | ---- |
| id          | string   |      |
| userId      | string   |      |
| performedAt | datetime | 実施日時 |
| createdAt   | datetime |      |

### WorkoutExercise(ワークアウト内の種目)

| カラム     | 型     | 備考           |
| ---------- | ------ | -------------- |
| id         | string |                |
| workoutId  | string |                |
| exerciseId | string |                |
| order      | int    | 表示順         |

### WorkoutSet(セット記録)

| カラム            | 型             | 備考                                              |
| ----------------- | -------------- | -------------------------------------------------- |
| id                | string         |                                                     |
| workoutExerciseId | string         |                                                     |
| setNumber         | int            |                                                     |
| weightKg          | float \| null  | recordType = weight_reps のときのみ使用            |
| reps              | int \| null    | recordType = weight_reps / bodyweight のときのみ使用 |
| durationSec       | int \| null    | recordType = time のときのみ使用                    |
| createdAt         | datetime       |                                                     |

`recordType`によって使うカラムが変わる(自重種目なら`weightKg`は使わない等)。可変長データを別テーブルに分けるほどの複雑さではないため、nullable列で対応する。

### BodyWeight(体重記録)

| カラム    | 型       | 備考                          |
| --------- | -------- | ----------------------------- |
| id        | string   |                                |
| userId    | string   |                                |
| date      | date     | 1日1件。`(userId, date)`にunique制約 |
| weightKg  | float    |                                |
| createdAt | datetime |                                |

## R2/R3で追加予定(設計のみ、今は実装しない)

- **WorkoutTemplate**: テンプレート機能用。R1では最小構成として「過去のワークアウトをコピーして開始」で済ませられる可能性があり、専用テーブルが要るかはR2着手時に判断する
- **PersonalRecord**: PR履歴。種目ごとに最大重量を更新するたびレコードを追加する想定
- **AiFeedbackCache**: 週次サマリーの生成結果をキャッシュする
