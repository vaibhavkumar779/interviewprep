# Docker - BASICS & FUNDAMENTALS
## Questions Only - Test Yourself

### Core Concepts
1. What is Docker? What problem does it solve?
2. What is the difference between a container and a virtual machine?
3. What is a Docker image? What is a Docker container?
4. What is the Docker Engine? Name its components.
5. What is the Docker daemon? What is the Docker CLI?
6. What is a Docker registry? Name 3 registries.
7. What is Docker Hub? Can you host a private registry?
8. What is the lifecycle of a Docker container? (create, start, stop, rm)
9. What is a Docker layer? How does layering work?
10. What is a union filesystem?
11. What command lists all running containers? All containers (including stopped)?
12. What command shows the logs of a container?
13. How do you execute a command inside a running container?
14. What is `docker inspect`? What information does it show?
15. How do you remove all stopped containers and unused images?

### Dockerfile Deep Dive
16. What is a Dockerfile? What is the build context?
17. Explain each Dockerfile instruction: FROM, RUN, CMD, ENTRYPOINT, COPY, ADD, WORKDIR, ENV, EXPOSE, ARG, VOLUME, USER, LABEL, HEALTHCHECK.
18. What is the difference between COPY and ADD? When to use each?
19. What is the difference between CMD and ENTRYPOINT?
20. What happens when you specify both CMD and ENTRYPOINT?
21. What is the difference between shell form and exec form? (`RUN command` vs `RUN ["command"]`)
22. What is a .dockerignore file? Why is it important?
23. What are build arguments (ARG)? How are they different from ENV?
24. What is a HEALTHCHECK? Write one for an HTTP service.
25. What is the USER instruction? Why should you never run as root?

### Interview-Style
26. Walk me through a Dockerfile you wrote. Explain each line.
27. You have an image that's 2GB. How do you reduce its size?
28. What's the difference between `docker run` and `docker exec`?
29. How do you debug a container that exits immediately on start?
30. Write a Dockerfile for a Python Flask app from scratch right now.
