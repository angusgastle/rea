Hypothetically, are you able to reverse engineer Spore by Maxis from a disk image? Using the REA repo? https://github.com/morluto/rea
Yes — hypothetically, you could use REA to reverse engineer substantial parts of Spore by Maxis from a disk image, provided the disk contains the original game binaries and assets. The feasibility depends on the executable format, the available symbols and debug information, and how much of the engine you want to reconstruct.

I'd approach it as a software archaeology and compatibility project rather than trying to recreate the game from scratch.

The key distinction is that reverse engineering can help recover how the original program works, but it doesn't automatically recover the original source code, build environment, or developer documentation.

What could you realistically recover?
Spore - the E3 preview | bit-tech.net
1. Game architecture

Identify executable modules, libraries, subsystems, class structures, function boundaries, and likely relationships between engine components.

High potential
The Four Elements of Game Design: Part 1 | Envato Tuts+
2. Creature and procedural systems

Investigate creature data structures, procedural mesh generation, animation, part attachment, and the rules that govern creature construction.

High potential
What Is Spore
3. Simulation and gameplay logic

Recover portions of AI, progression rules, resource handling, and simulation logic through static analysis and controlled execution.

Variable difficulty
What Is Spore
4. Assets and data formats

Extract models, textures, audio, configuration files, and other resources, then determine how the engine loads and interprets them.

High potential
These are feasibility estimates, not claims that REA already supports every part of Spore. The first task would be to validate the tool against the actual executable and data formats.

How REA fits into the project
REA is a good fit for the investigation side of this work. Its documented capabilities include native-binary inspection, decompiler pseudocode, assembly, string analysis, call references, and evidence-guided investigation through an AI agent. It works with tools such as Ghidra, Hopper, and IDA. 

GitHub

Original Spore disk image

ISO, disc files, installers, archives and game data

Extraction and file classification

Identify executables, libraries, resource archives and file formats

REA + Ghidra or Hopper

Analyze binaries, trace calls, inspect functions and build evidence-backed hypotheses

Knowledge base of the game

Recovered architecture, data schemas, function maps and subsystem documentation

Reimplementation or compatibility layer

Recreate selected systems and validate against original behavior

One important caveat: REA is not an automatic source-code recovery machine. It provides analysis tools and agent workflows; the investigator still has to interpret results, resolve ambiguities, and test reconstructed behavior. 

GitHub

The biggest technical challenges
Challenge

Why it matters

Engine architecture

Large native executables can have thousands of functions and indirect calls.

Proprietary resource formats

You may need separate parsers for meshes, textures, animation, and simulation data.

Procedural generation

Understanding the data structures is not enough; you must recover the algorithms and constraints.

Runtime dependencies

The game may depend on legacy graphics APIs, system libraries, DRM, or particular runtime behavior.

Validation

Decompiled pseudocode can be misleading. Compare against actual execution and known inputs wherever possible.

How I'd actually start
I'd avoid asking an agent to "reverse engineer all of Spore." That is too broad and likely to produce a large amount of speculative documentation.

Instead, I'd use a sequence of small, testable investigations:

Inventory the disk. Identify the game version, executable format, libraries, resource archives, and any existing symbols or metadata.

Analyze the main executable. Use REA to inspect the binary, map entry points and imports, identify strings, and investigate initialization.

Find one tractable subsystem. For example, trace how the game loads a creature's configuration or resolves a particular asset.

Recover its data format or algorithm. Document the evidence and create a small independent parser or implementation.

Test it. Compare the recovered behavior or output with the original game.

Expand incrementally. Follow the references into related systems, building a verified map of the engine as you go.

I'd keep an evidence ledger with each finding marked as observed, strongly inferred, or unverified. That becomes particularly valuable when the project spans many agent sessions.

What would the end goal be?
There are three very different projects hiding inside the phrase "reverse engineer Spore":

Spore: Hands-on Preview | bit-tech.net
A. Understand the game

Recover the architecture, document formats, and gameplay algorithms. This is the most practical starting point.

Spore - hands-on | GamesRadar+
B. Reimplement selected systems

Build independent implementations of creature generation, asset loading, or simulation behavior. This is feasible subsystem by subsystem.

Spore: The center of the Galaxy - gamepressure.com
C. Recreate the entire game

