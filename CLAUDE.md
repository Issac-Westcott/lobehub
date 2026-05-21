@AGENTS.md

## AI Assistant Guidelines

### Core Principle: Reuse Before Building

Before designing any new rendering, display, or data-flow mechanism, **first audit what LobeHub already provides**. The codebase is large and feature-rich; building a parallel system is almost always the wrong call.

Checklist before proposing a new subsystem:
1. Does a store slice or selector already manage this state?
2. Does a Portal or Drawer pattern already handle this display mode?
3. Does an existing Markdown plugin or rehype plugin already transform this content?
4. Does `@lobehub/ui` already expose the needed primitive?

If any answer is yes, integrate with the existing path rather than introducing a new one.

### Known Rendering Mechanisms

#### HTML / Artifact Rendering — `<lobeArtifact>` + Artifact Portal

When AI responses contain HTML, SVG, React, or other renderable content, LobeHub uses the **Artifact system** — do NOT build a separate rendering pipeline for this.

How it works:
- AI wraps output in `<lobeArtifact identifier="..." type="text/html" title="...">...</lobeArtifact>`
- `useChatMarkdown` → `LobeArtifact` rehype plugin detects the tag during streaming
- The Artifact Portal panel opens automatically (Ant Design Drawer, iframe-based rendering)
- Users can toggle between Preview and Code views
- State lives in `src/store/chat/slices/portal/` (`openArtifact`, `closeArtifact` actions)
- Key files:
  - Detection: `src/features/Conversation/Markdown/plugins/LobeArtifact/rehypePlugin.ts`
  - Portal UI: `src/features/Portal/Artifacts/Body/index.tsx`
  - HTML iframe render: `src/features/Portal/Artifacts/Body/Renderer/HTML.tsx`
  - Tag constants: `packages/const/src/plugin.ts` (`ARTIFACT_TAG`, `ARTIFACT_TAG_REGEX`)

**Implication**: Any feature that needs to display rendered HTML to the user should instruct the AI to wrap output in `<lobeArtifact type="text/html">` and let the existing portal handle display. No custom iframe, no custom drawer needed.

#### Chat Input Action Buttons

New toolbar buttons above the chat input belong in `src/features/ChatInput/ActionBar/`. Each action is a self-contained directory (e.g., `ActionBar/Memory/`, `ActionBar/Tools/`). Follow the existing pattern: add a directory, export the component, register it in `ActionBar/config.ts`.

