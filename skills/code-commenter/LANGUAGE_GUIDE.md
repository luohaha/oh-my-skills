# Language-Specific Commenting Guide

Detailed commenting conventions and examples for each language.

## C/C++

### Style
- Use Doxygen comments (`/** */`) for public APIs
- Use `//` for inline comments
- Comment headers extensively, implementations less so

### Key Points to Document
- Memory management (who owns what, when to free)
- Thread safety (which functions are thread-safe)
- Performance characteristics (O(n) complexity)
- Lifetime of objects

### Example
```cpp
/**
 * @brief Thread-safe LRU cache with configurable capacity.
 * 
 * All public methods use internal locking for thread safety.
 * Eviction is automatic when capacity is reached (LRU policy).
 * 
 * @tparam K Key type (must be hashable and comparable)
 * @tparam V Value type (stored by value, must be copyable)
 * 
 * Memory: O(capacity) space for stored items
 * 
 * Thread safety: All methods are thread-safe. Internal mutex
 * protects access to the cache and LRU ordering.
 */
template<typename K, typename V>
class LRUCache {
private:
    // Mutex for all cache operations - held during get/put/evict
    mutable std::mutex mutex_;
    
    // Actual cache storage: map for O(1) lookup
    std::unordered_map<K, V> cache_;
    
    // LRU tracking: recent accesses at front, old at back
    std::list<K> lru_order_;
};
```

---

## Python

### Style
- Use docstrings (`"""`) for modules, classes, functions
- Follow PEP 257 conventions
- Include type hints in function signatures
- Use `#` for inline comments

### Key Points to Document
- Function purpose and algorithm
- Parameter types and constraints
- Return values
- Exceptions that may be raised
- Examples for complex functions

### Example
```python
def merge_sorted_arrays(arr1: List[int], arr2: List[int]) -> List[int]:
    """
    Merge two sorted arrays using two-pointer technique.
    
    Both input arrays MUST be sorted in ascending order.
    Neither input array is modified.
    
    Algorithm: O(n+m) time, O(n+m) space
    Uses two pointers advancing through each array, always
    taking the smaller element.
    
    Args:
        arr1: First sorted array (ascending)
        arr2: Second sorted array (ascending)
        
    Returns:
        New sorted array containing all elements from both inputs
        
    Raises:
        ValueError: If either input array is not sorted
        
    Example:
        >>> merge_sorted_arrays([1, 3, 5], [2, 4, 6])
        [1, 2, 3, 4, 5, 6]
        
        >>> merge_sorted_arrays([1], [])
        [1]
    """
    result = []
    i, j = 0, 0
    
    # Two-pointer merge: compare elements and take smaller
    while i < len(arr1) and j < len(arr2):
        if arr1[i] <= arr2[j]:
            result.append(arr1[i])
            i += 1
        else:
            result.append(arr2[j])
            j += 1
    
    # Append remaining elements (at most one array has remaining)
    result.extend(arr1[i:])
    result.extend(arr2[j:])
    
    return result
```

---

## Java

### Style
- Use JavaDoc (`/** */`) for public APIs
- Document all public classes and methods
- Include `@param`, `@return`, `@throws` tags
- Explain thread safety explicitly

### Key Points to Document
- Class purpose and usage
- Thread safety guarantees
- Exceptions (checked and unchecked)
- Parameter constraints
- Side effects

### Example
```java
/**
 * Thread-safe bounded blocking queue for producer-consumer patterns.
 * 
 * <p>This queue blocks:
 * <ul>
 *   <li>Producers when full (until consumer removes item)
 *   <li>Consumers when empty (until producer adds item)
 * </ul>
 * 
 * <p>Ideal for rate-limiting and backpressure scenarios where
 * you want to prevent producers from overwhelming consumers.
 * 
 * <p><strong>Thread Safety:</strong> All methods are thread-safe.
 * Uses internal locks and condition variables.
 * 
 * <p><strong>Performance:</strong> O(1) enqueue/dequeue operations.
 * 
 * @param <E> type of elements in the queue (must not be null)
 * 
 * @author Casey Luo
 * @version 1.0
 * @since 2025-01-01
 */
public class BoundedQueue<E> {
    /**
     * Inserts element, blocking if queue is full.
     * 
     * <p>Blocks until space becomes available. Will not return
     * until the element is successfully enqueued or thread is interrupted.
     * 
     * @param element element to add (must not be null)
     * @throws InterruptedException if interrupted while waiting
     * @throws NullPointerException if element is null
     * @throws IllegalStateException if queue is shut down
     */
    public void put(E element) throws InterruptedException {
        if (element == null) {
            throw new NullPointerException("Cannot enqueue null");
        }
        
        lock.lock();
        try {
            // Wait while full - releases lock and blocks
            while (queue.size() == capacity) {
                notFull.await();
            }
            
            queue.add(element);
            notEmpty.signal();  // Wake up one waiting consumer
        } finally {
            lock.unlock();
        }
    }
}
```

