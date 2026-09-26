# Repository Guidelines for AI Agents

These guidelines ensure consistency, brand coherence, clean component architecture, and transparent, maintainable code across the entire Book My Event codebase.

---

## 1. Git & Workflow Etiquette (Strict Rule)
- **Never commit or push autonomously:** Do NOT run `git commit` or `git push` unless the user explicitly tells you to do so.
- **Leave changes in the working tree:** Make edits directly in the local files so the user can review them with `git status` and `git diff`.
- **Pre-flight checks:** Always run type checking (`npx tsc --noEmit`) and unit tests (`npx vitest run`) before reporting any task complete to ensure nothing is broken.

---

## 2. Plain-English Commenting & Code Readability
> **Core Principle:** Code is read far more often than it is written. Comments must be written in **clear, jargon-free English** so that a person with no coding background can read through any file and understand **what** the code does, **why** it was written that way, and **how** it affects the user.

- **Explain the "Why" and Real-World Impact:**
  - ❌ *Cryptic / Dev Jargon:* `// Inset clamp memoization`
  - ✅ *Plain English:* `// Make sure the bottom of the modal leaves a 16px cushion above the phone's navigation bar so buttons aren't hard to tap or cut off by the home indicator.`
- **Annotate Math & Calculations:** Whenever an offset, multiplier, or formula is used, explain the visual or business reason:
  - ✅ *Example:* `// We cap the gradient height at 80% of the tab bar so scrolling venue photos softly fade out behind the navigation bar without washing out the bottom half of the screen.`
- **Standard Plain-English Section Banners:** In components and screens, use clear structural headings so humans and AI search tools can find sections instantly without reading hundreds of lines:
  ```ts
  // ============================================================================
  // 1. STATE & DATA (What this screen tracks and remembers)
  // ============================================================================

  // ============================================================================
  // 2. USER ACTIONS (What happens when buttons, cards, or chips are tapped)
  // ============================================================================

  // ============================================================================
  // 3. SCREEN DISPLAY (The visual layout rendered for the user)
  // ============================================================================

  // ============================================================================
  // 4. STYLES & THEME (Colors, spacing, fonts, and borders)
  // ============================================================================
  ```

---

## 3. Single Source of Truth & Function Deduplication
> **Core Principle:** Never invent a second function to do something an existing function already does.

- **Search First:** Before writing any utility, calculation, date formatter, currency converter, or validation logic, search `packages/domain` and `src/utils` to see if one already exists.
- **Import & Reuse:** If a function exists, import and use it directly. Never copy-paste or write a local duplicate version inside a component or screen.
- **Extend, Don't Clone:** If an existing function is close but lacks an option (e.g. formatting a date range vs a single date), extend the original shared function with an optional parameter rather than creating a parallel helper.
- **Business Logic in Domain:** Any core business logic (booking calculations, pricing, refund rules, permissions) belongs strictly in `packages/domain`, not inside React components.

---

## 4. Token Discipline & "Zero Magic Numbers"
- **No hardcoded values:** Never write arbitrary raw hex color codes (e.g., `#111814`, `#eaedf0`), arbitrary pixel paddings/margins, or raw border radii into stylesheets or inline styles.
- **Always use `tokens.*`:** Import and use `tokens` from `@bmt/design-tokens` (e.g., `tokens.color.surface`, `tokens.spacing.lg`, `tokens.radius.xxxl`).
- **Update the token library first:** If a new design stop is genuinely needed (e.g. adding a new radius or spacing stop), add it to `packages/design-tokens/src/index.ts` first, then reference that token in your code.

---

## 5. Brand Coherence & Phosphor Icons
- **Exclusively use Phosphor SVG icons** from `apps/mobile/src/components/icons.tsx`.
- **Do not disregard the usage of Phosphor icons:** Omitting icons or using ad-hoc unicode symbols/emojis breaks brand coherence.
- Whenever options, preferences, filter categories, steppers, search actions, or status indicators are displayed, pair them with the appropriate Phosphor icon (e.g., `ForkKnife`, `Leaf`, `Wine`, `Prohibit`, `Users`, `Check`, `MapPin`, `CalendarCheck`, `FadersHorizontal`, `MagnifyingGlass`, `Star`, `X`, `CaretRight`, `ArrowRight`).
- Never introduce emojis anywhere in user-facing UI or prototypes.

