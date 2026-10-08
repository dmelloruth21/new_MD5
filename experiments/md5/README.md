# MD5 Hash Algorithm

## Experiment ID
`EXP03`

## Experiment Name
**MD5 Hash Algorithm** (Group 3 &mdash; `group-md5`)

## Group Allotment & Collaborative Team Roles
- **Member 1: Nicole Dabre** &mdash; Preprocessing, UTF-8 byte encoding, `0x80` padding, bit length calculation, and 512-bit block parsing.
- **Member 2: Alciya Dodti** &mdash; MD5 trigonometric constants $K[i]$, per-round left rotation amounts $S[i]$, and auxiliary non-linear Boolean functions $F, G, H, I$.
- **Member 3: Larissa Dabreo** &mdash; Initial register states, 64-round compression engine, Merkle&ndash;Damg&aring;rd feed-forward addition, Little-Endian hexadecimal formatting, and core pipeline assembly.
- **Member 4: Ruth Dmello** &mdash; Interactive simulation UI, dynamic salt nonce generation, Strict Avalanche Effect comparator with 128-cell heatmap, 6-stage step-by-step Processing Visualizer, and 10-question self-assessment quiz.

---

## Aim & Objectives
- **Aim:** To implement, trace, and evaluate the **MD5 (Message-Digest Algorithm 5)** cryptographic hashing algorithm; to examine message preprocessing, 512-bit block partitioning, and 64-step non-linear compression rounds; and to quantitatively demonstrate the **Avalanche Effect** and security boundaries through interactive simulation.
- **Objectives:**
  1. Understand the algorithmic foundation of iterative cryptographic hash functions.
  2. Implement native JavaScript preprocessing, padding, and 16 &times; 32-bit word division without third-party libraries.
  3. Trace the intermediate states of chaining registers $A, B, C, D$ across the 4 distinct non-linear compression rounds.
  4. Quantify bitwise Hamming distances and verify the Strict Avalanche Criterion (SAC) showing approximately 50% bit diffusion.
  5. Analyze real-world cryptanalytic collision attacks (Wang et al., Flame, SSL certificate forgery) and understand modern deprecation guidelines (RFC 6151 / NIST SP 800-131A).

---

## Input Specification
- **Data Type:** Text message (arbitrary length UTF-8 encoded string).
- **Optional Parameter:** Dynamic Salt / Nonce token (`0x...` 32-bit pseudorandom hex integer).
- **Edge Cases Supported:**
  - Empty string (`""`).
  - Standard ASCII strings (e.g., `"abc"`, `"hello"`).
  - Extended multi-byte UTF-8 Unicode characters (e.g., `"é"` / `\u00e9`).
  - Long multi-block sentences spanning multiple 512-bit blocks (e.g., `"The quick brown fox jumps over the lazy dog"`).

---

## Output Specification
- **Digest Size:** Fixed 128 bits (16 bytes).
- **Representation:** 32 lowercase hexadecimal characters (4 little-endian 32-bit words concatenated: $A \parallel B \parallel C \parallel D$).
- **Diagnostics:** Input character count, unpadded bit length, padded byte size, 512-bit block count, 32-bit chaining register values ($A, B, C, D$), bit-difference counts, percentage avalanche diffusion, and 128-cell visual heatmap matrix.

---

## Algorithm Architecture & Manual Implementation
The MD5 algorithm is implemented **purely in native JavaScript** (`script.js`) strictly following **RFC 1321** specifications, with **no external libraries** (no CryptoJS, no hash-wasm, and no Web Crypto API).

### 1. Preprocessing & Padding
1. **UTF-8 Byte Encoding:** Plaintext characters are converted into raw byte sequences using the browser's native `TextEncoder`.
2. **Padding Bit Delimiter (0x80):** A single `'1'` bit followed by seven `'0'` bits (`0x80` byte) is immediately appended.
3. **Zero-Byte Padding:** Zero bytes (`0x00`) are appended until the message byte length satisfies:
   $$\text{Length} \equiv 56 \pmod{64} \quad (\text{or } 448 \pmod{512} \text{ bits})$$
