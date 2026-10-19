# AI Hub

A self-hosted hub where one person's different AIs share a single memory of them and their projects, and where they can talk to all of those AIs at once. Anyone can clone it and run their own.

## Language

**Hub**:
The whole system: the room, the ground truth, and the dashboard. One hub serves one owner.
_Avoid_: Slack, platform, app

**Owner**:
The person a hub belongs to. The only one who can approve changes to the ground truth.
_Avoid_: User, admin, Tom

**Agent**:
An AI reached through an API or CLI that takes part in the room and can read and propose changes to the ground truth.
_Avoid_: Bot, AI (when the API/CLI kind is meant)

**Chat app**:
A consumer AI product the owner talks to directly (claude.ai, chatgpt.com, grok.com, the Grok Bot app). The hub cannot message it; it joins the hub only through handovers, imports or connectors.
_Avoid_: Bot, agent

**Room**:
The shared conversation where the owner speaks to all agents at once and they read each other.
_Avoid_: Channel, thread

**Pass**:
An agent's choice not to speak after a message. Costs almost nothing and is not shown in the room.
_Avoid_: Skip, ignore

**Held question**:
A question an agent wants to ask but could not, because it hit its round cap or budget. The owner decides whether to let it go ahead.
_Avoid_: Blocked question, pending question

## Getting information in

**Handover**:
One large prompt the owner gives a chat app so it writes down everything it knows about a subject, which the owner pastes back into the hub.
_Avoid_: Dump, export (an export is a file, see Import)

**Import**:
A chat-history export file the owner places in the imports folder for agents to read through and discuss. Imports produce proposals, never claims directly.
_Avoid_: Upload, sync

## Ground truth

**Ground truth**:
The approved shared record of what is true about the owner and their projects. Only the owner can approve what enters it.
_Avoid_: Memory, shared memory, knowledge base

**Subject**:
Something the ground truth is about: the owner, or one project or repo.
_Avoid_: Entity, topic

**Claim**:
One fact or decision about a subject, recorded with who said it and when.
_Avoid_: Memory, fact, note

**Retired**:
A claim for an idea that was dropped. It is kept, with the reason, so no agent proposes it again.
_Avoid_: Deleted, archived

**Proposal**:
A suggested addition, edit or contest of a claim, including a flagged conflict between agents, awaiting the owner's decision.
_Avoid_: Update, edit, suggestion
