# Code Commenter Examples

This document provides detailed before/after examples of code commenting across different languages and scenarios.

## Example 1: C++ Database Engine Code

### Before
```cpp
class Compaction {
    void run(int level) {
        std::vector<SSTable*> files = select_files(level);
        SSTable* output = new SSTable();
        
        for (auto f : files) {
            for (auto kv : f->scan()) {
                if (is_valid(kv)) {
                    output->add(kv);
                }
            }
        }
        
        output->finish();
        swap_files(files, output);
    }
};
```

### After
```cpp
/**
 * @brief Manages LSM-tree compaction operations.
 * 
 * Compaction merges multiple sorted SSTable files into fewer, larger files,
 * removing deleted keys and obsolete versions. This is critical for maintaining
 * read performance as the LSM-tree grows.
 * 
 * Thread safety: This class is NOT thread-safe. Callers must ensure exclusive
 * access during compaction operations.
 */
class Compaction {
    /**
     * Execute compaction for the specified level.
     * 
     * This is a blocking operation that typically takes 1-10 seconds per GB of data.
     * During compaction:
     * - Read amplification may temporarily increase by 2x
     * - Disk I/O spikes to ~200MB/s sustained
     * - Temporary disk usage = (input_size + output_size)
     * 
     * Algorithm: Size-tiered compaction
     * 1. Select overlapping SSTables from the level
     * 2. Merge-sort their contents using k-way merge
     * 3. Filter out tombstones and expired keys
     * 4. Write consolidated output to new SSTable
     * 5. Atomically swap old files for new file
     * 
     * @param level LSM tree level to compact (0 = memory, 1-6 = disk levels)
     * 
     * @throws IOException if disk write fails
     * @throws OutOfMemoryError if too many files selected (> 1000)
     */
    void run(int level) {
        // Select SSTables that overlap in key range
        // Uses greedy algorithm: picks largest files first to maximize space reclamation
        std::vector<SSTable*> files = select_files(level);
        
        // Create new SSTable for merged output
        // Pre-allocate bloom filter based on estimated key count to minimize memory reallocations
        SSTable* output = new SSTable();
        
        // K-way merge of all input SSTables
        // Files are already sorted, so this is essentially a merge-sort operation
        for (auto f : files) {
            for (auto kv : f->scan()) {
                // Filter: only keep the latest version of each key
                // is_valid() checks:
                // - Key hasn't been deleted (no tombstone marker)
                // - TTL hasn't expired
                // - Version number is latest (for MVCC)
                if (is_valid(kv)) {
                    output->add(kv);
                }
            }
        }
        
        // Finalize output SSTable: flush buffers, build index, compute checksums
        // After this point, the file is immutable and ready for reads
        output->finish();
        
        // Atomically replace old files with new consolidated file
        // This updates the manifest file and makes the change visible to readers
        // Old files are deleted after a grace period (default: 60s) to allow
        // in-flight reads to complete safely
        swap_files(files, output);
    }
};
```

---

## Example 2: Python Data Pipeline

### Before
```python
def process_events(events, window_size=300):
    result = []
    i = 0
    while i < len(events):
        window = []
        start_time = events[i]['timestamp']
        
        while i < len(events) and events[i]['timestamp'] - start_time < window_size:
            window.append(events[i])
            i += 1
        
        if len(window) > 10:
            result.append(aggregate(window))
    
    return result
```

