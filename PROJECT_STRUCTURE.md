# プロジェクト構造

```
corporate-learning-platform/
├── apps/
│   ├── web/                    # Next.js フロントエンド
│   │   ├── app/                # App Router
│   │   │   ├── (auth)/         # 認証関連ページ
│   │   │   ├── (dashboard)/    # ダッシュボード
│   │   │   ├── (admin)/        # 管理者画面
│   │   │   ├── api/            # API Routes
│   │   │   └── layout.tsx
│   │   ├── components/         # UIコンポーネント
│   │   ├── lib/               # ユーティリティ
│   │   ├── hooks/             # カスタムフック
│   │   └── public/
│   │
│   └── api/                    # Express バックエンド
│       ├── src/
│       │   ├── controllers/    # コントローラー
│       │   ├── services/       # ビジネスロジック
│       │   ├── models/         # データモデル
│       │   ├── middlewares/    # ミドルウェア
│       │   ├── utils/          # ユーティリティ
│       │   └── index.ts
│       └── prisma/
│           └── schema.prisma
│
├── packages/
│   ├── database/              # Prisma スキーマ & マイグレーション
│   ├── shared/                # 共有型定義・ユーティリティ
│   └── ui/                    # 共有UIコンポーネント
│
├── scripts/                   # ビルド・デプロイスクリプト
├── docker/                    # Docker設定
└── docs/                      # ドキュメント
```

## 各ディレクトリの役割

### apps/web (フロントエンド)
- **技術**: Next.js 14, TypeScript, Tailwind CSS
- **主要機能**: 
  - 社員向けUI (クイズ、学習、ダッシュボード)
  - 管理者向けUI (分析、コンテンツ管理)
  - リアルタイム通知

### apps/api (バックエンド)
- **技術**: Express, TypeScript, Prisma
- **主要機能**:
  - RESTful API
  - 認証・認可
  - ビジネスロジック
  - データベースアクセス

### packages/database
- データベーススキーマ定義
- マイグレーション管理
- シードデータ

### packages/shared
- 共通型定義
- バリデーションルール
- 定数定義
- ユーティリティ関数

### packages/ui
- 再利用可能なUIコンポーネント
- デザインシステム
- アイコンセット