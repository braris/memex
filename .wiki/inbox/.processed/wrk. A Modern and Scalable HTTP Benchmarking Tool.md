---
tags:
  - benchmarking
  - http
  - networking
  - performance
source: https://github.com/wg/wrk
---
# `wrk`: A Modern and Scalable HTTP Benchmarking Tool

## 🔗 Source
**Link:** [`wrk`: A Modern and Scalable HTTP Benchmarking Tool](https://github.com/wg/wrk)

## 📝 Summary
`wrk` is a highly performant, open-source HTTP benchmarking utility designed to generate massive amounts of load from a single multi-core CPU. By combining a multithreaded architecture with scalable event notification systems like `epoll` and `kqueue`, it can efficiently saturate web servers to test their limits. Additionally, it features a built-in LuaJIT scripting engine, allowing developers to customize request generation, process responses, and create tailored reporting without sacrificing raw performance.

## 📖 Key Points & Retelling
* **Massive Load Generation:** `wrk` is specifically engineered to test the limits of HTTP servers by maintaining thousands of concurrent connections and generating significant request volume using minimal local hardware resources (often just a single multi-core CPU).
* **High-Performance Architecture:** It achieves this scale by utilizing a multithreaded design paired with modern, asynchronous OS-level event notification mechanisms (such as `epoll` on Linux and `kqueue` on BSD/macOS), reducing context-switching overhead.
* **LuaJIT Scripting Integration:** One of its standout features is the integration of the LuaJIT scripting engine. Users can write custom Lua scripts to dynamically alter HTTP methods, generate custom headers or payloads, handle authentication, and calculate custom metrics/reports during the benchmark.
* **Customizable Parameters:** Through a simple command-line interface, users can easily configure the total number of threads (`-t`), the total number of concurrent HTTP connections (`-c`), the duration of the test (`-d`), and set specific timeout constraints.
* **Detailed Latency Reporting:** When run with the `--latency` flag, `wrk` provides a comprehensive breakdown of latency distribution (average, standard deviation, max, and percentiles) alongside raw requests-per-second and transfer-per-second metrics.