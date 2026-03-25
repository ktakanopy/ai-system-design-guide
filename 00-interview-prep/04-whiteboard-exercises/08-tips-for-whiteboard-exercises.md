# Tips for Whiteboard Exercises

### Drawing Tips

1. **Start with boxes and labels** before connecting with arrows
2. **Use consistent notation**: rectangles for services, cylinders for databases, arrows for data flow
3. **Label data on arrows**: what flows between components
4. **Leave space** for additions as you discuss

### Common Patterns to Know

| Pattern | When to Use | Draw As |
|---------|-------------|---------|
| Load balancer + service fleet | Any scaled service | LB → multiple boxes |
| Queue + workers | Async processing | Queue → worker pool |
| Cache layer | Read-heavy, latency-sensitive | Diamond before service |
| CDC/streaming | Real-time updates | Kafka/stream icon |
| Sidecar | Cross-cutting concerns | Small box attached to service |

### Phrases That Signal Strong Candidates

- "Before I design this, let me understand the scale..."
- "The tradeoff here is..."
- "In production, we would also need..."
- "One failure mode to consider is..."
- "Let me walk you through the latency budget..."
- "For evaluation, I would measure..."

### Time Management

- Do not spend more than 5 minutes on clarification
- Draw the complete high-level picture before deep diving
- Leave time for reliability and evaluation
- Check in with the interviewer on focus areas

---

*See also: [Question Bank](01-question-bank.md) | [Answer Frameworks](02-answer-frameworks.md) | [Common Pitfalls](03-common-pitfalls.md)*