4. **Append 64-Bit Length:** The original unpadded bit length is encoded as an 8-byte 64-bit integer using little-endian byte ordering (least significant byte first) and appended to reach an exact multiple of 512 bits (64 bytes).
5. **Partitioning into Words:** Each 512-bit block is parsed into sixteen 32-bit little-endian words: $M[0], M[1], \dots, M[15]$.

### 2. State Registers & Constants
- Four 32-bit chaining registers initialized with RFC 1321 IVs:
  - $A = \mathtt{0x67452301}$
  - $B = \mathtt{0xEFCDAB89}$
  - $C = \mathtt{0x98BADCFE}$
  - $D = \mathtt{0x10325476}$
- Table $K[0 \dots 63]$: 64 constants where $K[i] = \lfloor 2^{32} \times |\sin(i + 1)| \rfloor$.
- Shift table $S[0 \dots 63]$: Predefined per-round left circular rotation amounts.

### 3. Four-Round Compression (64 Operations)
Each 512-bit block is compressed across 4 sequential rounds (16 operations each = 64 operations total):

| Round | Operations | Auxiliary Function | Formula | Word Index $g$ |
|---|---|---|---|---|
| **Round 1** | $0 \dots 15$ | $F(B, C, D)$ | $(B \land C) \lor (\neg B \land D)$ | $g = i$ |
| **Round 2** | $16 \dots 31$ | $G(B, C, D)$ | $(B \land D) \lor (C \land \neg D)$ | $g = (5i + 1) \bmod 16$ |
| **Round 3** | $32 \dots 47$ | $H(B, C, D)$ | $B \oplus C \oplus D$ | $g = (3i + 5) \bmod 16$ |
| **Round 4** | $48 \dots 63$ | $I(B, C, D)$ | $C \oplus (B \lor \neg D)$ | $g = (7i) \bmod 16$ |

For each step $i \in [0, 63]$:
$$temp = D; \quad D = C; \quad C = B;$$
$$B = B + ((A + \text{Func}(B, C, D) + K[i] + M[g]) \lll S[i]) \pmod{2^{32}};$$
$$A = temp;$$

At the conclusion of the 64 operations, updated registers are added modulo $2^{32}$ to previous state values (feed-forward accumulation).

### 4. Final Digest Construction
Registers $A, B, C, D$ are serialized into little-endian hex (8 characters each) and concatenated to yield the 32-character MD5 hash.

---

## Interactive Features & Visualizer

### 1. Dynamic Salt & Nonce Engine
Allows students to toggle an automatic pseudo-random 32-bit salt (`0x...`) appended to the message, demonstrating keyed message hashing and defense against precomputed rainbow table attacks.

### 2. Strict Avalanche Effect Comparator with 128-Cell Heatmap
- Compares two messages differing by as little as 1 bit or 1 character (e.g. `"abc"` vs `"abd"`).
- Computes exact bitwise Hamming distance and character mismatches.
- Renders an interactive **128-Bit Heatmap Matrix** displaying all 128 output bits across Registers A, B, C, and D color-coded as flipped (red) vs unchanged (green), verifying ~50% bit diffusion.

### 3. Step-by-Step MD5 Processing Visualizer
Presents live authentic internal states across 6 collapsible stages:
1. **Stage 1 (Input Message):** Character length, byte count, and unpadded bit size.
2. **Stage 2 (UTF-8 Bytes):** Byte-by-byte hexadecimal sequence.
3. **Stage 3 (Preprocessing & Padding):** Color-coded breakdown of original bytes, `0x80` delimiter byte, zero-pad bytes, and 64-bit LE length.
4. **Stage 4 (512-bit Block Words):** 4&times;4 visual grid of sixteen 32-bit little-endian words ($M[0 \dots 15]$) in hex and decimal.
5. **Stage 5 (Four Compression Rounds):** Documents functions $F, G, H, I$, shift amounts, and displays exact intermediate register snapshots ($A, B, C, D$) after each round.
6. **Stage 6 (Final Digest):** Modulo $2^{32}$ addition step and little-endian hex concatenation.

