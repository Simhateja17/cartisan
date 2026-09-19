<div align="center">

# Cartisan

**Agentic commerce with a conscience: two Claude agents, one shared runtime, and a deterministic system of record that holds the final word.**

Built for the Razorpay Buildathon.

</div>

---

## The core principle

> Claude decides what to ask and how to explain it.
> Deterministic systems decide what is true, what is allowed, what is committed, what is paid, and what is recoverable.

Every design decision in this repository follows from that one sentence. The model is genuinely in charge of the conversation, and genuinely not in charge of the money.

---

## Where the code lives

| Folder | What it is |
| :--- | :--- |
| `frontend/` | Next.js app: the storefront chat, the merchant portal, the evidence viewer, and the operations console. |
| `backend/` | Python service: both agents, the shared agent runtime, the commerce core, and the Razorpay integration. |

This repository is the combined submission. `backend/` and `frontend/` are Git submodules that track the two source repositories, so their full commit histories live there:

- Frontend repo and history: https://github.com/Simhateja17/razorpay_frontend
- Backend repo and history: https://github.com/Simhateja17/razorpay_backend

Clone the mother repository with its children initialized:

```bash
git clone --recurse-submodules https://github.com/Simhateja17/cartisan.git
```

For an existing checkout, initialize them with `git submodule update --init --recursive`. The mother repository records which commit of each child is used; changes inside either child are committed and pushed in that child repository, then the updated submodule pointer is committed here.

---

## How a request flows

Every single request, on either agent, follows the same shape.

```mermaid
flowchart LR
    A["1. USER ASKS<br/><br/><i>I need a charger under<br/>Rs 3,000 that works<br/>with my phone.</i>"]
    B["2. HOST ADDS CONTEXT<br/><br/>Verified identity<br/>Current page and cart<br/>Conversation and memory<br/>Local time"]
    C["3. CLAUDE REASONS<br/><br/>Understands the goal<br/>Chooses what it needs<br/>Observes results<br/>Continues until done"]

    A --> B --> C

    C -.-> D["ALWAYS: SYSTEM PROMPT<br/><br/>Rules that always apply.<br/>Never invent product facts.<br/>Always search first.<br/>Checkout can only be staged."]
    C -.-> E["WHEN NEEDED: SKILL<br/><br/>A longer procedure, loaded<br/>only when it is relevant.<br/>Example: checking whether two<br/>products are compatible."]
    C -.-> F["TOOLS: THE AGENT'S HANDS<br/><br/>Real actions on real systems.<br/>Search products, add to cart,<br/>show a product card.<br/>The model picks the tool.<br/>Code runs it."]

    style A fill:#fff8e6,stroke:#b8860b,stroke-width:2px,color:#000
    style B fill:#fff8e6,stroke:#b8860b,stroke-width:2px,color:#000
    style C fill:#fff8e6,stroke:#b8860b,stroke-width:2px,color:#000
    style D fill:#eef4ff,stroke:#3b6fd4,stroke-width:2px,color:#000
    style E fill:#eef4ff,stroke:#3b6fd4,stroke-width:2px,color:#000
    style F fill:#eef4ff,stroke:#3b6fd4,stroke-width:2px,color:#000
```

**Why skills instead of one subagent per domain?** Shopping is one continuous conversation. Handing off between agents loses context and adds delay. A skill teaches the same agent a new trick without breaking the flow the customer is already in.

---

## Agent one: the shopping agent

From a vague need to a safe, human confirmed checkout.

It can help with search, comparison, compatible setups, the cart, checkout staging, and order and payment questions.

```mermaid
flowchart LR
    U1["UNDERSTAND<br/><br/>Already knows:<br/>facts need a tool call,<br/>prices come from the server,<br/>ask at most one question"]
    U2["LOAD PROCEDURE<br/><br/>Compatibility checking is rare,<br/>so that skill loads only<br/>when it is actually asked about"]
    U3["CALL TOOLS<br/><br/>Search catalogue,<br/>check compatibility,<br/>show product cards.<br/>Cards use real server data"]
    U4["CONTROLLED CART<br/><br/>Customer taps a shown card,<br/>the server resolves it,<br/>checks pass,<br/>the real cart updates"]
    U5["STAGE, THEN CONFIRM<br/><br/>The agent only stages checkout.<br/>No order yet.<br/>No money moved yet.<br/>A human confirms"]
    U6["RAZORPAY AND RECOVERY<br/><br/>One order is created.<br/>A payment attempt follows.<br/>Only a verified payment event<br/>marks it PAID.<br/>Retry is a new attempt<br/>on the same order"]

    U1 --> U2 --> U3 --> U4 --> U5 --> U6

    style U1 fill:#eaf7ee,stroke:#2e7d4f,stroke-width:2px,color:#000
    style U2 fill:#eaf7ee,stroke:#2e7d4f,stroke-width:2px,color:#000
    style U3 fill:#eaf7ee,stroke:#2e7d4f,stroke-width:2px,color:#000
    style U4 fill:#eaf7ee,stroke:#2e7d4f,stroke-width:2px,color:#000
    style U5 fill:#eaf7ee,stroke:#2e7d4f,stroke-width:2px,color:#000
    style U6 fill:#eaf7ee,stroke:#2e7d4f,stroke-width:2px,color:#000
```

