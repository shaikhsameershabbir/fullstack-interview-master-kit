## 18) Streams & Buffers (lifecycle, backpressure, pipe)

### Definition (technical)
A **Buffer** is Node’s representation of raw binary data (bytes). A **stream** is an abstraction for reading/writing data incrementally over time (Readable/Writable/Duplex/Transform), with **backpressure** providing flow control so producers do not overwhelm consumers, enabling scalable I/O for large or continuous data.

If you want to build “real Node apps”, streams are unavoidable.

Teacher truth:
> Streams are how Node stays fast and stable when data is big or continuous.

If you don’t understand streams, you’ll eventually write code that:
- loads huge files into memory
- crashes under load
- or becomes slow because you removed backpressure accidentally

---

### Basic intuition (simple)
- **Buffer** = bytes in memory.
- **Stream** = a flow of bytes over time.
- **Backpressure** = “slow down, I can’t handle more data yet.”

The difference between buffering and streaming:
- buffering: “give me everything, then I’ll start”
- streaming: “give me chunks, I’ll process as they arrive”

---

### Internal working (engine level)
#### Why streaming matters (memory + latency)
If you:
1) read a 1GB file fully into memory
2) transform it
3) write it

…you risk:
- huge memory spikes
- GC pressure
- slower performance

Streams avoid this by keeping only a small window of data in memory at any time.

#### Stream types (conceptual)
- **Readable**: produces data (file read stream, HTTP request)
- **Writable**: consumes data (file write stream, HTTP response)
- **Duplex**: readable + writable (sockets)
- **Transform**: duplex that transforms chunks (gzip, parsing, encryption)

Teacher checkpoint:
> Request (`req`) is readable, Response (`res`) is writable. That’s why piping works so naturally in Node servers.

#### Pipe mechanism (the “conveyor belt”)
`readable.pipe(writable)` connects a producer to a consumer and manages flow control.

Think of it like:
- readable pushes chunks
- writable signals when it’s full
- pipe coordinates pause/resume

#### Backpressure (core concept)
Backpressure is what keeps your server from dying.

When the writable side can’t keep up:
- it returns a signal (e.g., `write()` returns `false`)
- the readable should pause
- when writable drains, readable resumes

If you ignore backpressure:
- memory grows
- latency grows
- process can crash

---

### Edge cases
- Missing `error` handlers can crash processes or leak resources.
- Transform streams must respect chunk order; mixing async incorrectly can corrupt output.
- Buffer encoding mismatches (`utf8` vs `base64` vs raw bytes) corrupt data silently.

---

### Interview questions
- Why are streams better than `readFile` for large data?
- What is backpressure? How does Node handle it?
- Explain readable vs writable vs duplex vs transform.
- What happens if you ignore backpressure?

---

### Real-world usage (Node.js)
- File uploads/downloads
- Proxying requests (read from upstream, pipe to client)
- Log processing / ETL
- Compression (`zlib`) and encryption transforms

Streams help you:
- limit memory usage
- reduce latency (start responding before full body is read)
- remain stable under load

---

### Practice (do these in Node)
1) Copy a large file:
   - version A: `readFile` + `writeFile`
   - version B: `createReadStream().pipe(createWriteStream())`
   Compare memory usage.
2) Build a server endpoint that streams a file response.
3) Create a transform stream that uppercases text and pipe file → transform → output.
4) Intentionally ignore backpressure by writing in a tight loop, then fix it by respecting `drain`.

---

### Connections to other topics
- **12 fs**: file streams vs `readFile`.
- **14 HTTP**: request/response bodies are streams.
- **19 Performance**: backpressure prevents memory blowups and latency spikes.
