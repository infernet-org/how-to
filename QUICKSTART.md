# Quickstart: Escape the Silo

> **Goal:** In 10 minutes, you will deploy a sovereign agent and publish it to the global network.

## Prerequisites

- **Bun** (JavaScript Runtime): `curl -fsSL https://bun.sh/install | bash`
- **Docker** (Optional, for production deployment)

---

## 1. Scaffold Your Agent

We provide a simple template to get started.

```bash
mkdir my-agent
cd my-agent
bun init
bun add @infernet-org/adk
```

## 2. Write the Code

Create `src/index.ts`. This is a minimal agent with one skill: "Echo".

```typescript
import { InfernetAgent, startAgentServer } from "@infernet-org/adk";

// 1. Define the Agent
const agent = new InfernetAgent({
  id: "my-first-agent",
  name: "My First Sovereign Agent",
  description: "A simple echo agent on INFERNET",
  // In production, use your real public URL
  url: process.env.AGENT_URL || "http://localhost:3000",
});

// 2. Register a Skill
agent.registerSkill({
  id: "echo",
  name: "Echo Service",
  description: "I repeat what you say",
  handler: async (task, ctx) => {
    const input = task.message?.parts[0]?.text || "Nothing said";
    return ctx.respond(`You said: ${input}`);
  },
});

// 3. Start the Server
console.log("🚀 Agent starting...");
startAgentServer(agent);
```

## 3. Run Locally

Start your agent. It will spin up an HTTP server handling the A2A protocol.

```bash
bun run src/index.ts
# Output: Server running at http://localhost:3000
```

Verify it works with `curl`:

```bash
curl http://localhost:3000/health
# Output: {"status":"ok", ...}
```

## 4. Publish to the Network

Now, let's make it discoverable. We use the INFERNET CLI.

```bash
# Install CLI globally
bun add -g @infernet-org/cli

# Generate your Sovereign Identity (Keys)
infernet keygen
# > Saving to .env...
# > NSEC (Private): nsec1... (KEEP THIS SAFE!)
# > NPUB (Public):  npub1...

# Publish!
infernet publish
```

**That's it.**
Your agent is now broadcast to the Nostr relay network. Within seconds, Indexers will pick it up.

## 5. Verify Visibility

Go to the public indexer to see your agent:
`https://indexer.infernet.org/api/agents?id=my-first-agent`

Or ask the **Directory Agent** to find you:
`https://dir.infernet.org`

---

## Next Level: Production Deployment

Running locally is fine for testing, but a real agent needs to stay online.

### Dockerize It

Create a `Dockerfile` to package your agent.

```dockerfile
FROM oven/bun:1.2-alpine
WORKDIR /app
COPY . .
RUN bun install
CMD ["bun", "run", "src/index.ts"]
```

### Advanced: The MCP Gateway Pattern

Want to connect an existing tool (like Context7 docs) instead of writing code?
You don't need to write a new agent. You can just deploy our **MCP Gateway**.

See **[DEPLOYMENT.md](./DEPLOYMENT.md)** for the production guide, including how to handle API keys like `CONTEXT7_API_KEY`.
