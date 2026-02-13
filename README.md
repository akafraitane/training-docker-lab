# Container Fundamentals Workshop

**From Zero to Production-Ready Containerization**

A comprehensive hands-on workshop covering Docker and Podman containerization, from fundamental concepts to production deployment.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 📋 Workshop Overview

**Duration:** ~8 hours (480 minutes)
**Level:** Beginner to Intermediate
**Prerequisites:** Basic command-line familiarity

### What You'll Learn

- ✅ Container architecture (namespaces, cgroups, layers)
- ✅ Image management and optimization
- ✅ Building production-ready Dockerfiles
- ✅ Container lifecycle management
- ✅ Networking and service discovery
- ✅ Persistent storage strategies
- ✅ Security hardening best practices
- ✅ Multi-container applications

---

## 🚀 Quick Start

### Option 1: View on MOAW (Microsoft Open-source Azure Workshops)

The workshop is optimized for deployment on MOAW for the best experience.

### Option 2: Run Locally with Jekyll

```bash
# Clone the repository
git clone https://github.com/akafraitane/training-docker-lab.git
cd training-docker-lab

# Install dependencies
bundle install

# Serve locally
bundle exec jekyll serve

# Open browser to http://localhost:4000
```

### Option 3: Read the Source

All workshop content is in `_entries/` directory as markdown files.

---

## 📚 Workshop Content

### 1. Introduction
- Why containers matter
- Containers vs VMs
- Real-world use cases

### 2. Core Concepts
- Container architecture internals
- Linux namespaces
- cgroups and resource management
- Image layers and copy-on-write

### 3. Working with Images
- Image management basics
- Choosing the right base image
- Building optimized Dockerfiles
- Multi-stage builds
- Vulnerability scanning

### 4. Running Containers
- Container lifecycle management
- Resource limits and constraints
- Health checks and restart policies
- Debugging and troubleshooting

### 5. Networking
- Network types and isolation
- Custom bridge networks
- Service discovery with DNS
- Port mapping strategies
- Multi-tier network architecture

### 6. Persistent Storage
- Volumes vs bind mounts vs tmpfs
- Data persistence patterns
- Backup and restore strategies
- Storage best practices

### 7. Security & Production
- Image security
- Runtime hardening
- Rootless containers
- Capability management
- Production deployment checklist

### 8. Next Steps
- Container orchestration (Kubernetes)
- CI/CD integration
- Cloud container services
- Learning resources

---

## 🛠️ Sample Applications

The workshop includes production-ready examples:

### Flask API (`sample-app/flask-api/`)
- Multi-stage Dockerfile
- Non-root user
- Health checks
- Gunicorn for production

### E-commerce Stack (`sample-app/ecommerce-stack/`)
- Multi-container docker-compose setup
- Network isolation (frontend/backend)
- PostgreSQL database with persistence
- Redis caching
- nginx reverse proxy

---

## 📖 Prerequisites

### Required Tools

Choose **one** of:

**Podman (Recommended):**
```bash
# macOS
brew install podman
podman machine init --cpus 4 --memory 8192

# Windows (with WSL2)
# Download Podman Desktop from https://podman-desktop.io/

# Linux
sudo apt install podman  # Ubuntu/Debian
sudo dnf install podman  # Fedora/RHEL
```

**Docker:**
```bash
# Download Docker Desktop
# https://www.docker.com/products/docker-desktop/
```

### Optional Tools
- **Trivy** - Vulnerability scanning
- **VS Code** - Code editor
- **Git** - Version control

---

## 📂 Repository Structure

```
training-docker-lab/
├── _config.yml          # Jekyll configuration
├── _entries/            # Workshop content (markdown)
│   ├── 01 introduction.md
│   ├── 01-01 prerequisites.md
│   ├── 02 core-concepts.md
│   ├── 02-01 architecture.md
│   ├── 03 images.md
│   ├── 03-01 image-basics.md
│   ├── 03-02 dockerfiles.md
│   ├── 04 running-containers.md
│   ├── 04-01 running-basics.md
│   ├── 04-02 networking.md
│   ├── 04-03 storage.md
│   ├── 05 security.md
│   ├── 05-01 security-hardening.md
│   ├── 06 next-steps.md
│   └── 06-01 next-steps.md
├── _layouts/            # Jekyll templates
├── _includes/           # Reusable components
├── _sass/              # Stylesheets
├── css/                # Compiled CSS
├── js/                 # JavaScript
├── media/              # Images and diagrams
├── sample-app/         # Example applications
│   ├── flask-api/
│   └── ecommerce-stack/
├── index.html          # Workshop homepage
├── Gemfile             # Ruby dependencies
├── Makefile            # Build commands
└── README.md           # This file
```

---

## 🎯 Learning Objectives

By the end of this workshop, you will be able to:

1. **Understand** container architecture and how isolation works
2. **Build** optimized, secure container images
3. **Deploy** multi-container applications with proper networking
4. **Implement** persistent storage strategies
5. **Apply** security best practices and hardening techniques
6. **Troubleshoot** common container issues
7. **Design** production-ready containerized architectures

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-content`)
3. Commit your changes (`git commit -m 'Add amazing content'`)
4. Push to the branch (`git push origin feature/amazing-content`)
5. Open a Pull Request

### Content Guidelines

- Keep examples simple and focused
- Include hands-on exercises
- Explain the "why" not just the "how"
- Follow existing formatting conventions
- Test all code examples

---

## 📜 License

This workshop uses a dual license:

- **Documentation and workshop content:** [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE)
- **Code samples and applications:** [MIT License](LICENSE-CODE)

You are free to:
- Share and adapt the content
- Use code in your projects
- Create derivative works

**Attribution:** Please credit "Abdoul-Hakim Afraitane" and link back to this repository.

---

## 📞 Contact & Support

**Author:** Abdoul-Hakim Afraitane

- GitHub: [@akafraitane](https://github.com/akafraitane)
- LinkedIn: [abdoul-hakim-afraitane](https://www.linkedin.com/in/abdoul-hakim-afraitane/)
- Original Workshop: [docker-for-beginners-lab](https://github.com/akafraitane/docker-for-beginners-lab)

**Found an issue?** [Report it on GitHub](https://github.com/akafraitane/training-docker-lab/issues)

**Questions?** [Open a discussion](https://github.com/akafraitane/training-docker-lab/discussions)

---

## 🌟 Acknowledgments

- Original workshop inspiration: [docker-for-beginners-lab](https://github.com/akafraitane/docker-for-beginners-lab)
- MOAW template: [training-aks-lab1](https://github.com/lgmorand/training-aks-lab1)
- Container communities: Docker, Podman, Kubernetes
- All contributors and workshop participants

---

## 🗺️ Roadmap

## ⭐ If This Helped You

If you found this workshop useful:

- ⭐ Star this repository
- 🍴 Fork it for your own use
- 📢 Share it with others
- 🐛 Report issues
- 💡 Suggest improvements

---

**Happy containerizing!** 🐳 🚢

*Last updated: February 2024*
