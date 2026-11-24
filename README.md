# leakguard-ai
VERITAS AI HACKATHON · 
PROJECT WRITE-UP (that won me an Excellence Award+ $1000 scholarship) 
LeakageGuard AI: Quant Code Auditor
Track: AI + Finance · Author: Sairah Elizabeth George
This project is a static code analysis prototype for development verification purposes and is not financial advice.

The Problem
In algorithmic trading, lookahead bias is a silent backtest killer. When a Python strategy accidentally
references future price bars—such as shifting a Pandas series backward
(df['close'].shift(-1)) or indexing forward in a loop (t + 1)—backtests show unrealistic
profits that fail in live markets. Quantitative developers need a fast, automated pre-flight check to
catch data leakage before running backtests.

What I Built
LeakageGuard AI is a static code auditor that parses Python trading scripts into an Abstract Syntax
Tree (AST) and scans for structural time-series leakage. It returns a clear PASSED or FAILED
diagnostic report pinpointing the exact line numbers and violation types:

[*] Running LeakageGuard AI audit... ❌ AUDIT FAILED: Lookahead Violations
Detected! - Violation #1 (Line 5) [Pandas Data Leakage]: Negative shift shift(-1)
references future prices in backtest. - Violation #2 (Line 8) [Lookahead Indexing
Bias]: Illegal future offset (+1) added to time index 't'.

The tool runs end-to-end inside Google Colab with automated unit tests and includes a modular
Streamlit web app (app.py) for live browser auditing.
What I Added
Instead of basic text matching that throws false alarms on normal words or valid rolling windows, I
built a deterministic AST visitor (LookaheadVisitor) and orchestration engine (QuantAuditor). I
also created a custom benchmark suite of flawed and clean strategy scripts alongside unittest
verification tests.
How It Works
AST Parsing (CodeParser): Uses Python's native ast.parse() to convert raw code into a
syntax tree without executing untrusted scripts.
Structural Rules (LookaheadVisitor): Inspects .shift() calls for negative integer constants
(-1, -2) and flags forward additions on time indices (t + 1).
Diagnostics (QuantAuditor): Outputs exact line numbers, rule violations, and clean handling
for syntax errors.


Results & Baseline Comparison
I Tested across custom strategy scripts containing negative shifts, forward offsets, and valid historical
rolling windows (close.rolling(20).mean()):

Audit Method Leakage Caught False Alarms Diagnostics
Baseline 1: Manual review 0% automated 0 None
Baseline 2: Naive text search 100% High (flags valid
shift(1))

No line context

LeakageGuard AI (AST) 100% direct
patterns

0 false positives Exact line & type

Where It Fails
Because static AST analysis inspects code structure without running it, it currently misses dynamic
variable indirection—for example, if a negative offset is stored in an external config file or assigned
to a variable (offset = -1; df['close'].shift(offset)) rather than written as a literal
constant.

What I Would Do Next
Constant Propagation: Track variable assignments across the AST scope so indirect negative
offsets are resolved and flagged.

Expanded Rules: Add checks for backward-filling leakage (bfill()) and forward-looking
merge_asof() calls.

Git Pre-Commit Hook: Package the auditor as a CLI pre-commit check to block leaky code
automatically before merging.
