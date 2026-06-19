# Chat Components

The renderer chat UI. Modules are organized in **layers**: a module may import only
from the same or a lower layer, plus `@cherrystudio/ui`, `@shared/*`, and the renderer
utils/hooks already shared app-wide. This keeps each layer independently buildable and
lets the tree be carved in dependency order (see [Carve status](#carve-status)).

Import from the package entry `@renderer/components/chat` rather than deep paths, except
when working inside a module.

## Layers

### Foundation (leaf — no chat-internal dependencies)

The design-system floor everything else builds on. Pure presentation / contracts; no
business state, no data access.

- **`primitives/`** — stateless building blocks (`Panel`, `Toolbar`, `EmptyState`,
  `ErrorState`, `LoadingState`, `StatusBadge`, `Disclosure`, `ContextMenu`). Themed via
  `@cherrystudio/ui` tokens.
- **`tokens/`** — composer inline-token views (mentions / quotes): `ComposerToken` and
  the `ChatTokenView` contract. Depends only on `utils/`.
- **`layout/`** — React contexts for layout mode, viewport insets, and the immersive
  navbar, plus the `NarrowLayout` helper. Context plumbing only — no business state.
- **`utils/`** — pure helpers (e.g. `quoteToken`).

### Content (depends on Foundation)

- **`messages/`** — the message-rendering family: `blocks/`, `frame/`, `list/`
  (virtualized), `markdown/`, and `MessageContentProvider`. Projects a `MessageListItem`
  at the data boundary; virtualized lists rely on stable item identity.
- **`composer/`** — the input composer (`ComposerCore` / `ComposerSurface`, draft, paste,
  presets, schema). Depends on `messages/` for token/quote rendering.
- **`motion/`** — animation descriptors (composer dock, message enter).

### Orchestration & features (depends on Content)

- **`shell/`** — conversation and app shells: `ConversationShell`, `ChatAppShell`,
  `ConversationStageCenter`, `PageSidebar`, `OverlayHost`, `RightPaneHost`, `MainPane`.
- **`panes/`** — right-pane content (`ArtifactPane`, `PdfPreviewPanel`, `ReferencePanel`)
  and the `RightPaneRegistry`.
- **`resources/`** — the resource list (topics / sessions) with grouping and expansion.
- **`settings/`** — in-chat settings sections.
- **`actions/`** — message / resource action registries, menus, and confirm dialogs.
- **`adapters/`** — the thin contract layer that projects business entities (topic /
  session / message) into stable UI shapes and aggregates the pane / action registries.
  Fetches nothing and owns no cache. See [`adapters/README.md`](./adapters/README.md).

### Pages (top consumers — live in `src/renderer/pages/`)

`pages/home` (chat), `pages/agents` (agent sessions), and `pages/history` assemble the
above into routed surfaces.

## Dependency rule

> A module imports only from the same or a lower layer (+ `@cherrystudio/ui`, `@shared/*`,
> and renderer utils/hooks already on `main`). Foundation imports nothing chat-internal
> except foundation peers.

Concretely: `composer/` and `motion/` may import `messages/`, but `messages/` must not
import `composer/`; `adapters/` may import `actions/` / `panes/` / `composer/`, but none
of those may import `adapters/`.

## Carve status

The chat UI is being moved from `feat/chat-page` onto `main` in dependency order, one
self-contained, CI-green PR per layer:

1. **Foundation** — `primitives`, `tokens`, `layout`, `utils`. ← _this PR_
2. **Content** — `messages`, then `composer` + `motion`.
3. **Orchestration** — `shell`, `panes`, `resources`, `settings`, `actions`, `adapters`.
4. **Pages** — `pages/home`, `pages/agents`, `pages/history`.

Side components carve alongside the layer they depend on: `QuickPanel` and
`ModelSelector` are self-contained (foundation-independent); `GlobalSearch` depends on
`messages/` and lands with or after it.
