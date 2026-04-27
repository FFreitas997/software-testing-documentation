# Performance Testing

## What is Performance Testing?

**Performance testing** is a type of non-functional testing that evaluates how a system behaves under a particular workload. It measures responsiveness, stability, scalability, and resource usage to ensure the application meets its non-functional requirements (NFRs).

Performance issues that are not caught before production can result in degraded user experience, revenue loss, and reputational damage.

---

## Key Performance Metrics

| Metric | Definition | Unit |
|--------|-----------|------|
| **Response Time** | Time from sending a request to receiving the full response | milliseconds (ms) |
| **Latency** | Time for the first byte of the response to arrive | milliseconds (ms) |
| **Throughput** | Number of transactions processed per unit of time | requests/second (RPS) |
| **Error Rate** | Percentage of requests that result in an error | % |
| **Concurrent Users** | Number of users interacting with the system simultaneously | count |
| **CPU Utilisation** | Percentage of CPU capacity consumed | % |
| **Memory Usage** | Amount of RAM consumed by the application | MB / GB |
| **Apdex Score** | User satisfaction metric (0–1) based on response time thresholds | 0.0–1.0 |

### Apdex Score

The Application Performance Index (Apdex) classifies response times into three zones:

- **Satisfied:** response time ≤ T (threshold, e.g., 500 ms)
- **Tolerating:** T < response time ≤ 4T
- **Frustrated:** response time > 4T

```
Apdex = (Satisfied + Tolerating / 2) / Total Samples
```

---

## Types of Performance Tests

### 1. Load Testing

Simulates **expected peak load** to verify the system meets performance targets.

- **Goal:** Confirm that the system handles the anticipated number of concurrent users/transactions within acceptable response times.
- **Example:** Simulate 1,000 concurrent users on a checkout flow during a sales event.

```
Throughput
  ^
  |         ____________________
  |        /
  |       /
  |______/
  |
  +-----------------------------> Time
    Ramp up   Steady state
```

---

### 2. Stress Testing

Pushes the system **beyond its normal limits** to identify the breaking point and how it fails.

- **Goal:** Determine the maximum capacity; observe degradation and recovery behaviour.
- **Questions answered:** At what load does performance degrade? Does the system recover after load drops? Are there memory leaks or resource exhaustion issues?

---

### 3. Spike Testing

Applies a **sudden, dramatic increase** in load, then removes it.

- **Goal:** Test how the system handles unexpected bursts of traffic (e.g., a viral event, a marketing campaign going live).
- **Example:** Instantly increase load from 100 to 5,000 users, hold for 2 minutes, then drop back.

---

### 4. Endurance (Soak) Testing

Runs the system under **sustained load over an extended period** (hours or days).

- **Goal:** Detect memory leaks, connection pool exhaustion, file handle leaks, and other issues that only appear over time.
- **Example:** Simulate 500 concurrent users for 12 hours.

---

### 5. Scalability Testing

Tests the system's ability to **scale up or out** in response to increased load.

- **Vertical scaling:** Adding more CPU/RAM to a single server.
- **Horizontal scaling:** Adding more servers behind a load balancer.
- **Goal:** Verify that scaling mechanisms work and that performance improves proportionally.

---

### 6. Volume Testing

Tests the system's behaviour with a **large amount of data** in the database.

- **Goal:** Ensure performance does not degrade as data grows (e.g., querying a table with 100 million rows).

---

## Performance Testing Process

```
1. Define performance requirements (NFRs)
         ↓
2. Identify critical user journeys
         ↓
3. Design test scenarios
         ↓
4. Prepare test data and environment
         ↓
5. Execute baseline test (single user)
         ↓
6. Execute load / stress tests
         ↓
7. Analyse results and identify bottlenecks
         ↓
8. Optimise and retest
         ↓
9. Report findings
```

---

## Tools

### Apache JMeter

[JMeter](https://jmeter.apache.org/) is the most widely used open-source performance testing tool. It supports HTTP, HTTPS, JDBC, JMS, SOAP, REST, and more.

**Basic JMeter test plan:**
```
Test Plan
  └── Thread Group (100 users, ramp-up 10s, 5 loops)
        ├── HTTP Request (GET /api/products)
        ├── Response Assertion (status code 200)
        └── Aggregate Report Listener
```

**Run from CLI:**
```bash
jmeter -n -t test-plan.jmx -l results.jtl -e -o report/
```

---

### k6

[k6](https://k6.io/) is a modern, developer-friendly load testing tool with a JavaScript API.

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },   // ramp up to 100 users
    { duration: '5m', target: 100 },   // hold at 100 users
    { duration: '2m', target: 0   },   // ramp down to 0
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95th percentile < 500ms
    http_req_failed:   ['rate<0.01'],  // error rate < 1%
  },
};

export default function () {
  const res = http.get('https://api.example.com/products');
  check(res, { 'status is 200': (r) => r.status === 200 });
  sleep(1);
}
```

Run: `k6 run script.js`

---

### Gatling

[Gatling](https://gatling.io/) is a Scala-based tool with a fluent DSL and detailed HTML reports, well-suited for continuous performance testing in CI/CD.

```scala
val scn = scenario("Browse Products")
  .exec(
    http("Get Products")
      .get("/api/products")
      .check(status.is(200))
  )

setUp(
  scn.inject(
    rampUsers(100).during(60.seconds)
  )
).protocols(httpProtocol)
 .assertions(
   global.responseTime.percentile(95).lt(500),
   global.failedRequests.percent.lt(1)
 )
```

---

### Comparison

| Tool | Language | Best For | Free? |
|------|----------|----------|-------|
| Apache JMeter | GUI / XML | Legacy systems, broad protocol support | Yes |
| k6 | JavaScript | Developer workflows, CI integration | Yes (OSS) |
| Gatling | Scala / Java | Continuous performance testing | Yes (OSS) |
| Locust | Python | Custom scenarios | Yes |
| Artillery | YAML / JS | API and microservices | Yes (OSS) |
| BlazeMeter | Cloud | Managed JMeter at scale | Paid |

---

## Performance Testing in CI/CD

Integrating performance tests into the pipeline catches regressions before they reach production:

```yaml
# GitHub Actions example with k6
- name: Run performance tests
  uses: grafana/k6-action@v0.3.1
  with:
    filename: tests/performance/load-test.js
  env:
    K6_CLOUD_TOKEN: ${{ secrets.K6_CLOUD_TOKEN }}
```

**Tips:**
- Run a lightweight smoke-level performance test on every PR.
- Run full load tests nightly or before every release.
- Set thresholds and fail the build if they are breached.
- Track performance trends over time; a slow creep is as dangerous as a sudden spike.

---

## Analysing Results and Identifying Bottlenecks

| Symptom | Likely Cause |
|---------|-------------|
| Response time increases under load | Insufficient server resources, slow database queries |
| Memory grows continuously | Memory leak, unclosed connections |
| CPU pegged at 100% | Inefficient algorithms, missing caching |
| Error rate spikes at high load | Thread pool exhaustion, connection pool too small |
| Latency high but throughput low | Network bottleneck, synchronous blocking I/O |

---

## Further Reading

- [Types of Testing →](02-types-of-testing.md)
- [Security Testing →](08-security-testing.md)
- [Best Practices →](09-best-practices.md)
