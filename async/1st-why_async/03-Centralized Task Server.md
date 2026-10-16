# Solution B: Centralized Task Server

## 1\. Why Do We Need This Solution?

In Solution A, each application server stores pending email tasks in its own memory.

When we have multiple application servers, their memory is isolated. One server cannot automatically access the tasks stored in another server's memory.

Our goal is to introduce a separate server that centrally stores the email tasks received from the application servers.

## 2\. Architecture Diagram

``` mermaid
flowchart TD
    C[Client] --> LB[Load Balancer]

    LB --> S1[Application Server 1]
    LB --> S2[Application Server 2]
    LB --> S3[Application Server 3]

    S1 -->|Submit email task| TS[Centralized Task Server]
    S2 -->|Submit email task| TS
    S3 -->|Submit email task| TS

    TS -->|Provide pending tasks| W1[Email Worker 1]
    TS -->|Provide pending tasks| W2[Email Worker 2]

    W1 --> EP[Email Provider]
    W2 --> EP
```

## 3\. How It Works

1. A client requests an action that requires an email, such as signing up.
2. The load balancer routes the request to an application server.
3. The application server submits an email task to the centralized task server.
4. The task server stores the pending task in its own memory.
5. The application server returns a response to the client without waiting for the email to be sent.
6. An email worker retrieves a pending task from the task server.
7. The worker sends the email through the email provider.
8. After successful processing, the task is removed from the task server's pending tasks.

## 4\. Advantages

- **Shared access:** All application servers submit tasks to the same centralized server.
- **Separation of responsibilities:** Application servers handle user requests while workers handle email delivery.
- **Independent scaling:** We can increase the number of application servers or workers separately.
- **Centralized task handling:** Pending tasks are managed in one place rather than scattered across application server memory.

## 5\. Limitations

### Problem 1: The centralized task server becomes a single point of failure

If the task server crashes, application servers may no longer be able to submit or retrieve tasks.

### Problem 2: Pending tasks can be lost

Because tasks are stored only in the task server's memory, a restart or crash can erase pending tasks.

### Problem 3: Multiple workers may process the same task

If several workers retrieve tasks concurrently, the task server must coordinate which worker owns each task to prevent duplicate processing.

### Problem 4: The task server can become a bottleneck

As the number of application servers and email requests grows, one task server may struggle to accept and distribute all tasks.

## 6\. Conclusion

This design solves the isolated-memory problem from Solution A by centralizing pending email tasks.

However, it introduces a new dependency: the centralized task server itself must be highly available, scalable, and reliable.

Our next solution should address these limitations while preserving the benefits of centralized task handling.

We have not introduced a message queue or database in this design. The centralized task server is still an application component storing tasks in memory.

