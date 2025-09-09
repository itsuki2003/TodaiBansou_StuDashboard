# 監査ログ方針

## 概要

主要なデータ（生徒情報、成績、学習状況など）に対する全ての作成・更新・削除（CUD）操作を`audit_logs`テーブルに記録する。

## 実装方法

- Supabase の**トリガー関数**を活用し、対象テーブルへの操作を検知して自動的にログを挿入する。
- `auth.uid()`関数を使用して、操作を行ったユーザーの ID を`changed_by`カラムに記録する。
- 変更前後の値は`OLD`および`NEW`レコードから取得し、JSONB 形式で保存する。

## 対象テーブル

- `students`
- `guardians`
- `contacts`
- `grades`
- `schedules`
- `study_plans`
- `student_issues`
- `student_instructors`
