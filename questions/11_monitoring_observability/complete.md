# Monitoring & Observability - COMPLETE
## Questions Only - Test Yourself

### Fundamentals
1. What is the difference between monitoring and observability?
2. What are the three pillars of observability? (Metrics, Logs, Traces)
3. What is the difference between metrics and logs?
4. What is distributed tracing? When is it needed?
5. What is the difference between black-box and white-box monitoring?
6. What are SLIs, SLOs, and SLAs? Give concrete examples for a web API.
7. What is an error budget? How does it influence release decisions?
8. What are the RED method metrics? (Rate, Errors, Duration)
9. What are the USE method metrics? (Utilization, Saturation, Errors)
10. What are the four golden signals? (Latency, Traffic, Errors, Saturation)

### Prometheus
11. What is Prometheus? How does it collect metrics? (pull vs push)
12. What is the Prometheus architecture? (server, exporters, alertmanager, pushgateway)
13. What is an exporter? Name 5 common exporters.
14. What is PromQL? Write 5 example queries.
15. How do you write a PromQL query for: request rate per second?
16. How do you write a PromQL query for: 95th percentile latency?
17. How do you write a PromQL query for: error rate percentage?
18. What are metric types? (Counter, Gauge, Histogram, Summary)
19. When do you use Counter vs Gauge vs Histogram?
20. What is a recording rule? Why use one?
21. What is the Pushgateway? When would you use it?
22. How does Prometheus service discovery work in Kubernetes?
23. How do you instrument a Python/Go application for Prometheus?
24. What is the Prometheus retention period? How do you configure it?
25. What are the limitations of Prometheus for long-term storage?

### Grafana
26. What is Grafana? How does it integrate with Prometheus?
27. What types of panels/visualizations does Grafana support?
28. How do you create a dashboard in Grafana?
29. What is a Grafana data source? Name 5.
30. How do you set up alerts in Grafana?
31. How do you templatize dashboards with variables?
32. What is Grafana provisioning? How do you manage dashboards as code?
33. What is Grafana Loki? How is it different from ELK?

### Alerting
34. What is Alertmanager? How does it integrate with Prometheus?
35. What is an alerting rule? Write one for: CPU usage > 80% for 5 minutes.
36. What is alert routing? Grouping? Silencing? Inhibition?
37. How do you send alerts to Slack, PagerDuty, email?
38. What is alert fatigue? How do you prevent it?
39. What is the difference between warning and critical alerts?
40. How do you write good alerts? (actionable, relevant, not noisy)

### Logging
41. What is centralized logging? Why is it needed?
42. What is the ELK stack? (Elasticsearch, Logstash, Kibana)
43. What is the EFK stack? (Elasticsearch, Fluentd, Kibana)
44. What is Loki? How is it different from Elasticsearch?
45. What is structured logging? Why is it better than unstructured?
46. What log levels exist? (DEBUG, INFO, WARN, ERROR, FATAL)
47. How do you aggregate logs from Kubernetes pods?
48. How do you search and filter logs effectively?
49. What is log rotation? Why is it important?
50. How much logging is too much? How do you decide what to log?

### Tracing
51. What is Jaeger? What is Zipkin?
52. What is OpenTelemetry? How does it relate to tracing?
53. What is a span? What is a trace?
54. How do you instrument an application for tracing?
55. What is Kiali? When would you use it?

### Interview-Style
56. Describe your monitoring setup at your current organization.
57. A service has high latency. Walk through how you diagnose it using monitoring tools.
58. How do you set up monitoring for a new microservice from scratch?
59. You're getting 500 alerts per day. Most are noise. How do you fix this?
60. Write Prometheus alerting rules for a production web service.
61. Design a monitoring dashboard for a K8s cluster. What metrics would you show?
62. How do you implement on-call rotation and incident management?
63. What is a runbook? How do you create effective runbooks?
64. How do you monitor CI/CD pipeline health?
65. How do you correlate metrics, logs, and traces for a single request?
