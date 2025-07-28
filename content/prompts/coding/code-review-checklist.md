---
title: "Comprehensive Code Review Checklist Generator"
date: "2024-11-20"
tags: ["code-review", "quality-assurance", "typescript", "react", "best-practices"]
category: "development-process"
difficulty: "intermediate"
usage_count: 15
effectiveness: "high"
ai_models: ["claude-3.5-sonnet", "gpt-4"]
related_projects: ["codex-app", "portfolio-template"]
---

# Comprehensive Code Review Checklist Generator

## Prompt

```
You are an expert code reviewer conducting a thorough review of [LANGUAGE/FRAMEWORK] code. 
Please analyze the provided code changes and create a comprehensive review checklist covering:

**Context Information:**
- Project: [PROJECT_NAME]
- Framework/Language: [FRAMEWORK_LANGUAGE]
- Team Preferences: [TEAM_STANDARDS]
- Performance Requirements: [PERFORMANCE_NEEDS]

**Review Areas:**

1. **Functionality & Logic**
   - Does the code achieve its intended purpose?
   - Are edge cases handled appropriately?
   - Is error handling comprehensive and appropriate?

2. **Code Quality & Maintainability**
   - Is the code readable and well-structured?
   - Are naming conventions consistent with project standards?
   - Is the code properly commented where necessary?
   - Are functions/methods appropriately sized and focused?

3. **Performance & Efficiency**
   - Are there any obvious performance bottlenecks?
   - Is memory usage optimized?
   - Are expensive operations appropriately cached or memoized?

4. **Security**
   - Are inputs properly validated and sanitized?
   - Are authentication/authorization checks in place?
   - Are sensitive data and secrets handled securely?

5. **Testing**
   - Is test coverage appropriate for the changes?
   - Are tests meaningful and testing the right things?
   - Are integration points properly tested?

6. **Architecture & Design**
   - Does the code follow established architectural patterns?
   - Are dependencies managed appropriately?
   - Is the code loosely coupled and highly cohesive?

**For each area, provide:**
- ✅ Things that are done well
- ⚠️ Areas that need attention
- 🔧 Specific suggestions for improvement
- 📚 Learning opportunities or patterns to share

**Code to review:**
[PASTE_CODE_HERE]
```

## Usage Examples

### Example 1: React Component Review

**Input:**
```typescript
// Context: React TypeScript component for user profile editing
// Team prefers functional components with hooks
// Performance is critical (used on mobile)

const UserProfile = ({ userId }: { userId: string }) => {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data)
        setLoading(false)
      })
  }, [userId])
  
  if (loading) return <div>Loading...</div>
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  )
}
```

**AI Response:**
```markdown
## Code Review Checklist: UserProfile Component

### ✅ Things Done Well
- Clear component interface with typed props
- Proper use of hooks for state management
- Loading state handling

### ⚠️ Areas Needing Attention

**Functionality & Logic**
- [ ] No error handling for failed API requests
- [ ] Potential null reference error if user is null
- [ ] Missing cleanup for aborted requests

**Performance & Efficiency**  
- [ ] API call happens on every render if userId changes
- [ ] No request deduplication or caching

**Security**
- [ ] No input validation for userId parameter
- [ ] Direct API response used without sanitization

### 🔧 Specific Improvements

1. **Add Error Handling:**
```typescript
const [error, setError] = useState<string | null>(null)

useEffect(() => {
  const abortController = new AbortController()
  
  fetch(`/api/users/${userId}`, { signal: abortController.signal })
    .then(res => {
      if (!res.ok) throw new Error('Failed to fetch user')
      return res.json()
    })
    .then(data => {
      setUser(data)
      setError(null)
    })
    .catch(err => {
      if (err.name !== 'AbortError') {
        setError(err.message)
      }
    })
    .finally(() => setLoading(false))
    
  return () => abortController.abort()
}, [userId])
```

2. **Add Type Safety:**
```typescript
interface User {
  id: string
  name: string
  email: string
}

const [user, setUser] = useState<User | null>(null)
```

3. **Improve Performance:**
```typescript
// Use SWR or React Query for caching
import useSWR from 'swr'

const { data: user, error, isLoading } = useSWR(
  `/api/users/${userId}`,
  fetcher
)
```

### 📚 Learning Opportunities
- Consider custom hooks for data fetching logic
- Explore React Query for advanced caching scenarios
- Review team patterns for error boundary integration
```

### Example 2: Database Query Review

**Context:** PostgreSQL query optimization for content search

**Input:**
```sql
-- Search content across all types with filters
SELECT c.*, u.name as author_name
FROM content_items c
JOIN users u ON c.user_id = u.id
WHERE c.title ILIKE '%search_term%' 
   OR c.content ILIKE '%search_term%'
AND c.created_at >= '2024-01-01'
AND c.tags && ARRAY['typescript', 'react']
ORDER BY c.created_at DESC
LIMIT 20;
```

**AI Response Would Include:**
- Index recommendations for search performance
- SQL injection vulnerability assessment  
- Query performance analysis
- Alternative approaches (full-text search, etc.)

## Variations

### Quick Security Review
```
Focus specifically on security concerns in this [LANGUAGE] code:
- Input validation and sanitization
- Authentication/authorization checks  
- Data exposure risks
- Injection vulnerabilities
- Secrets handling

[CODE_HERE]
```

### Performance-Focused Review
```
Analyze this code for performance issues and optimization opportunities:
- Algorithmic complexity
- Memory usage patterns
- I/O operations efficiency
- Caching opportunities
- Scalability concerns

[CODE_HERE]
```

### Architecture Review
```
Evaluate this code from an architectural perspective:
- Design pattern adherence
- Separation of concerns
- Dependency management
- Testability
- Future extensibility

[CODE_HERE]
```

## Success Metrics

- **Code Quality**: 25% reduction in post-merge bugs
- **Review Efficiency**: 40% faster review process
- **Knowledge Sharing**: Junior developers report better understanding
- **Consistency**: More uniform code quality across team

## Tips for Best Results

1. **Provide Context**: Always include project details and team standards
2. **Be Specific**: Mention particular areas of concern
3. **Include Examples**: Reference similar code from your codebase
4. **Set Expectations**: Clarify what level of detail you want
5. **Follow Up**: Ask for clarification on suggestions you don't understand

## Integration with Workflow

```bash
# Git hook example
#!/bin/bash
# .git/hooks/pre-push

echo "Running AI code review on changed files..."
git diff --name-only HEAD~1 HEAD | grep -E '\.(ts|tsx|js|jsx)$' | while read file; do
  echo "Reviewing $file..."
  # Send file to AI review service
  ai-review --file "$file" --output "./reviews/$(basename $file .ts).md"
done
```

## Related Prompts

- [Bug Analysis Prompt](./bug-analysis-prompt.md)
- [Refactoring Suggestions](./refactoring-suggestions.md) 
- [Test Generation Prompt](./test-generation-prompt.md)