# omni-dev Commit Guidelines

This project follows conventional commit format with specific requirements:

## Commit Format
```
<type>: <description>

[optional body]

[optional footer(s)]
```

## Types
- `feat`: New features or enhancements
- `fix`: Bug fixes
- `docs`: Documentation changes
- `refactor`: Code refactoring without behavior changes
- `chore`: Maintenance tasks, dependency updates
- `test`: Test additions or modifications
- `ci`: CI/CD pipeline changes
- `build`: Build system changes

## Body Guidelines

### When to use concise vs verbose format:

**Use concise format (title only or minimal body) for:**
- Routine dependency updates
- Build configuration changes that don't affect behavior
- Simple bug fixes with obvious solutions
- Code formatting or style changes
- CI timeout adjustments or similar maintenance

**Use verbose format for:**
- New features or significant enhancements
- Breaking changes or API modifications
- Complex bug fixes requiring explanation
- Changes that affect user behavior
- Architectural decisions or refactoring

### Concise format guidelines:
- Focus on the primary change in the title
- One line commit messages are preferred for simple changes
- Secondary changes (e.g., CI adjustments to support main change) can be briefly mentioned
- Avoid explaining routine changes unless unusual
- Don't list obvious consequences of the main change (e.g., updating specific test files when upgrading a test dependency)
- Don't explain what competent engineers can infer (e.g., that flake.lock updates when hackage index changes)
- Don't describe the purpose of routine updates unless the choice is non-obvious

### Verbose format guidelines:
For significant changes, include:
- What was changed and why
- Impact on users or developers
- Any breaking changes or migration notes
- References to issues or tickets

### Obvious follow-on changes (don't document these):

When making certain changes, other files are automatically or necessarily updated. Don't list these obvious consequences:

**Dependency updates typically require:**
- Updating specific package files that use the dependency
- Updating hackage index-state to access newer versions
- Updating flake.lock when index-state changes
- Adjusting CI timeouts for larger dependency trees

**Hackage index updates typically require:**
- Updating flake.lock with new package hashes (both hackageNix node and related changes)
- Potential dependency version adjustments

Note: If index-state for hackage is updated, it implies updating the hackageNix node in flake.lock, so it is not necessary to mention it as a separate change. It is okay to mention it as part of the same change.

When unpinning dependencies (e.g., removing version pins like "2024.09.15"), it's acceptable to briefly mention what the change does (e.g., "updates to the latest haskell.nix infrastructure") but avoid speculating about benefits or improvements unless they are specific and measurable.

When updating library dependencies, focus on concrete changes required (e.g., "introduces breaking API changes requiring MonadBaseControl constraint additions") but avoid generic benefit statements like "provides access to improved utilities" or "ensures compatibility with newer frameworks" unless addressing a specific issue.

Avoid concluding commit messages with generic summary statements like "This update modernizes the infrastructure", "ensures compatibility with latest tooling", or "enables access to the latest improvements" - the concrete changes already convey this information. If the conclusion cannot be specific or provide new information beyond what's already known or inferrable, omit it altogether.

**New features typically require:**
- Adding corresponding test files
- Updating module exports
- Adding imports where the feature is used

**Refactoring typically requires:**
- Updating all call sites of renamed functions
- Adjusting imports and exports
- Updating related documentation

### General rules:
- Keep commit messages concise and signal-focused
- Focus on the primary change, not its obvious consequences
- Avoid repeating yourself
- Don't explain changes unless they are substantive or unusual

## Examples

### Concise format (preferred for routine changes):
```
chore: upgrade abcdef from ^>=0.8 to ^>=0.10

Also updates code to compile with the new library
```

```
fix: handle null pointer in user validation
```

```
chore: update hackage index to 2025-09-10
```

### Examples of overly verbose messages to avoid:
```
❌ TOO VERBOSE:
chore: upgrade hedgehog-extras from ^>=0.8 to ^>=0.10

Updates hedgehog-extras dependency across test suites to use the latest
version. This change ensures compatibility with newer testing frameworks
and provides access to improved testing utilities.

- Update cardano-api test suite dependency
- Update cardano-wasm test suite dependency  
- Update hackage index-state to 2025-09-10T10:05:13Z
- Update flake.lock with corresponding dependency changes
- Increase HLS workflow timeout from 60 to 90 minutes for larger builds

The timeout increase accommodates the additional build time required for
the dependency upgrades and ensures CI stability.

✅ BETTER:
chore: upgrade hedgehog-extras from ^>=0.8 to ^>=0.10

Also increases HLS workflow timeout from 60 to 90 minutes
```

### Verbose format (for significant changes):
```
feat(claude): add contextual intelligence for commit message improvement

Implements Phase 3 of the twiddle command enhancement with multi-layer
context discovery including project conventions, branch analysis, and
work pattern detection. This enables more comprehensive and detailed
commit messages similar to the Claude Code template workflow.

- Add project context discovery from .omni-dev/ configuration
- Implement branch naming pattern analysis
- Add work pattern detection across commit ranges
- Enhance Claude prompting with contextual intelligence
- Support verbosity levels based on change significance

Closes #12
```

<!-- Optional verbose template for unusual changes - uncomment and customize if needed:

<type>(<scope>): <concise description>

<Detailed explanation of what was changed and why this approach was taken>

- Specific change 1 with rationale if non-obvious
- Specific change 2 with rationale if non-obvious
- Any breaking changes or migration notes
- Performance implications or behavioral changes

References: <issue numbers, PRs, or external links>

-->
