# GeoVera Database Schema Documentation

**Project:** GeoVera AI Brand Intelligence Platform  
**Supabase Project Ref:** vozjwptzutolvkvfpknk  
**Generated:** 2026-02-08 14:43:47  
**Total Tables:** 90

---

## Table of Contents

1. [Schema Overview](#schema-overview)
2. [Core Tables](#core-tables)
3. [Pipeline & Data Processing](#pipeline--data-processing)
4. [Intelligence & Insights](#intelligence--insights)
5. [Social & Trends](#social--trends)
6. [Content Studio](#content-studio)
7. [Radar & Market Intelligence](#radar--market-intelligence)
8. [Chronicle & DNA](#chronicle--dna)
9. [Subscription & Quotas](#subscription--quotas)
10. [System & Configuration](#system--configuration)

---

## Schema Overview

The GeoVera database consists of **90 tables** organized into functional domains:

- **Core Tables (2):** brands, customers
- **gv_* Tables (82):** Main application tables with Row Level Security enabled
- **Supporting Tables (6):** api_keys, user_brands, usage_metrics, execution logs

### Key Schema Patterns

- **UUID Primary Keys:** All tables use `gen_random_uuid()` for primary keys
- **Row Level Security (RLS):** Enabled on all `gv_*` tables
- **Timestamps:** Standard `created_at` and `updated_at` tracking
- **Foreign Keys:** Extensive relationships linking brands → runs → jobs → artifacts → insights → tasks
- **JSONB Fields:** Flexible metadata storage for dynamic data
- **Check Constraints:** Enum validation and score ranges (0-100, 0-1)

---

## Core Tables

### brands
**Purpose:** Central brand registry  
**Rows:** 11 | **RLS:** ✓

**Columns:**

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | uuid | NOT NULL, DEFAULT gen_random_uuid() | Primary key (UUID) |
| `brand_name` | text | NOT NULL, DEFAULT 'Default Brand'::text | Brand display name |
| `created_at` | timestamp with time zone | NOT NULL, DEFAULT now() | Record creation timestamp |
| `category` | text | CHECK | Business category (fnb, fashion, fmcg, beauty, contractor, finance, other) |
| `business_type` | text | CHECK | Business model (offline, online, hybrid) |
| `description` | text | - | - |
| `web_url` | text | - | - |
| `whatsapp` | text | - | - |
| `logo_url` | text | - | - |
| `updated_at` | timestamp with time zone | NOT NULL, DEFAULT now() | Last update timestamp |
| `onboarding_completed` | boolean | NOT NULL, DEFAULT false | - |
| `google_maps_url` | text | - | - |
| `country` | text | - | - |
| `onboarding_step` | smallint | NOT NULL, DEFAULT 0 | - |
| `subscription_tier` | text | DEFAULT 'starter'::text | - |
| `subscription_started_at` | timestamp with time zone | - | - |
| `subscription_expires_at` | timestamp with time zone | - | - |

### customers
**Purpose:** Customer/user accounts
**Rows:** 7 | **RLS:** ✓

**Key Columns:**

- `id` (uuid) - Default: gen_random_uuid()
- `email` (text)
- `full_name` (text?)
- `company_name` (text?)
- `phone` (text?)
- `tier` (text) - Default: 'trial'::text
- `status` (text) - Default: 'active'::text
- `trial_ends_at` (timestamp with time zone?) - Default: (now() + '14 days'::interval)

*+ 6 more columns*

---

## Pipeline & Data Processing

The data pipeline processes raw content through multiple stages with quality gates and confidence scoring.


### gv_runs
**Purpose:** Execution runs tracking brand intelligence cycles
**Rows:** 29 | **RLS:** ✓

**Key Columns:**

- `id` (uuid) - Default: gen_random_uuid()
- `brand_id` (uuid)
- `mode` (text) - Default: 'auto'::text
- `status` (text) - Default: 'active'::text
- `cost_summary` (jsonb) - Default: '{}'::jsonb
- `created_at` (timestamp with time zone) - Default: now()
- `updated_at` (timestamp with time zone) - Default: now()
- `started_at` (timestamp with time zone?)

*+ 15 more columns*

### gv_jobs
**Purpose:** Individual processing jobs within runs
**Rows:** 34 | **RLS:** ✓

**Key Columns:**

- `id` (uuid) - Default: gen_random_uuid()
- `brand_id` (uuid)
- `run_id` (uuid?)
- `job_type` (text)
- `status` (text) - Default: 'queued'::text
- `attempts` (integer) - Default: 0
- `max_attempts` (integer) - Default: 3
- `scheduled_for` (timestamp with time zone) - Default: now()

*+ 12 more columns*

### gv_raw_artifacts
**Purpose:** Ingested raw content from various sources
**Rows:** 107 | **RLS:** ✓

**Key Columns:**

- `id` (uuid) - Default: gen_random_uuid()
- `brand_id` (uuid)
- `run_id` (uuid?)
- `source` (text)
- `platform` (text?)
- `source_category` (text)
- `signal_layer` (integer)
- `impact_level` (text)

*+ 14 more columns*

### gv_normalized_artifacts
**Purpose:** Cleaned and standardized content
**Rows:** 105 | **RLS:** ✓

**Key Columns:**

- `id` (uuid) - Default: gen_random_uuid()
- `raw_artifact_id` (uuid)
- `brand_id` (uuid)
- `run_id` (uuid?)
- `source` (text)
- `platform` (text?)
- `source_category` (text)
- `signal_layer` (integer)

*+ 24 more columns*

### gv_perplexity_payloads
**Purpose:** External Reality Engine queries and responses
**Rows:** 6 | **RLS:** ✓

**Key Columns:**

- `id` (uuid) - Default: gen_random_uuid()
- `brand_id` (uuid)
- `run_id` (uuid?)
- `cycle_window` (text)
- `assembled_at` (timestamp with time zone) - Default: now()
- `assembly_version` (text) - Default: 'PH3-PP1'::text
- `layer2_signals` (jsonb) - Default: '[]'::jsonb
- `layer2_count` (integer?) - Default: 0

*+ 16 more columns*

### gv_strategic_synthesis
**Purpose:** AI-synthesized strategic insights
**Rows:** 4 | **RLS:** ✓

**Key Columns:**

- `id` (uuid) - Default: gen_random_uuid()
- `brand_id` (uuid)
- `run_id` (uuid)
- `evidence_pack_id` (uuid)
- `confidence_score_id` (uuid)
- `question_set_id` (uuid?)
- `min_confidence_score` (numeric)
- `evidence_pack_refs` (ARRAY) - Default: ARRAY[]::uuid[]

*+ 19 more columns*

### gv_confidence_scores
**Purpose:** Quality and confidence metrics
**Rows:** 5 | **RLS:** ✓

**Key Columns:**

- `id` (uuid) - Default: gen_random_uuid()
- `evidence_pack_id` (uuid)
- `brand_id` (uuid)
- `run_id` (uuid)
- `total_evidence_items` (integer) - Default: 0
- `attributed_evidence_items` (integer) - Default: 0
- `layer2_evidence_items` (integer) - Default: 0
- `layer0_evidence_items` (integer) - Default: 0

*+ 14 more columns*