---

## JavaScript / React

### Style
- Use JSDoc (`/** */`) for functions
- Document component props with PropTypes or TypeScript
- Explain state management
- Document hooks usage and dependencies

### Key Points to Document
- Component purpose and usage
- Props and their types
- State management approach
- Side effects (API calls, timers)
- Performance considerations
- Event handlers

### Example
```javascript
/**
 * Searchable user list with click-to-select functionality.
 * 
 * Features:
 * - Real-time client-side filtering (case-insensitive)
 * - Controlled component (manages own filter state)
 * - Keyboard accessible
 * 
 * Performance: O(n) filtering on every keystroke.
 * For lists >1000 users, consider:
 * - Debouncing the filter (lodash.debounce)
 * - Virtualization (react-window)
 * - Server-side filtering
 * 
 * @param {Object} props
 * @param {Array<{id: string, name: string}>} props.users - User array
 * @param {Function} props.onSelect - Called when user clicks item: (user) => void
 * @returns {JSX.Element}
 * 
 * @example
 * <UserList 
 *   users={teamMembers} 
 *   onSelect={(user) => console.log('Selected:', user.name)}
 * />
 */
function UserList({ users, onSelect }) {
    // Local filter state - parent doesn't need to track this
    // Using controlled component pattern for the input
    const [filter, setFilter] = useState('');
    
    // Client-side filtering: case-insensitive substring match
    // Runs on every render (every keystroke in filter input)
    // 
    // Performance: O(n) where n = users.length
    // For small lists (<1000): ~1ms
    // For large lists (>10000): consider debouncing or server-side filtering
    const filtered = users.filter(u => 
        u.name.toLowerCase().includes(filter.toLowerCase())
    );
    
    return (
        <div className="user-list">
            {/* Controlled input - value from state, onChange updates state */}
            <input 
                type="text"
                value={filter}
                onChange={e => setFilter(e.target.value)}
                placeholder="Search users..."
                aria-label="Filter users by name"
            />
            
            <ul>
                {filtered.map(u => (
                    <li 
                        key={u.id}  // React key for efficient reconciliation
                        onClick={() => onSelect(u)}
                        className="user-item"
                        role="button"
                        tabIndex={0}  // Keyboard navigation
                        onKeyPress={(e) => {
                            // Handle Enter/Space for accessibility
                            if (e.key === 'Enter' || e.key === ' ') {
                                onSelect(u);
                            }
                        }}
                    >
                        {u.name}
                    </li>
                ))}
            </ul>
            
            {/* TODO: Show "No results" when filtered.length === 0 */}
        </div>
    );
}
```

---

## Go

### Style
- Use `//` for comments (Go convention)
- Comment all exported functions/types
- Explain concurrency patterns clearly
- Document goroutine lifecycles

### Key Points to Document
- Exported vs unexported distinction
- Goroutines and channel usage
- Lock holding patterns
- Context cancellation
- Error handling strategies

### Example
```go
// HandleRequest serves data with cache-first strategy and DB fallback.
//
// Flow:
//  1. Check Redis cache (~1ms p99)
//  2. On miss, query PostgreSQL (~50ms p99)
//  3. Cache successful DB results (60s TTL)
//
// Timeout: 5 seconds total (prevents resource exhaustion)
// Thread safety: Safe for concurrent calls
//
// Common errors:
//  - context.DeadlineExceeded: 5s timeout hit
//  - cache errors: non-fatal, falls through to DB
//  - DB errors: returns HTTP 500
func (s *Server) HandleRequest(w http.ResponseWriter, r *http.Request) {
    // Create 5s timeout context to prevent hanging on slow queries
    // Chosen based on: p99 DB latency (3s) + 2s buffer
    ctx, cancel := context.WithTimeout(r.Context(), 5*time.Second)
    defer cancel()  // Always release resources
    
    id := r.URL.Query().Get("id")
    // TODO: Validate ID format (currently accepts any string)
    
    // Try cache first (Redis) - serves ~85% of requests
    data, err := s.cache.Get(id)
    if err != nil {
        // Cache miss or cache down - both require DB query
        // We don't distinguish because both need same handling
        
        // Query database (source of truth, but slower)
        data, err = s.db.Query(ctx, id)
        if err != nil {
            // DB query failed - this is fatal
            // Common causes: DB down, timeout, invalid ID, network partition
            // TODO: Distinguish 404 (not found) from 500 (server error)
            log.Printf("DB query failed: %v", err)
            w.WriteHeader(http.StatusInternalServerError)
            return
        }
        
        // Populate cache for future requests (fire-and-forget)
        // TTL=60s balances freshness vs hit rate vs memory
        // Cache write errors are ignored - next request queries DB again
        s.cache.Set(id, data, 60*time.Second)
    }
    
    // Serialize and send response
    json.NewEncoder(w).Encode(data)
}
```

---

## Rust