### After
```python
def process_events(events, window_size=300):
    """
    Process event stream using sliding time windows.
    
    This function groups events into time-based windows and aggregates each window
    that exceeds the minimum threshold. This is commonly used for:
    - Detecting bursts of user activity
    - Rate limiting and throttling
    - Real-time analytics and alerting
    
    Algorithm: Sliding time window
    1. Scan events in chronological order (assumes pre-sorted input)
    2. Group events within window_size seconds of each other
    3. Aggregate windows with sufficient events (>10)
    4. Discard windows with too few events (likely noise)
    
    Args:
        events: List of event dictionaries, MUST be sorted by 'timestamp' field ascending.
               Each event dict must have 'timestamp' (Unix epoch seconds) and other fields.
        window_size: Window duration in seconds (default: 300 = 5 minutes).
                    Smaller windows = more responsive, but more overhead.
                    Larger windows = smoother aggregates, but higher latency.
    
    Returns:
        List of aggregated window results. Each element represents one time window
        that had >10 events. Empty list if no windows met the threshold.
    
    Performance:
        - Time complexity: O(n) single pass through events
        - Space complexity: O(w) where w = max events per window (typically < 1000)
        - Typical throughput: ~100K events/sec on single core
    
    Example:
        >>> events = [
        ...     {'timestamp': 1000, 'user': 'alice', 'action': 'click'},
        ...     {'timestamp': 1100, 'user': 'bob', 'action': 'view'},
        ...     # ... more events ...
        ... ]
        >>> windows = process_events(events, window_size=300)
        >>> print(f"Found {len(windows)} active windows")
    """
    result = []
    i = 0  # Current position in events list
    
    while i < len(events):
        window = []  # Events within current time window
        
        # Start of window is determined by first event's timestamp
        start_time = events[i]['timestamp']
        
        # Collect all events within window_size seconds of start_time
        # This implements a tumbling window (non-overlapping)
        # Events are guaranteed sorted, so we can stop when we exceed the window
        while i < len(events) and events[i]['timestamp'] - start_time < window_size:
            window.append(events[i])
            i += 1
        
        # Only process windows with sufficient events (>10)
        # Threshold of 10 filters out noise and ensures statistical significance
        # Tune this based on your false positive tolerance
        if len(window) > 10:
            # Aggregate window into summary statistics
            # aggregate() computes: event count, unique users, action distribution
            result.append(aggregate(window))
        # NOTE: Windows with ≤10 events are silently discarded
        # Consider logging these if you need to track dropped data
    
    return result
```

---

## Example 3: Go Microservice

### Before
```go
func (s *Server) HandleRequest(w http.ResponseWriter, r *http.Request) {
    ctx, cancel := context.WithTimeout(r.Context(), 5*time.Second)
    defer cancel()
    
    id := r.URL.Query().Get("id")
    
    data, err := s.cache.Get(id)
    if err != nil {
        data, err = s.db.Query(ctx, id)
        if err != nil {
            w.WriteHeader(500)
            return
        }
        s.cache.Set(id, data, 60*time.Second)
    }
    
    json.NewEncoder(w).Encode(data)
}
```

### After
```go
// HandleRequest serves data queries with caching and fallback to database.
//
// Request flow:
// 1. Check cache first (Redis) - typical hit rate: 85%
// 2. On cache miss, query PostgreSQL database
// 3. Cache successful DB queries for 60 seconds
// 4. Return error on failure
//
// Performance characteristics:
// - Cache hit: ~1ms p99 latency
// - Cache miss: ~50ms p99 latency (includes DB query + cache write)
// - Timeout: 5 seconds (prevents resource exhaustion from slow queries)
//
// Error handling:
// - Cache errors are non-fatal (fall through to DB)
// - DB errors return HTTP 500 to client
// - Context cancellation (client disconnect) short-circuits processing
func (s *Server) HandleRequest(w http.ResponseWriter, r *http.Request) {
    // Create timeout context to prevent slow queries from hanging indefinitely
    // 5 seconds chosen based on p99 DB latency (3s) + 2s buffer
    // If timeout fires, context.DeadlineExceeded error is returned to caller
    ctx, cancel := context.WithTimeout(r.Context(), 5*time.Second)
    defer cancel() // Ensure resources are released even if we return early
    
    // Extract resource ID from query parameters
    // TODO: Add validation - currently accepts any string, could validate format
    id := r.URL.Query().Get("id")
    
    // LAYER 1: Check cache (Redis)
    // Cache is first line of defense, serves ~85% of requests
    data, err := s.cache.Get(id)
    if err != nil {
        // Cache miss or cache unavailable - fall back to database
        // Note: We don't distinguish between cache miss vs cache error
        // Both cases require DB query, so we treat them identically
        
        // LAYER 2: Query database (PostgreSQL)
        // This is the source of truth, but slower (~50ms vs 1ms for cache)
        data, err = s.db.Query(ctx, id)
        if err != nil {
            // DB query failed - this is a fatal error
            // Common causes:
            // - Database is down
            // - Query timeout (5s exceeded)
            // - Invalid ID format (no matching row)
            // - Network partition
            
            // TODO: Add structured logging here with error details
            // TODO: Distinguish between 404 (not found) and 500 (server error)
            w.WriteHeader(500)
            return
        }
        
        // Successful DB query - populate cache for future requests
        // TTL of 60 seconds balances:
        // - Freshness: Data can be up to 1 minute stale
        // - Hit rate: Longer TTL = higher hit rate, but staler data
        // - Memory: Shorter TTL = less cache memory pressure
        //
        // Note: Cache write errors are silently ignored (fire-and-forget)
        // If cache write fails, next request will just query DB again
        s.cache.Set(id, data, 60*time.Second)
    }
    // At this point, 'data' contains valid result from either cache or DB
    
    // Serialize response as JSON and write to client
    // No error handling here - if encoding fails, client gets partial response
    // Consider using w.Write() with explicit error checking for production
    json.NewEncoder(w).Encode(data)
}
```