> **The boundary.** The agent can search, explain, and stage a checkout. It cannot create a payment link, confirm an order, mark an order paid, or issue a refund.

---

## Agent two: the merchant agent

From a business question to a governed, evidence backed proposal.

It can help with sales, inventory, unmet demand, pricing, campaigns, order issues, and pending changes.

```mermaid
flowchart LR
    M1["UNDERSTAND<br/><br/>Already knows:<br/>every number needs a real read,<br/>always state the time window,<br/>observed is not guessed"]
    M2["LOAD PROCEDURE<br/><br/><i>What should I restock?</i><br/>loads the restock skill,<br/>which reads sales and stock<br/>and shows the math<br/>and any gaps"]
    M3["CALL READ TOOLS<br/><br/>Reads real stock and sales data.<br/>Code, not the model,<br/>calculates days of cover<br/>and reorder amount"]
    M4["STAGE PROPOSAL<br/><br/>Writes a pending proposal<br/>with real numbers and<br/>the evidence behind it"]
    M5["HUMAN MAKER CHECKER<br/><br/>The agent has no approve button.<br/>A human approves in the portal.<br/>Limits are re-checked<br/>at that moment"]
    M6["SAFE BUSINESS CLAIMS<br/><br/>Numbers describe what happened,<br/>not ROI or impact.<br/>If data is missing it says so.<br/>It never assumes zero"]

    M1 --> M2 --> M3 --> M4 --> M5 --> M6

    style M1 fill:#f4eefc,stroke:#6b4bb8,stroke-width:2px,color:#000
    style M2 fill:#f4eefc,stroke:#6b4bb8,stroke-width:2px,color:#000
    style M3 fill:#f4eefc,stroke:#6b4bb8,stroke-width:2px,color:#000
    style M4 fill:#f4eefc,stroke:#6b4bb8,stroke-width:2px,color:#000
    style M5 fill:#f4eefc,stroke:#6b4bb8,stroke-width:2px,color:#000
    style M6 fill:#f4eefc,stroke:#6b4bb8,stroke-width:2px,color:#000
```

> **The boundary.** The agent can read, analyse, explain, and stage. It cannot approve, apply, move stock, change a price, or send money.

---

## The shared engineering foundation

Both agents sit on the same runtime, so both inherit the same speed and the same guarantees.

```mermaid
flowchart TB
    subgraph SF [" "]
    direction LR
    F1["FAST WITHOUT LOSING<br/>INTELLIGENCE<br/><br/>Stable parts of the prompt<br/>are cached<br/>User context comes after<br/>Independent reads run in parallel<br/>The UI streams in as it is ready"]
    F2["SAFETY IS CODE,<br/>NOT PROMPT HOPE<br/><br/>Identity comes from the host,<br/>never from the chat<br/>Outside text is sanitized first<br/>Writes only accept<br/>server issued references<br/>Limits are enforced on every write"]
    F3["SUPABASE IS THE<br/>SOURCE OF TRUTH<br/><br/>Catalog, specs, compatibility rules<br/>Cart versions, inventory, reservations<br/>Checkout stages, orders, attempts<br/>Conversations, turns, presentations<br/>Merchant proposals and approvals<br/>Outbox and deduplicated inbox"]
    F4["PROOF AND OPERABILITY<br/><br/>Evidence: who acted, on what,<br/>what was checked,<br/>and what happened<br/><br/>Operations: is the system healthy.<br/>Speed, errors, payments, recovery"]
    end

    style SF fill:#fafafa,stroke:#cccccc,color:#000
    style F1 fill:#fff0f0,stroke:#c25353,stroke-width:2px,color:#000
    style F2 fill:#fff0f0,stroke:#c25353,stroke-width:2px,color:#000
    style F3 fill:#fff0f0,stroke:#c25353,stroke-width:2px,color:#000
    style F4 fill:#fff0f0,stroke:#c25353,stroke-width:2px,color:#000
```

---

## System map

How the pieces actually wire together.

