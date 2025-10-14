# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Astro Starlight documentation site for the "Fixing Your Frontend: Performance Monitoring Best Practices" workshop. The site teaches developers how to implement comprehensive performance and error monitoring in e-commerce applications using Sentry, with a focus on preparing for high-traffic events like Black Friday.

## Development Commands

- `pnpm install` - Install dependencies (pnpm is the required package manager)
- `pnpm dev` or `pnpm start` - Start development server at localhost:4321
- `pnpm build` - Build for production (outputs to ./dist/)
- `pnpm preview` - Preview production build locally
- `pnpm format` - Format code with Prettier
- `pnpm format:check` - Check code formatting
- `pnpm lint` - Lint code
- `pnpm lint:fix` - Auto-fix linting issues

## Architecture

**Framework**: Astro 5.11+ with Starlight integration for documentation theming
**Content**: MDX files in `/src/content/docs/` with workshop modules
**Deployment**: Vercel adapter with image service and asset handling
**Package Manager**: pnpm 10.18+ (enforced via packageManager field)

## Workshop Content Structure

The workshop is organized into 5 main modules focused on frontend performance monitoring:

1. **Setting Up Performance & Error Monitoring** (`getting-started.mdx`)
   - Sentry SDK setup for React frontend and Node.js backend
   - Performance monitoring with Web Vitals tracking
   - Session Replay and error monitoring configuration
   - Basic alert rules setup

2. **Optimizing Performance with Web Vitals** (`optimizing-web-vitals.mdx`)
   - Identifying performance bottlenecks using LCP, TTFB, INP
   - Custom performance spans for business operations
   - Database query optimization (indexing)
   - Using Sentry's Seer AI for performance suggestions

3. **Distributed Tracing & Backend Performance** (`distributed-tracing.mdx`)
   - Linking frontend slowness to backend issues
   - Identifying and fixing N+1 query problems
   - Implementing caching strategies
   - Request waterfall analysis

4. **Session Replay & Production Monitoring** (`session-replay-alerts.mdx`)
   - Debugging user experience issues with Session Replay
   - Privacy configuration (masking sensitive data)
   - Advanced alert rules for Web Vitals and business metrics
   - Creating monitoring dashboards

5. **Production Readiness & Code Quality** (`production-readiness.mdx`)
   - Advanced alerting strategies
   - Audience-specific dashboards
   - Sentry Prevent AI for code review
   - On-call rotation and incident response

## Technology Focus

The workshop covers:

- **Frontend**: Vite + React + TypeScript with Sentry React SDK
- **Backend**: Node.js + Express with Sentry Node SDK
- **Database**: PostgreSQL with Drizzle ORM
- **Target Application**: E-commerce store (`sergical/unborked` repo)
- **Focus Areas**: Web Vitals, distributed tracing, Session Replay, performance optimization
- **AI Tools**: Sentry Seer for RCA and fix suggestions, Prevent AI for code review

## Key Configuration Files

- `astro.config.mjs` - Starlight theme with workshop navigation and Vercel adapter
- `src/content.config.ts` - Content collections using Astro's docsLoader and docsSchema
- `sentry.client.config.js` / `sentry.server.config.js` - Sentry configuration for the docs site
- Custom CSS in `/src/styles/custom.css` for Sentry-specific styling

## Content Guidelines

- All workshop content is in MDX format in `/src/content/docs/`
- Supports Starlight components: Steps, Card, CardGrid, and callout blocks (:::tip, :::note, :::caution)
- Code examples target the `unborked` e-commerce application
- Performance-focused scenarios (holiday traffic, high load)
- Assumes participants are new to performance monitoring
- Images stored in `/src/assets/img/` for screenshots and diagrams