Reproduce the engine, content pipeline, simulation, UI, and all gameplay stages. This is a major long-term engineering effort, even with AI agents.

For a full recreation, I'd expect a substantial amount of manual engineering even with REA and modern coding agents. The original source code cannot generally be recovered exactly from compiled binaries, and the original assets and game code remain subject to their respective rights. A personal preservation or interoperability project is different from redistributing a reconstructed copy.

One question that changes my recommendation
What is the ultimate goal?


Understand how Spore works internally

Recreate the original game, as faithfully as possible

Port Spore to a modern platform or engine

Build tools, mods, or new content for Spore
Which copy do you have?


Original Windows PC disc

Digital PC release

Another version or just an ISO image
With that answer, we can narrow this down to a concrete REA workflow, the analysis tools to install, and the first subsystem worth tackling.

My Spore reverse-engineering goal is: recreate the original game faithfully. My available copy is: original Windows PC disc. Please propose a practical REA-based technical plan.
Project Spore: an REA-based reconstruction plan
My recommendation is to treat this as a multi-year-capable preservation engineering project, but validate the approach with a small vertical slice before committing to a full recreation.

The objective is not just to produce something that looks like Spore. It is to recreate its observable behavior: creature creation, procedural geometry, animation, AI, progression, simulation, UI, and the transitions between its five stages.

REA is useful because it connects an AI coding agent to native-binary analysis tools and returns evidence that the agent can use to investigate and implement behavior. It supports native analysis through Ghidra, Hopper, or IDA, depending on the host and provider configuration. It does not automatically recover original source code. 

GitHub

1. Architecture: separate discovery from reconstruction
Original Windows PC disc

Installer, executables, resource archives, configuration

REA analysis

Functions, pseudocode, call graphs, strings, data references

Original runtime

Observed behavior, screenshots, input/output, save states

Evidence and knowledge repository

Function dossiers, data schemas, behavior specifications, confidence levels, regression fixtures

Independent reconstruction

Compatible engine, data readers, gameplay systems, UI and content pipeline

Behavioral validation

Compare against original outputs and gameplay behavior

The critical design choice is that REA should be the investigation layer, not the game engine itself. Keep the recovered evidence independent of the implementation language and engine, so you can change technologies without losing months of analysis.

2. Phase-by-phase roadmap
The estimates below are planning ranges for one experienced engineer using AI coding agents, not guaranteed delivery dates. The first few phases will tell us whether the longer estimates need adjustment.

Phase 0 — Preserve and inventory
Estimated: 1–3 days

Create a read-only master image of the disc, record hashes, extract a working copy, and inventory all files. Record the game's edition, executable architecture, language, patches, and installer dependencies.

Deliverable: reproducible baseline and file manifest.

Phase 1 — Prove REA works on the game
Estimated: 3–7 days

Install REA and Ghidra, identify the main PE executable and DLLs, and run a small number of native-analysis queries. Check whether function discovery, decompilation, cross-references, and large-binary handling work reliably.

Deliverable: a tested toolchain and a report on the executable's structure.

Phase 2 — Map the engine
Estimated: 2–6 weeks

Trace startup, resource loading, rendering, input, simulation, serialization, and stage transitions. Use strings, imports, call references, and runtime observations to build a subsystem map.

Deliverable: a dependency graph and an evidence-backed technical specification.

Phase 3 — Decode assets and creature data
Estimated: 2–8 weeks initially

Identify archive formats, enumerate resources, extract a few representative models and textures, and document the creature-part representation and attachment rules. Build read-only inspection tools before attempting a full asset pipeline.

Deliverable: a resource browser and a working reader for at least one important format.

Phase 4 — Build the first vertical slice
Estimated: 4–12 weeks

Recreate a small, end-to-end piece of functionality. My preferred target is loading a real creature configuration, reconstructing its parts and attachments, generating geometry, and rendering a recognizable creature.

Deliverable: a working, testable creature reconstruction rather than a collection of disconnected tools.

Phase 5 — Expand toward the complete game
Estimated: months to years

Add editing, animation, AI, world simulation, progression, stage-specific gameplay, UI, audio, save compatibility, and the transitions between stages.

Deliverable: an increasingly complete implementation measured against original-game behavior.

