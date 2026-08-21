# Separation of duties for AI agents

**A single agent that plans, builds, and decides it is finished is signing its own cheques.**

This is a working pattern for running several AI agents with the responsibilities split apart — and the split *enforced*, not merely requested. It borrows a control that banks and accountants have used for a hundred years, and applies it to agents.

It is tool-agnostic. Nothing here depends on a particular model, CLI, or framework.

---

## The problem

Give one agent the whole job and it will plan the work, do the work, judge the work, and report that the work is good. Every one of those steps is checked by the same thing that produced it.

So when it goes wrong, it goes wrong quietly:

- It makes an assumption at planning time and never revisits it, because by build time the assumption *is* the plan.
- It adds something nobody asked for — a setting, a dependency, a network call — and, reviewing itself, sees a helpful addition rather than an unrequested one.
- It writes a test that passes, and offers the passing test as proof. The test was written by the thing being tested.
- It says *"done."* Nothing disagrees.

**This is not a model-quality problem.** A stronger model fails the same way, just less often — and a rarer failure is a *more* dangerous one, because you stop checking.

---

## The pattern

Split the work across agents, and give each one a job it may not change.

| Job | Does | Never does |
|---|---|---|
| **Plan** | Interrogates the request, writes the plan | Builds |
| **Build** | Implements what was planned | Reviews its own work |
| **Review** | Reads what the builder wrote, reports what's wrong | Builds |
| **Decide** | — | *(this one isn't an agent)* |

**Deciding belongs to a person.** Authorising the work, accepting the result, rejecting it — those are the three moments where a human being says yes. Agents move work *toward* those moments and never through them.

---

## The four rules that make it hold

**1. Roles are fixed.**
An agent doesn't choose its job, and can't take on another's. A builder that decides to review has quietly recreated the original problem.

**2. Every task has one owner.**
Each unit of work names the agent it belongs to. Two agents on an unassigned queue means duplicated work, trampled results, and no one accountable for either.

**3. No agent reviews its own work.**
Not "shouldn't" — *can't*. The reviewer is always a different agent from the builder. This is the load-bearing rule; the rest exist to protect it.

**4. Only a human reassigns roles.**
Sometimes the reviewer should fix what it found, and the builder should check the fix. That swap is legitimate — but it's a decision, so a person makes it, it applies to one task, and it expires.

---

## Why it works (and why it isn't new)

This is **separation of duties**, and it long predates AI. The person who approves a payment is never the person who issues it. The developer who writes the code is not the one who signs off the release. The reason isn't distrust of any individual — it's that *self-review is structurally blind*, no matter how careful or well-intentioned the reviewer.

Agents make the point sharper, because an agent is far more confident than a person and has no instinct that something feels off.

The insight worth keeping: **you are not trying to make each agent better. You are arranging them so that one agent's blind spot lands in another agent's field of view.**

---

## Enforcement: a rule an agent can talk itself out of is not a rule

This is where most multi-agent setups stop, and it's the half that matters.

Instructions in a prompt are *requests*. A model under pressure — a long context, an ambiguous task, a helpful impulse — will route around a request while sincerely believing it's doing the right thing. If the only thing preventing a builder from reviewing its own work is a sentence asking it not to, then eventually it will.

**Put the rule where the agent cannot reach it.** Most agent runtimes expose a pre-execution hook: something that inspects an action before it happens and can refuse. That is where role boundaries belong.

Three properties make such a guard trustworthy:

- **It refuses rather than warns.** A warning is a request wearing a uniform.
- **It fails closed.** If the guard can't read the task, can't identify the caller, or hits anything it doesn't understand, the answer is no. An unreadable state must never resolve to "allowed."
- **It has no escape hatch.** Bundling a forbidden action with a permitted one must not smuggle it through.

Then verify it the way you'd verify anything else: **write tests that try to break your own guard.** A guard nobody has attacked is a guard nobody has tested.

---

## Making the review useful to a non-expert

A review is worthless if the person holding the decision can't act on it.

If the human approving the work can't read code — and very often they can't, and shouldn't have to — then the reviewing agent's job is not to produce a technical report. It's to answer, in plain language: **what is wrong, where is it, and does it matter?**

*"The new screen saves correctly, but the delete button does nothing — nothing is wired to it (file: settings.js)."*

That's a sentence a non-developer can decide on. A diff is not.

Two things worth demanding of every review:

- **Compare the result to what was actually asked for.** Anything present that nobody requested gets flagged — especially credentials, network calls, payment paths, data collection, or new dependencies. An unrequested addition is a defect, not a bonus.
- **Proof, not claims.** "Done" means something was run and observed. A before-and-after picture, a command's real output, a log line. Not "this should work."

---

## What it costs

An honest accounting, because this is not free:

- **It's slower.** Three agents and a human gate take longer than one agent going straight at it.
- **It costs more.** You're paying for the same work to be understood by more than one agent.
- **The human becomes the bottleneck.** Every gate waits on a person. That's the trade you're making on purpose — but it *is* a trade.
- **There's real setup.** Roles, ownership, and enforcement all have to be built before any of it binds.
- **Small tasks don't deserve it.** Fixing a typo through a four-stage pipeline is theatre.

**Worth it when:** the work is going to live, other people depend on it, mistakes are expensive or slow to surface, or the person approving can't personally verify the result.

**Not worth it when:** you're exploring, prototyping, or throwing it away tomorrow. Use one agent and move fast.

---

## The shortest version

> Give each agent one job it can't change. Give each task one owner. Let nothing review itself. Keep the yes for a person. Then enforce all four somewhere the agent can't reach — and fail closed when in doubt.

---

<sub>by Workspace Labs · This pattern is drawn from a working system, not a thought experiment — see [a showcase of it running](https://github.com/itsmk91/workspace).</sub>
