---
name: _ref-review-criteria
description: "Code review criteria and quality checklist reference"
user-invocable: false
---

# Review Criteria

## Security (Critical)
- No hardcoded secrets or API keys in source code
- Input validation at system boundaries (user input, external APIs)
- SQL injection prevention (parameterized queries only)
- Auth checks on every mutation endpoint (`@require_permission`)

## Architecture
- Module boundaries respected: no cross-module model imports
- Cross-module writes via EventBus, not direct DB
- Services.py is the public API — routes.py is thin HTTP layer
- Error handling uses WorkshopError hierarchy

## Code Quality
- Functions < 50 lines, files < 500 lines
- No dead code or unused imports
- Naming: snake_case for Python, camelCase for TypeScript
- Type hints on public function signatures

## Performance
- N+1 query prevention (use joinedload/selectinload)
- Pagination on list endpoints (PaginatedResponse)
- Redis cache for hot-path reads
- Embedding calls rate-limited (semaphore, max 4 concurrent)

## Testing
- Test data must be purged (hard delete) after tests
- Idempotent event handlers (same event twice = no side effects)
