# Supabase セキュリティ方針

## Row Level Security (RLS)

全てのテーブルに対して、デフォルトで RLS を有効にする。これにより、ポリシーで明示的に許可しない限り、全てのデータアクセスは拒否される。

### RLS の基本方針

- **`SELECT` (閲覧):** 講師は担当生徒の情報のみ、管理者は全情報を閲覧できる。
- **`INSERT` (作成):** ユーザーは自分自身に関連するデータのみ作成できる（例：講師が担当生徒の課題を作成）。
- **`UPDATE` (更新):** ユーザーは自分自身が作成したデータ、または担当する生徒のデータのみ更新できる。
- **`DELETE` (削除):** 原則として物理削除は行わず、`deleted_at`のようなフラグを立てる論理削除を推奨。ポリシーは UPDATE と同様。

---

### RLS ポリシーの具体例

#### `students` テーブル

```sql
-- 管理者（Admin, SystemAdmin）は全生徒を閲覧・操作できる
CREATE POLICY "admins_full_access_on_students" ON students FOR ALL
USING (EXISTS (SELECT 1 FROM instructors WHERE id = auth.uid() AND role IN ('Admin', 'SystemAdmin')));

-- 講師（Instructor）は担当生徒のみを閲覧できる
CREATE POLICY "instructors_select_assigned_students" ON students FOR SELECT
USING (auth.uid() IN (SELECT instructor_id FROM student_instructors WHERE student_id = students.id));
```

#### `schedules` テーブル

```sql
-- 管理者は全スケジュールを操作できる
CREATE POLICY "admins_full_access_on_schedules" ON schedules FOR ALL
USING (EXISTS (SELECT 1 FROM instructors WHERE id = auth.uid() AND role IN ('Admin', 'SystemAdmin')));

-- 講師は担当生徒のスケジュールのみを閲覧・操作できる
CREATE POLICY "instructors_access_assigned_schedules" ON schedules FOR ALL
USING (
  students.id IN (
    SELECT student_id FROM student_instructors
    WHERE instructor_id = auth.uid()
  )
);
```

#### `grades`, `study_plans`, `student_issues` テーブル

```sql
-- （schedules テーブルと同様のポリシーを適用）
```

### ファイルストレージ (Supabase Storage) のアクセスポリシー

- 成績表などの個人情報ファイルは、認証されたユーザーのみがアクセスできるよう、適切なバケットポリシーを設定する。
- ファイルへのアクセスは、有効期限付きの署名付き URL（Signed URL）を発行する方式を基本とする。
