# Rust Journald Replacement (`rust-journald`)

An educational project to replicate the functionality of `systemd-journald` inside an Artix Linux `runit` environment. 

**Goal:** Understand Linux standard I/O, Unix Domain Sockets, structured binary logging, and concurrency by building a centralized logging daemon in Rust.

---

## 🗺️ The Roadmap & Checklist

### [x] Milestone 1: Standard I/O & The Pipe (The Raw Material)
- [x] Create a `mirror` program that loops infinitely.
- [x] Read standard input (`stdin`) into a mutable `String` buffer.
- [x] Handle **EOF (End of File)** by checking if `bytes_read == 0` to break the loop cleanly.
- [x] Use Early Returns (`return;`) to process command-line arguments (`$1` via `env::args()`) without nesting logic in `else` blocks.
- [x] *Lesson Learned:* Rust expects valid UTF-8 text on `stdin`. Piping raw binaries (like `cat /bin/bash | ./mirror`) will cause a panic unless explicitly handled as raw bytes.

### [x] Milestone 2: Runit's Hidden Plumbing (Connecting the Source)
- [x] **The Goal:** Force `runit` to use this binary instead of `svlogd`.
- [x] Compile a release build: `cargo build --release`.
- [x] Copy the binary to a service's log directory (e.g., `/etc/runit/sv/cups/log/`).
- [x] Modify the `log/run` script to pipe the service's output into the Rust binary, redirecting the Rust output to a file:
  ```bash
  #!/bin/sh
  [ -d /var/log/cups ] || install -dm755 /var/log/cups
  exec ./mirror >> /var/log/cups/cups.log
  ```
- [x] Bring the service up and verify logs are written.

### [ ] Milestone 3: Inter-Process Communication (The Socket)

- [ ] The Goal: Move from text files to a client/server architecture.

-  [ ] Write my-daemon: Uses std::os::unix::net::UnixListener to open a Unix socket (e.g., /run/rust-journal.sock) and print received text.

-  [ ] Write my-forwarder: Uses UnixStream to connect to the socket, send a log line, and exit.

### [ ] Milestone 4: Structuring Data & File Formats (The "Journal")

-   [ ] The Goal: Save structured data, not plain text.

-   [ ] Define a Rust Struct for a log entry (Timestamp, Service ID, Message).

-   [ ] Convert the Struct to a byte array (Vec<u8>) and append it to a .journal binary file.

-   [ ] Write a mini journalctl reader tool that parses the binary file back into readable text.

### [ ] Milestone 5: Concurrency (The Master Daemon)

-   [ ] The Goal: Handle logs from multiple services simultaneously.

-   [ ] Implement multi-threading (std::thread) or async (tokio) in the central daemon.

-   [ ] Use Arc and Mutex to safely share write-access to the binary journal file across multiple incoming socket connections.
