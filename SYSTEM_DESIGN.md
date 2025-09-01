# 企業向け社員教育プラットフォーム - システム設計書

## 1. プロジェクト概要

### ビジョン
導入するだけで自動的に社員のスキルアップを促進し、組織全体の知識レベルを向上させる教育プラットフォーム

### 主要機能
- 毎日定時（10時）に5択問題を自動配信
- スコア管理とゲーミフィケーション
- 学習コンテンツ（動画・テキスト）管理
- 進捗トラッキングと分析
- 管理者向けダッシュボード

## 2. システムアーキテクチャ

### 技術スタック
```
フロントエンド:
- Next.js 14 (App Router)
- TypeScript
- Tailwind CSS
- Shadcn/ui
- React Query (データフェッチング)
- Chart.js (分析グラフ)

バックエンド:
- Node.js + Express
- TypeScript
- Prisma ORM
- PostgreSQL
- Redis (キャッシング・セッション管理)

認証・通知:
- NextAuth.js (認証)
- SendGrid (メール通知)
- Web Push API (ブラウザ通知)
- Slack API (Slack連携)

インフラ:
- Vercel (フロントエンド)
- Railway/Render (バックエンド)
- Supabase (PostgreSQL + Storage)
- Cloudflare R2 (動画配信)
```

## 3. データベース設計

### 主要テーブル

#### Companies (企業)
```sql
- id: UUID
- name: string
- subdomain: string (unique)
- plan: enum (starter, professional, enterprise)
- settings: json
- created_at: timestamp
- updated_at: timestamp
```

#### Users (ユーザー)
```sql
- id: UUID
- company_id: UUID (FK)
- email: string (unique)
- name: string
- role: enum (admin, manager, employee)
- department: string
- position: string
- avatar_url: string
- total_score: integer
- streak_days: integer
- last_answer_date: date
- notification_settings: json
- created_at: timestamp
```

#### Categories (カテゴリー)
```sql
- id: UUID
- company_id: UUID (FK)
- name: string
- description: text
- icon: string
- order: integer
- is_active: boolean
```

#### LearningContents (学習コンテンツ)
```sql
- id: UUID
- company_id: UUID (FK)
- category_id: UUID (FK)
- title: string
- type: enum (video, article, slide)
- content_url: string
- description: text
- duration_minutes: integer
- difficulty: enum (beginner, intermediate, advanced)
- tags: string[]
- created_by: UUID (FK to Users)
- created_at: timestamp
```

#### Questions (問題)
```sql
- id: UUID
- company_id: UUID (FK)
- category_id: UUID (FK)
- learning_content_id: UUID (FK, nullable)
- question_text: text
- question_type: enum (multiple_choice, true_false)
- difficulty: integer (1-5)
- points: integer
- explanation: text
- reference_links: json
- is_active: boolean
- created_by: UUID (FK to Users)
- created_at: timestamp
```

#### QuestionOptions (選択肢)
```sql
- id: UUID
- question_id: UUID (FK)
- option_text: string
- is_correct: boolean
- order: integer
```

#### DailyQuizzes (日次クイズ)
```sql
- id: UUID
- company_id: UUID (FK)
- question_id: UUID (FK)
- scheduled_date: date
- sent_at: timestamp
- category_id: UUID (FK)
```

#### UserAnswers (回答)
```sql
- id: UUID
- user_id: UUID (FK)
- question_id: UUID (FK)
- selected_option_id: UUID (FK)
- is_correct: boolean
- points_earned: integer
- answered_at: timestamp
- time_spent_seconds: integer
```

#### UserProgress (学習進捗)
```sql
- id: UUID
- user_id: UUID (FK)
- learning_content_id: UUID (FK)
- status: enum (not_started, in_progress, completed)
- progress_percentage: integer
- completed_at: timestamp
- last_accessed_at: timestamp
```

#### Achievements (実績)
```sql
- id: UUID
- name: string
- description: string
- icon: string
- condition_type: enum (score, streak, completion, perfect)
- condition_value: integer
- points: integer
```

#### UserAchievements (ユーザー実績)
```sql
- id: UUID
- user_id: UUID (FK)
- achievement_id: UUID (FK)
- earned_at: timestamp
```

## 4. 主要機能の詳細

### 4.1 問題配信システム
```typescript
// 毎日10時に実行されるCronジョブ
interface QuizScheduler {
  scheduleDaily(): Promise<void>
  selectQuestions(companyId: string): Promise<Question[]>
  sendNotifications(users: User[], quiz: DailyQuiz): Promise<void>
}

// 問題選択アルゴリズム
- ユーザーの正答率を分析
- 苦手分野を重点的に出題
- 新規コンテンツを優先
- 難易度を段階的に調整
```