---

## Example 4: SQL Query

### Before
```sql
SELECT 
    u.user_id,
    u.email,
    COUNT(o.order_id) as order_count,
    SUM(o.total_amount) as revenue,
    MAX(o.order_date) as last_order
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.user_id, u.email
HAVING COUNT(o.order_id) > 0
ORDER BY revenue DESC
LIMIT 100;
```

### After
```sql
-- Top 100 customers by revenue (2024 cohort analysis)
--
-- Purpose: Identify highest-value customers who joined in 2024
-- Used by: Marketing team for VIP customer campaigns
-- Runs: Daily at 2am via scheduled job
-- Typical runtime: 15-30 seconds for ~1M users, ~5M orders
--
-- Performance notes:
-- - Uses index on users.created_at (created_at_idx)
-- - Uses index on orders.user_id (orders_user_id_idx)
-- - LEFT JOIN ensures we include users with zero orders (filtered by HAVING)
-- - GROUP BY on both user_id and email because email is not functionally dependent
--
-- Business logic:
-- - "Customer" = user who made at least one order (HAVING clause)
-- - "Revenue" = sum of all order totals (doesn't subtract refunds/cancellations)
-- - Only considers users who joined after Jan 1, 2024
-- - Top 100 sorted by total revenue descending
--
SELECT 
    u.user_id,
    u.email,
    COUNT(o.order_id) as order_count,        -- Total number of orders placed
    SUM(o.total_amount) as revenue,           -- Lifetime revenue (doesn't account for refunds)
    MAX(o.order_date) as last_order           -- Date of most recent order
FROM users u
-- LEFT JOIN preserves all users even if they have no orders
-- This is filtered out by HAVING, but makes the query logic clearer
LEFT JOIN orders o ON u.user_id = o.user_id
WHERE u.created_at > '2024-01-01'             -- Only 2024 cohort
GROUP BY u.user_id, u.email                   -- Email included for display, not aggregation
HAVING COUNT(o.order_id) > 0                  -- Exclude users with no orders
ORDER BY revenue DESC                         -- Highest revenue customers first
LIMIT 100;                                    -- Top 100 only
```

---

## Example 5: JavaScript React Component

### Before
```javascript
function UserList({ users, onSelect }) {
    const [filter, setFilter] = useState('');
    
    const filtered = users.filter(u => 
        u.name.toLowerCase().includes(filter.toLowerCase())
    );
    
    return (
        <div>
            <input 
                value={filter} 
                onChange={e => setFilter(e.target.value)} 
            />
            {filtered.map(u => (
                <div key={u.id} onClick={() => onSelect(u)}>
                    {u.name}
                </div>
            ))}
        </div>
    );
}
```

### After
```javascript
/**
 * UserList - Filterable list of users with selection capability
 * 
 * This component provides a searchable list interface commonly used in:
 * - User management dashboards
 * - Recipient selection for messages/invites
 * - Team member assignment dialogs
 * 
 * Features:
 * - Real-time client-side filtering (case-insensitive substring match)
 * - Click-to-select interaction
 * - Controlled component pattern (filter state managed internally)
 * 
 * Performance: O(n) filtering on every keystroke. For large lists (>1000 users),
 * consider debouncing the filter or using virtualization.
 * 
 * @param {Object} props
 * @param {Array} props.users - Array of user objects with {id, name, ...}
 * @param {Function} props.onSelect - Callback when user clicks a list item: (user) => void
 * 
 * @example
 * <UserList 
 *   users={teamMembers} 
 *   onSelect={(user) => console.log('Selected:', user.name)}
 * />
 */
function UserList({ users, onSelect }) {
    // Local state for search filter
    // Using controlled component pattern - this component owns filter state
    // Parent doesn't need to track search text, simplifying parent logic
    const [filter, setFilter] = useState('');
    
    // Client-side filtering: case-insensitive substring match on name
    // This runs on every render (every keystroke in filter input)
    // 
    // Performance implications:
    // - OK for small lists (<1000 users): ~1ms filtering time
    // - Consider debouncing for medium lists (1000-10000 users)
    // - Consider server-side filtering for large lists (>10000 users)
    // 
    // Algorithm: Simple O(n) linear scan with string.includes()
    // For more advanced search (fuzzy matching, multiple fields), consider
    // libraries like fuse.js or implement Levenshtein distance
    const filtered = users.filter(u => 
        u.name.toLowerCase().includes(filter.toLowerCase())
    );
    
    return (
        <div>
            {/* Search input - updates filter state on every keystroke */}
            <input 
                value={filter}  // Controlled input - value comes from state
                onChange={e => setFilter(e.target.value)}
                placeholder="Search users..."  // Visual hint for users
                aria-label="Filter users by name"  // Accessibility
            />
            
            {/* Render filtered results */}
            {/* Each user div is clickable and triggers onSelect callback */}
            {filtered.map(u => (
                <div 
                    key={u.id}  // React key for efficient list rendering
                    onClick={() => onSelect(u)}  // Notify parent of selection
                    style={{ cursor: 'pointer' }}  // Visual affordance for clickability
                    role="button"  // Accessibility: marks div as interactive
                    tabIndex={0}   // Keyboard navigation support
                >
                    {u.name}
                </div>
            ))}
            
            {/* TODO: Add empty state when filtered.length === 0 */}
            {/* TODO: Add loading state while users are being fetched */}
            {/* TODO: Add virtualization for large lists (react-window) */}
        </div>
    );
}
```