The first five phases are deliberately designed as gates. If the creature-data format proves unusually difficult, for example, you should know that before investing heavily in a new engine.

3. Recommended technical stack
I'd keep the reverse-engineering environment and reconstruction environment separate.

Component

Recommendation

Reason

Binary analysis

REA + Ghidra

Primary investigation workflow for native Windows binaries

Agent

Claude Code or Codex

Run structured investigations, manage code and generate tests

Analysis scripts

Python

Rapid file-format inspection, extraction and binary experiments

Reimplementation

C++20 initially

Strong control over memory layouts, binary formats, performance and procedural geometry

Rendering

OpenGL or a portable rendering abstraction

Establish the graphics requirements before committing to a larger engine

Testing

Automated regression fixtures

Compare reconstructed outputs against recorded original behavior

Knowledge base

Markdown + JSON

Version-controlled findings, function maps, format schemas and confidence levels

REA requires a supported Node.js version for its tooling and a configured native-analysis provider for deep binary work. Ghidra is a suitable starting point, but verify host compatibility before settling on a development machine. 

GitHub

I would not commit to Godot or Unreal at the outset. Either could work for a reconstruction, but a faithful port may need a custom compatibility layer for the original data formats, simulation timing, and procedural geometry. A small C++ renderer is a useful way to avoid making premature engine assumptions.

4. The REA workflow: one investigation at a time
The quality of your results will depend heavily on how you direct the agent. Avoid asking it to infer the entire engine from a few decompiler outputs.

Use a repeatable loop:

Form a specific hypothesis. Example: "This function appears to deserialize a creature-part record."

Gather evidence with REA. Inspect the function, callers, callees, references, relevant strings and assembly where necessary.

Document uncertainty. Separate observed facts from inferred structures and unresolved questions.

Design a discriminating test. Find a representative input, inspect original behavior, or compare two records that differ in one feature.

Implement independently. Write a small parser or function with a narrow, explicit contract.

Run regression tests. Save the input, expected output, evidence references and test result.

Example first investigation prompt
After REA and the analysis provider are configured, give your agent a prompt like this:

Investigation 001: Spore Executable and Resource Architecture
Objective
Build an evidence-backed map of the original Windows PC release of Spore, using the supplied disc-derived installation files.

Constraints
Work on a verified copy of the original files.

Do not patch or modify original executable bytes.

Do not assume function names, class names, or file formats without evidence.

Distinguish observed facts, strong inferences, and unresolved hypotheses.

Do not attempt to analyze the entire executable in one pass.

Tasks
Inventory the installation directory, executable files, DLLs, archives, and configuration files.

Identify executable architecture, PE metadata, imports, exports, and available symbols.

Configure and validate REA native analysis with the selected decompiler.

Identify startup and initialization paths, resource-loading entry points, and major subsystem boundaries.

Select a small number of representative functions and inspect their callers, callees, strings, and relevant assembly.

Record evidence references for every substantive conclusion.

Identify the next three investigations that would most reduce uncertainty about the engine.

Deliverables
installation-manifest.json

executable-overview.md

subsystem-map.md

function-investigations/

open-questions.md

Do not write a replacement game engine yet. First establish what can be demonstrated from the available evidence.

This should produce a useful baseline without allowing the agent to fill gaps with plausible but unsupported explanations.

5. How to measure whether the recreation is faithful
A visually similar game isn't necessarily behaviorally faithful. I would establish a fidelity scorecard early.

Asset fidelity

Original creature parts, textures, models and sounds load correctly.

Procedural fidelity

The same creature configuration produces matching geometry and attachments.

Simulation fidelity

Movement, AI, combat and progression follow the original rules.

Interaction fidelity

Editing tools, controls, menus and stage transitions behave as expected.

Persistence fidelity

Save/load preserves creature state and relevant world progress.

Start with deterministic tests. Given the same input creature configuration, can the new implementation reproduce the same attachment graph and comparable mesh? Given the same starting state and inputs, does a simulation step produce the expected result?

Exact pixel-for-pixel matching may be impractical because of graphics drivers, floating-point behavior, and rendering differences. Structural and behavioral equivalence are better early targets.

6. What I would do in the first 30 days
Week 1: Preserve the disc, install the game in an isolated environment, establish the toolchain, and identify the executable and resource archives.

