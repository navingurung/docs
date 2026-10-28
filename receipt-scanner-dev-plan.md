# AI OCR Scanner — Developer Plan

> **Project:** AI OCR Scanner (P-SAM-67)
> **Target app:** `taimatsu-tax-free-frontend` (Vite + React + MUI)
> **Prototype:** `receipt-ai-scan` (Next.js + shadcn)
> **Scope:** Scan / Upload receipt → Gemini → fill existing Orders inputs. No DB, no backend.

---

## Folder Structure

```
src/features/steps/aiReceiptScanner/
├── receiptSchema.ts
├── receiptCalc.ts
├── receiptToOrder.ts
├── gemini.ts
├── receiptImage.ts
├── frameAnalysis.ts
├── ReceiptCamera.tsx
├── ReceiptScanButtons.tsx
└── __tests__/
    ├── receiptCalc.test.ts
    └── receiptToOrder.test.ts
```

**Why `features/steps/aiReceiptScanner/`?**
It follows the existing pattern: `steps/smaregi/`, `steps/square/` and `steps/shopify/` hold components used only by `Orders.tsx`. The scanner is the same kind of thing, an order data source for the Orders step.

---

## Build Order

```
Logic (1–4) → API & Image (5–7) → UI (8–9) → Wiring (10–12)
```

Pure logic comes first, so the money parts are tested before any UI exists.

---

## New Files

### Step 1 — `receiptSchema.ts`

> **What:** The `Receipt` type, the AI prompt, the JSON schema sent to Gemini, and `normalizeReceipt()`.
> **Source:** Ported from `lib/receipt-schema.ts`. Only the import paths change.
> **Why:** This is the contract with the AI. AI output is never guaranteed to match the type, so everything passes through `normalizeReceipt()` before the app uses it (missing fields → `null`, strings → numbers, invalid JAN codes dropped).

### Step 2 — `receiptCalc.ts`

> **What:** `summarizeReceipt()` checks the items against the printed tax and total, and returns warnings and errors.
> **Source:** Ported from `lib/receipt-calc.ts`.
> **Why:** This is the guard for *"total and tax must never be wrong."* It handles inclusive/exclusive tax, the 8% / 10% rates, subtotal discounts and rounding differences, and flags any mismatch so staff check it.

### Step 3 — `receiptToOrder.ts`

> **What:** `receiptToOrder()` converts a `Receipt` into the shape `Orders.tsx` already uses (`details`, `sellDate`, `orderId`, `tax`, `received`, `discount`).
> **Source:** New.
> **Why:** This is the only file that knows the SAMURAI TAX order format. It uses the same rules as the Smaregi / Shopify / Square imports (50-char names, `#2` for duplicate names, `reduced` flag, v2/v3 `goodsType`), so the rest of the scanner stays independent of Orders.

### Step 4 — `__tests__/receiptCalc.test.ts`, `__tests__/receiptToOrder.test.ts`

> **What:** Vitest tests for steps 2 and 3.
> **Source:** New.
> **Why:** These two files decide the amounts that go into a tax refund. Tests cover inclusive and exclusive tax, mixed 8% / 10%, discounts, and v2 / v3 sale dates. They follow the repo's existing `__tests__/` convention.

### Step 5 — `gemini.ts`

> **What:** Sends the image to the Gemini API and returns a normalized `Receipt`.
> **Source:** Adapted from `lib/gemini.ts` + `app/api/receipt-scan/route.ts`.
> **Why:** All API details live in one place (endpoint, model, timeout, error messages, `store: false`), so a model or vendor change touches only this file. It runs in the browser because there's no backend in this phase.
> **Note:** It reads `VITE_GEMINI_API_KEY`, which is visible in the browser. The key must be restricted by HTTP referrer in Google Cloud.

### Step 6 — `receiptImage.ts`

> **What:** `prepareImage()` converts any image to JPEG and resizes the long edge to ≤ 2048px.
> **Source:** Ported from `lib/image.ts`.
> **Why:** Smaller uploads make the scan faster (the ~3 s target) and use fewer AI tokens, so it costs less. It also handles HEIC from iPhone/iPad.

### Step 7 — `frameAnalysis.ts`

> **What:** Checks brightness, blur, motion and receipt position on camera frames.
> **Source:** Ported from `lib/frame-analysis.ts`.
> **Why:** It catches bad photos before they're sent, because bad input means bad accuracy. It's pure pixel math, so it's kept out of the component.

### Step 8 — `ReceiptCamera.tsx`

> **What:** A fullscreen camera dialog that auto-captures when the receipt is in frame and steady.
> **Source:** Ported from `components/receipt-scanner.tsx`, with Tailwind rewritten in MUI.
> **Why:** Only needed when staff press Scan, so it's lazy-loaded and doesn't slow down the Orders step.

### Step 9 — `ReceiptScanButtons.tsx`

> **What:** The only component `Orders.tsx` imports. It holds the Scan and Upload buttons, the loading state, the error message and the check warnings.
> **Source:** New (MUI).
> **Why:** It gives Orders one entry point, the same way Orders uses the POS selectors. It returns the result through `onScanned`, and Orders decides how to apply it.

---

## Changed Files

### Step 10 — `Orders.tsx`

> **What:** One import and `<ReceiptScanButtons />` below the receipt-number input, plus `handleReceiptScanned`.
> **Why:** It reuses `handleImportedOrderChange` (item-limit check), `onTaxChange`, `onOrderIdChange` and `onReceivedChange`, exactly like the POS imports, and sets `taxOverriddenRef` so the printed tax wins over the auto-calculated tax. Staff then review in the existing items table.

### Step 11 — `ja.json`, `en.json`

> **What:** Labels for the buttons, loading text, errors and warnings.
> **Why:** The app uses i18n everywhere; no hard-coded text.

### Step 12 — `.env.sample`

> **What:** Adds `VITE_GEMINI_API_KEY` and `VITE_GEMINI_MODEL`.
> **Why:** Other developers and Render need to know these exist. Like the other `VITE_` values, they're read at build time.

---

## Not Ported (and why)

> **`app/page.tsx`, `receipt-review.tsx`, `final-confirm.tsx`:** The Orders items table already does review and editing.
> **`draft-storage.ts`:** No localStorage drafts; the Orders flow already holds state.
> **`proxy.ts`, API route:** Next.js server only; there's no server in this app.
> **`step-header.tsx`, `action-tile.tsx`, `icons.tsx`:** Replaced by MUI and `@mui/icons-material`.
> **Tailwind:** The app uses MUI.

---

## Rules While Building

> - One step at a time: explain → confirm → write.
> - Japanese comments, matching the repo.
> - No new dependencies.
> - Mapping follows the existing POS imports; the printed tax and total always win.
> - Small PRs: Logic (1–5) → UI (6–9) → Wiring (10–12).