### 4.2 スコアリングシステム
```typescript
interface ScoringSystem {
  calculatePoints(answer: UserAnswer): number
  updateStreak(userId: string): Promise<void>
  checkAchievements(userId: string): Promise<Achievement[]>
  getLeaderboard(companyId: string, period: 'daily' | 'weekly' | 'monthly'): Promise<LeaderboardEntry[]>
}

// ポイント計算ロジック
- 基本ポイント: 10点
- 正解ボーナス: +難易度×5点
- 連続正解ボーナス: +連続数×2点
- 速答ボーナス: 30秒以内で+5点
- ストリークボーナス: 連続日数×1点
```

### 4.3 学習推奨システム
```typescript
interface LearningRecommendation {
  analyzeWeakPoints(userId: string): Promise<Category[]>
  recommendContent(userId: string): Promise<LearningContent[]>
  generateLearningPath(userId: string): Promise<LearningPath>
}
```

### 4.4 管理者ダッシュボード
```typescript
interface AdminDashboard {
  // 分析機能
  getCompanyMetrics(): Promise<CompanyMetrics>
  getDepartmentComparison(): Promise<DepartmentStats[]>
  getEngagementTrends(): Promise<EngagementData>
  
  // コンテンツ管理
  uploadContent(content: LearningContent): Promise<void>
  createQuestion(question: Question): Promise<void>
  reviewAnswerStats(questionId: string): Promise<QuestionStats>
}
```

## 5. API設計

### RESTful API エンドポイント
```
認証:
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/refresh
GET    /api/auth/me

問題:
GET    /api/quiz/daily - 本日の問題取得
POST   /api/quiz/answer - 回答送信
GET    /api/quiz/history - 回答履歴

学習コンテンツ:
GET    /api/contents - コンテンツ一覧
GET    /api/contents/:id - コンテンツ詳細
POST   /api/contents/:id/progress - 進捗更新

スコア・実績:
GET    /api/scores/me - 自分のスコア
GET    /api/scores/leaderboard - リーダーボード
GET    /api/achievements - 実績一覧

管理者:
GET    /api/admin/analytics - 分析データ
POST   /api/admin/contents - コンテンツ作成
POST   /api/admin/questions - 問題作成
GET    /api/admin/users - ユーザー管理
```

## 6. UI/UXデザイン

### 画面構成
```
社員向け:
- ダッシュボード (今日の問題、スコア、ストリーク)
- クイズ画面 (5択問題、タイマー、ヒント)
- 学習ライブラリ (動画、記事、カテゴリー別)
- マイページ (進捗、実績、履歴)
- ランキング (全社、部署別、月間/週間)

管理者向け:
- 分析ダッシュボード
- コンテンツ管理
- 問題管理
- ユーザー管理
- 設定 (通知時間、カテゴリー、ポイント設定)
```

## 7. 通知戦略

### マルチチャネル通知
```typescript
interface NotificationStrategy {
  channels: {
    email: boolean
    slack: boolean
    browser: boolean
    mobile: boolean
  }
  timing: {
    dailyQuiz: '10:00'
    reminder: '14:00'  // 未回答者向け
    weeklyReport: 'friday 17:00'
  }
  content: {
    personalized: boolean
    includeLeaderboard: boolean
    motivationalMessage: boolean
  }
}
```

## 8. セキュリティ

### 実装項目
- JWT認証 + Refresh Token
- Rate Limiting
- SQL Injection対策 (Prisma ORM)
- XSS対策 (React自動エスケープ)
- CORS設定
- データ暗号化 (保存時・通信時)
- 監査ログ
- GDPR/個人情報保護法準拠

## 9. スケーラビリティ

### 対応策
- マイクロサービス対応可能な設計
- Redis キャッシング
- CDN活用 (静的アセット・動画)
- データベース読み書き分離対応
- 非同期処理 (Bull Queue)
- Webhook対応

## 10. 導入フロー

### Phase 1: MVP (2週間)
- 基本的な問題配信機能
- シンプルなスコアリング
- Web版のみ

### Phase 2: 拡張機能 (1ヶ月)
- 学習コンテンツ管理
- 詳細な分析機能
- Slack連携

### Phase 3: エンタープライズ (2ヶ月)
- マルチテナント対応
- API公開
- モバイルアプリ
- AI問題生成

## 11. 成功指標 (KPI)

```typescript
interface KPIs {
  engagement: {
    dailyActiveRate: number  // 目標: 80%以上
    averageAnswerTime: number  // 目標: 2分以内
    streakMaintenance: number  // 目標: 60%が7日以上
  }
  learning: {
    averageScore: number  // 目標: 70点以上
    contentCompletionRate: number  // 目標: 50%以上
    knowledgeRetention: number  // 1ヶ月後の再テスト正答率
  }
  business: {
    employeeSatisfaction: number
    managerSatisfaction: number
    knowledgeGapReduction: number
  }
}
```