```mermaid
flowchart TB
    subgraph CLIENT ["FRONTEND (Next.js)"]
        direction LR
        S["Storefront<br/>shopping chat"]
        P["Merchant portal<br/>approvals"]
        E["Evidence viewer<br/>journeys"]
        O["Operations<br/>health and recovery"]
    end

    API["FastAPI edge<br/>/chat/storefront, /chat/portal, /cart,<br/>/checkout, /orders, /portal, /evidence"]

    subgraph RT ["SHARED AGENT RUNTIME (cartisan_agent)"]
        direction LR
        LOOP["Agent loop<br/>prompt assembly, streaming,<br/>skills, memory"]
        GATE["Gates and fences<br/>identity, sanitization,<br/>server issued references,<br/>limit checks"]
    end

    subgraph AG ["AGENTS"]
        direction LR
        SA["Shopping agent<br/>search_products, get_product_details,<br/>add_to_cart, stage_checkout,<br/>get_order_status"]
        MA["Merchant agent<br/>get_business_snapshot, read_metrics,<br/>get_inventory_alerts, stage_price_update,<br/>stage_promotion, stage_campaign"]
    end

    subgraph CORE ["COMMERCE CORE (deterministic)"]
        direction LR
        CART["Carts and inventory<br/>reservations"]
        CO["Checkout stages<br/>and orders"]
        PAY["Payments<br/>attempts and webhook"]
        CHG["Merchant changes<br/>maker checker"]
        EV["Evidence, metrics,<br/>recovery"]
    end

    DB[("Supabase / Postgres<br/>single source of truth")]
    RZP{{"Razorpay<br/>payment links and webhooks"}}
    HUMAN(["Human confirmation<br/>required to commit"])

    CLIENT --> API --> RT --> AG
    AG -->|read and stage only| CORE
    CORE --> DB
    PAY <-->|verified events only| RZP
    CO -.->|must pass through| HUMAN
    CHG -.->|must pass through| HUMAN
    HUMAN ==>|commits| CORE

    style CLIENT fill:#eef4ff,stroke:#3b6fd4,color:#000
    style RT fill:#fff8e6,stroke:#b8860b,color:#000
    style AG fill:#eaf7ee,stroke:#2e7d4f,color:#000
    style CORE fill:#f4eefc,stroke:#6b4bb8,color:#000
    style DB fill:#e8f6f6,stroke:#2b7f7f,stroke-width:2px,color:#000
    style RZP fill:#e8f6f6,stroke:#2b7f7f,stroke-width:2px,color:#000
    style HUMAN fill:#fff0f0,stroke:#c25353,stroke-width:3px,color:#000
```

The source diagram is in [`architecture.excalidraw`](architecture.excalidraw). Open it at [excalidraw.com](https://excalidraw.com) to read the full annotated version.

---

## The payment path, precisely

This is the part that has to be exactly right, so it is worth stating on its own.

1. The agent stages a checkout. Nothing is charged and no order exists yet.
2. A human confirms. Only now is a single order created.
3. A payment attempt is created against that order. A retry creates a **new attempt on the same order**, never a second order.
4. The order becomes `PAID` only when a verified Razorpay webhook event says so. Not when the agent says so, and not when the browser redirect says so.
5. Provider events land in a deduplicated inbox, so a replayed or duplicated webhook cannot double apply.
6. Anything stuck is visible and replayable from the operations console.

---

## Repository layout

```
cartisan/
├── frontend/                  Next.js app
│   ├── app/
│   │   ├── storefront/        Customer shopping chat
│   │   ├── portal/            Merchant console and approvals
│   │   ├── evidence/          Per journey audit trail
│   │   └── operations/        Health, payments, recovery
│   ├── components/            UI and presentation components
│   └── lib/                   API client, stores, formatting
│
└── backend/                   Python service
    ├── api/                   FastAPI edge and request lineage
    ├── cartisan_agent/        Shared runtime: loop, prompts,
    │                          gates, fences, presentations
    │   ├── skills/            Shopping skills
    │   └── merchant_skills/   Merchant skills
    ├── shopping_agent/        Shopping agent wiring
    ├── merchant_agent/        Merchant agent wiring
    ├── commerce_common/       Prompt assembly, streaming,
    │                          grounding, memory, MCP
    ├── marketplace_backend/   Deterministic commerce core:
    │                          carts, checkout, payments,
    │                          inventory, evidence, recovery
    ├── supabase/              Schema and migrations
    └── tests/                 Test suite
```

---

## Skills

Skills are procedures loaded only when the conversation actually needs them, which keeps the base prompt small and the agent fast.

**Shopping:** `compatibility-check`, `build-a-setup`, `checkout-and-payment`, `order-and-payment-help`

**Merchant:** `restock-decision`, `price-change`, `campaign-review`, `daily-briefing`

---

## Running it locally

**Backend**

```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # fill in Supabase, Anthropic, and Razorpay keys
uvicorn api.main:app --reload
```

**Frontend**

```bash
cd frontend
npm install

cat > .env.local <<'ENV'
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_SUPABASE_URL=your-supabase-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-supabase-anon-key
ENV

npm run dev
```

The storefront is at `/storefront`, the merchant portal at `/portal`, the evidence viewer at `/evidence`, and the operations console at `/operations`.

---

<div align="center">

Cartisan lets an agent be genuinely useful in commerce without ever letting it be the thing that moves the money.

</div>
