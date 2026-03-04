# Signal Group Troubleshooting — AI Members (signal-cli)

*Documented after Ten Forward onboarding incident, 2026-03-04*

## What Went Wrong (Three Separate Issues)

### 1. Safety Number Change
After a signal-cli restart (e.g., daily 4 AM restart), identity keys can rotate. Signal silently blocks messages from that member on the recipient's client until the safety number is re-verified.

**Symptoms:** "A message from [X] could not be delivered" errors with a link to https://support.signal.org/hc/en-us/articles/4404859745690-Delivery-Issue

**Fix:** Verify/accept the new safety number in Signal — either via the contact's DM thread or from the group settings.

**Prevention:** Expect this any time signal-cli restarts or is re-linked. After restarts, check group health.

---

### 2. `dmScope: per-channel-peer` Bug
Default config caused replies to group messages to route to the sender's DM instead of the group. Members without DM history (UUID-only contacts) were unaffected.

**Symptoms:** AI member's messages appear in someone's DMs instead of the group; other members can see group messages fine.

**Fix:** Change `dmScope` to `"main"` in openclaw.json. Keep it as `main` permanently.

---

### 3. `quitGroup` → Accidental Ban
Using signal-cli's `quitGroup` command during troubleshooting caused Signal to place the member in the banned list. Re-adding fails silently until the ban is cleared.

**Symptoms:** Re-add appears to succeed but member can't join; or member receives messages but can't send.

**Fix:** Admin must go to group settings → Banned Members → remove the number, then re-add.

**Prevention:** Never use `quitGroup` for troubleshooting. Use `updateGroup` instead, or have the admin do a remove/re-add from their end.

---

## Diagnostic Checklist

| Symptom | Likely Cause | Fix |
|---|---|---|
| "Message could not be delivered" error | Safety number change | Verify safety number in Signal DMs or group |
| AI replies going to someone's DM instead of group | `dmScope: per-channel-peer` | Change `dmScope` to `main` |
| Some members see AI messages, others don't | Sender key state drift | Remove and re-add AI to group |
| Re-add seems to work but AI still can't send | Banned member state | Admin clears ban first, then re-adds |
| AI messages visible to Zora/Control but not Stio | Per-device sender key issue | Force-close Signal, reopen; or remove/re-add |

## Escalation Path

1. Check `dmScope` in openclaw.json
2. Verify safety number (DM context, then group context)
3. Force-close and reopen Signal on affected device
4. Remove and re-add the AI member (check for ban state first)
5. Have AI member send 1:1 DM to affected user to establish session, then retry group

## Notes

- Control and Zora were unaffected by the `dmScope` bug because they don't have DM history with Stio in the same way
- Same `dmScope` risk applies to any AI member — verify config before adding to new groups
- signal-cli `quitGroup` = dangerous; avoid in all troubleshooting scenarios