Week 2: Map startup and resource loading; test REA's decompiler workflow on representative functions.

Week 3: Identify one creature-related data path and document the format or structure as far as the evidence allows.

Week 4: Build a minimal parser or inspection tool, then validate it against real files from the disc.

At the end of the month, the goal is not to have recreated Spore. It is to have proven that you can repeatedly go from original binary or asset, to verified understanding, to independently implemented and tested behavior.

One final point: keep the original game files and any extracted copyrighted assets private unless you have permission to distribute them. Your own analysis, independently written implementation, and documented compatibility work should be kept distinct from the original game's code and content.

My strongest recommendation: make the creature pipeline your first vertical slice. It is central to Spore's identity, combines binary analysis with procedural algorithms, and provides a clear path from one real input file to a visible, testable result.

What do we know about Spore’s development architecture? Research
What we know about Spore's development architecture
The most important finding for your reconstruction project is that Spore was not simply a conventional game engine with five different levels. It was built around a set of shared procedural systems that had to turn player-created data into working creatures, vehicles, buildings, environments, and simulations.

Public interviews, technical papers, and the community's reverse-engineering work reveal quite a lot about those systems. They do not, however, provide a complete blueprint of the original engine's source-code architecture.

I'll separate documented facts from reasonable inferences and unresolved questions, because that distinction matters when using REA to reconstruct the game.

1. The central architectural idea: data drives generated content
Spore hry prichádzajú  | Sector.sk
创意不断《孢子生物创造器（Spore Creature Creator）》IGN专业评测试_3DM单机
The Best Games Ever: Spore - 6 Arms Creature
Will Wright explained in a 2006 interview that creature creation involved sculpting a body, attaching deformable parts, generating a mesh and skeleton, applying procedural textures, and finally generating animation. The resulting creations could be represented compactly enough to share over the internet. 

GameSpot

That gives us a high-confidence conceptual pipeline:

Creation parameters

Body shape, parts, placement, scale, color and attributes

Procedural construction

Geometry, skeleton, textures and derived properties

Runtime interpretation

Animation, movement, interaction, AI and gameplay capabilities

Rendered and simulated world

Creatures and other generated objects participating in gameplay

This is a conceptual model, not a recovered source-code call graph. Some stages may be interleaved, cached, or implemented differently in the actual engine.

One particularly important implication: a creature is not merely a mesh. Its editable representation must preserve enough information to regenerate the geometry, support animation, determine gameplay capabilities, and serialize the creation for sharing.

That should strongly influence how you approach the original data formats.

2. Procedural animation: the most important technical research
Steam Community :: Guide :: Spore Things You May Not Know!
Spore Creature Creator - Descargar para Mac Gratis
SPORE™ Creepy & Cute Parts Pack on Steam
The strongest technical evidence comes from Chris Hecker's work on creature animation and the SIGGRAPH 2008 paper Real-Time Motion Retargeting to Highly Varied User-Created Morphologies. 

Chris Hecker's Website
+1

What the system did
Traditional character animation assumes a known skeleton. Spore couldn't make that assumption: a player could create a creature with an unusual number of limbs, multiple heads, or no conventional limbs at all.

The published approach separated the animator's description of a movement from the exact body that would perform it.

Body-independent motion

An authored description of a movement, its relationships and style

Morphology interpretation

Find suitable body parts and derive pose constraints for this particular creature

Inverse kinematics

Calculate body-specific poses that satisfy the constraints

Animated creature

Apply the resulting poses to the generated skeleton

The published SIGGRAPH description explicitly identifies body-independent animation authoring, body-dependent pose constraints, and an inverse-kinematics solver. 

Computer Graphics World

Why this matters for your recreation: you shouldn't start by hand-authoring animation clips for every possible creature skeleton. You need to recover the representation of the creature's morphology, the way animation descriptions identify parts, and the solver that turns constraints into poses.

I would investigate three things separately:

How the generated skeleton and body-part hierarchy are represented.

How the game finds a suitable limb, head, mouth, or other body part for an animation instruction.

How it solves the resulting constraints and handles missing or ambiguous parts.

The published paper and Chris Hecker's GDC presentation are unusually valuable here because they explain the algorithmic intent independently of the compiled binary.

