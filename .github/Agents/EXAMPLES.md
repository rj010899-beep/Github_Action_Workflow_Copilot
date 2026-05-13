# PR Review Agent - Example Outputs

This document shows example outputs from the PR Review Agent for different scenarios.

---

## Example 1: Security Issues Found

**Scenario**: PR introduces SQL injection vulnerability and hardcoded API key

**Agent Comment**:

```markdown
## 🔍 Code Review Findings

### 🚨 Security Issues

#### 1. SQL Injection Vulnerability
- **[src/database/users.js:45]**: SQL Injection vulnerability detected
  - **Severity**: Critical
  - **Issue**: User input `email` is directly concatenated into SQL query without sanitization
  - **Code**: 
    ```javascript
    const query = `SELECT * FROM users WHERE email = '${email}'`;
    ```
  - **Recommendation**: Use parameterized queries to prevent SQL injection
    ```javascript
    const query = 'SELECT * FROM users WHERE email = ?';
    db.execute(query, [email]);
    ```
  - **Reference**: [OWASP A03:2021 - Injection](https://owasp.org/Top10/A03_2021-Injection/)
  - **CWE**: [CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)

#### 2. Hardcoded API Key
- **[src/config/api.js:12]**: Hardcoded API key detected
  - **Severity**: Critical
  - **Issue**: API key `AKIAIOSFODNN7EXAMPLE` is exposed in source code
  - **Code**:
    ```javascript
    const API_KEY = 'AKIAIOSFODNN7EXAMPLE';
    ```
  - **Recommendation**: Move sensitive credentials to environment variables
    ```javascript
    const API_KEY = process.env.API_KEY;
    if (!API_KEY) {
      throw new Error('API_KEY environment variable is required');
    }
    ```
  - **Reference**: [CWE-798: Use of Hard-coded Credentials](https://cwe.mitre.org/data/definitions/798.html)

### ⚠️ Code Quality Issues

#### 3. Missing Error Handling
- **[src/api/fetch-data.js:78]**: Unhandled promise rejection
  - **Issue**: Async operation without error handling can cause unhandled rejection
  - **Code**:
    ```javascript
    const data = await fetch(url);
    return data.json();
    ```
  - **Suggestion**: Add try-catch block
    ```javascript
    try {
      const data = await fetch(url);
      return await data.json();
    } catch (error) {
      logger.error('Failed to fetch data:', error);
      throw new FetchError('Data fetch failed', { cause: error });
    }
    ```
  - **Impact**: Silent failures can lead to undefined behavior and difficult debugging

---

**Actions Taken**:
- 🏷️ Added label: `security`
- 🏷️ Added label: `bug`
- 🏷️ Added label: `needs-review`
```

---

## Example 2: Breaking Changes Detected

**Scenario**: PR modifies public API by removing a parameter

**Agent Comment**:

```markdown
## 🔍 Code Review Findings

### 💥 Breaking Changes

#### 1. Function Signature Changed
- **[src/api/user-service.ts:34]**: Public API method signature modified
  - **Change**: Removed `options` parameter from `createUser()`
  - **Before**:
    ```typescript
    async createUser(userData: UserData, options?: CreateOptions): Promise<User>
    ```
  - **After**:
    ```typescript
    async createUser(userData: UserData): Promise<User>
    ```
  - **Impact**: External consumers relying on the `options` parameter will experience runtime errors
  - **Migration**: Update all call sites to remove the `options` parameter, or introduce a new method name
    ```typescript
    // Option 1: Keep old method for backward compatibility
    async createUser(userData: UserData, options?: CreateOptions): Promise<User> {
      return this.createUserV2(userData);
    }
    
    async createUserV2(userData: UserData): Promise<User> {
      // new implementation
    }
    
    // Option 2: Deprecate with clear migration path
    /** @deprecated Use createUserV2() instead. Will be removed in v3.0 */
    async createUser(userData: UserData, options?: CreateOptions): Promise<User>
    ```

#### 2. Environment Variable Renamed
- **[config/env.ts:15]**: Environment variable name changed
  - **Change**: `DATABASE_URL` → `DB_CONNECTION_STRING`
  - **Impact**: Deployment configurations and CI/CD pipelines need updates
  - **Migration**: 
    1. Support both variable names temporarily
    ```typescript
    const dbUrl = process.env.DB_CONNECTION_STRING || process.env.DATABASE_URL;
    if (process.env.DATABASE_URL) {
      logger.warn('DATABASE_URL is deprecated, use DB_CONNECTION_STRING');
    }
    ```
    2. Update all deployment configurations
    3. Add migration guide to CHANGELOG.md

### ✨ Suggestions

- Consider adding deprecation warnings for one version before removing the old API
- Update CHANGELOG.md with migration guide
- Add API versioning to prevent future breaking changes

---

**Actions Taken**:
- 🏷️ Added label: `breaking-change`
- 🏷️ Added label: `documentation`
```

---

## Example 3: Performance Issues

**Scenario**: PR introduces N+1 query problem

**Agent Comment**:

```markdown
## 🔍 Code Review Findings

### ⚠️ Code Quality Issues

#### 1. N+1 Query Problem
- **[src/services/order-service.ts:56-62]**: Database N+1 query anti-pattern detected
  - **Issue**: Fetching related data in a loop causes multiple database queries
  - **Code**:
    ```typescript
    const orders = await Order.findAll();
    for (const order of orders) {
      order.customer = await Customer.findById(order.customerId);
      order.items = await OrderItem.findByOrderId(order.id);
    }
    ```
  - **Suggestion**: Use eager loading or join queries
    ```typescript
    const orders = await Order.findAll({
      include: [
        { model: Customer, as: 'customer' },
        { model: OrderItem, as: 'items' }
      ]
    });
    ```
  - **Impact**: 
    - For 100 orders: 1 + (100 * 2) = 201 queries
    - With fix: 1 query with joins
    - Potential 200x performance improvement
  - **Performance**: Can cause significant slowdown with large datasets

#### 2. Memory Leak in Event Listener
- **[src/components/dashboard.tsx:89]**: Event listener not cleaned up
  - **Issue**: `resize` event listener added but never removed
  - **Code**:
    ```typescript
    useEffect(() => {
      window.addEventListener('resize', handleResize);
    }, []);
    ```
  - **Suggestion**: Add cleanup function
    ```typescript
    useEffect(() => {
      window.addEventListener('resize', handleResize);
      return () => {
        window.removeEventListener('resize', handleResize);
      };
    }, [handleResize]);
    ```
  - **Impact**: Memory accumulates with component mount/unmount cycles

---

**Actions Taken**:
- 🏷️ Added label: `performance`
- 🏷️ Added label: `code-quality`
```

---

## Example 4: Clean PR (No Issues)

**Scenario**: Well-written PR with no issues detected

**Agent Comment**:

```markdown
## ✅ Code Review Complete

This PR looks good! No security vulnerabilities or code quality issues detected.

**Reviewed:**
- Security: ✓
- Code Quality: ✓
- Error Handling: ✓
- Performance: ✓
- Breaking Changes: ✓

**Summary:**
- 5 files changed
- Proper error handling implemented
- Tests included for new functionality
- Documentation updated

Great work! 🎉
```

---

## Example 5: Draft PR (Light Review)

**Scenario**: Draft PR opened for early feedback

**Agent Comment**:

```markdown
## 🔍 Draft PR Review (High-Level)

This is a draft PR - providing architectural feedback only.

### 📐 Architecture Notes

**Overall Approach**: ✓ Good
- Clean separation of concerns between API and service layers
- Proper use of dependency injection

**Consider**:
- Current approach uses synchronous file operations - might want to make async for better performance
- Error handling strategy: ensure consistent error types across modules

**Security Note**:
- ⚠️ [src/auth/token.js:23]: Token generation uses `Math.random()` - consider using `crypto.randomBytes()` for security-sensitive tokens

---

When ready for detailed review, convert from draft to ready for review.
```

---

## Example 6: Large PR

**Scenario**: PR with 25+ files changed

**Agent Comment**:

```markdown
## ⚠️ Large PR Detected

This PR contains **27 files changed** - consider splitting into smaller, focused PRs for easier review.

### High-Risk Files Reviewed

#### Security Critical:
✓ **src/auth/middleware.js** - No issues found
✓ **src/api/payments.ts** - No issues found

#### Configuration Changes:
⚠️ **config/production.yml:15** - Exposed database host
- Consider using environment variable for `db.host`

### Recommendations

1. **Split this PR** into logical chunks:
   - Authentication changes (src/auth/*)
   - API endpoints (src/api/*)
   - Configuration updates (config/*)
   - Tests (tests/*)

2. **Benefits of smaller PRs**:
   - Faster review cycles
   - Easier to identify issues
   - Lower risk of merge conflicts
   - Clearer commit history

3. **For next time**: Aim for PRs with <15 files changed

---

**Partial Review Complete** - Focus on high-risk files only due to PR size.
```

---

## Example 7: Documentation-Only PR

**Scenario**: PR only changes markdown files

**Agent Comment**:

```markdown
## 📚 Documentation Review

### ✓ Documentation Quality
- Clear and well-structured
- Code examples are accurate
- No broken links detected

### ✨ Suggestions

1. **README.md:45** - Consider adding a "Quick Start" section at the top
2. **API.md:78** - Code example missing error handling:
   ```javascript
   // Current
   const result = api.getData();
   
   // Suggested
   try {
     const result = await api.getData();
   } catch (error) {
     console.error('Failed to fetch data:', error);
   }
   ```

3. **CONTRIBUTING.md** - Might want to add a section on commit message conventions

---

**Actions Taken**:
- 🏷️ Added label: `documentation`
```

---

## Example 8: Dependency Update

**Scenario**: PR updates npm packages

**Agent Comment**:

```markdown
## 📦 Dependency Update Review

### Dependencies Updated
- `axios`: 0.27.2 → 1.6.0 ⚠️
- `lodash`: 4.17.20 → 4.17.21 ✓
- `express`: 4.17.1 → 4.18.2 ✓

### ⚠️ Breaking Change Alert

**axios 1.x.x** includes breaking changes:

1. **Response Interceptor Change**:
   - Error responses now need explicit handling
   - Update interceptors in `src/api/client.js`

2. **Timeout Behavior**:
   - Timeout now applies to entire request cycle
   - Review timeout values in API calls

### Security Notes
✓ **lodash 4.17.21** - Fixes prototype pollution vulnerability (CVE-2020-8203)
✓ **express 4.18.2** - Fixes query parser DoS (CVE-2022-24999)

### Testing Recommendations
- [ ] Run full integration test suite
- [ ] Test error handling paths
- [ ] Verify timeout behavior
- [ ] Check API interceptor logic

---

**Actions Taken**:
- 🏷️ Added label: `breaking-change`
- 🏷️ Added label: `needs-review`
```

---

## Summary

The PR Review Agent provides:
- **Specific**: Line numbers and code references
- **Actionable**: Concrete suggestions with examples
- **Educational**: Explanations of why issues matter
- **Prioritized**: Security issues highlighted first
- **Consistent**: Structured format across all reviews

These examples demonstrate the agent's ability to catch real issues while providing valuable feedback to improve code quality.
