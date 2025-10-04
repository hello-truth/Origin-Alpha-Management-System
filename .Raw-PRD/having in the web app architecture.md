having in the web app architecture of the modular where the app raw description under @Origin-Alpha-Management-System/.Raw-PRD and mostly taking reference from @E:\\Coolify\\htdocs\\Origin-Alpha-Management-System\\Reference-Project\\Refer-Project-3 {note: for all of its functionality} then after modules for additional helpful functionality that might be needed :

 The modular architecture to be alike following in : "E:\\Coolify\\htdocs\\Caldron Pixer Enovator Project\\Caldron-Pixer-Admin"

"E:\\Coolify\\htdocs\\Caldron Pixer Enovator Project\\Caldron-Pixer-Api"

"E:\\Coolify\\htdocs\\Caldron Pixer Enovator Project\\Caldron-Pixer-Shop"



But in the architecture of the :

### Clients

client that to be hold on caldronflex.com.np and their access functionality \& dashboard for whole usage in the clients.caldronflex.com.np(but the domain and envto be changable in .env) {as already the refer project 3 have its ui \& functionality to used as reusable component} \& its tech stack to be alike :

**Clients Dashboard**

NextJs

Typescript

Tailwindcss

React Hook Form

React-Query



**Frontend**

NextJs

Typescript

Tailwindcss

React Hook Form

React-Query

Next SEO

Next PWA



### Api

api also with architecture alike the caldron pixer-api with graphql with Laravel and the logic of the project-3 with some customization (at last of the project

 in Laravel 12 architecture,



### Super Admin

**superadmin Dashboard**

NextJs

Typescript

Tailwindcss

React Hook Form

React-Query



first of all the files to be analyzed under @Origin-Alpha-Management/.Raw-PRD \& Organize the files and complete the Missing Component(try it to cover from every aspect and perspective ) with available details And by asking,at last

Similarly the techstack to be latest and the agents(both orchestrator agent and sub agent work on  to work on same takand create the pull request(one session for each task):must follow these rules:

Key practices:



File size limits: 300 lines max for components. If it's bigger, break it down.

Domain boundaries: Related functionality stays together. No mixing World logic with Character logic.

PR-based workflow: No direct commits to main branches. Every change gets reviewed.

KISS principle: Simple solutions over clever ones. The codebase should be readable six months later.,try to use the

Testing Principles

The testing approach focuses on behavior over implementation. Test what users actually experience, not how the code works internally.
Core guidelines:

Test WHAT the feature does for users, not HOW the code implements it

Focus on acceptance criteria and core functionality

Avoid testing implementation details like CSS classes or internal state

Use user-centric queries (getByRole, getByText) over test IDs when possible

Code Standards



Single responsibility for components and functions

Domain boundaries must be respected

Type safety is mandatory: no any types

UI Components: Always use shadcn/ui components instead of raw HTML elements:

Design Token Enforcement

The codebase enforces our 23-color design token system through automated linting:



Stylelint: Catches hardcoded hex colors, rgb/hsl values, and non-token color names in CSS

Tailwind Config: Restricts available colors to only our design token palette

Automatic Detection: Violations are caught during development and in CI

To fix color violations:



npm run lint:css        # See what's wrong

npm run lint:css:fix    # Auto-fix simple issues

The linter will flag things like color: #ff0000 or background: rgb(255,0,0) and point youTailwind classes instead.



Documentation Standards

Documentation should sound like explaining something to a colleague, not writing for a corporate wiki.



Context first: Start with why this exists, then what it does


Skip corporate language: No "comprehensive solutions" or "leveraging synergies"

Keep it practical: Focus on implementation over theory

GitHub Workflow

Always link commits to issues

Use semantic commit messages

PR descriptions should reference issues

All tests must pass before merge

Feature branches are created from and merged back to the develop branch

Always use PR template from .github/PULL\_REQUEST\_TEMPLATE.md

Always target the develop branch in PRs, NEVER target main

Domain Boundaries



also the frontend and the clients side to be in different containeer as the 

