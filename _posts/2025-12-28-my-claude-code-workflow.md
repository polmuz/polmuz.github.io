---
layout: post
title: "My Claude Code Workflows"
date: 2025-12-28
---

Nowadays I exclusively use Claude Code as my agent. I used Copilot, Copilot Chat, and Aider in the past. This guide assumes you're already familiar with Claude Code basics. If not, start with the [official docs](https://docs.anthropic.com/en/docs/claude-code). Here's what my workflows look like.

## Research and small changes

I use the default mode for this. Asking questions about the codebase. Asking it to research some API (pointing it to the official docs helps). Fixing some copy or a small bug that requires one unit test. Beyond that, it becomes hard to get good results.

## Medium-sized changes

This is the most common scenario. Making changes to an endpoint, rearranging part of the UI, fixing bugs that require refactoring. I use Plan Mode (a mode where Claude creates a plan before implementing, cycle modes with Shift+Tab) for this.

1. I give it a detailed prompt that looks like the description of a properly triaged ticket/issue. Something you would tell a colleague if you were offloading this task to them.
2. I review the plan, add clarifications, ask for more implementation details where necessary.
3. I ask it to "review the plan for gaps and inconsistencies". It usually finds a couple of things, often things I hadn't even considered.
4. I usually prompt it one last time with "thoroughly review the plan" to see if it picks up anything important.
5. If I'm happy with it, I let it implement it in "accept all edits" mode (edits are applied without confirmation). You might want to manually accept edits at first since you obviously can't trust this thing, but you'll start trusting it and letting it edit freely soon enough.
6. Once it finishes, I ask it to "review the implementation for gaps and inconsistencies". It always finds multiple things. It seems LLMs are better at reviewing code than generating code.
7. I ask it to give me alternative ways to address each issue it found. I select my preferred solution, tell it to skip the issue, or explain how I want it addressed.
8. I repeat steps 6-7 until I'm satisfied. I usually do a couple of iterations.
9. Commit and push the branch!

## Large changes

When a task is too big for a single session, I need to break it down. I use Plan Mode if what I'm working on is large but well-defined.

1. I use the same initial workflow as a medium-sized task, reviewing and improving the plan.
2. When I'm happy with it, I ask it to split the plan into stages.
3. I ask it to move tasks around as I see fit.
4. I ask it to write the plan to a markdown file.
5. I use a new session to implement each stage as a medium-sized change.

If it's not well-defined or it requires architectural decisions, I use [OpenSpec](https://openspec.dev/) (a spec-driven framework for planning code changes through specification documents) instead.

1. I generate a proposal with the corresponding design document, specs, and tasks.
2. I iterate on the proposal, make the design decisions, review the specs, and verify the tasks.
3. I ask it to "review the proposal for gaps and inconsistencies". It always finds things.
4. Once I'm happy with it, I accept it and move to implementation.
5. I either let it implement the full thing, or implement up to a stage where I want to manually verify (e.g., implement Tasks 1.x to 3.x, then pause for manual testing).
6. I handle each implementation stage like a medium-sized change, with the review cycles and committing.

## Skills

I create [skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) for things I do often that become annoying.

I have a skill to address PR feedback. It uses the GitHub CLI to pull all the PR comments. It has interactive and bulk modes. It makes addressing comments so much easier. I don't have to go back and forth between the PR, the IDE, and the terminal.

In interactive mode:
1. It shows me each comment's content, with the code context and a link.
2. It gives me multiple choices for how to address it (or I can write my own).
3. It implements the fix, commits, and replies with a short description of how it was addressed and the commit hash.
4. It moves to the next comment.

In bulk mode, it writes a markdown document with all comments. I mark my choices, and it processes them all at once.

I have a "thorough code review" skill with a list of things I want it to look for and what the workflow should look like. It has similar interactive and bulk modes to the PR feedback skill.

## Tools for the agent

To make all of this work well, you need to set up the agent properly. Having a decent CLAUDE.md file is important for the agent to be effective. The automatically generated one is a good start. It should include how to use tools to generate good and correct code.

Your CLAUDE.md should include:
- How to run tests and linters
- Anything that would help any engineer produce consistent quality code
- Anything that would help verify the agent's work is correct
- Workarounds for things the agent repeatedly fails at (e.g., how to run a specific build, how to start the correct emulator, how to run tests in a specific way, how to create common seed data)

## Context management

Another thing to keep in mind is [context management](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents). Agents are getting better at this, but it's something that I keep an eye on. Sometimes if I'm close to the auto-compact threshold (when Claude automatically summarizes the conversation to free up context space), I will ask it to dump the session's relevant context and progress to a markdown file, check that nothing important is missing, and start a new session. I recently added a rule to my CLAUDE.md: tests and builds should use Tasks (which run in a separate context) when detailed output isn't needed. This keeps the main context clean.

## Decay

Depending on when you are reading this, you might want to ignore some or all the things I mentioned. LLMs and agents are improving at such a rapid pace that I expect most of these things will be unnecessary or different in six months.
