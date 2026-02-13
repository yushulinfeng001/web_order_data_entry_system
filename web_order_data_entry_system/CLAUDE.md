# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Chinese-language web-based order management system (订单管理系统) built with Flask. Manages price lists, products, customers, and orders with CSV-based file storage. Single-page app with all frontend code in one HTML template.

## Commands

```bash
# Install dependencies (uses uv with Python 3.11)
uv sync

# Run the dev server (http://127.0.0.1:5001)
uv run web-order
# or directly:
uv run python -m order_management.app
```

No test suite or linter is configured.

## Architecture

**Single-module monolith** — the entire backend is `order_management/app.py` (~580 lines), and the entire frontend is `order_management/templates/index.html` (~31K, inline CSS + vanilla JS).

### Data Layer
- All data stored as CSV files in `order_management/data/`:
  - `price_lists.csv` — price list definitions (id, name)
  - `products.csv` — products tied to price lists (id, list_id, name, unit, price)
  - `customers.csv` — customers tied to price lists (id, name, list_id)
  - `orders.csv` — order records (id, date, customer, product, unit, price, quantity, total)
- Read/write via `read_csv()` / `write_csv()` helpers with a global `threading.Lock` for concurrency
- IDs are auto-incrementing integers (`next_id()`)

### API Structure
All JSON REST endpoints under `/api/`:
- `/api/pricelists` — CRUD + copy (`/api/pricelists/<id>/copy`)
- `/api/products` — CRUD, filterable by `list_id` query param
- `/api/customers` — CRUD
- `/api/orders` — CRUD + search (`/api/orders/search`) + import/export
- `/api/orders/export/csv`, `/api/orders/export/excel` — filtered export
- `/api/data/export` — full data export as multi-sheet Excel
- `/api/orders/import` — CSV/XLSX import with header mapping (Chinese ↔ English)

### Key Design Decisions
- Cascade delete: deleting a price list also removes all its products
- Products are unique within a price list by (name, unit) pair
- Units are restricted to `['套', '个']` (sets, pieces)
- Order search supports regex for customer names, and flexible date range granularity (YYYY, YYYY-MM, YYYY-MM-DD)
- Export files use `utf-8-sig` encoding for Excel compatibility with Chinese characters

### Frontend
- Single-page with tab navigation: 货物清单 (Products), 客户管理 (Customers), 订单管理 (Orders)
- Inline editing in tables (no separate edit forms)
- All API calls via vanilla `fetch()`

## Change Log

### 2026-02-13
- 新增 `CLAUDE.md`：记录项目架构、常用命令、数据层设计、API 结构等，供后续开发者快速上手
- 重写 `README.md`：替换空白占位内容，补充中文项目简介、快速启动、功能列表、技术栈说明
