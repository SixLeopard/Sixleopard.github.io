---
title: Making Web Apps Entirely in C#
date: 2000-06-20 22:00:00 +1000
categories: [Development, Web]
tags: [ASP.NET Core Blazor, C#, Web Development, Backend, Frontend]
---

# 🚧 UNDER CONSTRUCTION 🚧

Will cover:
- My experience trying out ASP.NET Core Blazor

<!--TL;DR

[3–4 sentences. What you built, how long you used Blazor, and your verdict in one line. Don't bury the lede — readers want to know up front whether you recommend it.]

Stack: [e.g., .NET 9, Blazor Web App with Auto render mode, EF Core, Postgres, MudBlazor]
Project scope: [e.g., internal CRUD app for ~50 users, or public-facing SaaS, or hobby project]

Why I went all-in on C#
[Set up the personal motivation. Why did you reach for Blazor instead of the React/Vue/Svelte default? Pick the threads that are actually true for you:]

One language across the stack — fewer context switches
Strong typing all the way from database row to button click
Existing .NET expertise you didn't want to throw away
Hesitation about JavaScript ecosystem churn
You wanted to see if the hype was real


Be honest: if part of the motivation was "I just don't enjoy JavaScript," say so. Readers respect that more than a fabricated technical justification.


The Blazor hosting models, briefly
[A reader landing here from search may not know the lay of the land. Keep this short — link to Microsoft's docs for the deep dive. Just enough to make the rest of the post make sense.]
ModelHow it runsTrade-offBlazor ServerRenders on the server, syncs DOM diffs over SignalRFast initial load, tiny client; needs a persistent connectionBlazor WebAssemblyApp ships as WASM, runs entirely in the browserWorks offline; large initial download, slower startupBlazor Web App (.NET 8+)Mix-and-match: static SSR, Server, WASM, or Auto per componentBest of both, but more complex mental modelBlazor HybridRuns in a native shell (MAUI, WPF, WinForms)For desktop/mobile, not web
[Mention which one you picked and why — this frames every opinion that follows.]

What I actually built
[A short section grounding the post in a concrete project. Without this, the rest reads as abstract opinion.]

Domain / problem space
Approximate size (pages, components, users, requests/day)
Team size (solo? two people? larger?)
Anything notable about the constraints (offline support, real-time updates, strict latency targets, regulated industry)

[A screenshot here helps a lot, if you can share one.]

What worked well
One language, one type system, end to end
[The headline feature. Make it concrete:]

Sharing DTOs between client and server — no codegen, no schema drift
Refactors crossing the network boundary in a single Rider/VS rename
The same validation attributes running on the form and the API

The component model is genuinely good
[Razor components, parameters, cascading values, render fragments. Pick what stood out to you.]
razor@* Tiny example showing what makes the model click for you *@
<Card Title="@product.Name">
    <Actions>
        <Button OnClick="Edit">Edit</Button>
    </Actions>
    @product.Description
</Card>
Tooling

Hot reload [how well did it actually work for you? Be specific.]
Debugger experience [breakpoints in .razor files just work — or did they not?]
IDE support [Rider vs VS vs VS Code — what was your daily driver?]

The .NET ecosystem under it

EF Core, MediatR, FluentValidation, Serilog — none of this changes because you switched to Blazor
Identity / auth that doesn't require gluing together five npm packages
Background services in the same process

Specific wins from your project
[2–3 bullets specific to what you built. These are the most credible parts of the whole post.]

What was rough
This section is what makes a retrospective worth reading. Don't soften it.
Initial load size (WASM) or persistent connection cost (Server)
[Whichever applies. If WASM: the runtime download, AOT compilation tradeoffs, trimming, the first-paint experience. If Server: SignalR reconnects, the dreaded "Attempting to reconnect" overlay, what happens on flaky mobile networks, scaling SignalR across servers.]
JavaScript interop, when you can't avoid it
You went into this hoping for "no JavaScript." The reality:

File uploads beyond a point
Anything involving the clipboard, geolocation, browser storage, or third-party widgets
Charting libraries [unless you used a Blazor-native one — say which]
[Whatever bit you in your project]


Blazor's JS interop is fine, but every JS module you import is a piece of the dream you gave up.

Render mode debugging in .NET 8+
[If you used the unified Blazor Web App: write about the "which render mode is this component running in right now" mental tax. Prerendering subtleties — components running twice, services behaving differently, OnInitializedAsync firing in places you didn't expect.]
State management
[Pick your honest take:]

Cascading parameters work but get unwieldy
DI-scoped services are the de facto state container — but "scoped" means different things in Server vs WASM
Third-party options (Fluxor, Blazored.LocalStorage) — used any?

Ecosystem maturity

Component libraries: MudBlazor, Radzen, Telerik, Syncfusion — [your honest assessment of the one you used]
Smaller pool of Stack Overflow answers than React
Some npm-equivalent gaps you had to fill yourself

Performance gotchas

ShouldRender overrides you didn't know you needed
Re-rendering parent components nuking child state
Large lists without virtualization
[Anything you measured and fixed — concrete numbers make this section land]


Things I wish someone had told me on day one
[The most valuable section of any retrospective. Make these specific, not generic. Examples to consider:]

"Decide your render mode strategy before you write components, not after"
"If you're using Blazor Server, plan for SignalR scale-out from day one if you'll have more than one node"
"Don't reach for StateHasChanged until you understand why your UI isn't updating — usually it's a parameter binding issue"
"Inject PersistentComponentState if you don't want prerendered components to fetch data twice"
"Treat @key as load-bearing, not optional"
"Pick a component library on day one and commit — switching is painful"


Would I do it again?
[The verdict. Avoid the wishy-washy "it depends" non-answer. Give a clear take, with the caveat conditions.]
I'd reach for Blazor again when:

[Concrete situation 1 — e.g., internal tools, line-of-business apps, .NET shop]
[Concrete situation 2]
[Concrete situation 3]

I'd reach for [React / Next / SvelteKit / whatever] when:

[Concrete situation 1 — e.g., public-facing marketing site where first-paint matters most]
[Concrete situation 2 — e.g., the team doesn't know C#]
[Concrete situation 3 — e.g., you need a specific npm-only library]


Who Blazor is actually for
[The frank summary your reader scrolled looking for.]
✅ Good fit if you:

Already work in .NET
Are building internal tools, dashboards, admin panels, line-of-business apps
Value type safety over ecosystem size
Have a small team that can't afford a separate frontend specialty

❌ Probably not a fit if you:

Need maximum SEO and first-paint speed on public marketing pages
Depend on a specific npm-only library with no Blazor equivalent
Are building primarily for users on slow mobile networks (WASM payload)
Your team has no C# background and no appetite to learn it


What I'd try next
[Forward-looking section. Keeps engaged readers in your orbit.]

AOT compilation properly — [did you try it? bundle size impact?]
[Server-side rendering improvements landing in newer .NET versions]
Pairing Blazor with a real-time backend (SignalR-driven dashboards)
Trying Blazor Hybrid for a desktop port


Closing thoughts
[Pull it together in 2–3 paragraphs. The honest, lived-in summary. Avoid the urge to end on "and you should try it too" — readers will decide based on the body of the post.]
[A good closing move: name the thing Blazor changed about how you think about web development, regardless of whether you keep using it.]
-->