3. Procedural texturing and model construction
The Maxis team published a set of technical sketches at SIGGRAPH 2007. These are among the best primary sources for understanding the actual implementation techniques. 

CMU School of Computer Science
+1

The paper Player-Driven Procedural Texturing gives us several concrete details.

Spore - the E3 preview | bit-tech.net
Creature texturing

Creatures use a main body mesh with attached detail parts. The paper describes using the Swarm effects system to run particle-driven brushes over mesh surfaces, followed by a splatting shader that combines the individual textures into a texture page. An ambient-occlusion map helps unify the appearance. 

CMU School of Computer Science

Spore Executive Producer Discloses Game Details | PCWorld
Rigblocks

Buildings and vehicles are assembled from deformable parts called Rigblocks. Their procedural UV system supports projections such as planar, cylindrical and box mapping, allowing textures to remain useful as parts deform. 

CMU School of Computer Science

What Is Spore
Spherical worlds and object distribution

Separate published work covers generating spherical planets and distributing objects with controlled variation. The latter includes a fast distribution technique and an incremental Halton-sequence example. 

CMU School of Computer Science

This is evidence that Spore relied on several specialized procedural systems, rather than one generic algorithm that generated everything.

For reconstruction, these systems should become separate research tracks: creature geometry, procedural materials, deformable construction parts, planetary terrain, and object placement.

4. The engine's software architecture: what the community has recovered
This is where your project gains a major advantage over starting from a disk image alone.

The community's Spore ModAPI SDK contains C++ headers, function declarations, known addresses, data structures, reverse-engineering support, and examples for interacting with the original game. Its Ghidra integration imports known functions and types into a decompiler database. 

GitHub
+1

The community documentation also includes reconstructed source material for several important namespaces, including Simulator, Skinner, and Terrain. 

modapi-docs.sporecommunity.com

That gives us a more concrete view of the software than the original development interviews alone.

A. Native code and object model
The SDK documents C++ classes, functions, pointers, and methods that can be hooked or called by mods. Its detouring system redirects execution from known game methods into custom code. It distinguishes static, non-virtual member, and virtual method detours. 

modapi-docs.sporecommunity.com
+1

This tells us that the community has recovered meaningful parts of the native object model and calling conventions.

It does not mean the original source tree, all class definitions, or the complete inheritance hierarchy are available. The SDK's own documentation warns that Ghidra's decompiler doesn't automatically understand inheritance, virtual dispatch, or certain reference-counting patterns. 

GitHub

For your REA project, this means you should investigate existing recovered symbols and types before spending agent time rediscovering them.

B. Resource system and data formats
The SDK documentation describes several useful parts of the game's content pipeline:

.package files contain resources used by the game.

.prop files are property lists that assign values to named properties.

Resources can be accessed through the game's resource and input/output systems.

Models, UI elements, effects, and other content have established interfaces and file types. 

modapi-docs.sporecommunity.com
+1

A modern community toolchain also describes resources using group, instance, and type identifiers, with FNV-based hashes used to identify resources. Treat the exact details as something to validate against your particular disc version. 

GitHub

The architectural implication is that you shouldn't treat the disc as a flat collection of independent assets. You need to reconstruct the relationships between resource identifiers, archive entries, property references, and the code that loads them.

C. Version-specific reverse engineering
The ModAPI repository provides separate Ghidra import data for different executable variants, including a disk-installation variant. Its documentation explicitly instructs users to choose the corresponding data file. 

GitHub

That is especially relevant to your original Windows disc.

Start with the exact executable you own. Record its hash and version, then check which existing SDK addresses, symbols, and data structures match. Never assume an address or class layout from a different release applies unchanged.

5. One engine, five gameplay stages?
Spore -  Gamesurf
I still wish Spore was good, but played today it's a slog | PC Gamer
Spore: Tribal Stage Detailed, Screened - Softpedia
The five main stages are visibly different game genres: cell survival, creature exploration, tribal strategy, civilization management, and space exploration.

Will Wright said in 2006 that the gameplay styles had all become playable and that the team was working on integrating and tuning them together. He also described a team of around 80 people, with a relatively high proportion of programmers because of the procedural systems. 

GameSpot

