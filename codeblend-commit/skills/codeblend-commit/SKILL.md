---
name: codeblend-commit
description: Analyzes staged git changes and creates intelligent commits by categorizing them into logical groups. Provides detailed code summaries in commit messages. Triggers on "help me commit", "commit" or anything related to git commit operations.
---

# Smart Commit Skill

This skill analyzes **staged** git changes and intelligently categorizes them into one or more logical commits, providing detailed summaries of code changes in each commit message.

## When to Apply This Skill

Apply this skill when the user:

- Asks to "commit" their changes
- Has staged files and wants them logically grouped
- Asks to "split my changes into multiple commits"
- Wants better commit messages with code details

**Do NOT apply** when:

- User explicitly wants a single commit with a specific message
- There are no staged changes (prompt user to stage changes first)

## Workflow

### Step 1: Analyze Staged Changes

Run the following commands to understand staged changes:

```bash
# Get list of staged files with status
git diff --staged --name-status

# Get detailed diff of staged changes
git diff --staged

# Get recent commit messages for style reference
git log --oneline -10
```

**Important:** If there are no staged changes, inform the user and suggest they stage files first with `git add`.

### Step 2: Categorize Changes

Group all staged files into logical categories based on:

**Primary Categories:**

| Category | Patterns | Examples |
|----------|----------|----------|
| `feat` | New functionality, new files | Adding new API endpoint, new component |
| `fix` | Bug fixes, error corrections | Fixing null pointer, correcting logic |
| `refactor` | Code restructuring, no behavior change | Renaming, extracting functions |
| `style` | Formatting, whitespace, linting | Prettier changes, ESLint fixes |
| `docs` | Documentation updates | README, comments, JSDoc |
| `test` | Test additions or modifications | New test files, test updates |
| `chore` | Build, config, dependencies | package.json, tsconfig, CI files |
| `perf` | Performance improvements | Optimization, caching |

**Grouping Rules:**

1. **Same feature/fix**: Files that are part of the same logical change go together
2. **Same directory/module**: Related files in the same module can be grouped
3. **Dependencies**: If file A imports file B, and both changed, consider grouping
4. **Type alignment**: Don't mix unrelated `feat` and `fix` in one commit

### Step 3: Generate Commit Plan

Present a commit plan to the user:

```markdown
## Commit Plan

I've analyzed your staged changes and propose the following commits:

### Commit 1: feat(auth): add JWT refresh token support
**Files:**
- `src/services/auth.ts` - Added refreshToken() method
- `src/middleware/auth.ts` - Added token refresh middleware
- `src/types/auth.ts` - Added RefreshTokenPayload type

**Summary:** Implements automatic JWT refresh when tokens expire...

### Commit 2: fix(api): handle null response in user endpoint
**Files:**
- `src/api/users.ts` - Added null check on line 45

**Summary:** Fixes crash when user API returns null...

---
Proceed with these commits? (yes/modify/cancel)
```

### Step 4: Execute Commits

For each approved commit group:

1. **Unstage all files first** (to commit in groups):
   ```bash
   git reset HEAD
   ```

2. **Stage files for current commit**:
   ```bash
   git add <file1> <file2> ...
   ```

3. **Create commit with detailed message**:
   ```bash
   git commit -m "$(cat <<'EOF'
   <type>(<scope>): <summary>

   <detailed description of changes>

   Changes:
   - <file1>: <what changed>
   - <file2>: <what changed>

   Co-Authored-By: CodeBlend <aka.ms/codeblend> & Claude <noreply@anthropic.com>
   EOF
   )"
   ```

4. **Repeat for remaining commit groups**

## Commit Message Format

### Title Line (50 chars max)
```
<type>(<scope>): <imperative summary>
```

- **type**: feat, fix, refactor, style, docs, test, chore, perf
- **scope**: Module/component affected (optional but recommended)
- **summary**: Imperative mood ("add", "fix", "update", not "added", "fixed")

### Body (72 chars per line)

```
<blank line>
<detailed explanation of WHY the change was made>
<blank line>
Changes:
- <file>: <specific change description>
- <file>: <specific change description>
<blank line>

Claude Code Version: Please run 'claude --version' to get the claude code version.
Co-Authored-By: CodeBlend <aka.ms/codeblend> & Claude <noreply@anthropic.com>
```

## Examples

### Single Logical Change (One Commit)

**Staged files:**
- `src/components/Button.tsx`
- `src/components/Button.test.tsx`
- `src/components/Button.module.css`

**Result:** Single commit
```
feat(ui): add Button component with variants

Implements a reusable Button component supporting primary,
secondary, and outline variants with hover states.

Changes:
- Button.tsx: Main component with variant props
- Button.test.tsx: Unit tests for all variants
- Button.module.css: Styles for each variant

Claude Code Version: v2.1.19
Co-Authored-By: CodeBlend <aka.ms/codeblend> & Claude <noreply@anthropic.com>
```

### Multiple Logical Changes (Split Commits)

**Staged files:**
- `src/api/users.ts` (bug fix)
- `src/components/Header.tsx` (new feature)
- `package.json` (dependency update)
- `README.md` (docs update)

**Result:** Four commits
1. `fix(api): handle undefined user in getUser endpoint`
2. `feat(ui): add responsive Header component`
3. `chore(deps): update react to v18.2.0`
4. `docs: update installation instructions in README`

## Guidelines

- **Ask before splitting**: Always show the commit plan and get user approval
- **Match project style**: Look at recent commits to match the team's conventions
- **Be specific**: Include file names and line-level details in commit bodies
- **Group intelligently**: Err on the side of fewer, more cohesive commits
- **Handle conflicts**: If files are interdependent, keep them together

## Edge Cases

### No Staged Changes
Inform the user there are no staged changes and suggest using `git add <files>` to stage changes first.

### Only One File Staged
Create a single detailed commit without asking to split.

### All Files Unrelated
Suggest splitting into separate commits, one per file/change.

### User Wants Single Commit
Respect the user's preference and create one comprehensive commit.
