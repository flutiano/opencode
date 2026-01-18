# OpenCode Architecture Overview

OpenCode is an open-source AI coding agent with a modular, client/server architecture. This document provides a high-level overview of the major modules and their relationships.

## Core Architecture

### 1. `packages/opencode` - Core CLI & Agent Engine

The heart of OpenCode - a provider-agnostic AI coding agent.

**Tech Stack**: Bun + TypeScript

**Key Components**:

- `src/agent/` - Agent definitions and prompt templates (build/plan agents)
- `src/tool/` - Capabilities: bash, read, write, grep, lsp, webfetch, etc.
- `src/session/` - Conversation state and LLM processing
- `src/cli/` - Command-line interface with TUI (SolidJS + OpenTUI)
- `src/server/` - Hono-based server for client/server architecture
- `src/mcp/` & `src/acp/` - Model Context Protocol & Agent Client Protocol support
- `src/provider/` - Multi-provider LLM support (OpenAI, Anthropic, Google, etc.)
- `src/lsp/` - Language Server Protocol integration

**Key Features**:

- Orchestrates AI Agents (build with edit-access, plan with read-only)
- Executes code operations safely (bash commands, file edits, codebase searches)
- Provides rich Terminal UI for chatting with the agent
- Manages MCP/ACP connections to external tools
- Serves as an API backend for multiple frontends

---

### 2. `packages/app` - Frontend Application

The primary graphical client for OpenCode.

**Tech Stack**: SolidJS + Vite

**Key Features**:

- Session-based AI chat interface
- Terminal emulation component
- File tree navigation
- Multi-agent switching UI
- SDK context for backend communication

**Entry Point**: `src/entry.tsx`

---

### 3. `packages/ui` - UI Component Library

Shared SolidJS UI kit used across all OpenCode clients.

**Key Features**:

- Interactive code diffs (unified/split view)
- AI-ready messaging components
- 15+ built-in developer themes (Dracula, Nord, Tokyo Night, etc.)
- Rich iconography (file types, AI providers)
- Markdown rendering with syntax highlighting (Shiki)
- Audio assets for UI interactions

**Key Directories**:

- `src/components/` - Core UI components and domain-specific components
- `src/theme/` - Multi-theme management system
- `src/assets/` - Fonts, icons, and audio assets
- `src/context/` - Shared state and providers

---

### 4. `packages/sdk` - Client/Server SDK

Type-safe JavaScript/TypeScript SDK for OpenCode communication.

**Key Components**:

- Auto-generated from `openapi.json` specification
- V1 and V2 versions available
- `openapi.json` - Source of truth for API definition
- Build script generates typed clients from spec

**Entry Point**: `packages/sdk/js/src/index.ts`

**Build Command**: `./packages/sdk/js/script/build.ts`

---

## Platform & Management

### 5. `packages/console` - Management Platform

Full-stack platform for user/workspace management and billing.

**Tech Stack**: SolidStart (Solid.js), Drizzle ORM, Postgres/PlanetScale

**Sub-modules**:

#### `packages/console/app`

- SolidStart frontend and **Zen API gateway**
- OpenAI-compatible proxy to various LLM providers
- Authentication flows using `@openauthjs/openauth`
- Stripe webhooks and payment processing
- Key routes: `/workspace`, `/zen`, `/auth`, `/stripe`

#### `packages/console/core`

- Drizzle ORM database layer
- Database schemas: `user`, `workspace`, `billing`, `key`, `model`
- Business logic for core entities
- Database migrations

#### `packages/console/mail`

- Transactional email templates using `@jsx-email/render`
- Workspace invitation emails

#### `packages/console/function`

- Serverless utility functions
- Authentication and logging functions
- Deployed as AWS Lambda or Cloudflare Workers

#### `packages/console/resource`

- Infrastructure and environment definitions
- Cloudflare and Node environment support

**Key Features**:

- User & Workspace management
- Monetization (subscriptions, credits, billing via Stripe)
- API Gateway (Zen) - abstracts multiple AI providers into unified interface
- Identity provider for session/token management

---

### 6. `packages/desktop` - Native Desktop App

Cross-platform desktop application using Tauri v2.

**Tech Stack**: Tauri v2 (Rust) + SolidJS frontend