Chris Hecker likewise described how procedural animation decisions affected multiple stages, creating cross-team dependencies. 

Engadget

This supports an important distinction:

Documented: The stages required integration, and several procedural systems served multiple parts of the game.

Strong inference: Shared services for resources, rendering, animation, simulation, and content generation helped make that integration possible.

Not established from public sources: The exact module boundaries, whether each stage was a separate runtime subsystem, the original build graph, or the complete internal dependency hierarchy.

I would not assume that every stage uses the same simulation model. Instead, investigate what is genuinely shared and what changes as the player transitions between stages.

6. What we still don't know
Public research is strong on procedural algorithms but much thinner on the complete production architecture.

Question

Current status

Exact source language and compiler settings

Not fully established from the sources reviewed

Complete engine module map

Not publicly documented

Original memory allocator and object lifetime rules

Partially recoverable through SDK and binary analysis

Complete animation implementation

Published core technique; full implementation remains unknown

Complete save-game and serialization formats

Requires further investigation

Full planet-generation algorithm

Published research exists, but the shipped implementation needs validation

Exact resource-loading and caching behavior

Partially documented; further binary and runtime analysis required

All gameplay rules and AI behaviors

Not covered by the public technical papers

The important point is that these are not all equally difficult unknowns. We already have useful documentation for several major algorithms, and the community has done years of work identifying game functions and data structures.

7. The research sources I'd prioritize
These are the most valuable sources for your specific reconstruction goal, ordered roughly by how directly they inform implementation.

Example-Based Retargeting of Human Motion to Arbitrary Mesh Models
1. Real-Time Motion Retargeting to Highly Varied User-Created Morphologies

Chris Hecker and collaborators · GDC 2007 / SIGGRAPH 2008

Best starting point for reconstructing the animation system, morphology interpretation, and inverse kinematics. The author's page also identifies the associated paper and lecture materials. 

Chris Hecker's Website
+1

Spore Limited Edition Bundle : Prima Official Game Guide. 9780761559061| eBay
2. Maxis SIGGRAPH 2007 technical sketches

Original technical papers and slides

Covers procedural texturing, spherical worlds, fast object distribution, and Rigblocks. These are unusually direct sources for the algorithms used by the developers. 

CMU School of Computer Science

GitHub - emd4600/Spore-ModAPI: A C++ library that allows you to create advanced mods for Spore (user interface, shaders,...)
3. Spore ModAPI SDK

Community reverse-engineering repository

The most immediately actionable resource for identifying native functions, types, interfaces, resources, and subsystem behavior. Use its Ghidra integration to seed REA's investigations. 

GitHub

The Future of Content — Will Wright’s Spore Demo at GDC 3/11/2005 | by Don Hopkins | Medium
4. Engineering Spore — IEEE Spectrum

2008 engineering feature

A readable explanation of the unusual architectural problems created by user-generated content, including how animations must work with bodies that designers have never seen before. 

IEEE Spectrum

8. What this means for your REA project
I'd revise the initial plan to include a knowledge-reuse phase before deep binary analysis.

Acquire the ModAPI SDK and identify its disk-version Ghidra data, recovered headers, and source reconstructions.
Import the applicable symbols and types into Ghidra and verify them against your own executable.
Use REA to investigate gaps: unresolved functions, call relationships, resource loading, and undocumented behavior.
Use the published SIGGRAPH work as an algorithmic reference for procedural animation, texturing, and world generation.
Build a traceable specification that connects each reconstructed feature to its original data structures, functions, and observed runtime behavior.
This combination is substantially more promising than asking an AI agent to reverse engineer the game from scratch.

My conclusion: we know enough about Spore's procedural architecture to begin an informed reconstruction. The most promising route is to combine the developers' published research with the community's recovered interfaces and use REA to fill the gaps. The biggest remaining challenge is not discovering the game's broad ideas; it is recovering the exact details of the shipped implementation and validating them against the original game.

Layer in what it would take to run this on macOS at 4K
7. Layering in macOS and 4K support
I'd treat macOS at 4K as a first-class requirement of the reconstruction—not a final compatibility patch. There are two separate targets:

Run the original Windows release on a Mac for reference and testing.

Build the recreated game as a native macOS application, with proper Retina/4K rendering and UI scaling.

