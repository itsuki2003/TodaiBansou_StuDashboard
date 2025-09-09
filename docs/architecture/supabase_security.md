# Supabase セキュリティ方針

## Row Level Security (RLS)

全てのテーブルに対して、デフォルトで RLS を有効にする。アクセス許可はポリシーを通じて明示的に行う。

### `students` テーブルのポリシー例

```sql
-- RLSを有効化
ALTER TABLE students ENABLE ROW LEVEL SECURITY;

-- 管理者（Admin, SystemAdmin）は全生徒を閲覧可能
CREATE POLICY "admins_select_all_students" ON students
FOR SELECT TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM instructors
    WHERE id = auth.uid() AND role IN ('Admin', 'SystemAdmin')
  )
);

-- 講師（Instructor）は担当生徒のみ閲覧可能
CREATE POLICY "instructors_select_assigned_students" ON students
FOR SELECT TO authenticated
USING (
  auth.uid() IN (
    SELECT instructor_id FROM student_instructors
    WHERE student_id = students.id
  )
);

-- 講師は担当生徒の情報を更新可能
CREATE POLICY "instructors_update_assigned_students" ON students
FOR UPDATE TO authenticated
USING (
  auth.uid() IN (
    SELECT instructor_id FROM student_instructors
    WHERE student_id = students.id
  )
);
```

### ファイルストレージ (Supabase Storage) のアクセスポリシー

- 成績表などの個人情報ファイルは、認証されたユーザーのみがアクセスできるよう、適切なバケットポリシーを設定する。
- ファイルへのアクセスは、有効期限付きの署名付き URL（Signed URL）を発行する方式を基本とする。