---

## 6. Component Reuse & Standard Selection Controls
> **Core Principle:** Use existing components when possible. Do not invent new layouts to achieve a goal. Reference the web and mobile designs for existing components. If a component exists in the library, use it; otherwise, create or extend a reusable variant.

- **Do not create bespoke one-off containers** to solve a layout need when a standard component exists in the library.
- If a component does not support a needed variation (e.g. multi-select, icon prefix, subtle tone, size):
  - **Extend the existing component** by adding props/variants, or
  - **Create a reusable component** in the design system (`components/common/`) and export it through `components/ui.tsx`.
- Keep parity and reference features between Web (`apps/web`) and Mobile (`apps/mobile`).
- **Standard Selection & Input Controls:** For options, preferences, tags, filters, and numeric counts, use the design system's established controls (e.g., `SelectChip`, `FilterChip`, `MultiSelectChipGroup` for tag selections, `Stepper` for quantities) rather than inventing arbitrary card grids or custom box containers.
  - Selection controls must support both **single-select** and **multi-select** behaviors cleanly.
  - Active states must provide crisp visual distinction using brand tokens (`tokens.color.cta`, high-contrast active icons, and clear typography).

---

## 7. Component API & Prop Conventions
> **Core Principle:** Every component in the design system must feel like it was built by the same author. Prop names, event callbacks, variants, and element slots must follow predictable, uniform conventions across the entire library.

- **Standardized Prop Naming:**
  - **User Actions / Callbacks:** Always use `onPress`, `onSelect`, `onChange`, `onClose`, `onDismiss`. *(Never use `onClick`, `handlePress`, `actionFn`, or `callback`)*.
  - **Text & Copy:** Use `label` for buttons, chips, and single-line controls. Use `title` and `subtitle` for cards, modals, and headers. Use `description` or `helperText` for supporting copy.
  - **Boolean States:** Use standard adjective flags: `selected`, `disabled`, `loading`, `visible`, `fullWidth`.
  - **Variants & Sizes:** Use `variant?: "primary" | "secondary" | "subtle" | "ghost" | "outline"` and `size?: "sm" | "md" | "lg"`.
- **Element Slots & Icon Composition (ReactNode over Enums):**
  - Always accept `icon?: React.ReactNode` or `trailingIcon?: React.ReactNode`. *(Never pass string icon names like `icon="heart"` that require internal switch statements; accept the Phosphor component directly so callers have full control over icon, weight, and styling)*.
  - Use `children` or header/footer slots so parent screens can customize accessories without hacking internal styles.
- **Explicit, Exported TypeScript Interfaces:**
  - Every reusable component must define and export an explicit props interface (e.g., `export interface SelectChipProps { ... }`). Never use inline anonymous type blocks or `any`.
- **Controlled State Discipline:**
  - Any selectable or input component (chips, steppers, date pickers, search fields) must follow standard controlled patterns: pair value props (`selected`, `value`) with callback props (`onSelect`, `onChange`).
- **Barrel Export Discipline (`components/ui.tsx`):**
  - Every reusable component created in `components/common/`, `components/cards/`, or `components/modals/` must be exported through `apps/mobile/src/components/ui.tsx` so screens can import them cleanly from a single entry point.

---

## 8. Mobile Safe Areas, Viewports & Touch Targets
- **Safe Area Insets:** Always account for safe area insets (`useSafeAreaInsets()`) for both top and bottom edges.
- **Android Navigation:** On Android, never assume the bottom inset is 0 or low; account for the 48dp system 3-button navigation panel and gesture navigation bars.
- **Floating Bars & Modals:** Modals, bottom sheets, sticky footers (`PinnedFooter`), and bottom navigation (`BottomTabs`) must never be covered by or collide with the system navigation panel or camera notches.
- **Touch Targets:** Interactive elements must have a minimum touch target size of 44×44dp (using `hitSlop` or minimum dimensions) so buttons are easy to tap.
- **Accessibility:** Every `Pressable` must include an explicit `accessibilityRole="button"` and a meaningful, plain-English `accessibilityLabel`.

---

## 9. Offline Persistence
- Persist customer onboarding, search criteria, and event preferences to offline storage (`appStorage`) so that app reloads preserve user state without requiring repeated onboarding.