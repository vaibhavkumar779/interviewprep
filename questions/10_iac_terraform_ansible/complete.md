# Terraform & Ansible - COMPLETE
## Questions Only - Test Yourself

### Terraform Basics
1. What is Terraform? What is Infrastructure as Code?
2. What is the difference between Terraform and Ansible?
3. What is declarative vs imperative IaC? Which is Terraform?
4. What are Terraform providers? Name 5.
5. What is the Terraform workflow? (init, plan, apply, destroy)
6. What is `terraform init`? What does it download?
7. What is `terraform plan`? Why run it before apply?
8. What is `terraform apply`? What is `-auto-approve`?
9. What is `terraform destroy`?
10. What is HCL (HashiCorp Configuration Language)?

### Terraform State
11. What is Terraform state? Where is it stored by default?
12. Why should you NOT store state in Git?
13. What is remote state? Name 3 remote backends.
14. What is state locking? Why is it important?
15. What is `terraform state list`? `terraform state show`?
16. How do you move a resource in state? (`terraform state mv`)
17. How do you import existing resources into Terraform? (`terraform import`)
18. What happens if you lose your state file?
19. What is state file encryption?

### Resources & Modules
20. What is a Terraform resource? Write one for an Azure VM.
21. What is a data source? When would you use one?
22. What is a Terraform module? Why use modules?
23. How do you create a module? What is the directory structure?
24. How do you use a module from the Terraform Registry?
25. What are input variables? Output values? Locals?
26. What is `variable` block? What types are supported?
27. How do you pass variables? (CLI, tfvars file, env vars, default values)
28. What is `terraform.tfvars`? How is it different from `variables.tf`?
29. What are output values? When do you need them?
30. What is `depends_on`? When is it needed?

### Advanced Terraform
31. What are provisioners? Why does HashiCorp discourage them?
32. What is `count`? What is `for_each`? When to use which?
33. What are dynamic blocks?
34. What is a Terraform workspace? When would you use one?
35. How do you handle sensitive variables?
36. What is `lifecycle` block? (create_before_destroy, prevent_destroy, ignore_changes)
37. How do you handle Terraform drift?
38. What is `terraform taint`? What replaced it?
39. What is `terraform refresh`?
40. How do you manage multiple environments with Terraform? (workspaces vs directories vs terragrunt)

### Ansible Basics
41. What is Ansible? What is it used for?
42. What is the difference between Ansible and Terraform?
43. What is agentless architecture? How does Ansible connect to hosts?
44. What is an Ansible inventory? (static vs dynamic)
45. What is a playbook? What is a play?
46. What is a task? What is a module?
47. Name 10 Ansible modules you know.
48. What is `ansible.cfg`?
49. What are handlers? When do they run?
50. What is the difference between `copy` and `template` modules?

### Ansible Advanced
51. What are Ansible roles? What is the directory structure?
52. What is Ansible Galaxy? How do you use it?
53. What are Ansible variables? What is variable precedence?
54. What is Jinja2 templating in Ansible?
55. What are Ansible facts? How do you gather custom facts?
56. What are conditionals in Ansible? (`when`)
57. What are loops in Ansible? (`loop`, `with_items`)
58. What is Ansible Vault? How do you encrypt secrets?
59. What are tags in Ansible? How do you use them?
60. How do you test Ansible playbooks? (Molecule, ansible-lint)

### Interview-Style
61. Write a Terraform config to create: VPC + Subnet + Security Group + EC2 instance on AWS.
62. Write a Terraform config to create: Resource Group + AKS cluster on Azure.
63. Write a Terraform module for a reusable web application stack.
64. Write an Ansible playbook to: Install Docker, pull an image, run a container.
65. Write an Ansible playbook to: Set up a monitoring stack (Prometheus + Grafana).
66. How do you handle Terraform state in a team of 10 developers?
67. A Terraform apply fails halfway. Some resources are created, some aren't. What do you do?
68. How do you implement blue-green deployment using Terraform?
69. Describe your IaC setup at your current/previous organization.
70. How do you test IaC before applying to production?
