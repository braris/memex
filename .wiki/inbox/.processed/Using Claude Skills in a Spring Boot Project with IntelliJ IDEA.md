
# [Using Claude Skills in a Spring Boot Project with IntelliJ IDEA](https://senoritadeveloper.medium.com/using-claude-skills-in-a-spring-boot-project-with-intellij-idea-5a98003a121d)

*A practical, step-by-step guide to structuring backend development with reusable AI skills and real-world Spring Boot workflows*

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*H1FFr4D-QCzRb51p)

Photo by Customerbox on Unsplash

### Install Claude Code with Homebrew

Install [Homebrew](https://brew.sh/) on your Mac if you haven’t already:

```c
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After Homebrew installation, run the following command:

```c
brew install --cask claude-code
```

Verify your Claude installation:

```c
claude --version
```

To run Claude:

```c
claude
```

Now follow the browser prompts for authentication.

### Install Claude Code Plugin to Your IntelliJ IDEA

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*AiiOuhDVduMGsWYbwU1FYQ.png)

It provides features such as the following:

- **Diff viewing:** Open file diffs in the IDE diff viewer for reviewing and modifying proposed changes
- **Selection context:** The current selection in the IDE is automatically shared with Claude Code

### Create Your Project

First, we start with creating our Spring Boot project. For this, click on File > New Project. Select Generators > Spring Boot. You can also create by visiting [https://start.spring.io/](https://start.spring.io/), if you are more accustomed to that.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*g1bf9Jl_-NNo6D3VqtEamA.png)

Click “Next” after filling out.

I plan to develop a simple Notes application so I chose the following dependencies from the list:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*skWpYZ4HrFkWAdzHI75_4g.png)

Click “Create”.

Allow IntelliJ IDEA to open the recently created project. Now we are ready to go.

### Initialize Claude Code for Your Project

Now click on Claude Code plugin icon. It will open a Claude session in a terminal:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*KgbIIFEcn5mH1_zGpmWqcg.png)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*uH80F79f8DgVG8nWXReRuA.png)

You don’t have to run “ */init* ” or create a “ *CLAUDE.md* ” file for Skills to work. What you’ve already set up with “*.claude/skills/* ” will be fully sufficient.

- (`.claude/skills/...`) → Skills — Behavior modules
- `CLAUDE.md` → Project-level instructions

CLAUDE.md is a single file with global guidance. Best for always-on rules (to reduce repeated prompting even further), like:

- project context — “This project uses Java 21 and Spring Boot 3”
- high-level constraints — “We follow layered + modulith architecture”
- “how we work in this repo” — “Never expose entities directly in APIs”

### Adding Skills To Your Project

Don’t rely on a single repository. Instead, compose your own skill set inside your project, reflecting your team’s actual architecture, coding standards, and constraints.

To start with

- Use the official Anthropic repo for structure and correctness
- Add dr-jskill for Spring Boot conventions
- Bring in claude-code-java for general backend discipline
- Use everything-claude-code for workflows and testing mindset

Each folder can come from a different source and you can refine them over time:

```c
.claude/skills/ 
  spring-boot-core/ ← from jdubois/dr-jskill
  java-best-practices/ ← from decebals/claude-code-java
  spring-boot-testing/ ← from everything-claude-code
  architecture/ ← from alirezarezvani/claude-skills
  custom-project-rules/ ← your own
```

**1\. Anthropic Official Skills** *(Baseline)*  
anthropics/skills [https://github.com/anthropics/skills](https://github.com/anthropics/skills)

This is the official reference implementation for Claude Skills:

- Real production-grade SKILL.md examples
- Complex multi-step skills (document creation, workflows, etc.)
- Patterns for structuring instructions and triggers

It doesn’t focus on Spring Boot specifically, but it teaches you how Skills are *supposed* to be structured and written.

Study the structure of `SKILL.md` files and copy formatting patterns (especially the YAML frontmatter).

**2\. dr-jskill** *(Spring Boot–Focused)  
*jdubois/dr-jskill [https://github.com/jdubois/dr-jskill](https://github.com/jdubois/dr-jskill)

Created by Julien Dubois (known for JHipster), this repo is one of the few that directly targets Spring Boot workflows. It encodes real-world best practices for building production-grade applications.

Generates:

- Spring Boot 4.x projects
- PostgreSQL setup
- production-ready structure

It is the most “plug-and-play” Spring Boot skill available right now so drop it into your project like:

```c
.claude/skills/spring-boot-generator/
```

Now write the following prompt to Claude:

```c
“Generate a Spring Boot module following dr-jskill patterns.”
```

**3\. claude-code-java** *(Java Patterns)  
*decebals/claude-code-java [https://github.com/decebals/claude-code-java](https://github.com/decebals/claude-code-java)

This is your “core backend skill pack”. This repo fills in the broader Java ecosystem (after adding *dr-jskill*) by complementing Spring Boot skills with general Java discipline — something many repos overlook.

It includes reusable skills for:  
\- architecture decisions  
\- clean code practices  
\- testing approaches

Extract relevant skills:

```c
java-architecture
spring-patterns
testing-guidelines
```

Then place into (or keep as is):

```c
.claude/skills/java-*
```

**4\. everything-claude-code** *(Real-world Workflows)  
*affaan-m/everything-claude-code [https://github.com/affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)

“Spring Boot + Engineering discipline baked into Claude” — this repository helps enforce *how* you build software. It teaches Claude to behave more like a disciplined engineer rather than just a code generator. It includes Spring Boot-specific skills (rare).

It includes skills around:  
\- security practices  
\- TDD  
\- architecture  
\- verification workflows

**5\. awesome-claude-skills** *(Your Discovery Hub)*  
travisvn/awesome-claude-skills [https://github.com/travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)

It’s a curated list of many different skill repositories (as an ecosystem map). As the Skills ecosystem grows, you’ll want a way to discover new patterns and ideas (to expand your skillset). This repo helps you do exactly that. Skills are modular folders with instructions and resources.

Helps you discover:

- backend skills
- API design
- testing workflows
- dev tooling

**6\. claude-skills** *(Massive Production-Ready Skill Library)* — Bonus  
alirezarezvani/claude-skills [https://github.com/alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills)

“Enterprise upgrade pack” — This is one of the most comprehensive skill collections available right now. You get a *library of proven patterns* that you can mix into your Spring Boot workflow. Pick relevant ones.

- 170+ skills across multiple domains
- Architecture patterns
- Development workflows
- CI/CD guidance

**7\. Spring AI Agent Utils** *(Highly Relevant for Modern Backends)* — Bonus  
spring-ai-community/spring-ai-agent-utils [https://github.com/spring-ai-community/spring-ai-agent-utils](https://github.com/spring-ai-community/spring-ai-agent-utils)

If your Spring Boot applications are moving toward AI or agent-based capabilities, this repo becomes extremely relevant. It bridges the Spring ecosystem with agent-style interactions

- Build AI-powered features inside Spring Boot
- Want backend services that cooperate with agents
- Are exploring autonomous or semi-autonomous workflows

**8.Spring Modulith–Focused Skills** — Bonus  
sivaprasadreddy/sivalabs-agent-skills [https://github.com/sivaprasadreddy/sivalabs-agent-skills](https://github.com/sivaprasadreddy/sivalabs-agent-skills)

For details about Spring Modulith, you can visit [my blog post](https://senoritadeveloper.medium.com/modular-monolith-with-spring-boot-spring-modulith-6687c234daab).

If you’re working with Spring Modulith or modular monolith architecture, this repository includes guidance for:

- REST API development with Spring MVC
- Modular monolith patterns using Spring Modulith
- Focused testing strategies and tooling

You can copy the relevant skill folder into:

```c
.claude/skills/spring-boot/
```

### Installing Claude Skills in Your Spring Boot Project

Now we’ve found a few good skill repositories, the next step is actually getting them into your project.

Claude needs to run inside your project directory to detect your skills. Once it starts, it will automatically scan:

```c
.claude/skills/
```

Navigate to your project root folder in your terminal in IntelliJ Idea (View → Tool Windows → Terminal).

```c
mkdir -p .claude/skills
```

Pause implementing changes for now. I will be giving some extra information.

To add a skill (manual way), copy the relevant skill folder from the repository and paste into your project such as:

```c
.claude/skills/spring-boot-standards/SKILL.md
```

You know, the key file is always “ *SKILL.md* ”. It contains:

- metadata (name, description)
- instructions
- rules and best practices

Once it’s there, Claude can discover and use it automatically.

Some repositories support installing skills via a CLI command so you can add a skill with the CLI Shortcut:

```c
npx skills add <repo-url>
```

More specifically:

```c
npx skills add https://github.com/... --skill spring-boot
```

Prefer the manual approach to:

- see exactly what you’re adding
- to be able to tweak it immediately

### Each folder = one skill (or one responsibility)

Organizing your skills matters more than you think. As you add more skills, things can get messy quickly if you don’t organize them.

Don’t just dump 20 skills into your project at once. Start with:

- 1 Spring Boot skill
- 1 testing skill
- 1 general Java skill

Then grow gradually, because Claude performs better when:

- instructions are focused
- there’s less overlap
- intent is clearer

A clean structure might look like this:

```c
.claude/skills/
  spring-boot-core/
  spring-boot-testing/
  java-best-practices/
  architecture/
  ci-cd/
```

This helps Claude:

- understand context better
- pick the right skill automatically
- avoid conflicting instructions

Now, let’s continue to make changes to the project.

Create the following folders and create a “SKILL.md” file inside each:

```c
.claude/skills/
  spring-boot-core/
  java-best-practices/
  persistence-jpa/
  flyway-migrations/
  spring-security/
  spring-boot-testing/
```
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*iUEbn_vXLAAI9tyHnIdfqw.png)

I cloned all the repositories to a folder and opened in IntelliJ IDEA in a new window:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*euGfTYANontqf5iGiFXz8g.png)

From “jdubois/dr-jskill” repo, use the following files’ content selectively to fill your SKILL.md file under “spring-boot-core” and “persistence-jpa”:

```c
SKILL.md
AGENTS.md
references/
```

From “decebals/claude-code-java” repo, use the SKILL.md files under the following folders selectively to fill your SKILL.md file under “java-best-practices”, “spring-boot-testing” and “persistence-jpa”:

```c
.claude/skills/
  java-code-review/
  api-contract-review/
  concurrency-review/
  performance-smell-detection/
  test-quality/
  security-audit/
  maven-dependency-audit/
```

From “affaan-m/everything-claude-code” repo, you can use the files under the following folders selectively to fill your SKILL.md file under “spring-security” and “flyway-migrations”:

```c
skills/
  api-design
  security-review
  springboot-security
  database-migrations
  postgres-patterns
```

### Use the Skills — Prompts

***Phase 1 — Project foundation:***

Prompt 1 — Project structure

```c
Create a Spring Boot Notes application.

Requirements:
- Java 17+
- Use Spring Boot best practices
- Follow layered architecture (controller, service, repository)
- Use REST APIs
- Keep the project clean and modular

Explain the structure briefly.
```

Prompt 2 — Domain model

```c
Design a Note entity for a Notes application.

Requirements:
- id (Long)
- title (String)
- content (String)
- createdAt (timestamp)
- updatedAt (timestamp)

Follow JPA best practices.
Use proper annotations and types.
```

***Phase 2 — Persistence + migrations:***

Prompt 3 — Flyway migration

```c
Create a Flyway migration for the Note entity.

Requirements:
- Use PostgreSQL
- Follow safe migration practices
- Use proper data types
- Include timestamps
- Avoid unsafe operations

Name the migration properly (V1__...).
```

Prompt 4 — Repository

```c
Create a JPA repository for Note.

Requirements:
- Use Spring Data JPA
- Add basic CRUD support
- Add a method to search notes by title (case-insensitive)
```

***Phase 3 — Business logic:***

Prompt 5 — Service layer

```c
Create a NoteService.

Requirements:
- create note
- get all notes
- get note by id
- update note
- delete note

Follow clean service design.
Handle edge cases (not found, validation).
```

***Phase 4 — API layer:***

Prompt 6 — Controller

```c
Create a REST controller for Note.

Requirements:
- CRUD endpoints
- Proper HTTP status codes
- Validation support
- Clean request/response structure

Do not expose entity directly if not recommended.
```

***Phase 5 — Security:***

Prompt 7 — Basic security setup

```c
Add Spring Security to the Notes application.

Requirements:
- Stateless authentication (basic or JWT for simplicity)
- Protect all endpoints except a health endpoint
- Use proper HTTP status codes (401, 403)
- Configure security filter chain explicitly
```

Prompt 8 — Authorization rules

```c
Enhance security:

Requirements:
- Only authenticated users can access notes
- Prepare structure for user ownership (even if not fully implemented)
- Use method-level security where appropriate
```

***Phase 6 — Testing:***

Prompt 9 — Unit tests

```c
Write unit tests for NoteService.

Requirements:
- Use JUnit 5 and AssertJ
- Mock dependencies
- Cover:
  - create
  - get by id
  - update
  - delete
- Test edge cases
```

Prompt 10 — Slice tests

```c
Write controller tests using @WebMvcTest.

Requirements:
- Test API endpoints
- Mock service layer
- Validate responses and status codes
```

Prompt 11 — Integration test

```c
Write an integration test using @SpringBootTest.

Requirements:
- Test full flow (create → fetch → update → delete)
- Use Testcontainers for PostgreSQL
```

***Phase 7 — Improvements:***

Prompt 12 — Refactor

```c
Review and refactor the Notes application.

Focus on:
- code clarity
- duplication
- best practices
- clean architecture
```

Prompt 13 — Security review

```c
Perform a security review of the Notes application.

Check for:
- missing validation
- improper error handling
- security misconfigurations
- sensitive data exposure
```

Prompt 14 — Performance & DB

```c
Review persistence and database design.

Focus on:
- indexes
- query performance
- JPA usage
- potential bottlenecks
```

Once we write the prompt and hit Enter, it shows the used skills:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*-5j3gym2tLUFeE1QRt8neA.png)

I allowed all edits for the session. You can also set:

```c
claude --dangerously-skip-permissions
```

After running all of the prompts one by one in order, I tried to run the tests. There were failing ones so I wrote a simple prompt such as “run all the tests and fix failing”.

***Tip:*** Use /btw to ask a quick side question without interrupting Claude’s current work.

Voilà! The final project version is available at my GitHub:

## [GitHub - senoritadeveloper01/claude-skills: A Spring Boot Notes Project built entirely with Claude…](https://github.com/senoritadeveloper01/claude-skills?source=post_page-----5a98003a121d---------------------------------------)