---

## Knowledge Check & Self-Assessment Quiz
A comprehensive 10-question quiz covering all laboratory learning outcomes:
1. MD5 output digest length (128 bits / 32 hex chars).
2. Initial padding delimiter byte (`0x80`).
3. Message compression block size (512 bits / 16 words).
4. Total compression operations per block (64 operations).
5. Avalanche effect and Strict Avalanche Criterion (SAC).
6. Auxiliary Boolean function $F(B, C, D)$ used in Round 1.
7. Mathematical derivation of table $K[i] = \lfloor 2^{32} \times |\sin(i + 1)| \rfloor$.
8. Appending original bit length as 64-bit little-endian integer.
9. Byte ordering convention (Little-Endian).
10. Modern cryptanalytic collision attacks and security deprecation.

Includes automatic scoring, per-question explanation cards, and retake functionality.

---

## Test Cases & RFC 1321 Verification Vectors

The native implementation passes all official RFC 1321 test vectors:

| Test Case | Input Message | Expected MD5 Digest | Result |
|---|---|---|---|
| **TC1** | `""` (empty string) | `d41d8cd98f00b204e9800998ecf8427e` | Passed |
| **TC2** | `"abc"` | `900150983cd24fb0d6963f7d28e17f72` | Passed |
| **TC3** | `"hello"` | `5d41402abc4b2a76b9719d911017c592` | Passed |
| **TC4** | `"abd"` (1-bit changed input) | `4911e516e5aa21d327512e0c8b197616` | Passed |
| **TC5** | `"é"` (Unicode UTF-8 `0xC3 0xA9`) | `66ddcd97cfdeabb2f6fb8a999b4bc76f` | Passed |
| **TC6** | `"The quick brown fox jumps over the lazy dog"` | `9e107d9d372bb6826bd81d3542a419d6` | Passed |

---

## Cryptographic Limitations & Security Warning
- **Collision Vulnerabilities:** MD5 is **cryptographically broken** for collision resistance. In 2004, Wang et al. demonstrated practical collisions; modern systems can produce chosen-prefix collisions in seconds.
- **Real-World Exploits:** MD5 collisions enabled rogue SSL/TLS Certificate Authority forgery (2008) and Microsoft code-signing bypass in the Flame malware (2012).
- **Deprecation Standards:** **RFC 6151** and NIST **SP 800-131A** formally deprecate MD5 for digital signatures, certificates, and secure protocol negotiation.
- **Educational Context:** MD5 is studied strictly for understanding hash function design, block processing, and the avalanche effect. Production systems must utilize **SHA-256**, **SHA-3**, or password hashing functions like **Argon2id**.

---

## References
1. **RFC 1321:** R. Rivest, *The MD5 Message-Digest Algorithm*, MIT Laboratory for Computer Science and RSA Data Security, Inc., April 1992. [https://www.ietf.org/rfc/rfc1321.txt](https://www.ietf.org/rfc/rfc1321.txt)
2. **RFC 6151:** S. Turner, L. Chen, *Updated Security Considerations for the MD5 Message-Digest and HMAC-MD5 Algorithms*, March 2011. [https://www.ietf.org/rfc/rfc6151.txt](https://www.ietf.org/rfc/rfc6151.txt)
3. **William Stallings:** *Cryptography and Network Security: Principles and Practice*, Pearson Education.
4. **Xiaoyun Wang and Hongbo Yu (2005):** *How to Break MD5 and Other Hash Functions*, EUROCRYPT 2005, LNCS vol 3494, Springer.
5. **NIST SP 800-131A Rev. 2:** *Transitioning the Use of Cryptographic Algorithms and Key Lengths*, NIST Computer Security Division.
