# Low-latency trading systems in C++20

I build and measure the parts of an electronic trading system that sit on the critical path:
market data in, matching, pre-trade risk, order entry out, and the queues between them. Each
is written from public specifications, checked against a simple reference implementation, and
measured open loop on a dedicated, isolated machine where the latency path calls for it.

## The repositories

| Repository | What it is | One figure |
|---|---|---|
| [lockfree-ipc](https://github.com/Mukachi/lockfree-ipc) | Bounded SPSC and MPMC queues, shared memory between processes, and the measurement harness every other repo uses | SPSC one-way p99.9 250–264 ns at 1M msgs/s, against moodycamel 301–360 and Boost 281–294 |
| [fix-order-gateway](https://github.com/Mukachi/fix-order-gateway) | FIX 4.4 session layer, crash-safe recovery and order entry; conformance shown case by case against the FIX Trading Community's test cases | Order to ExecutionReport p50 19.0–19.1 µs at 10k orders/s; QuickFIX 1.16.0 32.2–34.8 µs with nothing persisted |
| [itch-feed-handler](https://github.com/Mukachi/itch-feed-handler) | Nasdaq TotalView-ITCH 5.0 over MoldUDP64: recovery, every order book, GLIMPSE snapshots, publication to other processes | Packet to published update p50 11.3–11.7 µs at 25k packets/s; the whole day at 4.23M msgs/s |
| [matching-engine](https://github.com/Mukachi/matching-engine) | Price-time limit order book and matching from Nasdaq's rulebook, with a real trading day replayed through it as order entry | Matches the very order Nasdaq executed for 98.55% of the day's 8,555,084 executions; the day at 8.74–8.82M steps/s |
| [tick-to-trade](https://github.com/Mukachi/tick-to-trade) | ITCH in through the feed handler, a strategy and risk checks from the text of RTS 6 Art. 15 and SEC 15c3-5, FIX out through the gateway | Market data to order read by the venue p50 22.0–22.2 µs over loopback, 53.9–55.6 µs over a 10GBASE-T link, at 25k packets/s |

Figures are ranges over repeated runs, copied from each repository's README as of 4 November
2026. The repositories are the source: each README's tables are generated from committed run
files, and a test fails if they disagree.

## How the work is done

- A design note for every component before its code, with what it must meet; the results are
  reviewed against it.
- The real trading day as oracle: Nasdaq's sample TotalView-ITCH 5.0 file of 30 January 2020.
- More than one form of each structure, measured against the others and against the reference;
  the losing forms stay in the repository with their numbers.
- Percentiles over every message, never averages; the machine, its isolation and its noise
  floor recorded in every run.
- Open items listed in each repository with a decision on each, including what is unexplained.
