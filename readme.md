# Handling High Concurrency in Java APIs: Interview Notes

## 1. The Core Concept (The "Why")
**The Problem:** When thousands of users hit your API at the exact same time, the server runs out of resources (Threads, CPU, RAM, or DB Connections).
**The Goal:** We need to **Reject** excess traffic, **Defer** slow tasks, and **Reuse** previous results.

---

## 2. Solution 1: Rate Limiting (The "Bouncer")
**Concept:**
Imagine a club bouncer. He only lets in 5 people per minute. If a 6th person tries to enter, they are turned away immediately. This protects the server from crashing.

**Tools:** `Bucket4j` (Java Library), API Gateway (Kong/AWS), Redis.

**Java Code Implementation (Using Bucket4j):**
*In this example, we allow 10 requests per minute.*

```java
import io.github.bucket4j.Bandwidth;
import io.github.bucket4j.Bucket;
import io.github.bucket4j.Bucket4j;
import io.github.bucket4j.Refill;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.time.Duration;

@RestController
public class RateLimitController {

    private final Bucket bucket;

    public RateLimitController() {
        // Define the limit: 10 requests allowed, refill 10 tokens every 1 minute
        Bandwidth limit = Bandwidth.classic(10, Refill.greedy(10, Duration.ofMinutes(1)));
        this.bucket = Bucket4j.builder().addLimit(limit).build();
    }

    @GetMapping("/api/high-traffic")
    public ResponseEntity<String> handleRequest() {
        // Try to consume 1 token
        if (bucket.tryConsume(1)) {
            return ResponseEntity.ok("Success! Request processed.");
        } else {
            // Return 429 Too Many Requests
            return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS)
                                 .body("Too many requests - Try again later.");
        }
    }
}
