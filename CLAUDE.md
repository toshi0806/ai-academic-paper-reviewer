# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is **AI Academic Paper Reviewer** - an AI-powered academic paper review bot that uses Google's Gemini AI to provide educational feedback on student papers. It's designed as a GitHub Action that automatically reviews pull requests containing academic papers (LaTeX/Markdown) when they're opened, synchronized, or reopened.

## Key Commands

### Build
```bash
npm run build
```
Compiles TypeScript source files to JavaScript in the `dist/` directory.

### Local Testing
```bash
npm run exec
```
Runs the bot locally for testing. Requires a `.env` file with the necessary environment variables.

### Install Dependencies
```bash
npm install
```

## Architecture

### Core Components

1. **Entry Points**
   - `src/index.ts`: Production entry point used by GitHub Actions
   - `src/local.ts`: Local development entry point with dotenv configuration

2. **Main Utilities (`src/utils/index.ts`)**
   - **Review Generation**: 
     - `generateReviewCommentObject()`: Generates structured reviews with line-specific comments (code review mode)
     - `generateAcademicReviewObject()`: Generates academic paper reviews with educational feedback
     - `generateReviewCommentText()`: Generates simple text-based reviews
     - `generateAcademicReviewText()`: Generates academic text reviews
   - **Prompt Generation**:
     - `createReviewPrompt()`: Creates prompts for code reviews
     - `createAcademicReviewPrompt()`: Creates prompts for academic paper reviews
   - **GitHub Integration**:
     - `fetchPullRequest()`: Retrieves PR metadata
     - `fetchPullRequestFiles()`: Gets changed files in PR
     - `filterFiles()`: Applies exclude patterns to filter out files
   - **Diff Processing**:
     - `parseFiles()`: Parses Git patches into structured format
     - `createParsedDiffText()`: Converts parsed diffs to readable format with line numbers
   - **AI Integration**:
     - Uses Vercel AI SDK with Google's Gemini models
     - Implements retry logic for handling `NoObjectGeneratedError`
     - Falls back to text generation if structured generation fails

### Key Design Patterns

1. **Dependency Injection**: The main `runReviewBotVercelAI()` function accepts generation and posting functions as parameters, enabling dry-run mode for testing.

2. **Structured Output with Zod**: Uses Zod schemas to ensure AI generates reviews in the correct format with path, line number, body, and priority.

3. **Error Handling**: Implements exponential backoff retry for AI generation failures with configurable temperature adjustment.

4. **Line Number Mapping**: Maintains accurate line number tracking in diffs to ensure comments are placed correctly on GitHub.

## Environment Variables

Required for local development (in `.env`):
- `GITHUB_TOKEN`: GitHub personal access token
- `GITHUB_OWNER`: Repository owner
- `GITHUB_REPO`: Repository name
- `GITHUB_PR_NUMBER`: Pull request number to review
- `GOOGLE_GENERATIVE_AI_API_KEY`: Gemini API key
- `LANGUAGE`: Review comment language (default: "English")
- `MODEL_CODE`: Gemini model to use (default: "models/gemini-2.5-flash-preview-04-17")
- `EXCLUDE_PATHS`: Comma-separated patterns to exclude from review
- `REVIEW_MODE`: Review mode - "ACADEMIC" for academic papers, "CODE" for code reviews (default: "CODE")

## GitHub Action Configuration

The action expects these inputs:
- `GEMINI_API_KEY`: Required
- `GITHUB_TOKEN`: Required
- `MODEL_CODE`: Optional (default: "models/gemini-2.0-flash-exp")
- `LANGUAGE`: Optional (default: "English")
- `EXCLUDE_PATHS`: Optional
- `REVIEW_MODE`: Optional ("ACADEMIC" for academic papers, "CODE" for code reviews, default: "CODE")

The action automatically sets up the Node environment, installs dependencies, and runs the compiled JavaScript from the `dist/` directory.

## Academic Paper Review Features

When `REVIEW_MODE` is set to "ACADEMIC", the bot provides:
- Educational feedback from an engineering faculty perspective
- Review categories: Academic Accuracy, Structure, Language Quality, Novelty, Format
- Priority levels: CRITICAL, IMPORTANT, SUGGESTION, GOOD_POINT
- Support for LaTeX and Markdown files
- Language quality assessment for both Japanese and English papers