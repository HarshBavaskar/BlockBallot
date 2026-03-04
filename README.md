<p align="center">
  <img src="docs/images/banner.png" alt="BlockBallot Banner" width="100%">
</p>

<p align="center">
  <b>A blockchain-secured digital election platform built with Spring Boot & Java</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Java-17+-blue?logo=openjdk&logoColor=white" alt="Java 17+">
  <img src="https://img.shields.io/badge/Spring_Boot-3.2-6DB33F?logo=springboot&logoColor=white" alt="Spring Boot 3.2">
  <img src="https://img.shields.io/badge/Blockchain-SHA--256_+_SHA3--256-black?logo=bitcoin&logoColor=white" alt="Blockchain">
  <img src="https://img.shields.io/badge/PoW-Proof_of_Work-orange?logo=ethereum&logoColor=white" alt="Proof of Work">
  <img src="https://img.shields.io/badge/License-MIT-green" alt="License">
</p>

---

## Table of Contents

- [Overview](#overview)
- [Demo](#demo)
- [Features](#features)
- [Architecture](#architecture)
- [Security Model](#security-model)
- [Database Design](#database-design)
- [OOP Concepts](#oop-concepts)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [API Endpoints](#api-endpoints)
- [Project Structure](#project-structure)
- [Contributing](#contributing)

---

## Overview

**BlockBallot** is a full-stack blockchain-secured digital voting platform that demonstrates how cryptographic blockchain technology can be applied to election systems. Every vote is anonymized, encrypted with multi-layer hashing, mined with Proof-of-Work, and permanently sealed on an immutable ledger.

The platform features a modern monochrome UI with glassmorphism effects, a real-time WebGL particle network background, and a comprehensive security architecture including rate limiting, input validation, audit logging, and hardened HTTP security headers.

### Key Highlights

- **One voter, one vote** — enforced via irreversible cryptographic token derivation
- **Proof-of-Work mining** — every block requires computational effort to create
- **Merkle tree** — O(log N) tamper evidence across the entire chain
- **5-layer hash pipeline** — SHA-256 → SHA3-256 → PBKDF2 (310K iterations) → HMAC-SHA512 → SHA3-256
- **Real-time results** — live vote tallying from the blockchain
- **Full chain verification** — verify integrity of every block at any time

---

## Demo

<p align="center">
  <img src="docs/images/Voting Demo.gif" alt="Vote Page" width="700">
  <br>
  <em>Cast Your Vote</em>
</p>

### Pages

| Page | Description |
|------|-------------|
| `/` | **Cast Vote** — Select a candidate and submit your ballot securely |
| `/login` | **Login Portal** — Secure access for registered electors |
| `/register` | **Elector Registration** — Enroll securely and receive auto-assigned EPIC ID |
| `/results` | **Live Results** — Real-time vote tally with chain statistics dashboard |
| `/ledger` | **Ledger Explorer** — Inspect every block, hash, and timestamp |
| `/audit` | **Audit Log** — Security event monitor (Restricted to `ROLE_ADMIN`) |
| `/admin` | **Admin Panel** — Electoral roll management and status verification (Restricted to `ROLE_ADMIN`) |

---

## Features

### Authentication & Role Flow

- **Elector Registration** — users enroll their details and receive mathematically unique, auto-assigned EPIC ID numbers
- **Dynamic Session Security** — all voting flow pages are shielded behind robust `HttpSession` validations and Interceptor firewalls
- **Role-Based Access Control (RBAC)** — strict `ROLE_ADMIN` / `ROLE_VOTER` segregation preventing unauthorized lateral movement
- **Safe Session Takedown** — dynamic "Sign Out" actions available across every portal for secure session closure

### Voting System

- **Voter registry validation** — only registered EPIC IDs with valid credentials can vote
- **Locked Input Authentication** — active session safely pipes known keys without error-prone manual repetitive inputs
- **One vote per person** — duplicate detection via irreversible cryptographic tokens
- **Candidate listing** with party affiliations
- **Instant receipt** — blockchain hash as proof of participation
- **Copy-to-clipboard** for receipt hashes

### Blockchain Engine

- **Genesis block** — immutable chain origin
- **Proof-of-Work** — blocks must satisfy hash difficulty (3 leading hex zeros)
- **Dual-hash model** — fast SHA-256 for mining + strong multi-layer hash for chain integrity
- **Chain linkage** — each block contains the previous block's hash
- **Block index validation** — sequential index enforcement
- **Merkle root computation** — chain-wide tamper evidence
- **Full chain verification** — re-verify every block's integrity on demand

### Security

- **5-layer cryptographic hashing** (SHA-256 → SHA3-256 → PBKDF2-310K → HMAC-SHA512 → SHA3-256)
- **Timing-safe comparisons** — prevents side-channel attacks
- **Rate limiting** — sliding-window per IP (60 req/min general, 20 votes/5min)
- **Input validation** — regex whitelisting for voter IDs (blocks SQL injection & XSS)
- **Security headers** — CSP, X-Frame-Options, HSTS, Referrer-Policy, Permissions-Policy
- **Audit logging** — immutable event log with IP masking for privacy
- **H2 console disabled** — no database exposure
- **Error details suppressed** — no stack traces or binding errors leaked

### UI/UX

- **Monochrome design system** with CSS custom properties
- **WebGL particle network** background (Three.js) interacting dynamically with inputs
- **Dynamic Interaction Glow & Burst Animations** — visual kinetic feedback connecting DOM input focus vs 3D render depth
- **Glassmorphism** card effects with backdrop blur
- **Micro-animations** — card entrances, hover effects, page transitions
- **Google Fonts** — DM Serif Display, Inter, JetBrains Mono
- **Responsive** — mobile-friendly layout
- **Custom scrollbar** and text selection styling

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     PRESENTATION LAYER                       │
│   Thymeleaf Templates  ·  CSS  ·  JavaScript  ·  Three.js   │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                     CONTROLLER LAYER                         │
│   VotingController (REST + MVC)                              │
│   Depends on VotingService INTERFACE (Dependency Inversion)  │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                      SERVICE LAYER                           │
│   VotingService (interface) → VotingServiceImpl (concrete)   │
│   • Rate limiting     • Input validation     • Audit logging │
│   • Voter registry    • Blockchain ops       • DB persistence│
└──────────┬───────────────────────────────┬──────────────────┘
           │                               │
┌──────────▼──────────────┐   ┌────────────▼──────────────────┐
│   BLOCKCHAIN ENGINE      │   │   DATA ACCESS LAYER           │
│   VotingBlockchain       │   │   JPA Repositories            │
│   ├── Block (Auditable)  │   │   ├── PollOptionRepository    │
│   ├── Merkle Tree        │   │   ├── PollRepository          │
│   └── Chain Verification │   │   ├── VoterRepository         │
│                          │   │   └── VoteRecordRepository    │
│   CRYPTO ENGINE          │   │                               │
│   CryptoLayers           │   │   DATABASE (H2 — 7 tables)    │
│   ├── SHA-256 / SHA3-256 │   │   Normalized to BCNF          │
│   ├── PBKDF2-HMAC-SHA256 │   └───────────────────────────────┘
│   ├── HMAC-SHA512        │
│   └── Proof-of-Work      │
└──────────────────────────┘
```

---

## Security Model

### Voter Anonymization Pipeline

```
Voter ID (e.g. "ABC1234567")
    │
    ▼ Normalize + Uppercase
"ABC1234567"
    │
    ▼ SHA-256
"a1b2c3d4..."
    │
    ▼ SHA3-256 + Election Scope ("POLL-101")
"e5f6a7b8..."
    │
    ▼ PBKDF2-HMAC-SHA256 (310,000 iterations)
[32 bytes]
    │
    ▼ HMAC-SHA512 (server pepper)
"irreversible_anonymous_token"
```

> **Result:** The original voter ID is mathematically irrecoverable. Even the system administrator cannot trace a vote back to a voter.

### Block Mining (Proof-of-Work)

```
payload = previousHash | optionId | anonymizedVoter | timestamp | blockIndex
    │
    ▼ Mine nonce (fast SHA-256, must produce 3 leading zeros)
powHash = "000a7b3c..."  ← PoW proof
    │
    ▼ Compute strong block hash (5-layer pipeline)
currentHash = multiLayerHash(payload | nonce | powHash)
    │
    ▼ Verify: powHash satisfies difficulty AND currentHash matches
✅ Block accepted into chain
```

---

## Database Design

### Normalized to BCNF — 7 Tables

```
┌──────────────┐     ┌──────────────────┐     ┌───────────────┐
│   parties    │     │  constituencies  │     │    polls      │
│──────────────│     │──────────────────│     │───────────────│
│ party_id  PK │◄──┐ │ constituency_id PK│◄──┐│ poll_id    PK │
│ party_name   │   │ │ constituency_name │   ││ title         │
│ party_abbr   │   │ │ state            │   ││ poll_type     │
│ founded_year │   │ │ district         │   ││ start_date    │
└──────────────┘   │ └──────────────────┘   ││ end_date      │
                   │          │              ││ is_active     │
                   │          ▼              │└───────┬───────┘
                   │ ┌──────────────────┐    │        │
                   │ │     voters       │    │        │
                   │ │──────────────────│    │        │
                   │ │ voter_id      PK │    │        │
                   │ │ voter_name      │    │        │
                   │ │ constituency_id FK├───┘        │
                   │ │ date_of_birth   │             │
                   │ │ registered_on   │             │
                   │ │ is_active       │             │
                   │ └──────────────────┘             │
                   │                                   │
              ┌────┴───────────────┐    ┌──────────────┴──────┐
              │   poll_options     │    │   vote_records      │
              │────────────────────│    │─────────────────────│
              │ option_id       PK │◄───│ record_id        PK │
              │ poll_id         FK ├───►│ poll_id          FK │
              │ option_text       │    │ option_id        FK │
              │ party_id        FK ├──┐ │ anonymized_voter    │
              │ constituency_id FK │  │ │ block_hash          │
              └────────────────────┘  │ │ voted_at            │
                                      │ └─────────────────────┘
                                      │
                              ┌───────┴───────────┐
                              │   audit_events    │
                              │───────────────────│
                              │ event_id       PK │
                              │ event_type        │
                              │ detail            │
                              │ client_ip         │
                              │ created_at        │
                              └───────────────────┘
```

### Normalization Forms Demonstrated

| Form | How |
|------|-----|
| **1NF** | Every column is atomic (separate `state`, `district`), every table has a PK |
| **2NF** | All single-column PKs → no partial dependencies possible |
| **3NF** | Party data stored via FK only (`party_id`), not as redundant text |
| **BCNF** | Every determinant (`party_abbr` UNIQUE, `(constituency_name, state)` UNIQUE) is a candidate key |

---

## OOP Concepts

### Encapsulation

All model fields are `private final` with getters only. Business logic is hidden inside classes:

```java
// Voter.java — eligibility rules hidden inside the class
public boolean isEligible() {
    if (!Boolean.TRUE.equals(isActive)) return false;
    return dateOfBirth.plusYears(18).isBefore(LocalDate.now());
}
```

### Abstraction (Abstract Class + Interfaces)

```java
// AbstractPoll.java — defines WHAT a poll is, not HOW it behaves
public abstract class AbstractPoll {
    public abstract String getPollType();
    public abstract int getMaxVotesPerVoter();
}

// Auditable.java — interface contract for integrity verification
public interface Auditable {
    String generateCryptographicHash();
    boolean verifyIntegrity();
}
```

### Inheritance

```
AbstractPoll (abstract)
    ├── GeneralElection   → getPollType() returns "GENERAL_ELECTION"
    └── Referendum         → getPollType() returns "REFERENDUM"
```

### Polymorphism

```java
// Same interface, different runtime behavior
VotingService service = /* Spring injects VotingServiceImpl */;
service.castVote(...);  // dispatches to concrete implementation

// Same method, different output
AbstractPoll poll1 = new GeneralElection(...);
AbstractPoll poll2 = new Referendum(...);
poll1.getPollType();  // → "GENERAL_ELECTION"
poll2.getPollType();  // → "REFERENDUM"
```

### Interfaces (7 total)

| Interface | Purpose |
|-----------|---------|
| `Auditable` | Cryptographic integrity contract |
| `VotingService` | Business logic abstraction |
| `PollOptionRepository` | JPA data access |
| `PollRepository` | JPA data access |
| `VoterRepository` | JPA data access |
| `VoteRecordRepository` | JPA data access |
| `Filter` (Jakarta) | HTTP security headers |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Language** | Java 17+ |
| **Framework** | Spring Boot 3.2 |
| **Template Engine** | Thymeleaf |
| **Database** | H2 (in-memory) |
| **ORM** | Spring Data JPA / Hibernate |
| **Crypto** | Java Security API (SHA-256, SHA3-256, PBKDF2, HMAC-SHA512) |
| **Frontend** | HTML5, CSS3, Vanilla JS |
| **3D Background** | Three.js (WebGL) |
| **Fonts** | Google Fonts (DM Serif Display, Inter, JetBrains Mono) |
| **Build Tool** | Apache Maven |

---

## Getting Started

### Prerequisites

- **Java 17** or higher
- **Apache Maven 3.8+**

### Run

```bash
# Clone the repository
git clone https://github.com/yourusername/BlockBallot.git
cd BlockBallot

# Build and run
mvn clean spring-boot:run
```

The app will start at **http://localhost:8080**

### Available Accounts

The following profiles are pre-seeded in the database to quickly tour the environment:

| ID / EPIC Number | Password | Role |
|----------|------|-----|
| `ADMIN` | `admin123` | `ROLE_ADMIN` |
| `ABC1234567` | `password123` | `ROLE_VOTER` |
| `DEF7654321` | `password123` | `ROLE_VOTER` |
| `GHI9876543` | `password123` | `ROLE_VOTER` |
| `JKL4567890` | `password123` | `ROLE_VOTER` |

### Configuration

| Property | Default | Description |
|----------|---------|-------------|
| `securevote.pepper` | Built-in | Server-side pepper for HMAC (override via `-Dsecurevote.pepper=YOUR_SECRET`) |
| `server.port` | `8080` | HTTP port |

---

## API Endpoints

### Web Pages

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/` | Voting ballot page |
| `POST` | `/cast-vote` | Submit a vote |
| `GET` | `/results` | Live results with chain stats |
| `GET` | `/ledger` | Blockchain ledger explorer |
| `GET` | `/audit` | Security audit log |

### REST APIs

| Method | Path | Response | Description |
|--------|------|----------|-------------|
| `GET` | `/api/verify-chain` | `{ "valid": true, "blocksVerified": 5, "merkleRoot": "..." }` | Full chain integrity verification |
| `GET` | `/api/chain-stats` | `{ "totalBlocks": 5, "totalVotes": 4, "merkleRoot": "..." }` | Chain statistics |

---

## Project Structure

```
src/main/java/com/securevote/
├── SecureVotingApplication.java          # Spring Boot entry point
│
├── config/
│   ├── AppConfig.java                    # Bean configuration & Startup Seeding
│   └── WebConfig.java                    # Interceptor wiring
│
├── controller/
│   ├── AuthController.java               # Login / Register endpoints
│   ├── AdminController.java              # RBAC Admin Panel
│   └── VotingController.java             # MVC + REST controller
│
├── model/
│   ├── Auditable.java                    # Interface — integrity contract
│   ├── AbstractPoll.java                 # Abstract class — poll template
│   ├── GeneralElection.java              # Concrete — extends AbstractPoll
│   ├── Referendum.java                   # Concrete — extends AbstractPoll
│   ├── Block.java                        # Blockchain block (implements Auditable)
│   ├── VotingBlockchain.java             # In-memory blockchain engine
│   ├── Poll.java                         # JPA entity — polls table
│   ├── PollOption.java                   # JPA entity — candidates table
│   ├── Party.java                        # JPA entity — parties table
│   ├── Constituency.java                 # JPA entity — constituencies table
│   ├── Voter.java                        # JPA entity — voters table (stores credentials)
│   └── VoteRecord.java                   # JPA entity — vote records table
│
├── repository/
│   ├── PollOptionRepository.java         # JPA interface — candidates
│   ├── PollRepository.java               # JPA interface — polls
│   ├── VoterRepository.java              # JPA interface — voters
│   ├── ConstituencyRepository.java       # JPA interface — constituencies
│   └── VoteRecordRepository.java         # JPA interface — vote records
│
├── security/
│   ├── CryptoLayers.java                 # 5-layer hashing + PoW + Merkle tree
│   ├── InputValidator.java               # Regex whitelisting for voter IDs
│   ├── AuthInterceptor.java              # Session state route firewalls
│   ├── RateLimiter.java                  # Sliding-window rate limiter
│   ├── SecurityHeadersFilter.java        # CSP, HSTS, X-Frame-Options filter
│   └── AuditLog.java                     # Immutable security event log
│
└── service/
    ├── VotingService.java                # Interface — voting business logic
    └── VotingServiceImpl.java            # Concrete implementation

src/main/resources/
├── application.properties                # Server + DB configuration
├── schema.sql                            # Normalized database schema (7 tables)
├── data.sql                              # Seed data
├── templates/
│   ├── login.html                        # Secure authentication portal
│   ├── register.html                     # Elector enrollment interface
│   ├── vote.html                         # Locked voting booth
│   ├── receipt.html                      # Vote confirmation + receipt hash
│   ├── results.html                      # Live results + chain stats
│   ├── ledger.html                       # Blockchain explorer
│   ├── audit.html                        # Restricted security audit log
│   └── admin.html                        # Restricted management panel
└── static/
    ├── css/app.css                        # Monochrome design system
    └── js/app.js                          # WebGL background + interactions
```

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

<p align="center">
  <br>
  <b>BlockBallot</b> — Because democracy deserves cryptographic certainty.
  <br><br>
  <sub>Built with ♠ using Spring Boot & Java</sub>
</p>
