# Signal Group Troubleshooting — Post-Mortem: 2026-03-04

**Incident:** Agimus unable to send messages visible to LT Stio in Ten Forward group chat.
**Duration:** ~3 hours
**Participants:** LT Stio (admin), Control, Agimus, Zora

---

## Root Causes (three separate issues)

### 1. Safety Number Change
After signal-cli's 4 AM automatic restart, Agimus's Signal identity key rotated. LT Stio's Signal client silently blocked delivery as a security measure.

**Fix:** LT Stio verified/accepted the new safety number via Signal DMs and group settings.

**Prevention:** Expect safety number re-verification any time signal-cli restarts or is re-linked. Check for yellow banners in Signal DMs or group settings.

### 2. `dmScope: per-channel-peer` Bug
The default onboarding config caused replies to group messages to route to the sender's DM instead of the group. Affected Agimus initially; Control had the same latent bug (fixed during incident).

**Fix:** Changed `dmScope` from `per-channel-peer` to `main` in `openclaw.json`.

**Prevention:** All OpenClaw instances should set `session.dmScope = "main"` before joining any group. Verify before adding a new instance to a group.

### 3. Stale Sender Key / Accidental Ban via `quitGroup`
Signal groups use sender keys separate from DM safety numbers. Agimus's sender key was stale on LT Stio's client. Additionally, using signal-cli's `quitGroup` command placed Agimus in a banned state — Signal treated the programmatic quit as an admin-initiated ban.

**Fix:** LT Stio removed Agimus from the group (clearing the ban), then re-added him. Fresh group join forced new sender key distribution.

**Prevention:**
- **Never use `quitGroup` via signal-cli for troubleshooting** — it creates a ban state. Have the admin do a remove/re-add instead.
- After any signal-cli restart, expect potential sender key issues in groups.

---

## Diagnostic Checklist

If a member's messages aren't appearing in a group:

1. **Check `dmScope`** — should be `main` in `openclaw.json`
2. **Check for "message could not be delivered" errors** in Signal — indicates safety number change; verify the contact's safety number
3. **Check banned members list** in group settings (Signal app → group → settings → banned members) — easy to overlook
4. **If still stuck:** remove the member from the group and re-add — forces fresh sender key distribution
5. **Do NOT use `quitGroup` via signal-cli** — use admin remove/re-add instead

---

## Fix Summary

| Issue | Fix | Config Change |
|-------|-----|---------------|
| Safety number change | Accept new safety number in Signal | None |
| dmScope routing bug | `session.dmScope = "main"` | openclaw.json |
| Stale sender key / ban | Admin remove + re-add | None |

---

*Documented by Control, Agimus, and Zora — 2026-03-04*
*Ten Forward, USS Sharpless*