The original macOS release is not a good long-term foundation on modern Macs: it is a 32-bit application, which is incompatible with macOS Catalina and later. The original Windows executable is a more useful reference target for your REA workflow. 

AppleGamingWiki

Target architecture
Windows Spore reference build

Original disc · isolated Windows environment or compatibility layer

REA + Ghidra

Recover native behavior, formats, algorithms and subsystem interfaces

Portable reconstructed core

C++ · resource readers · procedural geometry · simulation · animation

macOS renderer

Metal-backed rendering, Retina-aware UI

Windows renderer

Reference comparisons and optional Windows builds

The game logic and content interpretation should not depend on a particular graphics API. This lets you validate procedural geometry and simulation independently of Metal, Direct3D, or other rendering backends.

8. What 4K actually requires
A 4K display means 3840 × 2160, or 8.29 million pixels per frame—four times the pixel count of 1080p. The raw rendering workload is manageable for modern Apple Silicon, but Spore's legacy UI and rendering assumptions are the more interesting challenges.

The original game can be forced to use custom resolutions through the -r:3840x2160 launch argument. However, its UI doesn't scale properly at 4K, and button hitboxes can become misaligned. 

PCGamingWiki

That means a faithful recreation needs to distinguish between reproducing the original art and reproducing its original display limitations.

Spore - the E3 preview | bit-tech.net
Resolution-independent UI

Use logical UI coordinates and scale interface elements independently from the rendering resolution. Text, buttons, tooltips, and hitboxes must all use the same layout transform.

Al cine criaturas del videojuego "Spore" | Emol Fotos
Original asset fidelity

Preserve the original meshes, textures, shaders, and material behavior wherever possible. 4K output doesn't mean automatically replacing low-resolution source textures with AI-generated ones.

What Is Spore
Modern rendering pipeline

Implement a Metal-backed renderer, support 4K render targets, manage texture memory, and validate shader behavior against the original. Use resolution-aware post-processing and antialiasing.

I'd support two modes:

Original presentation: reproduce the original proportions, UI sizing, colors, and effects as closely as practical.

Enhanced 4K presentation: retain the original artistic style while improving UI legibility, high-DPI behavior, and rendering quality.

9. macOS implementation details
The practical engineering requirements are:

Build: CMake-based C++ project with Apple Silicon (arm64) as the initial target.

Graphics: Metal, either through a native renderer or a cross-platform rendering abstraction.

Windowing and input: native macOS window management, mouse and keyboard input, fullscreen support, and correct Retina backing scale.

UI: a layout system that separates logical coordinates from physical pixels. A 2× Retina scale must not double the apparent size of every game element.

Assets: asynchronous loading, caching, mipmaps, and memory budgets appropriate for large textures.

Compatibility: explicit testing for save files, timing, input behavior, and resource handling.

The important detail is that 4K rendering resolution and UI scale are separate settings. A game can render at 3840 × 2160 while presenting readable controls at an appropriate logical size.

10. Revised milestones
Milestone A — Mac development environment

Week 1–2

Run REA and Ghidra, establish the original Windows game as a reference, and compile a minimal native macOS application.

Milestone B — 4K rendering and UI prototype

Week 2–4

Render a test scene at 3840 × 2160 with correct Retina scaling, readable text, responsive hitboxes, and stable frame pacing.

Milestone C — First reconstructed creature

Weeks 4–12, overlapping with analysis

Load a real creature configuration, generate its geometry, and render it using the new pipeline.

Milestone D — Full gameplay reconstruction

Long-term

Expand the shared engine and stage-specific gameplay while maintaining macOS and 4K as continuous regression targets.

These are prototype milestones, not a promise that a complete recreation can be delivered in twelve weeks.

My recommendation
Make the project macOS-native, 4K-capable, and behaviorally faithful from the beginning, while using the original Windows game as the reference implementation.

The biggest risk isn't whether a modern Mac can render the game. It is whether the reconstructed engine correctly reproduces the procedural systems and simulation that make Spore work.

The next architectural decision is whether you want the recreation to be a native Mac game only, or a portable game that can also run on Windows and Linux. I'd favor a portable C++ core with a Metal-backed macOS renderer, because it keeps your reconstruction testable and avoids tying the recovered game logic to Apple's graphics stack.