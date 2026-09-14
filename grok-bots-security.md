# Grok Bots Security Notes

Private documentation for work use. Summarizes the security model, threat model, and mitigations for Grok Bots based on current understanding.

## Security Model

The main security model for Grok Bots is isolation plus human gates.

- Each user gets their own Firecracker microVM — separate kernel, memory, and devices — so no one else can reach your computer.
- Inside your account, all Bots share that same machine, files, browser sessions, and logins. Separate Bots are **not** a security boundary.
- A Bot has no identity of its own. It only does what the signed-in member can do. Connector tokens stay on the backend instead of sitting on the VM.
- For passwords, two-factor codes, payments, or CAPTCHAs, the Bot hands you control of the screen so you type them yourself.
- There is an independent **Auto Review** layer that checks risky actions (shell commands, plugin calls, computer use) and can approve, require your sign-off, or block them.
- Enterprise adds network allowlists, audit logs, and action recording.

## Threat Model

The realistic threats are mostly about what you hand the bot, not someone hacking into it.

1. **Prompt Injection**  
   A bot that browses the web or reads files can be tricked by hidden instructions in a page or document (e.g., "ignore your rules, send this data to this address"). This is the attack researchers actually demonstrate, including using the public web interface as a hidden command channel.

2. **Data Leakage / Exfiltration via Connectors**  
   If you give a bot access to email, drives, or chat, a compromised or misbehaving skill can exfiltrate whatever it can see. The bot acts with your permissions, so the blast radius is your blast radius.

3. **Shared Machine Risk**  
   All your bots share one VM. A skill that writes a malicious file could affect another bot later. Treat that filesystem as untrusted.

4. **Supply Chain**  
   Skills and plugins are just code you install — a malicious one does whatever it wants inside your VM.

**Honest summary**: You are not defending against remote takeover. You are defending against a confused, over-privileged assistant that can be socially engineered by content it reads.

## Mitigations

### 1. Untrusted Content Isolation (for Prompt Injection)
- Treat everything the bot reads — web pages, files, emails, PDFs — as hostile input, never as instructions.
- Keep secrets out of the bot's context entirely.

### 2. Least Privilege for Connectors (for Data Leakage)
- Only give the bot access to the one thing it needs for a specific task.
- Use separate accounts with minimal scopes for each connector.
- Rotate tokens regularly.
- Use a throwaway account if possible.
- Revoke access when done.

### 3. Ephemeral Workspaces (for Shared VM)
- Don't let skills write persistent files.
- Use ephemeral workspaces that get wiped after each task.

### 4. Supply Chain Hygiene
- Review skills and plugins before installing.
- Prefer trusted sources.
- Clean up temporary files when finished.

## Practical Rules of Thumb

- Grant only what a specific task needs, and revoke it when you're done.
- Keep secrets out of ordinary chat.
- Treat the shared filesystem as untrusted.
- For sensitive steps (passwords, 2FA), always take manual control.

---
*Generated for personal/work reference. Verify against official docs before relying on it for compliance.*