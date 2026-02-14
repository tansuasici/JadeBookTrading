# JADE Book Trading - Multi-Agent Book Trading System

A **Multi-Agent System (MAS)** built with the JADE (Java Agent DEvelopment Framework), where autonomous software agents negotiate with each other to buy and sell books.

## About

This is **not** a traditional web application. There is no central server. Instead, independent **autonomous agents** communicate, negotiate, and trade using FIPA (Foundation for Intelligent Physical Agents) standard messaging protocols.

### What You Will Learn

- Fundamentals of **Agent-Oriented Programming**
- **JADE Framework** usage and agent lifecycle
- **FIPA ACL** (Agent Communication Language) messaging protocol
- **Behaviour models**: `TickerBehaviour`, `CyclicBehaviour`, `OneShotBehaviour`
- **Yellow Pages** service discovery via the Directory Facilitator
- **Negotiation protocol**: CFP (Call for Proposal) → PROPOSE → ACCEPT_PROPOSAL → INFORM

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     JADE Platform (RMA)                       │
│                                                               │
│  ┌───────────────┐    FIPA ACL    ┌───────────────────────┐  │
│  │ BookBuyer     │◄─────────────►│ BookSeller             │  │
│  │ Agent         │               │ Agent                   │  │
│  │               │   1. CFP      │                         │  │
│  │  Target:      │─────────────►│  Catalogue:              │  │
│  │  "Java 101"   │               │  ┌───────────┬────────┐ │  │
│  │               │   2. PROPOSE  │  │ Book      │ Price  │ │  │
│  │               │◄─────────────│  ├───────────┼────────┤ │  │
│  │               │               │  │ Java 101  │  50    │ │  │
│  │               │  3. ACCEPT    │  │ JADE Pro  │  75    │ │  │
│  │               │─────────────►│  └───────────┴────────┘ │  │
│  │               │               │                         │  │
│  │               │   4. INFORM   │  ┌───────────────────┐  │  │
│  │               │◄─────────────│  │   Seller GUI      │  │  │
│  └───────────────┘               │  │  [Title] [Price]  │  │  │
│                                  │  │      [Add]        │  │  │
│         ┌──────────────────┐     │  └───────────────────┘  │  │
│         │ Directory        │     └───────────────────────┘  │
│         │ Facilitator      │                                 │
│         │ (Yellow Pages)   │                                 │
│         └──────────────────┘                                 │
└──────────────────────────────────────────────────────────────┘
```

## Negotiation Protocol (Step by Step)

```
Buyer Agent                    Directory Facilitator              Seller Agent(s)
     │                                │                                │
     │  1. Search "book-selling"       │                                │
     │────────────────────────────────►│                                │
     │                                │                                │
     │  2. Return seller list          │                                │
     │◄────────────────────────────────│                                │
     │                                                                  │
     │  3. CFP: "Do you have Java 101?"                                 │
     │─────────────────────────────────────────────────────────────────►│
     │                                                                  │
     │  4a. PROPOSE: "Price: 50"  (book available)                      │
     │◄─────────────────────────────────────────────────────────────────│
     │  4b. REFUSE: "not-available"   (book not found)                  │
     │◄─────────────────────────────────────────────────────────────────│
     │                                                                  │
     │  [Select lowest price]                                           │
     │                                                                  │
     │  5. ACCEPT_PROPOSAL: "I'll buy Java 101"                         │
     │─────────────────────────────────────────────────────────────────►│
     │                                                                  │
     │  6a. INFORM: "Purchase successful"                               │
     │◄─────────────────────────────────────────────────────────────────│
     │  6b. FAILURE: "Already sold to another buyer"                    │
     │◄─────────────────────────────────────────────────────────────────│