### Style
- Use doc comments (`///`) for public items
- Use `//` for inline comments
- Explain ownership and borrowing
- Document panic conditions

### Key Points to Document
- Ownership and lifetime parameters
- Panic vs Result error handling
- Unsafe code justification
- Performance characteristics
- Thread safety

### Example
```rust
/// Parse structured log line into LogEntry.
///
/// Format: `timestamp|level|message|metadata_json`
/// Example: `1234567890|INFO|User login|{"user_id": 42}`
///
/// Design: Optimized for high-throughput log processing.
/// Favors speed over detailed error reporting.
/// Invalid lines are silently dropped (returns None).
///
/// # Arguments
/// * `line` - UTF-8 log line (borrowed, zero-copy where possible)
///
/// # Returns
/// * `Some(LogEntry)` - Successfully parsed
/// * `None` - Malformed line (wrong format, bad JSON, etc.)
///
/// # Performance
/// * Zero-copy parsing for string slices
/// * JSON deserialization is bottleneck (~80% of CPU)
/// * Throughput: ~5M lines/sec single core
///
/// # Errors
/// Fails fast on first error (using `?` operator):
/// - <4 pipe-delimited fields
/// - Invalid timestamp (not a valid i64)
/// - Malformed JSON metadata
///
/// # Examples
/// ```
/// let valid = "1234567890|INFO|test|{}";
/// assert!(parse_log_line(valid).is_some());
///
/// let invalid = "malformed";
/// assert!(parse_log_line(invalid).is_none());
/// ```
fn parse_log_line(line: &str) -> Option<LogEntry> {
    // Split on pipe - collect() allocates Vec
    // Consider split().nth() for zero-alloc if this is hot path
    let parts: Vec<&str> = line.split('|').collect();
    
    // Validate field count (need exactly 4: timestamp|level|message|metadata)
    // Early return prevents panic in parts[] access below
    if parts.len() < 4 {
        return None;  // Silently drop malformed (no logging for perf)
    }
    
    // Construct LogEntry using ? for fail-fast error handling
    Some(LogEntry {
        // Parse Unix timestamp: &str -> i64
        // .parse().ok() converts Result -> Option
        // ? unpacks Option or returns None on failure
        timestamp: parts[0].parse().ok()?,
        
        // to_string() allocates owned String
        // Consider &'static str or Cow<str> if levels are fixed set
        level: parts[1].to_string(),
        
        message: parts[2].to_string(),
        
        // Deserialize JSON metadata (slowest operation ~80% CPU)
        // from_str can fail on invalid JSON
        // .ok() converts Result -> Option, ? unpacks or fails
        metadata: serde_json::from_str(parts[3]).ok()?,
    })
}
```

---

## SQL

### Style
- Use `--` for line comments
- Use `/* */` for block comments
- Comment before queries, not inline
- Explain business logic clearly

### Key Points to Document
- Query purpose and business logic
- Performance characteristics and typical runtime
- Index usage
- Join logic and filtering rules
- Data quality assumptions
- When/how often query runs

### Example
```sql
-- Top 100 revenue-generating customers (2024 cohort)
--
-- Purpose: Identify high-value customers for VIP marketing campaigns
-- Owner: Marketing team
-- Schedule: Runs daily at 2am via scheduled job
-- Runtime: 15-30 seconds for ~1M users, ~5M orders
--
-- Performance:
--   - Uses index: users.created_at_idx
--   - Uses index: orders.user_id_idx
--   - LEFT JOIN preserves all users (filtered by HAVING)
--
-- Business logic:
--   - "Customer" = user with at least 1 order (HAVING COUNT > 0)
--   - "Revenue" = sum of order totals (excludes refunds/cancellations)
--   - "2024 cohort" = users created on/after Jan 1, 2024
--   - Results sorted by lifetime revenue, descending
--
-- Data quality notes:
--   - Does NOT filter test accounts (add WHERE user.is_test = false)
--   - Does NOT exclude cancelled orders
--   - Assumes order.total_amount is in USD
--
SELECT 
    u.user_id,
    u.email,
    
    -- Total number of completed orders
    COUNT(o.order_id) as order_count,
    
    -- Lifetime revenue (gross, doesn't account for refunds)
    SUM(o.total_amount) as revenue,
    
    -- Most recent order date (for recency analysis)
    MAX(o.order_date) as last_order_date
    
FROM users u

-- LEFT JOIN to preserve users even if zero orders
-- (These get filtered out by HAVING clause below)
LEFT JOIN orders o ON u.user_id = o.user_id

-- Filter to 2024 cohort only
WHERE u.created_at >= '2024-01-01'

-- Group by both user_id and email
-- (Email not functionally dependent on user_id in this schema)
GROUP BY u.user_id, u.email

-- Exclude users with no orders
-- (Convert LEFT JOIN into effective INNER JOIN)
HAVING COUNT(o.order_id) > 0

-- Sort by revenue, highest first
ORDER BY revenue DESC

-- Top 100 only
LIMIT 100;
```
