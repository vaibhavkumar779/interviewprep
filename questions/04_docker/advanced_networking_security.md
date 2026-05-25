# Docker - ADVANCED: Networking, Storage, Security, Compose, Optimization
## Questions Only - Test Yourself

### Multi-Stage Builds
1. What is a multi-stage Docker build? Why use it?
2. Write a multi-stage Dockerfile for a Go application.
3. Write a multi-stage Dockerfile for a Node.js application.
4. How do you copy files from one stage to another?
5. Can you name build stages? How?
6. How many stages can you have? Is there a limit?

### Networking
7. What are the Docker networking modes? (bridge, host, none, overlay, macvlan)
8. What is the default network in Docker?
9. How do containers on the same bridge network communicate?
10. How do containers on different networks communicate?
11. What is `docker network create`? When would you create a custom network?
12. How do you expose a port? Difference between EXPOSE and `-p`?
13. What is port mapping? What does `-p 8080:80` mean?
14. How do you link containers? (legacy --link vs modern networks)

### Storage
15. What is a Docker volume? Why not use the container filesystem?
16. What is the difference between a volume, a bind mount, and a tmpfs mount?
17. How do you create and manage volumes?
18. How do you share data between containers?
19. What is a named volume vs an anonymous volume?
20. How do you backup a Docker volume?

### Docker Compose
21. What is Docker Compose? When would you use it?
22. Write a docker-compose.yml for a web app + PostgreSQL + Redis.
23. What is the difference between `docker-compose up` and `docker-compose up -d`?
24. How do you scale a service in Docker Compose?
25. What is `depends_on`? Does it wait for the service to be ready?
26. How do you handle environment variables in Compose? (.env file)
27. What is the difference between Compose v1 and v2?

### Security & Optimization
28. What are 10 Docker security best practices?
29. What is a distroless image? When would you use it?
30. What is the difference between alpine, slim, and full base images?
31. How do you scan Docker images for vulnerabilities? Name 3 tools.
32. How does Docker layer caching work? How do you optimize it?
33. Why should you order Dockerfile instructions from least to most frequently changing?
34. What is BuildKit? What features does it add?
35. What is `docker system prune`? When would you run it?
36. How do you set memory and CPU limits for a container?
37. What is Docker Content Trust? What does it do?
38. How do you prevent privilege escalation in containers?

### Troubleshooting
39. A container starts but the app inside is not reachable. What do you check?
40. A Docker build fails at a RUN step. How do you debug it?
41. Your Docker images are eating all disk space. How do you clean up?
42. `docker logs` shows nothing but the container is running. Why?
43. A container is consuming 100% CPU. How do you diagnose?

### Interview-Style
44. How do you manage Docker images in a CI/CD pipeline?
45. How do you tag Docker images? What tagging strategy do you use?
46. Explain how you containerized an existing application.
47. Docker vs Podman - what are the differences?
48. What is OCI? What are OCI-compliant images?
49. How do you handle secrets in Docker? (build-time vs runtime)
50. Write a Docker Compose file for a microservices setup with 3 services right now.