```

## Project Structure

```
BookTrading/
├── src/booktrading/
│   ├── BookBuyerAgent.java      # Buyer agent - searches for and purchases books
│   ├── BookSellerAgent.java     # Seller agent - manages catalogue and handles sales
│   ├── BookSellerGui.java       # Swing GUI for the seller agent
│   └── BookTrading.java         # Main class (entry point)
├── dist/
│   ├── BookTrading.jar          # Compiled JAR file
│   └── lib/
│       ├── jade.jar             # JADE framework library
│       └── commons-codec-1.3.jar
├── build.xml                    # Apache Ant build script
└── nbproject/                   # NetBeans IDE configuration
```

## Class Details

### BookBuyerAgent

An autonomous agent that searches for and purchases a target book.

| Property | Description |
|----------|-------------|
| **Behaviour** | `TickerBehaviour` - searches for sellers every 60 seconds |
| **Strategy** | Sends CFP to all sellers, accepts the lowest price offer |
| **Startup Argument** | Target book title (e.g. `"Java 101"`) |
| **Termination** | Shuts down automatically after a successful purchase |

**Key Behaviours:**
- `TickerBehaviour`: Periodically queries the Directory Facilitator for available sellers
- `RequestPerformer` (inner class): A 4-step state machine that drives the negotiation

### BookSellerAgent

An agent that manages a book catalogue and handles incoming sales.

| Property | Description |
|----------|-------------|
| **Catalogue** | `Hashtable<String, Integer>` - maps book title to price |
| **Service Registration** | Registers as `"book-selling"` in the Directory Facilitator |
| **GUI** | Uses `BookSellerGui` for adding books via a graphical interface |

**Key Behaviours:**
- `OfferRequestsServer` (`CyclicBehaviour`): Responds to CFP messages with a price proposal or refusal
- `PurchaseOrdersServer` (`CyclicBehaviour`): Processes purchase orders, removes sold books from catalogue

### BookSellerGui

A Swing-based graphical interface for the seller agent.

- Text fields for book title and price
- "Add" button to insert books into the catalogue
- Closing the window terminates the agent

## Prerequisites

| Requirement | Version |
|-------------|---------|
| **Java JDK** | 1.8 or higher |
| **Apache Ant** | 1.8.0+ (for building) |
| **JADE** | 4.5.0 (included in `dist/lib/`) |

## Getting Started

### 1. Build the Project

```bash
cd BookTrading
ant clean build
ant jar
```

### 2. Start the JADE Platform

```bash
cd BookTrading/dist

# Option 1: Start the platform with GUI, then create agents manually
java -cp "lib/jade.jar:lib/commons-codec-1.3.jar:BookTrading.jar" \
  jade.Boot -gui

# Option 2: Start with seller agents from the command line
java -cp "lib/jade.jar:lib/commons-codec-1.3.jar:BookTrading.jar" \
  jade.Boot -gui \
  "seller1:booktrading.BookSellerAgent;seller2:booktrading.BookSellerAgent"
```

### 3. Add a Buyer Agent

From the JADE RMA (Remote Management Agent) GUI:

1. Go to **Actions** → **Start New Agent**
2. Agent Name: `buyer1`
3. Class Name: `booktrading.BookBuyerAgent`
4. Arguments: `Java 101` (the book title to search for)

Or directly from the command line:

```bash
java -cp "lib/jade.jar:lib/commons-codec-1.3.jar:BookTrading.jar" \
  jade.Boot -gui \
  "seller1:booktrading.BookSellerAgent;buyer1:booktrading.BookBuyerAgent(Java 101)"
```

### 4. Add Books via the Seller GUI

When a seller agent starts, a GUI window will open:

1. Enter a book title in the **Book title** field (e.g. `Java 101`)
2. Enter a price in the **Price** field (e.g. `50`)
3. Click **Add**

The buyer agent will discover this book within 60 seconds and automatically initiate the purchase process.

## Example Run

```
Seller-agent seller1@192.168.0.100:1099/JADE is ready.
Java 101 inserted into catalogue. Price = 50

Hallo! Buyer-agent buyer1@192.168.0.100:1099/JADE is ready.
Target book is Java 101
Trying to buy Java 101
Found the following seller agents:
seller1@192.168.0.100:1099/JADE
Java 101 successfully purchased from agent seller1@192.168.0.100:1099/JADE
Price = 50
Buyer-agent buyer1@192.168.0.100:1099/JADE terminating.
```

## Glossary

| Term | Description |
|------|-------------|
| **Agent** | An autonomous, goal-driven software entity |
| **Behaviour** | A unit of work that an agent performs |
| **ACL Message** | Agent Communication Language message (FIPA standard) |
| **CFP** | Call for Proposal - a request for price offers |
| **PROPOSE** | A price offer reply |
| **ACCEPT_PROPOSAL** | Acceptance of a seller's offer |
| **INFORM** | Confirmation that a transaction completed successfully |
| **REFUSE** | Rejection (requested item not available) |
| **FAILURE** | Transaction failed (item already sold) |
| **Directory Facilitator (DF)** | Yellow pages service where agents register their services |
| **RMA** | Remote Management Agent - JADE platform management GUI |
| **FIPA** | Foundation for Intelligent Physical Agents - agent standards body |
| **MAS** | Multi-Agent System |

## Resources

- [JADE Official Website](https://jade.tilab.com/)
- [JADE Programmer's Guide](https://jade.tilab.com/doc/programmersguide.pdf)
- [FIPA Specifications](http://www.fipa.org/specifications/)

## License

This project is based on JADE examples and is licensed under the **GNU Lesser General Public License v2.1**.