---

## Example 6: Rust Systems Code

### Before
```rust
fn parse_log_line(line: &str) -> Option<LogEntry> {
    let parts: Vec<&str> = line.split('|').collect();
    if parts.len() < 4 {
        return None;
    }
    
    Some(LogEntry {
        timestamp: parts[0].parse().ok()?,
        level: parts[1].to_string(),
        message: parts[2].to_string(),
        metadata: serde_json::from_str(parts[3]).ok()?,
    })
}
```

### After
```rust
/// Parse a single structured log line into a LogEntry.
///
/// Expected format: `timestamp|level|message|metadata_json`
/// Example: `1234567890|INFO|User login|{"user_id": 42, "ip": "1.2.3.4"}`
///
/// This parser is designed for high-throughput log processing (millions of lines/sec)
/// and favors speed over error reporting. Invalid lines are silently dropped.
///
/// # Arguments
/// * `line` - Raw log line as UTF-8 string slice (borrowed, not owned)
///
/// # Returns
/// * `Some(LogEntry)` if parsing succeeds
/// * `None` if line is malformed (wrong format, invalid JSON, etc.)
///
/// # Performance
/// * Zero-copy parsing where possible (string slices, not allocations)
/// * JSON deserialization is the bottleneck (~80% of CPU time)
/// * Typical throughput: ~5M lines/sec on single core (mostly JSON overhead)
///
/// # Error Handling
/// Uses `?` operator for fail-fast semantics. First parse error returns None.
/// Errors include:
/// - Insufficient fields (< 4 delimiters)
/// - Invalid timestamp (not a valid i64 Unix epoch)
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
    // Split on pipe delimiter to extract fields
    // collect() allocates a Vec - consider split().nth() for zero-alloc if hot path
    let parts: Vec<&str> = line.split('|').collect();
    
    // Validate we have all required fields
    // Format: timestamp|level|message|metadata (4 fields minimum)
    // Early return on malformed input to avoid panics downstream
    if parts.len() < 4 {
        return None;  // Silently drop malformed lines (no error logging)
    }
    
    // Construct LogEntry by parsing each field
    // Using ? operator for fail-fast: any parse error returns None immediately
    Some(LogEntry {
        // Parse Unix timestamp (seconds since epoch)
        // .parse() converts &str -> i64, .ok() converts Result -> Option
        // ? operator unpacks Option or returns None if parsing failed
        timestamp: parts[0].parse().ok()?,
        
        // Log level as string (e.g., "INFO", "ERROR")
        // to_string() allocates a new String (owned data)
        // Consider using &'static str or Cow<str> if log levels are limited set
        level: parts[1].to_string(),
        
        // Log message body
        // to_string() allocates - necessary because LogEntry owns its data
        message: parts[2].to_string(),
        
        // Deserialize JSON metadata into HashMap
        // This is the slowest part (~80% of CPU time)
        // serde_json::from_str can fail on invalid JSON
        // .ok() converts Result<T, E> -> Option<T>, discarding error details
        metadata: serde_json::from_str(parts[3]).ok()?,
    })
}
```

---

## Key Takeaways

1. **Context is king**: Explain WHY decisions were made, not just WHAT the code does
2. **Document non-obvious behavior**: Edge cases, performance characteristics, threading
3. **Help future maintainers**: Include debugging hints, common pitfalls, related code
4. **Balance detail and conciseness**: Enough to understand, not so much it obscures
5. **Keep comments in sync**: Outdated comments are worse than no comments
