# Contributing Guidelines

## How to Contribute

We welcome contributions! Whether it's bug fixes, documentation improvements, new configurations, or examples.

### Getting Started

1. **Fork the repository**
   ```bash
   git clone https://github.com/YOUR_USERNAME/radarr-nginx-proxy.git
   cd radarr-nginx-proxy
   ```

2. **Create a branch**
   ```bash
   git checkout -b feature/your-feature-name
   git checkout -b fix/your-fix-name
   ```

3. **Make your changes**
   - Update files
   - Add documentation
   - Test thoroughly

4. **Commit your work**
   ```bash
   git add .
   git commit -m "Description of your changes"
   ```

5. **Push and create a Pull Request**
   ```bash
   git push origin your-branch-name
   ```

---

## Types of Contributions

### Documentation
- Fix typos or unclear explanations
- Add examples and use cases
- Improve troubleshooting guides
- Translate documentation

### Configuration Examples
- Add configurations for different scenarios
- Document alternative approaches
- Share working setups
- Include reasoning behind choices

### Bug Reports
- Describe the issue clearly
- Include steps to reproduce
- Share error messages and logs
- Note your environment (OS, versions)

### Feature Requests
- Explain the use case
- Describe the desired behavior
- Suggest implementation approaches
- Link related issues

---

## Before You Submit

- [ ] Test your changes thoroughly
- [ ] Follow the existing code/documentation style
- [ ] Update README or relevant docs
- [ ] Check for spelling and grammar
- [ ] Ensure configurations are commented
- [ ] Include examples for new features
- [ ] Reference any related issues

---

## Documentation Style Guide

### Headers
```markdown
# Main Title (H1 - used only once per file)
## Section (H2)
### Subsection (H3)
```

### Code Blocks
```markdown
# Include language identifier
\`\`\`nginx
location /radarr {
    proxy_pass http://localhost:7878;
}
\`\`\`

# For multi-line examples
\`\`\`bash
# Command with comments
sudo systemctl restart nginx
\`\`\`
```

### Lists and Tables
```markdown
# Unordered list
- Item 1
- Item 2

# Ordered list
1. First
2. Second

# Table
| Column | Value |
|--------|-------|
| Key    | Data  |
```

---

## Nginx Configuration Standards

### Comments
```nginx
# Single line comment
# Describe what this does

# Multi-line explanation
# Line 1
# Line 2
```

### Formatting
```nginx
# Use consistent indentation (4 spaces)
server {
    listen 80;
    
    # Logical grouping with blank lines
    location /radarr {
        proxy_pass http://radarr_backend;
        proxy_set_header Host $host;
    }
}
```

### Variables
```nginx
# Use meaningful variable names
upstream radarr_backend {
    server localhost:7878;
}

# Not: upstream backend { server localhost:7878; }
```

---

## Testing Guidelines

### Before Submitting Configuration
```bash
# Validate Nginx syntax
sudo nginx -t

# Test locally
curl http://localhost:7878/radarr

# Verify through reverse proxy (if running)
curl http://localhost/radarr
```

### Before Submitting Examples
- [ ] Test in Docker
- [ ] Test on bare metal
- [ ] Verify paths are correct
- [ ] Ensure reproducibility
- [ ] Document prerequisites

---

## Pull Request Process

1. **Title**: Be descriptive
   - ✅ Good: "Add Authelia SSO integration example"
   - ❌ Bad: "Fix stuff"

2. **Description**: Explain your changes
   - What problem does this solve?
   - How have you tested this?
   - Are there any breaking changes?

3. **Link issues**: Reference related issues
   ```
   Closes #123
   Related to #456
   ```

4. **Review process**
   - Maintainers will review your PR
   - Make requested changes
   - PR gets merged when approved

---

## Issue Reporting Template

```markdown
### Description
Brief description of the issue

### Steps to Reproduce
1. Step 1
2. Step 2
3. Step 3

### Expected Behavior
What should happen

### Actual Behavior
What actually happens

### Environment
- OS: Ubuntu 22.04
- Nginx version: 1.25.0
- Radarr version: 5.0.0

### Error Messages
(Include full error messages and logs)

### Additional Context
Any other relevant information
```

---

## Feature Request Template

```markdown
### Description
Brief description of the feature

### Use Case
Why would this be useful?

### Proposed Solution
How should this work?

### Alternatives Considered
Other approaches?

### Additional Context
Links, screenshots, etc.
```

---

## Code Review Checklist

For maintainers reviewing contributions:

- [ ] Code/config follows project style
- [ ] Documentation is clear and complete
- [ ] Examples are tested and reproducible
- [ ] No security vulnerabilities
- [ ] No breaking changes (or properly documented)
- [ ] Commit messages are descriptive
- [ ] Changes are focused (not mixing concerns)

---

## Community Guidelines

### Be Respectful
- Treat all contributors with respect
- Accept constructive criticism
- Help others learn

### Be Helpful
- Answer questions thoroughly
- Provide examples when possible
- Link to relevant resources

### Be Professional
- No spam or self-promotion
- Keep discussions on-topic
- Report inappropriate behavior

---

## Areas We Need Help With

- **Documentation**
  - Clarity improvements
  - Translation to other languages
  - Additional examples

- **Testing**
  - Test on different OS versions
  - Report compatibility issues
  - Document tested environments

- **Configuration Examples**
  - Alternative reverse proxies
  - Different authentication methods
  - Unique use cases

- **Maintenance**
  - Keep dependencies updated
  - Review and update outdated info
  - Help with issue triage

---

## Questions?

- Check existing issues and discussions
- Review relevant documentation
- Ask in GitHub Discussions
- Open a new issue for clarification

---

## License

By contributing, you agree that your contributions will be licensed under the same license as the project.

## Attribution

Contributors will be recognized in the CONTRIBUTORS file and project README.

---

Thank you for helping make this project better! 🎉
