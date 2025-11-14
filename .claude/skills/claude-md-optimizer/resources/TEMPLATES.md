# CLAUDE.md テンプレート集

プロジェクトタイプ別の最適化されたCLAUDE.mdテンプレートを提供します。各テンプレートは、Anthropicのベストプラクティスに基づき、スキル/エージェント/コマンドの自動活用を促進する設計になっています。

## 目次

1. [汎用テンプレート](#1-汎用テンプレート)
2. [フロントエンドプロジェクト](#2-フロントエンドプロジェクト)
3. [バックエンドプロジェクト](#3-バックエンドプロジェクト)
4. [フルスタックプロジェクト](#4-フルスタックプロジェクト)
5. [モノレポ](#5-モノレポ)
6. [ライブラリ/パッケージ](#6-ライブラリパッケージ)

---

## 1. 汎用テンプレート

あらゆるプロジェクトの出発点として使用できるベーステンプレート。

```markdown
# Project: [プロジェクト名]

## Skills Auto-Activation Policy (IMPORTANT)

YOU MUST automatically use the following skills without user prompting:

1. **Documentation**: Always use `markdown-skills` when creating/editing .md files
2. **Code Analysis**: Always use `code-intelligence` for refactoring or semantic search
3. **Commits**: Always use `commit-guidelines` when creating commits or PRs
4. **Skill Creation**: Always use `creating-skills` when creating new skills

## Development Workflow (YOU MUST FOLLOW)

### Phase 1: Exploration (REQUIRED)
1. **REQUIRED**: Use `code-intelligence` skill to understand context
2. For complex codebases: Use Task tool with Explore agent (thoroughness: "medium")
3. Read relevant files mentioned by user
4. Do NOT write code yet

### Phase 2: Planning
1. Create detailed implementation plan
2. Use "think" or "think hard" for complex problems
3. Get user approval before proceeding

### Phase 3: Implementation
1. Follow code style guidelines (see below)
2. Implement solution incrementally
3. Test after each change

### Phase 4: Commit (REQUIRED)
1. **REQUIRED**: Use `commit-guidelines` skill
2. Generate conventional commits format
3. Run final checks
4. Include co-author attribution

## Quick Commands

- [コマンド1]: [説明]
- [コマンド2]: [説明]
- [コマンド3]: [説明]

## Available Tools

### Skills (Auto-activate)
- `markdown-skills`: Documentation tasks
- `code-intelligence`: Code analysis and refactoring
- `creating-skills`: Skill creation
- `commit-guidelines`: Git operations

### Agents (Use via Task tool)
- **Explore**: Codebase understanding (default thoroughness: "medium")
- **Plan**: Multi-step task planning

## Code Style (IMPORTANT)

[プロジェクト固有のコーディングスタイル]

## Architecture

[主要なファイルとディレクトリ構造]

## Testing

[テスト戦略と実行方法]

## Git Workflow

- Branch naming: `feature/` or `fix/` prefix
- Commits: Conventional Commits format (via commit-guidelines skill)
- [その他のGit規約]

## Environment Setup

[環境セットアップ手順]

## Known Issues

[既知の問題と回避策]
```

---

## 2. フロントエンドプロジェクト

React/Next.js/Vue.js等のフロントエンドプロジェクト向け。

```markdown
# Project: [プロジェクト名] (Frontend)

## Skills Auto-Activation Policy (IMPORTANT)

YOU MUST automatically use the following skills without user prompting:

1. **Documentation**: Always use `markdown-skills` for .md files and Storybook docs
2. **Component Refactoring**: Always use `code-intelligence` for component changes
3. **Commits**: Always use `commit-guidelines` for git operations
4. **Visual Testing**: Auto-use Puppeteer MCP for screenshots when available

## Development Workflow (YOU MUST FOLLOW)

### Phase 1: Component Planning
1. **REQUIRED**: Use `code-intelligence` to understand existing component patterns
2. Review design mock or requirements
3. Plan component structure and props
4. Get user approval

### Phase 2: Implementation
1. Create component file in appropriate directory
2. **RECOMMENDED**: Create Storybook story alongside component
3. Implement component logic
4. Apply styling (see Code Style below)

### Phase 3: Visual Verification
1. **RECOMMENDED**: Use Puppeteer MCP to take screenshots
2. Compare with design mock
3. Test on multiple screen sizes (mobile, tablet, desktop)
4. Iterate until match

### Phase 4: Testing & Commit
1. Write component tests (see Testing section)
2. Run test suite
3. **REQUIRED**: Use `commit-guidelines` for commit
4. Create PR if ready

## Quick Commands

- `npm run dev` - Start development server (http://localhost:3000)
- `npm run build` - Build for production
- `npm run storybook` - Start Storybook (http://localhost:6006)
- `npm test` - Run test suite
- `npm run lint` - Run linter

## Available Tools

### Skills (Auto-activate)
- `markdown-skills`: Documentation and Storybook
- `code-intelligence`: Component refactoring
- `commit-guidelines`: Git operations

### Custom Commands
[カスタムコマンドがあれば記載]

### Agents
- **Explore**: Use "medium" thoroughness for component discovery
- **Plan**: Use for complex feature planning

### MCP Servers
- **Puppeteer**: Screenshot testing and visual comparison
  - Auto-use when user provides design mock
  - Use for responsive testing

## Code Style (IMPORTANT)

### React Components
- Use function components with arrow functions only
- Props destructuring in function signature
- Early returns for conditional rendering
- Example:
```typescript
export const Button = ({ variant, children, onClick }: ButtonProps) => {
  if (!children) return null
  return <button className={styles[variant]} onClick={onClick}>{children}</button>
}
```

### Styling
- Use [Tailwind CSS / CSS Modules / Styled Components]
- Mobile-first approach
- Follow design system tokens

### State Management
- Local state: `useState` / `useReducer`
- Global state: [Redux / Jotai / Zustand / Context]
- Server state: [React Query / SWR]

### Imports
- Group imports: React → External → Internal → Styles
- Use absolute imports via path aliases
- Example:
```typescript
import { useState } from 'react'
import { motion } from 'framer-motion'
import { Button } from '@/components/Button'
import styles from './Component.module.css'
```

## Architecture

```
src/
├── app/              [Next.js App Router pages / Route components]
├── components/       [Reusable components]
│   ├── ui/          [Atomic UI components]
│   ├── layout/      [Layout components]
│   └── features/    [Feature-specific components]
├── hooks/           [Custom React hooks]
├── lib/             [Utility functions]
├── styles/          [Global styles and theme]
└── types/           [TypeScript type definitions]
```

Key files:
- `src/app/layout.tsx` - Root layout
- `src/components/ui/` - Design system components
- `src/styles/globals.css` - Global styles

## Testing

### Unit Tests
- Framework: [Jest / Vitest]
- React: React Testing Library
- Location: `__tests__/` or `*.test.tsx` alongside component
- **REQUIRED**: Test all new components

### Visual Testing
- Storybook for component showcase
- **RECOMMENDED**: Use Puppeteer MCP for screenshot comparison

### E2E Tests
- Framework: [Playwright / Cypress]
- Location: `e2e/` directory
- Run before major releases

## Git Workflow

- Branch: `feature/component-name` or `fix/issue-description`
- Commits: Conventional Commits (via commit-guidelines skill)
  - feat: New component or feature
  - fix: Bug fix
  - style: Styling changes
  - refactor: Component refactoring
- **IMPORTANT**: Rebase, not merge
- PR must include: Screenshot, test coverage

## Environment Setup

- Node.js: v18+ (use nvm: `nvm use`)
- Package manager: [npm / yarn / pnpm]
- **REQUIRED**: Run `npm install` after pulling

Environment variables: Copy `.env.example` to `.env.local`

Required:
- `NEXT_PUBLIC_API_URL` - API endpoint
- [その他の環境変数]

## Known Issues

- Hot reload may fail after large changes: Restart dev server
- [その他の既知の問題]
```

---

## 3. バックエンドプロジェクト

Node.js/Python/Go等のバックエンドAPI向け。

```markdown
# Project: [プロジェクト名] (Backend API)

## Skills Auto-Activation Policy (IMPORTANT)

YOU MUST automatically use the following skills without user prompting:

1. **Documentation**: Always use `markdown-skills` for API docs
2. **API Refactoring**: Always use `code-intelligence` for endpoint changes
3. **Commits**: Always use `commit-guidelines` for git operations
4. **Database Changes**: REQUIRED to commit migrations separately

## Development Workflow (YOU MUST FOLLOW)

### Phase 1: API Design
1. **REQUIRED**: Use `code-intelligence` to understand existing API patterns
2. Define API contract (OpenAPI / tRPC schema / GraphQL schema)
3. Plan database schema changes if needed
4. Get user approval

### Phase 2: Test-Driven Development (REQUIRED)
1. Write integration tests FIRST (define expected behavior)
2. Run tests - confirm they fail
3. **REQUIRED**: Commit tests before implementation

### Phase 3: Implementation
1. Implement endpoint/service logic
2. Follow code style (see below)
3. Run tests after each change
4. Iterate until all tests pass

### Phase 4: Database Migrations (if applicable)
1. **REQUIRED**: Create migration file
2. Test migration up/down
3. **REQUIRED**: Commit migration separately from code

### Phase 5: Commit & Deploy
1. **REQUIRED**: Use `commit-guidelines` for commit
2. Update API documentation
3. Create PR with API changes documented

## Quick Commands

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm test` - Run test suite
- `npm run db:migrate` - Run database migrations
- `npm run db:seed` - Seed database with test data

## Available Tools

### Skills (Auto-activate)
- `markdown-skills`: API documentation
- `code-intelligence`: API refactoring and analysis
- `commit-guidelines`: Git operations

### Custom Commands
[カスタムコマンドがあれば記載]

### Agents
- **Explore**: Use "medium" thoroughness for API discovery
- **Plan**: Use for complex feature planning

## Code Style (IMPORTANT)

### General
- Language: [TypeScript / Python / Go]
- Async: [async/await / promises / goroutines]
- Error handling: Explicit error returns, no silent failures

### API Endpoints ([Framework name])
```[language]
[フレームワーク固有の例]

// Express.js example:
router.post('/api/users', async (req, res) => {
  try {
    const user = await createUser(req.body)
    res.json({ data: user })
  } catch (error) {
    res.status(400).json({ error: error.message })
  }
})
```

### Database Access
- ORM: [Prisma / TypeORM / SQLAlchemy / GORM]
- Always use transactions for multi-step operations
- Use prepared statements (SQL injection prevention)

### Validation
- Request validation: [Zod / Joi / class-validator]
- Validate all user inputs
- Return clear error messages

## Architecture

```
src/
├── server.ts         [Entry point]
├── routes/           [API route definitions]
├── controllers/      [Request handlers]
├── services/         [Business logic]
├── models/           [Data models / ORM schemas]
├── middleware/       [Custom middleware]
├── utils/            [Utility functions]
└── types/            [TypeScript types]
```

Key patterns:
- Routes → Controllers → Services → Models
- Authentication: [JWT / Session / OAuth]
- Authorization: [Role-based / Permission-based]

Database schema: `prisma/schema.prisma` or `models/`

## Testing

### Integration Tests (REQUIRED)
- Framework: [Jest / Vitest / pytest / Go testing]
- Test API endpoints end-to-end
- Location: `__tests__/` or `tests/`
- **REQUIRED**: Write tests BEFORE implementation (TDD)

### Unit Tests
- Test business logic in services
- Mock database calls
- Aim for >80% coverage

### Load Testing
- Tool: [k6 / Apache Bench]
- Test before production deployment

## Git Workflow

- Branch: `feature/endpoint-name` or `fix/issue-description`
- Commits: Conventional Commits (via commit-guidelines skill)
  - feat: New endpoint or feature
  - fix: Bug fix
  - perf: Performance improvement
  - refactor: Code refactoring
- **IMPORTANT**: Database migrations in separate commits
- **IMPORTANT**: Rebase, not merge

## Environment Setup

### Prerequisites
- [Runtime]: [Node.js v18+ / Python 3.11+ / Go 1.21+]
- Database: [PostgreSQL 14+ / MySQL 8+ / MongoDB 6+]
- Redis: [If applicable]

### Setup Steps
1. Install dependencies: `npm install` / `pip install -r requirements.txt`
2. Setup database: `npm run db:setup`
3. Copy `.env.example` to `.env`
4. Run migrations: `npm run db:migrate`
5. Start server: `npm run dev`

### Environment Variables
Required:
- `DATABASE_URL` - Database connection string
- `JWT_SECRET` - JWT signing secret
- `PORT` - Server port (default: 3000)
- [その他の環境変数]

## API Documentation

- Format: [OpenAPI / tRPC / GraphQL Schema]
- Location: `docs/api.md` or auto-generated
- **REQUIRED**: Update after API changes

View docs: [URL or command]

## Known Issues

- [既知の問題1]
- [既知の問題2]
```

---

## 4. フルスタックプロジェクト

フロントエンド + バックエンドの統合プロジェクト向け。

```markdown
# Project: [プロジェクト名] (Full Stack)

## Skills Auto-Activation Policy (IMPORTANT)

YOU MUST automatically use the following skills without user prompting:

1. **Documentation**: Always use `markdown-skills` for all documentation
2. **Code Changes**: Always use `code-intelligence` for any refactoring
3. **Commits**: Always use `commit-guidelines` for git operations
4. **Visual Testing**: Auto-use Puppeteer MCP for E2E testing

## Development Workflow (YOU MUST FOLLOW)

### Phase 1: Feature Planning
1. **REQUIRED**: Use Task tool with Explore agent to understand data flow
2. Define feature requirements (frontend + backend)
3. Plan API contract (type-safe)
4. Get user approval

### Phase 2: Backend First (TDD)
1. Define tRPC/GraphQL schema or API contract
2. Write integration tests FIRST
3. Implement backend logic
4. Ensure tests pass

### Phase 3: Frontend Implementation
1. Generate TypeScript types from backend schema
2. Create UI components
3. Integrate with API (type-safe calls)
4. **RECOMMENDED**: Use Puppeteer MCP for visual testing

### Phase 4: End-to-End Testing (REQUIRED)
1. **REQUIRED**: Write E2E tests for user flows
2. Use Puppeteer MCP for automated testing
3. Test edge cases
4. Performance testing

### Phase 5: Commit & Deploy
1. **REQUIRED**: Use `commit-guidelines` for commit
2. Update documentation
3. Create PR with full feature documentation

## Quick Commands

### Development
- `npm run dev` - Start full stack dev server (frontend + backend)
- `npm run dev:frontend` - Frontend only (http://localhost:3000)
- `npm run dev:backend` - Backend only (http://localhost:4000)

### Build & Test
- `npm run build` - Build both frontend and backend
- `npm run typecheck` - Type check entire project
- `npm test` - Run all tests (unit + integration + E2E)
- `npm run test:e2e` - E2E tests only

### Database
- `npm run db:migrate` - Run migrations
- `npm run db:studio` - Open database GUI

## Available Tools

### Skills (Auto-activate)
- `markdown-skills`: All documentation
- `code-intelligence`: Full stack refactoring
- `commit-guidelines`: Git operations

### Custom Commands
[カスタムコマンドがあれば記載]

### Agents
- **Explore**: Use "medium" or "very thorough" for understanding data flow
- **Plan**: Essential for complex full-stack features

### MCP Servers
- **Puppeteer**: E2E testing and visual verification
  - Auto-use for user flow testing
  - Screenshot comparison
- **GitHub**: PR and issue management

## Code Style (IMPORTANT)

### Type Safety (CRITICAL)
- **REQUIRED**: End-to-end type safety
- Frontend and backend share types
- No `any` types without explicit justification

### Frontend
- Framework: [Next.js / React / Vue]
- Styling: [Tailwind / CSS Modules]
- State: [Jotai / Redux / Context]
- API Client: [tRPC / GraphQL / REST with generated types]

### Backend
- Framework: [Next.js API Routes / Express / Fastify]
- Database: [Prisma / TypeORM]
- API: [tRPC / GraphQL / OpenAPI]

### Shared
```typescript
// packages/types/ or shared/types/
export type User = {
  id: string
  email: string
  name: string
}

// Frontend uses same types
// Backend uses same types
// NO duplication
```

## Architecture

```
[プロジェクト構造]
/
├── apps/
│   ├── web/              (Frontend - Next.js)
│   │   ├── src/
│   │   │   ├── app/      (Pages)
│   │   │   ├── components/
│   │   │   └── lib/
│   │   └── package.json
│   └── api/              (Backend - if separate)
│       ├── src/
│       │   ├── routes/
│       │   ├── services/
│       │   └── models/
│       └── package.json
├── packages/
│   ├── types/            (Shared TypeScript types)
│   ├── ui/               (Shared components)
│   └── config/           (Shared configs)
└── package.json          (Root)
```

Key patterns:
- Type sharing between frontend/backend
- API layer: [tRPC for type safety / GraphQL for flexibility]
- Authentication: [NextAuth.js / Auth0 / Custom JWT]

## Testing

### Unit Tests
- Frontend: React Testing Library + Vitest
- Backend: Integration tests with test database
- Location: `__tests__/` alongside source

### E2E Tests (REQUIRED)
- Framework: Playwright + Puppeteer MCP
- Location: `e2e/` directory
- **REQUIRED**: Test critical user flows
- Run before production deployment

### Test Strategy
1. Backend: Integration tests for all endpoints
2. Frontend: Component tests for UI logic
3. E2E: Critical user journeys

## Git Workflow

- Branch: `feature/feature-name` or `fix/issue-description`
- Commits: Conventional Commits (via commit-guidelines skill)
  - feat: New feature (full stack)
  - fix: Bug fix
  - refactor: Refactoring
  - test: Test additions
- **IMPORTANT**: Rebase, not merge
- PR must include: Screenshots, test coverage, migration notes

## Environment Setup

### Prerequisites
- Node.js v18+
- [Database]: PostgreSQL 14+ / MySQL 8+
- Redis (if using)

### Setup
1. Clone repository
2. Install dependencies: `npm install` (root, installs all workspaces)
3. Setup database: `npm run db:setup`
4. Copy `.env.example` to `.env`
5. Run migrations: `npm run db:migrate`
6. Start dev server: `npm run dev`

### Environment Variables
Required:
- `DATABASE_URL` - Database connection
- `NEXTAUTH_SECRET` - Auth secret
- `NEXTAUTH_URL` - Auth callback URL
- [その他の環境変数]

## Known Issues

- Type generation: Run `npm run generate` after schema changes
- [その他の既知の問題]
```

---

## 5. モノレポ

複数アプリケーション/パッケージを含むモノレポ向け。

### 5.1 ルート CLAUDE.md

```markdown
# Monorepo: [プロジェクト名]

## Skills Auto-Activation Policy (IMPORTANT)

YOU MUST automatically use the following skills without user prompting:

1. **Documentation**: Always use `markdown-skills` for all .md files
2. **Code Analysis**: Always use `code-intelligence` for refactoring
3. **Commits**: Always use `commit-guidelines` for git operations

These policies apply to ALL modules unless overridden by module-specific CLAUDE.md.

## Repository Structure

```
/
├── apps/
│   ├── web/          (Next.js frontend - see apps/web/CLAUDE.md)
│   ├── mobile/       (React Native - see apps/mobile/CLAUDE.md)
│   ├── admin/        (Admin dashboard - see apps/admin/CLAUDE.md)
│   └── api/          (Fastify backend - see apps/api/CLAUDE.md)
├── packages/
│   ├── ui/           (Shared React components)
│   ├── types/        (Shared TypeScript types)
│   ├── config/       (Shared configs: ESLint, TypeScript)
│   └── database/     (Prisma schema and migrations)
└── tools/
    └── scripts/      (Build and deployment scripts)
```

## Development Workflow (YOU MUST FOLLOW)

### When Working on Specific Module:
1. **REQUIRED**: Read module-specific CLAUDE.md first
2. Module guidelines take precedence over this file
3. Use Task tool with Explore agent (thoroughness: "medium") to understand module

### When Working Across Modules:
1. **REQUIRED**: Use `code-intelligence` to understand dependencies
2. Plan changes to avoid breaking other modules
3. Run affected tests: `npm test --scope=<module>`

## Quick Commands

### Development
- `npm run dev` - Start all dev servers
- `npm run dev --scope=web` - Start specific app
- `npm run build` - Build all packages and apps
- `npm run build --scope=api` - Build specific package

### Testing
- `npm test` - Run all tests
- `npm run test --scope=<module>` - Test specific module
- `npm run test:affected` - Test affected by changes

### Database
- `npm run db:migrate` - Run migrations
- `npm run db:studio` - Open Prisma Studio

## Available Tools

### Skills (Auto-activate across all modules)
- `markdown-skills`: Documentation
- `code-intelligence`: Cross-module refactoring
- `commit-guidelines`: Git operations

### Agents
- **Explore**: Use "very thorough" for cross-module understanding
- **Plan**: Essential for changes affecting multiple modules

## Common Workflows

### Adding New Package
1. Create package directory: `packages/<name>`
2. Add package.json with proper dependencies
3. Add to workspace in root package.json
4. Create package-specific CLAUDE.md

### Cross-Module Changes
1. **REQUIRED**: Use Task tool with Explore agent
2. Identify all affected modules
3. Update each module
4. Run `npm run test:affected`
5. Single commit for atomic change

## Shared Conventions

### Code Style
- TypeScript strict mode enabled
- ESLint config: `packages/config/eslint`
- Prettier config: `packages/config/prettier`

### Imports
- Use workspace protocol: `"@repo/ui": "workspace:*"`
- Absolute imports within each package

### Testing
- Unit tests: Alongside source files
- Integration tests: `__tests__/` directory
- E2E tests: `apps/*/e2e/`

## Git Workflow

- Branch: `feature/<module>/<description>` or `fix/<module>/<description>`
- Commits: Conventional Commits with scope
  - Format: `type(scope): description`
  - Scope: Module name (e.g., `feat(web): add login page`)
- **IMPORTANT**: Rebase, not merge
- PR: Must pass all affected tests

## Module-Specific Guidelines

Each app/package has its own CLAUDE.md with detailed guidelines:

### Apps
- `apps/web/CLAUDE.md` - Next.js frontend guidelines
- `apps/mobile/CLAUDE.md` - React Native guidelines
- `apps/admin/CLAUDE.md` - Admin dashboard guidelines
- `apps/api/CLAUDE.md` - Backend API guidelines

### Packages
- `packages/ui/CLAUDE.md` - Component library guidelines
- `packages/database/CLAUDE.md` - Database and migration guidelines

**When working in a specific module, BOTH this file and the module-specific CLAUDE.md apply.**
**Module-specific guidelines take precedence in case of conflicts.**

## Environment Setup

### Prerequisites
- Node.js v18+
- pnpm v8+ (workspace manager)

### Setup
1. Clone repository
2. Install dependencies: `pnpm install` (installs all workspaces)
3. Setup database: `pnpm db:setup`
4. Copy `.env.example` to `.env` in each app
5. Run migrations: `pnpm db:migrate`
6. Start all dev servers: `pnpm dev`

## Known Issues

- Turborepo cache: Run `pnpm clean` if builds are stale
- [その他の既知の問題]
```

### 5.2 モジュール固有 CLAUDE.md 例

```markdown
# Module: Web App (Frontend)

## Module-Specific Skills

Additional to root CLAUDE.md policies:
- **ALWAYS use** Puppeteer MCP for visual testing
- **RECOMMENDED**: Use Storybook for component development

## Inherits From

This module inherits from `/CLAUDE.md` (root).
Module-specific guidelines below override root guidelines when conflicts occur.

## Technology Stack

- Framework: Next.js 14 (App Router)
- Styling: Tailwind CSS
- State: Jotai
- API: tRPC (type-safe calls to `@repo/api`)

## Code Style (Module-Specific)

[Web app固有のスタイルガイド]

## Architecture

[Web app固有のアーキテクチャ]

## Testing

[Web app固有のテスト戦略]

For general guidelines, see `/CLAUDE.md`
```

---

## 6. ライブラリ/パッケージ

公開ライブラリやnpmパッケージ向け。

```markdown
# Package: [パッケージ名]

## Skills Auto-Activation Policy (IMPORTANT)

YOU MUST automatically use the following skills without user prompting:

1. **Documentation**: Always use `markdown-skills` for README and API docs
2. **Refactoring**: Always use `code-intelligence` for API changes
3. **Commits**: Always use `commit-guidelines` (conventional commits REQUIRED)

## Development Workflow (YOU MUST FOLLOW)

### Phase 1: API Design
1. Design public API carefully (breaking changes are costly)
2. **REQUIRED**: Document API before implementation
3. Get feedback on API design

### Phase 2: Test-Driven Development (REQUIRED)
1. Write tests FIRST for public API
2. Include usage examples in tests
3. Achieve >90% coverage

### Phase 3: Implementation
1. Implement functionality
2. Keep implementation private (don't expose internals)
3. Run tests continuously

### Phase 4: Documentation (REQUIRED)
1. **REQUIRED**: Update README with examples
2. **REQUIRED**: Update API documentation
3. **REQUIRED**: Update CHANGELOG

### Phase 5: Versioning & Release
1. **REQUIRED**: Follow semantic versioning strictly
2. **REQUIRED**: Use `commit-guidelines` for version bump commits
3. Review breaking changes checklist

## Quick Commands

- `npm run dev` - Build in watch mode
- `npm run build` - Build for production
- `npm test` - Run tests with coverage
- `npm run docs` - Generate API documentation
- `npm run lint` - Run linter
- `npm run release` - Create release (bump version, tag, publish)

## Available Tools

### Skills (Auto-activate)
- `markdown-skills`: README and documentation
- `code-intelligence`: API refactoring
- `commit-guidelines`: Conventional commits (REQUIRED for releases)

## Code Style (IMPORTANT)

### Public API Design
- **CRITICAL**: Backward compatibility
- Avoid breaking changes in minor/patch versions
- Deprecate before removing
- Clear error messages

### Code Organization
```
src/
├── index.ts          (Main entry point - public API)
├── core/             (Core functionality)
├── utils/            (Internal utilities)
└── types/            (TypeScript type definitions)
```

### Exports
- Only export public API from `index.ts`
- Keep implementation details private
- Document all public exports

### TypeScript
- Strict mode enabled
- Export types for library users
- No `any` in public API

## Documentation (CRITICAL)

### README.md (REQUIRED)
Must include:
- Clear description
- Installation instructions
- Quick start example
- API overview
- Links to full documentation

### API Documentation
- Document all public functions/classes
- Include usage examples
- Document edge cases
- Update on every API change

### CHANGELOG.md (REQUIRED)
- Follow Keep a Changelog format
- Group by version
- Categorize: Added, Changed, Deprecated, Removed, Fixed, Security

## Testing

### Coverage (REQUIRED)
- Minimum: >90% coverage
- Test all public API
- Include edge cases
- Test error handling

### Test Types
- Unit tests: All functions
- Integration tests: Public API workflows
- Type tests: TypeScript type correctness

## Versioning & Releases

### Semantic Versioning (REQUIRED)
- MAJOR: Breaking changes
- MINOR: New features (backward compatible)
- PATCH: Bug fixes (backward compatible)

### Breaking Changes Checklist
Before releasing breaking change:
- [ ] Update MAJOR version
- [ ] Document migration guide
- [ ] Deprecate old API first if possible
- [ ] Announce in advance

### Release Process
1. Run all tests: `npm test`
2. Update CHANGELOG.md
3. Bump version: `npm version [major|minor|patch]`
4. Build: `npm run build`
5. Publish: `npm publish`
6. Create GitHub release with notes

## Git Workflow

- Branch: `feature/<description>` or `fix/<description>`
- Commits: Conventional Commits (REQUIRED)
  - feat: New feature
  - fix: Bug fix
  - docs: Documentation only
  - refactor: Code refactoring
  - test: Test additions
  - chore: Maintenance
- **CRITICAL**: Commit messages drive CHANGELOG generation
- **IMPORTANT**: Rebase, not merge

## Environment Setup

### Prerequisites
- Node.js v18+
- npm or pnpm

### Setup
1. Clone repository
2. Install dependencies: `npm install`
3. Build: `npm run build`
4. Run tests: `npm test`

### Publishing (Maintainers Only)
- Requires npm authentication
- Follow release process above

## Known Issues

[既知の問題]
```

---

## 使用方法

### テンプレートの選択

プロジェクトタイプに応じて適切なテンプレートを選択:

1. **汎用**: 小規模プロジェクトまたは不明な場合
2. **フロントエンド**: React/Vue/Next.js等のUIプロジェクト
3. **バックエンド**: API/サーバーサイドプロジェクト
4. **フルスタック**: フロントエンド + バックエンド統合
5. **モノレポ**: 複数アプリ/パッケージ
6. **ライブラリ**: 公開パッケージやライブラリ

### カスタマイズ手順

1. 該当テンプレートをコピー
2. `[...]`部分を実際の情報で置換
3. 不要なセクションを削除
4. プロジェクト固有の情報を追加
5. チームでレビュー
6. Git にコミット

### テンプレート取得コマンド

Claude Code内で:
```
claude-md-optimizerスキルを使って、[プロジェクトタイプ]用の
CLAUDE.mdを生成してください。
```

---

バージョン: 1.0.0
最終更新: 2025-11-15
