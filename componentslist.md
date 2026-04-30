# RNT Flow Frontend - Component Specification

## Overview

This document catalogs all React components in the RNT Flow frontend (`src/`), organized by feature module. Each entry includes the file path, component name, purpose, and parent page/layout.

---

## Table of Contents

1. [App Root & Entry](#1-app-root--entry)
2. [App Shell (Layout)](#2-app-shell-layout)
3. [Authentication](#3-authentication)
4. [Dashboard](#4-dashboard)
5. [Strategies](#5-strategies)
6. [Deployments](#6-deployments)
7. [Backtest](#7-backtest)
8. [Instruments](#8-instruments)
9. [Risk](#9-risk)
10. [Settings](#10-settings)
11. [Audit](#11-audit)
12. [Admin](#12-admin)
13. [Trading](#13-trading)
14. [Strategic Builder](#14-strategic-builder)
15. [Canvas (ReactFlow Nodes)](#15-canvas-reactflow-nodes)
16. [Layout Components](#16-layout-components)
17. [Rail Components](#17-rail-components)
18. [UI Primitives](#18-ui-primitives)
19. [Hooks](#19-hooks)

---

## 1. App Root & Entry

### `src/main.tsx`
- **Purpose**: Application bootstrap - mounts React app, initializes MSW (Mock Service Worker) for API mocking in development
- **Page**: N/A (entry point)

### `src/App.tsx`
- **Component**: `App` (default export)
- **Purpose**: Root component defining all React Router routes. Registers nested routes under `AdminShell` for `/admin/*` paths. Handles route guards.
- **Page**: N/A (router root)

### `src/app/error-boundary.tsx`
- **Component**: `ErrorBoundary`
- **Purpose**: Class component that catches React subtree errors, displays a friendly error UI with retry button
- **Page**: Applied globally around the app

---

## 2. App Shell (Layout)

### `src/app/layout.tsx`
- **Component**: `Layout` (default export)
- **Purpose**: Main authenticated app shell - renders `<Header>`, `<Sidebar>`, and `<Outlet>` for page content
- **Page**: Root layout wrapping all authenticated routes (`/`)

### `src/app/header.tsx`

#### `Header`
- **Purpose**: Top navigation bar containing breadcrumbs, notification bell, connection status indicator, and user menu
- **Page**: Layout (renders in every authenticated page)

#### `Breadcrumbs` (sub-component)
- **Purpose**: Dynamic breadcrumb trail derived from current route path using `ROUTE_LABELS` mapping
- **Page**: Header

#### `ConnectionStatus` (sub-component)
- **Purpose**: Live WebSocket connection indicator (green dot = connected, red = disconnected)
- **Page**: Header

#### `NotificationsBell` (sub-component)
- **Purpose**: Bell icon with unread notification count badge, opens dropdown list of recent notifications
- **Page**: Header

#### `UserMenu` (sub-component)
- **Purpose**: Avatar dropdown with links to Profile (`/profile`), Place Order (`/orders/place`), Settings, and Logout
- **Page**: Header

### `src/app/sidebar.tsx`
- **Component**: `Sidebar`
- **Purpose**: Collapsible left navigation sidebar with icon+label nav items. NAV_ITEMS includes: Dashboard, Strategies, Deployments, Backtest, Instruments, Risk, Audit, Settings, Admin (ORG_ADMIN only), Place Order, Profile
- **Page**: Layout

### `src/app/providers.tsx`

#### `Providers` (default export)
- **Purpose**: Wrapper component composing React Query client, `AuthGuard`, `ThemeApplier`, and `ApiClientSetup`
- **Page**: N/A (wraps app)

#### `ThemeApplier` (sub-component)
- **Purpose**: Applies `dark` or `light` class to `<html>` element based on user preference
- **Page**: Providers

#### `AuthGuard` (sub-component)
- **Purpose**: HOC that checks authentication state, redirects to `/login` if unauthenticated
- **Page**: Providers

#### `ApiClientSetup` (sub-component)
- **Purpose**: Configures Axios API client with `Authorization: Bearer` token and `X-Workspace-ID` header from auth state
- **Page**: Providers

---

## 3. Authentication

### `src/features/auth/LoginPage.tsx`
- **Component**: `LoginPage` (default export)
- **Purpose**: Full-page login screen. Step 1: email + password form. Step 2: MFA/TOTP 6-digit code. Handles Goodwill OAuth redirect flow (receives `access_token` from `https://api.gwcindia.in/login-response`)
- **Page**: `/login` (standalone, no layout)

### `src/features/auth/components/LoginForm.tsx`
- **Component**: `LoginForm`
- **Purpose**: Email/password form with react-hook-form + zod validation. Submits to identity service. On success, stores JWT in memory/Zustand.
- **Page**: LoginPage

### `src/features/auth/components/MfaStep.tsx`
- **Component**: `MfaStep`
- **Purpose**: 6-digit TOTP input with auto-submit on complete, paste support for authenticator app codes
- **Page**: LoginPage (shown after password step succeeds)

### `src/features/auth/components/StepUpModal.tsx`
- **Component**: `StepUpModal`
- **Purpose**: Radix UI Dialog modal for step-up (MFA) authentication confirmation. Used before sensitive operations (kill switch, broker connect/disconnect)
- **Page**: Used by KillSwitchPanel, BrokerAccountsTab, and other sensitive action confirmations

---

## 4. Dashboard

### `src/features/dashboard/DashboardPage.tsx`
- **Component**: `DashboardPage` (default export)
- **Purpose**: Main landing page after login. Shows summary cards (active deployments P&L, daily P&L, open positions, margin used), active deployments list, market status bar
- **Page**: `/` (root route)

#### `SummaryCard` (sub-component)
- **Purpose**: Reusable metric card displaying icon, large value text, label, and optional subtext/trend indicator
- **Page**: DashboardPage

---

## 5. Strategies

### `src/features/strategies/StrategiesPage.tsx`
- **Component**: `StrategiesPage` (default export)
- **Purpose**: Paginated strategy list with search input, status filter dropdown, category filter, and sort options. Shows total strategies count.
- **Page**: `/strategies`

### `src/features/strategies/components/StrategyCard.tsx`
- **Component**: `StrategyCard`
- **Purpose**: Card showing strategy name, status badge, category tag, last modified date, P&L (if deployed), and overflow menu (Edit, Version History, Settings, Archive)
- **Page**: StrategiesPage

### `src/features/strategies/components/StrategyFilters.tsx`
- **Component**: `StrategyFilters`
- **Purpose**: Search text input + status dropdown (All/Draft/Validated/Active/Paused/Archived) + category multiselect
- **Page**: StrategiesPage

### `src/features/strategies/TemplateBrowserPage.tsx`
- **Component**: `TemplateBrowserPage` (default export)
- **Purpose**: Browse predefined strategy templates grouped by category (Momentum, Mean Reversion, Breakout, Options). Click to preview and create strategy from template.
- **Page**: `/strategies/templates`

### `src/features/strategies/ImportPage.tsx`
- **Component**: `ImportPage` (default export)
- **Purpose**: Multi-step wizard: (1) Upload JSON file, (2) Validate schema against Strategy AST spec, (3) Policy acknowledgment, (4) Confirm and import
- **Page**: `/strategies/import`

### `src/features/strategies/VersionHistoryPage.tsx`
- **Component**: `VersionHistoryPage` (default export)
- **Purpose**: Timeline view of strategy versions with version number, author, timestamp, change summary. Allows restore previous version.
- **Page**: `/strategies/:id/versions`

### `src/features/strategies/StrategySettingsPage.tsx`
- **Component**: `StrategySettingsPage` (default export)
- **Purpose**: Edit strategy name, description, tags. Archive/delete strategy with confirmation dialog.
- **Page**: `/strategies/:id/settings`

---

## 6. Deployments

### `src/features/deployments/DeploymentsPage.tsx`
- **Component**: `DeploymentsPage` (default export)
- **Purpose**: Paginated list of deployments with status filter (All/Active/Paused/Stopped/Failed), search, market status bar
- **Page**: `/deployments`

### `src/features/deployments/DeploymentDetailPage.tsx`
- **Component**: `DeploymentDetailPage` (default export)
- **Purpose**: Tabbed deployment detail view. Tabs: Overview (status, strategy info, P&L summary), Positions, Orders, Performance. Includes kill switch panel and deployment status badge.
- **Page**: `/deployments/:id`

#### `StatusBadge` (sub-component)
- **Purpose**: Colored pill showing deployment state (QUEUED/DEPLOYING/ACTIVE/PAUSED/STOPPING/STOPPED/FAILED) with icon
- **Page**: DeploymentDetailPage, DeploymentCard

### `src/features/deployments/components/DeploymentCard.tsx`
- **Component**: `DeploymentCard`
- **Purpose**: Summary card for a deployment showing strategy name, status badge, P&L, position count, deployment date, quick actions
- **Page**: DeploymentsPage, DashboardPage (active deployments widget)

### `src/features/deployments/components/PositionTable.tsx`
- **Component**: `PositionTable`
- **Purpose**: Sortable table of open positions: symbol, quantity, avg price, LTP, unrealized P&L, day P&L, exposure
- **Page**: DeploymentDetailPage (Positions tab)

### `src/features/deployments/components/OrderTable.tsx`
- **Component**: `OrderTable`
- **Purpose**: Sortable table of orders: order ID, timestamp, symbol, type, direction, qty, price, status, filled qty
- **Page**: DeploymentDetailPage (Orders tab)

### `src/features/deployments/components/PnlChart.tsx`
- **Component**: `PnlChart`
- **Purpose**: TradingView Lightweight Charts line series showing cumulative P&L over the deployment lifetime with trade entry/exit markers
- **Page**: DeploymentDetailPage (Performance tab)

### `src/features/deployments/components/MarketStatusBar.tsx`
- **Component**: `MarketStatusBar`
- **Purpose**: Horizontal bar showing NSE/BSE open/closed/pre-open status with colored indicators and current IST time
- **Page**: DeploymentsPage, DashboardPage, DeploymentDetailPage

### `src/features/deployments/components/KillSwitchPanel.tsx`
- **Component**: `KillSwitchPanel`
- **Purpose**: Three-scope kill switch toggles (Global, Workspace, Deployment) with step-up MFA confirmation. Shows last triggered timestamp.
- **Page**: DeploymentDetailPage

#### `KillSwitchToggle` (sub-component)
- **Purpose**: Individual kill switch toggle row with scope label, active/inactive styling, and trigger timestamp
- **Page**: KillSwitchPanel

---

## 7. Backtest

### `src/features/backtest/BacktestResultsPage.tsx`
- **Component**: `BacktestResultsPage` (default export)
- **Purpose**: Full backtest results display with equity curve, drawdown chart, metrics table, and trade log. Accessed after backtest simulation completes.
- **Page**: `/backtest/:id`

### `src/features/backtest/components/BacktestSummary.tsx`
- **Component**: `BacktestSummary`
- **Purpose**: 4-card grid: Total P&L (green/red), Win Rate %, Max Drawdown %, Sharpe Ratio
- **Page**: BacktestResultsPage

#### `SummaryCard` (sub-component)
- **Purpose**: Individual metric card with icon, value, label, and subvalue (e.g., "+12.5% vs benchmark")
- **Page**: BacktestSummary

### `src/features/backtest/components/EquityCurve.tsx`
- **Component**: `EquityCurve`
- **Purpose**: TradingView Lightweight Charts area series showing equity curve with buy/sell trade markers overlaid
- **Page**: BacktestResultsPage

### `src/features/backtest/components/DrawdownChart.tsx`
- **Component**: `DrawdownChart`
- **Purpose**: TradingView Lightweight Charts area (red) showing drawdown percentage over time
- **Page**: BacktestResultsPage

### `src/features/backtest/components/MetricsTable.tsx`
- **Component**: `MetricsTable`
- **Purpose**: Categorized performance metrics table: Returns (CAGR,净, gross), Risk (Max DD, Volatility, VaR), Trade Stats (Total, Winners, Losers, Avg), Efficiency (Sharpe, Sortino, Calmar)
- **Page**: BacktestResultsPage

### `src/features/backtest/components/TradeLogTable.tsx`
- **Component**: `TradeLogTable`
- **Purpose**: Virtualized, sortable, filterable table of individual trades with entry/exit prices, P&L, holding duration. Includes CSV export button.
- **Page**: BacktestResultsPage

---

## 8. Instruments

### `src/features/instruments/InstrumentsPage.tsx`
- **Component**: `InstrumentsPage` (default export)
- **Purpose**: Searchable instrument list with type filter (EQUITY/FUTURES/OPTIONS/INDEX) and exchange filter (NSE/NFO/BSE). Displays instrument cards in a responsive grid.
- **Page**: `/instruments`

### `src/features/instruments/components/InstrumentCard.tsx`
- **Component**: `InstrumentCard` (default export)
- **Purpose**: Expandable card showing instrument name, symbol, exchange badge, type badge, lot size, tick size, expiry (if F&O). Click to expand and show lazy-loaded chart.
- **Page**: InstrumentsPage

#### `ExchangeBadge` (sub-component)
- **Purpose**: Colored badge for exchange (NSE=blue, NFO=green, BSE=orange)
- **Page**: InstrumentCard

#### `TypeBadge` (sub-component)
- **Purpose**: Colored badge for instrument type (EQUITY, FUTURES, OPTIONS, INDEX)
- **Page**: InstrumentCard

### `src/features/instruments/components/InstrumentChart.tsx`
- **Component**: `InstrumentChart` (default export)
- **Purpose**: Candlestick chart (TradingView Lightweight Charts) showing OHLC price history for the instrument. Lazy-loaded when card expands.
- **Page**: InstrumentCard (lazy, on expand)

---

## 9. Risk

### `src/features/risk/RiskPage.tsx`
- **Component**: `RiskPage` (default export)
- **Purpose**: Risk dashboard displaying: risk limit cards (daily loss, capital per trade, max positions), daily loss meter (SVG arc), exposure chart, margin utilization bar, kill switch panel
- **Page**: `/risk`

### `src/features/risk/components/RiskLimitCards.tsx`
- **Component**: `RiskLimitCards`
- **Purpose**: 3-card grid showing configured vs used values for: Daily Loss Limit, Capital Per Trade, Max Positions
- **Page**: RiskPage

#### `LimitCard` (sub-component)
- **Purpose**: Individual limit card showing label, configured limit value, current used value, usage percentage bar
- **Page**: RiskLimitCards

### `src/features/risk/components/DailyLossMeter.tsx`
- **Component**: `DailyLossMeter`
- **Purpose**: SVG arc meter showing today's P&L loss as percentage of daily loss limit (green→yellow→red gradient as limit approaches)
- **Page**: RiskPage

### `src/features/risk/components/ExposureChart.tsx`
- **Component**: `ExposureChart`
- **Purpose**: Horizontal bar chart grouped by instrument showing current exposure vs maximum allowed exposure per instrument
- **Page**: RiskPage

### `src/features/risk/components/MarginBar.tsx`
- **Component**: `MarginBar`
- **Purpose**: Horizontal stacked progress bar showing Margin Used (red) / Margin Available (yellow) / Total Margin
- **Page**: RiskPage

### `src/features/risk/components/KillSwitchPanel.tsx`
- **Component**: `KillSwitchPanel`
- **Purpose**: Three-scope kill switch (Global/Workspace/Deployment) with step-up MFA modal for confirmation
- **Page**: RiskPage

#### `KillSwitchRow` (sub-component)
- **Purpose**: Individual kill switch row with scope label, description, toggle switch, and last triggered info
- **Page**: KillSwitchPanel

---

## 10. Settings

### `src/features/settings/SettingsPage.tsx`
- **Component**: `SettingsPage` (default export)
- **Purpose**: Tabbed settings page with tabs: Profile, Workspace, Broker Accounts, Notifications
- **Page**: `/settings`

### `src/features/settings/components/ProfileTab.tsx`
- **Component**: `ProfileTab` (default export)
- **Purpose**: User profile form: display name, email (read-only), change password form, TOTP setup link (QR code for authenticator app)
- **Page**: SettingsPage (Profile tab)

### `src/features/settings/components/WorkspaceTab.tsx`
- **Component**: `WorkspaceTab` (default export)
- **Purpose**: Workspace settings: workspace name, member list with roles, risk limits configuration (daily loss, capital per trade)
- **Page**: SettingsPage (Workspace tab)

### `src/features/settings/components/BrokerAccountsTab.tsx`
- **Component**: `BrokerAccountsTab` (default export)
- **Purpose**: Broker account list (Goodwill) with connect/disconnect buttons, connection status badges, and API key management
- **Page**: SettingsPage (Broker Accounts tab)

#### `StatusBadge` (sub-component)
- **Purpose**: Colored status indicator (Active=green, Disconnected=gray, Expired=red)
- **Page**: BrokerAccountsTab

---

## 11. Audit

### `src/features/audit/AuditPage.tsx`
- **Component**: `AuditPage` (default export)
- **Purpose**: ORG_ADMIN-only audit log page showing all organization events. Falls back to AccessDenied component for non-admin users.
- **Page**: `/audit`

#### `AccessDenied` (sub-component)
- **Purpose**: Shown when user lacks ORG_ADMIN role. Displays lock icon + "Access Denied" message.
- **Page**: AuditPage

### `src/features/audit/components/AuditTable.tsx`
- **Component**: `AuditTable` (default export)
- **Purpose**: Filterable/sortable audit event table with columns: Timestamp, Actor, Event Type, Resource, IP Address. Click row to open detail drawer.
- **Page**: AuditPage

#### `SortIcon` (sub-component)
- **Purpose**: Chevron up/down icon for sortable column headers
- **Page**: AuditTable

### `src/features/audit/components/AuditDetailDrawer.tsx`
- **Component**: `AuditDetailDrawer` (default export)
- **Purpose**: Right-side drawer panel showing full event details: event type, actor, timestamp, resource type+ID, request payload, response status, IP
- **Page**: AuditPage (opens on row click)

#### `Field` (sub-component)
- **Purpose**: Label + value row with optional copy-to-clipboard button
- **Page**: AuditDetailDrawer

---

## 12. Admin

### `src/features/admin/AdminPage.tsx`
- **Component**: `AdminPage` (default export)
- **Purpose**: Legacy tabbed admin panel (Users, Feature Flags, System Config) - shown when accessing `/admin` directly. ORG_ADMIN role gate.
- **Page**: `/admin`

### `src/features/admin/components/AdminShell.tsx`
- **Component**: `AdminShell` (named export)
- **Purpose**: Admin layout wrapper with collapsible 220px sub-nav sidebar showing 10 admin domains, breadcrumbs, and ORG_ADMIN role gate. Uses React Router `Outlet` for nested routes.
- **Page**: `/admin/*` (wrapper for all admin routes)

#### `AdminBreadcrumbs` (sub-component)
- **Purpose**: Breadcrumb trail for admin sub-pages (Admin > Users > User Detail)
- **Page**: AdminShell

#### `AdminNav` (sub-component)
- **Purpose**: Collapsible admin navigation sidebar with icon+label items for all 10 domains
- **Page**: AdminShell

### Admin Pages (10 total)

All admin pages are wrapped by AdminShell and require ORG_ADMIN role.

#### `src/features/admin/pages/UsersPage.tsx`
- **Component**: `UsersPage` (default export)
- **Purpose**: Full user management. List view with search/filter + detail view with activity timeline, role history, workspace permissions. Actions: change role, disable/enable account, reset MFA, delete user.
- **Page**: `/admin/users`

#### `src/features/admin/pages/SubscriptionsPage.tsx`
- **Component**: `SubscriptionsPage` (default export)
- **Purpose**: Subscription management. List subscriptions + detail with plan change, seat count update, invoice history, cancel subscription.
- **Page**: `/admin/subscriptions`

#### `src/features/admin/pages/GlobalStrategiesPage.tsx`
- **Component**: `GlobalStrategiesPage` (default export)
- **Purpose**: Cross-organization strategy management. List view + detail with versions tab, deployments tab, performance tab.
- **Page**: `/admin/strategies`

#### `src/features/admin/pages/OrganizationsPage.tsx`
- **Component**: `OrganizationsPage` (default export)
- **Purpose**: Organization management. List organizations + detail with general settings, workspace management (create/archive workspace), create new organization modal.
- **Page**: `/admin/organizations`

#### `src/features/admin/pages/BrokerConnectionsPage.tsx`
- **Component**: `BrokerConnectionsPage` (default export)
- **Purpose**: Broker connection management. List connections + detail with refresh token, disconnect broker, error log table.
- **Page**: `/admin/broker-connections`

#### `src/features/admin/pages/AdminAuditPage.tsx`
- **Component**: `AdminAuditPage` (default export)
- **Purpose**: Organization-wide audit log. Filters by date range, event type, actor. Paginated event table. Event detail panel. CSV export.
- **Page**: `/admin/audit`

#### `src/features/admin/pages/FeatureFlagsPage.tsx`
- **Component**: `FeatureFlagsPage` (default export)
- **Purpose**: Full feature flag management. List flags + detail with per-workspace override toggles. Create flag modal with key, name, description, category, default value.
- **Page**: `/admin/feature-flags`

#### `src/features/admin/pages/SystemConfigPage.tsx`
- **Component**: `SystemConfigPage` (default export)
- **Purpose**: System-wide configuration. Maintenance mode toggle, exchange trading hours editor (NSE/BSE/NFO), market freeze windows (add/delete), alert thresholds editor.
- **Page**: `/admin/system-config`

#### `src/features/admin/pages/ApiKeysPage.tsx`
- **Component**: `ApiKeysPage` (default export)
- **Purpose**: API key management. Create key (name, scopes), reveal prefix, revoke key, rotate key, copy-on-create modal.
- **Page**: `/admin/api-keys`

#### `src/features/admin/pages/WebhooksPage.tsx`
- **Component**: `WebhooksPage` (default export)
- **Purpose**: Webhook configuration. Create webhook (URL, event types checkboxes), enable/disable toggle, delete webhook, delivery history table, redeliver failed delivery.
- **Page**: `/admin/webhooks`

### Legacy Admin Components

#### `src/features/admin/components/UsersTab.tsx`
- **Component**: `UsersTab` (default export)
- **Purpose**: Legacy user management tab (superseded by UsersPage)
- **Page**: AdminPage

#### `src/features/admin/components/FeatureFlagsTab.tsx`
- **Component**: `FeatureFlagsTab` (default export)
- **Purpose**: Legacy feature flags tab (superseded by FeatureFlagsPage)
- **Page**: AdminPage

#### `src/features/admin/components/SystemConfigTab.tsx`
- **Component**: `SystemConfigTab` (default export)
- **Purpose**: Legacy system config tab (superseded by SystemConfigPage)
- **Page**: AdminPage

---

## 13. Trading

### `src/features/trading/OrderPlacementPage.tsx`
- **Component**: `OrderPlacementPage` (default export)
- **Purpose**: Manual order entry form. Fields: broker account selector, InstrumentSelector (symbol search), BUY/SELL direction chips, order type (Market/Limit/SL/SLM), quantity, price (for limit), trigger price (for SL), product type (MIS/NRML/CNC), validity (DAY/IOC). Shows live LTP from broker API + bid/ask. Order preview before submit.
- **Page**: `/orders/place`

### `src/features/trading/TraderProfilePage.tsx`
- **Component**: `TraderProfilePage` (default export)
- **Purpose**: Trader profile view. Shows: client info card (name, account ID, broker), account balance with margin utilization bar, portfolio summary (total holdings value, invested amount, total P&L), holdings table (symbol, qty, avg price, LTP, P&L), open orders table with cancel button and status chips.
- **Page**: `/profile`

#### `StatusChip` (sub-component)
- **Purpose**: Colored chip for order status (FILLED=green, ACKNOWLEDGED=blue, REJECTED=red, PENDING=gray, CANCELLED=orange)
- **Page**: TraderProfilePage

#### `PnLDisplay` (sub-component)
- **Purpose**: Colored P&L value display (green for profit, red for loss, with % change)
- **Page**: TraderProfilePage

---

## 14. Strategic Builder

### `src/features/strategic-builder/StrategicBuilderPage.tsx`
- **Component**: `StrategicBuilderPage` (default + named export)
- **Purpose**: Main visual strategy editor page. Routed at `/strategies/:id/edit` via VsbRoute wrapper. Contains mode indicator (Draft/Live/Paused), StrategyTestingStage with live chart (collapsible), and StrategyCanvas. Manages 3-state mode toggle (Draft→Live→Paused→Live).
- **Page**: `/strategies/:id/edit`

#### `StrategicBuilderPageInner` (sub-component)
- **Purpose**: Inner component containing all canvas logic, mode state, and child component composition
- **Page**: StrategicBuilderPage

#### `ModeIndicator` (sub-component)
- **Purpose**: Colored status indicator showing current mode: Draft (gray), Live (green), Paused (yellow) with pulsing dot for live
- **Page**: StrategicBuilderPageInner

#### `StrategyTestingStage` (sub-component)
- **Purpose**: ReactFlowProvider wrapper for the testing stage - enables ReactFlow context for the nested canvas
- **Page**: StrategicBuilderPageInner

### `src/features/strategic-builder/StrategyTestingStage.tsx`
- **Component**: `StrategyTestingStageInner` (sub-component)
- **Purpose**: Backtesting simulation stage with live TradingView chart, instrument selector, time range controls, and simulated order flow. Runs useStrategySimulator hook with 1.5s tick interval.
- **Page**: `/strategies/:id/test` (rendered within StrategicBuilderPage)

### `src/features/strategic-builder/components/BlockPalette.tsx`
- **Component**: `BlockPalette` (named + default export)
- **Purpose**: Searchable block category palette sidebar for drag-and-drop onto canvas. Shows all 77 block types grouped by category with icons. Collapsible category sections.
- **Page**: StrategicBuilderPageInner, StrategyCanvas

### `src/features/strategic-builder/components/NodeConfigPanel.tsx`
- **Component**: `NodeConfigPanel` (named + default export)
- **Purpose**: Right-side configuration panel for selected node. Shows block type info, parameter inputs by type (slider, select, text, number, multiselect, instrument, time), live value display when in live mode, error messages inline.
- **Page**: StrategicBuilderPageInner, StrategyCanvas

### `src/features/strategic-builder/components/StrategyCanvas.tsx`
- **Component**: `StrategyCanvas` (named + default export)
- **Purpose**: ReactFlow canvas wrapper managing nodeTypes (CanvasNode) and edgeTypes (AnimatedEdge/StaticEdge). Handles undo/redo stack, node selection, drag-drop from palette, canvasToAST() conversion, fitView on load.
- **Page**: StrategicBuilderPageInner

#### `StrategyCanvasInner` (sub-component)
- **Purpose**: Inner canvas with full ReactFlow onNodesChange/onEdgesChange handlers, connection logic, mode-based edge animation switching
- **Page**: StrategyCanvas

### `src/features/strategic-builder/components/CanvasControls.tsx`
- **Component**: `CanvasControls` (named + default export)
- **Purpose**: Floating toolbar on canvas with: zoom in/out/fit, undo/redo buttons, export strategy JSON button
- **Page**: StrategicBuilderPageInner, StrategyCanvas

#### `ControlButton` (sub-component)
- **Purpose**: Individual toolbar button with icon, tooltip, and disabled state
- **Page**: CanvasControls

### `src/features/strategic-builder/components/InstrumentSelector.tsx`
- **Component**: `InstrumentSelector` (named + default export)
- **Purpose**: Searchable instrument dropdown with exchange tabs (NFO/BFO/CDS/MCX), grouped by underlying symbol, shows lot size, option type/strike/expiry for derivatives. Uses useInstruments API hook.
- **Page**: OrderPlacementPage, StrategicBuilderPage (via StrategyTestingStage), NodeConfigPanel (for instrument-type params)

### `src/features/strategic-builder/components/StrategyChart.tsx`
- **Component**: `StrategyChart` (named + default export)
- **Purpose**: TradingView Lightweight Charts v5 candlestick chart with entry/exit arrow markers (green up arrow for buy, red down arrow for sell). Collapsible in StrategicBuilderPage header. Uses ResizeObserver for responsive sizing.
- **Page**: StrategicBuilderPageInner (collapsible panel)

### `src/features/strategic-builder/index.tsx`
- **Purpose**: Re-exports StrategicBuilderPage as named export for use in routing
- **Page**: N/A

---

## 15. Canvas (ReactFlow Nodes)

### `src/components/canvas/CanvasNode.tsx`
- **Component**: `CanvasNode` (default export)
- **Purpose**: ReactFlow custom node component registered in nodeTypes. Renders BaseNode with category-colored header band, node type label, port handles (source/target), and preview of configured parameters.
- **Page**: Used by ReactFlow in StrategyCanvas

### `src/components/canvas/BaseNode.tsx`
- **Component**: `BaseNode` (default export)
- **Purpose**: Visual node rendering with: category-colored left border band, node name header, parameter preview (key=value pairs), port handles (colored by port type: signal=blue, value=green, event=amber, position_info=purple, order_ref=red, leg_spec=indigo), live mode value display (green), error state (red border), unknown field type warning banner
- **Page**: CanvasNode (uses BaseNode internally)

### `src/components/canvas/PortHandle.tsx`
- **Component**: `PortHandle` (default export)
- **Purpose**: ReactFlow Handle wrapper with port-type-based coloring and tooltips showing port name/type
- **Page**: BaseNode (renders PortHandle for each port)

### `src/components/canvas/NodeQuickEditPopup.tsx`
- **Component**: `NodeQuickEditPopup` (default export)
- **Purpose**: Inline popup for quick parameter editing directly on the canvas node. Supports field types: operator (dropdown), slider, select, text, number, time, multiselect, instrument. Shows yellow warning banner for unknown field types.
- **Page**: CanvasNode (opens on node double-click)

### `src/components/canvas/AnimatedEdge.tsx`
- **Component**: `AnimatedEdge` (default export)
- **Purpose**: ReactFlow custom edge with GSAP-animated flowing particle (blue dot) along dashed smoothstep path. Used when canvas is in live mode.
- **Page**: StrategyCanvas (registered in edgeTypes)

### `src/components/canvas/StaticEdge.tsx`
- **Component**: `StaticEdge` (default export)
- **Purpose**: Simple static ReactFlow edge with smoothstep path. Used in draft/paused mode.
- **Page**: StrategyCanvas (registered in edgeTypes)

---

## 16. Layout Components

### `src/components/layout/CommandStudio.tsx`
- **Component**: `CommandStudio` (default export)
- **Purpose**: Studio layout wrapper composing StudioHeader + BlockPalette + canvas area + RightRail + StatusBar in a dark-themed studio environment
- **Page**: CanvasPage

### `src/components/layout/StudioHeader.tsx`
- **Component**: `StudioHeader` (default export)
- **Purpose**: Top bar for canvas studio with logo, breadcrumb, Validate button, Deploy button, and settings icon
- **Page**: CommandStudio

### `src/components/layout/BlockPalette.tsx`
- **Component**: `BlockPalette` (default export)
- **Purpose**: Dark-themed searchable sidebar showing block categories and block types as items. Used in CommandStudio layout.
- **Page**: CommandStudio

### `src/components/layout/RightRail.tsx`
- **Component**: `RightRail` (default export)
- **Purpose**: GSAP-animated slide-in panel from right side showing node configuration by category (Entry/Exit/Indicator/Greeks/Logic/Action rails). Shows selected node's category-specific parameters.
- **Page**: CommandStudio

### `src/components/layout/StatusBar.tsx`
- **Component**: `StatusBar` (default export)
- **Purpose**: Bottom status bar showing: market open/closed status, node count, edge count, validation status (x errors), strategy last saved time, live ticker tape
- **Page**: CommandStudio

---

## 17. Rail Components

Rail components render inside RightRail and provide category-specific parameter editing forms.

### `src/components/rail/EntryConditionRail.tsx`
- **Component**: `EntryConditionRail` (default export)
- **Purpose**: Entry condition parameters: reference price type (LTP/OHLC/Close), operator (above/below/crosses_above/crosses_below), indicator name, threshold value
- **Page**: RightRail (Entry Condition category)

### `src/components/rail/ExitConditionRail.tsx`
- **Component**: `ExitConditionRail` (default export)
- **Purpose**: Exit condition parameters: exit type (Fixed Target/Fixed Stop Loss/Trailing Stop/Time-Based), target/stop values, trailing activation
- **Page**: RightRail (Exit Condition category)

### `src/components/rail/IndicatorRail.tsx`
- **Component**: `IndicatorRail` (default export)
- **Purpose**: Indicator parameters: indicator name (SMA/EMA/RSI/MACD/etc), period, operator, threshold, timeframe (1m/5m/15m/1h/1d)
- **Page**: RightRail (Indicator category)

### `src/components/rail/LogicRail.tsx`
- **Component**: `LogicRail` (default export)
- **Purpose**: Logic gate parameters: gate type (AND/OR/NOT/If-Then-Else), input count, short-circuit mode
- **Page**: RightRail (Logic category)

### `src/components/rail/ActionRail.tsx`
- **Component**: `ActionRail` (default export)
- **Purpose**: Order action parameters: direction (BUY/SELL chips), order type (MARKET/LIMIT/SL/SLM), quantity value, validity (DAY/IOC), product type (MIS/NRML/CNC), exit order type (for brackets)
- **Page**: RightRail (Action category)

### `src/components/rail/GreeksRail.tsx`
- **Component**: `GreeksRail` (default export)
- **Purpose**: Greeks condition parameters: greek type (Delta/Gamma/Theta/Vega), operator (>, <, >=, <=), aggregation (per leg/per position), threshold value, AI-generated hint text
- **Page**: RightRail (Greeks Condition category)

### `src/components/rail/PositionMgmtRail.tsx`
- **Component**: `PositionMgmtRail` (default export)
- **Purpose**: Position sizing parameters: sizing method (Fixed Qty/Percent of Capital/Risk-Based), value, max exposure %
- **Page**: RightRail (Position Management category)

### `src/components/rail/TimeRail.tsx`
- **Component**: `TimeRail` (default export)
- **Purpose**: Time filter parameters: session type (pre-open/regular/post-close), from time, to time, days of week
- **Page**: RightRail (Time Filter category)

### `src/components/rail/MultiLegRail.tsx`
- **Component**: `MultiLegRail` (default export)
- **Purpose**: Multi-leg parameters: instrument type (FUTURES/OPTIONS), strike type (ATM/OTM/ITM/custom), expiry, ratio (for spreads), leg count
- **Page**: RightRail (Multi Leg category)

### `src/components/rail/StrategyAdjRail.tsx`
- **Component**: `StrategyAdjRail` (default export)
- **Purpose**: Strategy adjustment parameters: adjustment type (Adjustment-on-Event/Roll Position), trigger P&L % threshold, action (add/reduce/exit)
- **Page**: RightRail (Strategy Adjustment category)

### `src/components/rail/AnalyticsRail.tsx`
- **Component**: `AnalyticsRail` (default export)
- **Purpose**: Analytics condition parameters: metric (Max Pain/GEX/Probability of Profit/MMI/Unusual Activity), operator, threshold value
- **Page**: RightRail (Analytics category)

### `src/components/rail/UtilityRail.tsx`
- **Component**: `UtilityRail` (default export)
- **Purpose**: Utility block parameters: block type (Comment/Variable/Wait and Trade), name/text, initial value for variables
- **Page**: RightRail (Utility category)

---

## 18. UI Primitives

### `src/components/ui/badge.tsx`
- **Component**: `Badge` (named export)
- **Purpose**: CVA (class-variance-authority) based badge with variants: draft, validated, active, success, danger, warning, info, outline
- **Page**: Used throughout all features

### `src/components/ui/dialog.tsx`
- **Components**: `Dialog`, `DialogContent`, `DialogHeader`, `DialogTitle`, `DialogDescription` (all named exports)
- **Purpose**: Radix UI Dialog-based modal system. Dialog wraps overlay+content. DialogContent handles close on escape/click-outside. DialogHeader/Title/Description are structural sub-components.
- **Page**: VersionHistoryPage, StrategySettingsPage, and any feature needing modals

---

## 19. Hooks

### `src/hooks/useStrategySimulator.ts`
- **Hook**: `useStrategySimulator` (named export)
- **Purpose**: Canvas mode simulation hook. Generates ticks at 1.5s interval. Computes indicator values (RSI/SMA/EMA/MACD/Bollinger/ATR/VWAP/SuperTrend/Stochastic/ADX). Propagates entry/logic/exit signal flow through graph. Fires simulated orders. Updates nodeState to 'live' for value display. Manages tick buffer for chart.
- **Used by**: StrategyTestingStage

---

## Component Count Summary

| Category | Count |
|----------|-------|
| Page components | 30 |
| Layout/Shell components | 6 |
| Feature sub-components | 80+ |
| Canvas/Node components | 6 |
| Rail components | 12 |
| UI primitives | 5 (Badge, Dialog + 4 sub-components) |
| Hooks | 1 |
| **Total** | **~140+** |

---

## Page Route Summary

| Route | Page Component |
|-------|---------------|
| `/` | DashboardPage |
| `/login` | LoginPage |
| `/strategies` | StrategiesPage |
| `/strategies/templates` | TemplateBrowserPage |
| `/strategies/import` | ImportPage |
| `/strategies/:id/edit` | StrategicBuilderPage |
| `/strategies/:id/versions` | VersionHistoryPage |
| `/strategies/:id/settings` | StrategySettingsPage |
| `/deployments` | DeploymentsPage |
| `/deployments/:id` | DeploymentDetailPage |
| `/backtest/:id` | BacktestResultsPage |
| `/instruments` | InstrumentsPage |
| `/risk` | RiskPage |
| `/settings` | SettingsPage |
| `/audit` | AuditPage |
| `/admin` | AdminPage (legacy) |
| `/admin/*` | Admin pages (10) via AdminShell |
| `/orders/place` | OrderPlacementPage |
| `/profile` | TraderProfilePage |
