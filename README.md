# RelayStore

**RelayStore** is a high-performance warm storage buffer designed for managing and versioning files. 

Conceptually, RelayStore acts just like RAM sitting between a CPU and a Hard Drive. It serves as an intermediary layer between your fast clients and high-latency cold storage (like Storj/Cloud), ensuring instant file retrieval and non-blocking asynchronous uploads.

## Core Architecture

RelayStore implements a **Hierarchical Storage Management (HSM)** strategy with three distinct tiers:

- **L1 (Hot) - In-Memory LRU Cache:** Holds the most frequently requested files for sub-millisecond retrieval.
- **L2 (Warm) - Local Filesystem:** Acts as the persistent local buffer.
- **L3 (Cold) - Storj / Cloud:** The decentralized, durable source of truth. High latency, low cost.

When a client requests a file, RelayStore checks `L1 -> L2 -> L3`. If fetched from L3, the file is automatically "thawed" into L2 and L1 for subsequent requests.

## Technical Features

### 1. Asynchronous Write Buffering
Writing directly to decentralized storage (L3) is a blocking, high-latency operation. Handling this synchronously would tie up server resources and risk client timeouts. RelayStore offloads uploads to background workers and instantly returns a `JobID`, keeping the client API highly responsive while allowing clients to poll for real-time progress.

### 2. Immutable Versioning
Files in RelayStore are treated as immutable artifacts. Updates generate new, unique version numbers instead of overwriting existing files. This ensures backward compatibility for downstream services, prevents accidental breaking changes, and allows instant rollbacks without restoring from backups.

### 3. Strict Concurrency Control
Because of the async upload nature, background workers might write a new file to the L1/L2 cache at the exact same millisecond that HTTP requests are trying to read it. To handle this, all read/write operations across the storage layers are heavily wrapped with Go's synchronization primitives (`RWMutex`) to prevent race conditions, dirty reads, and map panics.

## Implementation Status

- [x] Storj integration for durable, decentralized storage.
- [x] Persistent local filesystem buffering.
- [x] Version control
- [x] Thread-safe operations via Mutex/RWMutex. 
- [ ] In-memory LRU cache for fastest retrieval.
- [ ] Friendly SDK
