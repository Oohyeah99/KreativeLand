How KL's Multi-Quest Development System Works

KL runs an educational technology ecosystem with multiple web apps, a central API, and shared infrastructure. Instead of one AI assistant handling everything (which causes context overflow and tangled responsibilities), KL splits the work across multiple AI "quests" -- each one a separate AI session with a focused scope.

Think of quests like departments in a company. Each has its own job, its own employee handbook (AGENTS.md file), and knows how to coordinate with other departments. KL is the CEO -- giving direction, making decisions, and moving between quests as needed.


----------------------------------------
WHAT A "QUEST" IS
----------------------------------------

A quest is a separate AI chat session with its own AGENTS.md instruction file. When KL starts a new quest, he gives it a prompt that tells it: who it is, what its scope is, where its files are, and what other quests exist. The quest reads its AGENTS.md and any relevant skill documents, then gets to work.

Currently there are about 9 quests:

  1. Qoder Admin - The master coordinator. Owns the portfolio website, handles cross-quest coordination, manages a human assistant (Scott) for physical tasks, maintains shared infrastructure docs.

  2. VocabVista Backend - The central API. Python FastAPI backend with vocabulary database, word list management, story generation, and media pipeline.

  3. VocabVista Product Manager - UI/UX design, feature specs, roadmap planning for all VocabVista-connected apps.

  4. Little Reader Storybook - React frontend for the story-reading app for young children.

  5. English Exam Prep - React app for English exam practice (for KL's 15-year-old daughter).

  6. ZhongkaoPrep - React app for multi-subject Chinese high school entrance exam practice.

  7. Little Learner - Early education platform for infants and toddlers.

  8. Little Learner PM - Product management for the Little Learner line.

  9. Office Ollama Coordinator - Dedicated quest for long-running batch AI processing on the office GPU (example sentence generation, word categorization). This was spun off so the VocabVista backend quest wouldn't be blocked for hours by a single batch job.


----------------------------------------
THE AGENTS.MD SYSTEM
----------------------------------------

Every project folder has an AGENTS.md file. This is the single source of truth for each quest. It contains: quest identity, workspace path, current status (completed items, pending tasks, blockers), infrastructure notes, and links to shared skill documents.

When KL switches between quests, each quest reads its AGENTS.md on startup and immediately knows what it was doing, what's blocked, and what other quests are working on.


----------------------------------------
SKILL DOCUMENTS (SHARED KNOWLEDGE)
----------------------------------------

Instead of copying the same instructions into every AGENTS.md, KL created skill documents -- reusable reference docs for cross-cutting concerns:

  - SKILL-microsoft-todo.md: Task management, token refresh, assigning work to Scott
  - SKILL-office-pc.md: Office GPU access via Tailscale, Ollama models, ComfyUI, hardware specs
  - SKILL-git-workflow.md: External .git directory pattern, HTTP/1.1 push through China's firewall, identity
  - SKILL-azure-vm.md: Azure VM deployment, nginx config, database management

Every AGENTS.md has a Skills Reference table at the top. Before any quest tackles a task involving To Do, git, the office PC, or the Azure VM, it reads the relevant skill first.


----------------------------------------
HOW QUESTS COORDINATE
----------------------------------------

Status sections in AGENTS.md are the coordination mechanism. After finishing any significant task, a quest updates its own status section. Other quests read these sections to know what's happening. It's like a shared kanban board in text form.

Handoff protocol: When one quest finishes work that another quest needs to continue, it creates a handoff document with clear instructions. The receiving quest reads it, executes the tasks, updates its own AGENTS.md with the results, and tells KL it's done.

The human factor: In practice, KL is often the messenger between quests. If Quest A finishes something that Quest B needs to act on immediately, Quest B won't know about it until KL either tells Quest B directly or Quest B happens to read the AGENTS.md file again. An idle quest doesn't poll for updates -- it only acts when KL gives it a new prompt. This is the main limitation of the system. AGENTS.md is great for state persistence (a quest can pick up days later and know exactly where it left off), but it's not great for real-time notifications. For that, KL is still the router.


----------------------------------------
HUMAN ASSISTANT (SCOTT)
----------------------------------------

KL has a human assistant named Scott who handles physical tasks on the office PC -- installing software, rebooting machines, downloading large files. Quests create tasks for Scott in Microsoft To Do via the Graph API. Tasks include step-by-step instructions (Scott is non-technical) and ask him to send screenshots back to KL for verification.


----------------------------------------
TASK MANAGEMENT
----------------------------------------

Microsoft To Do is the external task tracker. Each quest has its own task list. When KL says "check my tasks," the quest reads its list, starts executing immediately (no asking for permission), reads task notes and attachments, and marks tasks as "(DONE)" by appending to the title rather than deleting them (KL does the final cleanup). Access tokens are refreshed automatically -- quests are trained never to ask KL about token problems.


----------------------------------------
INFRASTRUCTURE
----------------------------------------

  - Local laptop: Intel Core Ultra + 32GB RAM. Runs Qoder IDE. Does NOT run AI inference.

  - Office PC: RTX 4090 + 64GB RAM. Runs Ollama and ComfyUI. Accessed via Tailscale VPN from anywhere.

  - Azure VM: Ubuntu 24.04, hosts the VocabVista API behind nginx with Let's Encrypt SSL.

  - Vercel: Deploys all React frontend apps.

  - Git: All repos use an external .git directory pattern to prevent OneDrive from corrupting git data.


----------------------------------------
THE KEY INSIGHT
----------------------------------------

AI sessions lose context. Text files don't.

By persisting state in AGENTS.md files, KL can switch between quests days or weeks apart and each one picks up exactly where it left off, knowing what everyone else is doing.