**Key Directories**:

- `src-tauri/` - Rust-based native backend
- `src/` - Frontend UI layer (SolidJS)
- `scripts/` - Build and preparation scripts

**Key Features**:

- Native OS integration (window management, system notifications, file system access)
- CLI installation directly from GUI
- Auto-update support
- Renders web-based frontend inside native shell

**Platforms**: macOS, Windows, Linux

---

### 7. `packages/web` - Website & Documentation

Astro-based marketing site and documentation portal.

**Tech Stack**: Astro + Starlight theme + SolidJS components

**Key Features**:

- Documentation hub for OpenCode usage
- Landing page (`Lander.astro`)
- Session/content sharing via unique URLs
- Deployment configured for Cloudflare

**Key Directories**:

- `src/content/docs/` - Technical documentation (.mdx files)
- `src/pages/` - Dynamic routing for docs and shared content
- `src/components/` - Astro and SolidJS components
- `src/assets/` - Branding and screenshots

---

## Supporting Modules

### 8. `packages/slack` - Slack Integration

Bot integration for OpenCode Slack workflow.

---

### 9. `packages/enterprise` - Enterprise Features

BETA enterprise functionality and features.

---

### 10. `packages/plugin` - Plugin System

Extensibility framework for adding custom functionality.

---

### 11. `packages/function` - Serverless Functions

Utility functions for cloud deployment scenarios.

---

### 12. `packages/util` - Shared Utilities

Common utility functions used across multiple packages.

---

### 13. `packages/script` - Build Scripts

Build and automation scripts for the monorepo.

---

### 14. `packages/extensions` - IDE Extensions

Editor integrations, such as Zed extension.

**Key Directory**: `packages/extensions/zed/`

---

### 15. `packages/docs` - Documentation Source

Markdown/MDX documentation files for the project.

**Key Files**:

- Documentation in `.mdx` format
- Links to OpenAPI spec (`openapi.json` -> `../sdk/openapi.json`)

---

## Architecture Summary

OpenCode follows a **client/server architecture**:

1. **Core Engine** (`packages/opencode`) - Provides the core AI agent engine and can run as a local server
2. **Multiple Clients** connect to the core engine:
   - TUI (terminal interface)
   - Desktop app (Tauri)
   - Web app (SolidJS)
3. **Management Layer** (`packages/console`) provides:
   - Authentication and user management
   - Billing and subscriptions
   - **Zen API gateway** that abstracts multiple LLM providers
4. **SDK** (`packages/sdk`) enables type-safe communication between all components
5. **UI Library** (`packages/ui`) provides shared components across all clients
6. **Website** (`packages/web`) serves as documentation and marketing hub

## Technology Stack

| Layer       | Technology                                                |
| ----------- | --------------------------------------------------------- |
| Runtime     | Bun                                                       |
| Language    | TypeScript                                                |
| Frontend    | SolidJS, Vite                                             |
| Backend API | Hono, SST (Cloudflare)                                    |
| Database    | Postgres/PlanetScale, Drizzle ORM                         |
| Desktop     | Tauri v2, Rust                                            |
| Styling     | Tailwind CSS                                              |
| LSP         | vscode-languageserver-types, web-tree-sitter              |
| AI SDK      | Vercel AI SDK (multiple providers)                        |
| Protocol    | MCP (Model Context Protocol), ACP (Agent Client Protocol) |

## Development Workflow

- **Install**: `bun install`
- **Run OpenCode**: `bun dev` (from root, runs `packages/opencode`)
- **Typecheck**: `bun turbo typecheck`
- **Build**: Per-package build commands
- **Regenerate SDK**: `./packages/sdk/js/script/build.ts`

## Key Design Principles

- **Provider Agnostic**: Works with Claude, OpenAI, Google, and local models
- **Client/Server**: Engine can run separately from clients, enabling remote usage
- **Modular**: Each package has a single responsibility
- **Extensible**: Plugin system and MCP/ACP support for integrations
- **Type-Safe**: Heavy use of TypeScript and auto-generated SDKs
- **Performance**: Built on Bun for fast execution
- **TUI-First**: Emphasis on terminal-based workflows

---

_Last Updated: 2026-01-18_
