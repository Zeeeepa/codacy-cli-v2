# Comprehensive Codacy CLI v2 Usage Guide

## Table of Contents
1. [Overview](#overview)
2. [Installation](#installation)
3. [Core Concepts](#core-concepts)
4. [Configuration Management](#configuration-management)
5. [Command Reference](#command-reference)
6. [Supported Tools & Languages](#supported-tools--languages)
7. [Usage Workflows](#usage-workflows)
8. [CI/CD Integration](#cicd-integration)
9. [Troubleshooting](#troubleshooting)
10. [Best Practices](#best-practices)

## Overview

Codacy CLI v2 is a comprehensive Go-based command-line tool that provides static code analysis capabilities for multiple programming languages. It operates in two modes:

- **Local Mode**: Uses local configuration files or Codacy's default configurations
- **Remote Mode**: Syncs configuration from your Codacy cloud account

### Key Features
- 🔧 **Multi-tool Support**: ESLint, PMD, Trivy, PyLint, Dart Analyzer, Semgrep, Lizard, Revive
- 📋 **Runtime Management**: Automatically installs and manages Node.js, Python, Java, Go, Dart
- 🎯 **Flexible Configuration**: Local defaults or cloud-synchronized settings
- 📊 **Multiple Output Formats**: Console, SARIF, JSON
- 🚀 **CI/CD Ready**: Built for automation and integration
- 🔒 **Security Focused**: Includes Trivy for vulnerability scanning

## Installation

### macOS (Homebrew)
```bash
brew install codacy/codacy-cli-v2/codacy-cli-v2
```

### Linux / Windows (WSL)
```bash
# One-time installation
bash <(curl -Ls https://raw.githubusercontent.com/codacy/codacy-cli-v2/main/codacy-cli.sh)

# Or create an alias for convenience
alias codacy-cli="bash <(curl -Ls https://raw.githubusercontent.com/codacy/codacy-cli-v2/main/codacy-cli.sh)"
```

### Version Pinning
For reproducible builds, pin to a specific version:
```bash
export CODACY_CLI_V2_VERSION="1.0.0-main.133.3607792"
```

## Core Concepts

### Directory Structure
After initialization, Codacy CLI creates this structure:
```
.codacy/
├── codacy.yaml              # Main configuration (runtimes & tools)
├── cli-config.yaml          # CLI mode configuration
├── .gitignore              # Ignores for Codacy files
├── logs/                   # CLI operation logs
└── tools-configs/          # Tool-specific configurations
    ├── languages-config.yaml
    ├── eslint.config.mjs
    ├── pylintrc
    └── ...
```

### Configuration Hierarchy
1. **codacy.yaml**: Defines runtimes and tools with versions
2. **Tool-specific configs**: Individual tool configurations
3. **Language mapping**: Maps file extensions to tools
4. **CLI mode**: Local vs remote operation mode

### Runtime Management
The CLI automatically manages these runtimes:
- **Node.js**: For ESLint and JavaScript tools
- **Python**: For PyLint and Python tools  
- **Java**: For PMD and Java tools
- **Go**: For Revive and Go tools
- **Dart**: For Dart Analyzer

## Configuration Management

### Sample codacy.yaml
```yaml
runtimes:
    - go@1.22.3
    - java@17.0.10
    - node@22.2.0
    - python@3.11.11
    - dart@3.7.2
tools:
    - eslint@8.57.0
    - pmd@7.11.0
    - pylint@3.3.6
    - trivy@0.66.0
    - semgrep@1.78.0
    - lizard@1.17.31
    - revive@1.7.0
    - dartanalyzer@3.7.2
```

### Language Configuration
The `languages-config.yaml` maps file extensions to tools:
```yaml
tools:
  - name: eslint
    languages: ["JavaScript", "TypeScript"]
    extensions: [".js", ".jsx", ".ts", ".tsx", ".mjs"]
    files: []
  - name: pylint
    languages: ["Python"]
    extensions: [".py"]
    files: []
  - name: trivy
    languages: ["Docker", "Kubernetes"]
    extensions: []
    files: ["Dockerfile", "docker-compose.yml", "*.yaml", "*.yml"]
```

## Command Reference

### `init` - Bootstrap Project Configuration

Initialize with local defaults:
```bash
codacy-cli init
```

Initialize with Codacy cloud integration:
```bash
codacy-cli init \
  --api-token <your-api-token> \
  --provider gh \
  --organization <your-org> \
  --repository <your-repo>
```

**Flags:**
- `--api-token`: Codacy API token (enables remote mode)
- `--provider`: Git provider (`gh`, `gl`, `bb`)
- `--organization`: Organization/owner name
- `--repository`: Repository name

### `config` - Configuration Management

#### `config discover` - Auto-detect Languages
```bash
# Discover languages in current directory
codacy-cli config discover .

# Discover in specific path
codacy-cli config discover /path/to/project
```

This command:
- Scans files to detect programming languages
- Updates `languages-config.yaml` with findings
- Enables relevant tools in `codacy.yaml`
- Creates tool-specific configuration files

#### `config reset` - Reset Configuration
```bash
# Reset to local defaults
codacy-cli config reset

# Reset to remote configuration
codacy-cli config reset \
  --api-token <token> \
  --provider gh \
  --organization <org> \
  --repository <repo>
```

### `install` - Install Dependencies

```bash
# Install all runtimes and tools
codacy-cli install

# Use custom registry
codacy-cli install --registry <custom-registry-url>
```

The install process:
1. Downloads and extracts runtimes (Node, Python, Java, etc.)
2. Installs tools using appropriate package managers
3. Handles platform-specific details
4. Shows progress and reports failures
5. Skips already installed components

### `analyze` - Run Code Analysis

```bash
# Run all configured tools
codacy-cli analyze

# Run specific tool
codacy-cli analyze --tool eslint

# Output in SARIF format
codacy-cli analyze --tool eslint --format sarif

# Save results to file
codacy-cli analyze --tool eslint --format sarif --output results.sarif

# Auto-fix issues when possible
codacy-cli analyze --tool eslint --fix

# Analyze specific files
codacy-cli analyze path/to/file.js
codacy-cli analyze --tool eslint src/components/
```

**Flags:**
- `--tool, -t`: Specific tool to run
- `--format`: Output format (`sarif`, `json`)
- `--output, -o`: Output file path
- `--fix`: Apply automatic fixes

### `upload` - Upload Results to Codacy

With project token:
```bash
codacy-cli upload \
  --sarif-path results.sarif \
  --commit-uuid <commit-sha> \
  --project-token <project-token>
```

With API token:
```bash
codacy-cli upload \
  --sarif-path results.sarif \
  --commit-uuid <commit-sha> \
  --api-token <api-token> \
  --provider gh \
  --owner <owner> \
  --repository <repo>
```

### `update` - Update CLI
```bash
codacy-cli update
```

### `version` - Show Version
```bash
codacy-cli version
```

## Supported Tools & Languages

| Tool | Languages | Purpose | Config File |
|------|-----------|---------|-------------|
| **ESLint** | JavaScript, TypeScript | Linting & Code Quality | `eslint.config.mjs` |
| **PMD** | Java, Apex, JavaScript | Static Analysis | `pmd.xml` |
| **Trivy** | Docker, K8s, IaC | Security Scanning | `trivy.yaml` |
| **PyLint** | Python | Code Quality & Style | `pylintrc` |
| **Dart Analyzer** | Dart, Flutter | Static Analysis | `analysis_options.yaml` |
| **Semgrep** | Multi-language | Security & Bug Detection | `semgrep.yml` |
| **Lizard** | Multi-language | Complexity Analysis | N/A |
| **Revive** | Go | Linting & Style | `revive.toml` |

### Tool Versions
The CLI supports multiple versions of some tools:
- **ESLint**: v8.x and v9.x
- **PMD**: v6.x and v7.x

Version selection is automatic based on your project's dependencies.

## Usage Workflows

### 1. New Project Setup (Local Mode)
```bash
# 1. Initialize with defaults
codacy-cli init

# 2. Auto-discover languages
codacy-cli config discover .

# 3. Install dependencies
codacy-cli install

# 4. Run analysis
codacy-cli analyze

# 5. Fix issues automatically where possible
codacy-cli analyze --fix
```

### 2. Existing Project with Codacy Account
```bash
# 1. Initialize with cloud sync
codacy-cli init \
  --api-token $CODACY_API_TOKEN \
  --provider gh \
  --organization myorg \
  --repository myrepo

# 2. Install dependencies
codacy-cli install

# 3. Run analysis and upload results
codacy-cli analyze --format sarif --output results.sarif
codacy-cli upload \
  --sarif-path results.sarif \
  --commit-uuid $(git rev-parse HEAD) \
  --project-token $CODACY_PROJECT_TOKEN
```

### 3. Focused Security Scanning
```bash
# Run only security tools
codacy-cli analyze --tool trivy --format sarif --output security.sarif
codacy-cli analyze --tool semgrep --format sarif --output semgrep.sarif
```

### 4. Language-Specific Analysis
```bash
# JavaScript/TypeScript projects
codacy-cli analyze --tool eslint --fix

# Python projects  
codacy-cli analyze --tool pylint

# Java projects
codacy-cli analyze --tool pmd

# Go projects
codacy-cli analyze --tool revive
```

## CI/CD Integration

### GitHub Actions Example
```yaml
name: Code Quality

on: [push, pull_request]

jobs:
  codacy-analysis:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Codacy CLI
        run: |
          bash <(curl -Ls https://raw.githubusercontent.com/codacy/codacy-cli-v2/main/codacy-cli.sh)
          
      - name: Initialize Codacy
        run: |
          codacy-cli init --api-token ${{ secrets.CODACY_API_TOKEN }} \
            --provider gh --organization ${{ github.repository_owner }} \
            --repository ${{ github.event.repository.name }}
            
      - name: Install dependencies
        run: codacy-cli install
        
      - name: Run analysis
        run: |
          codacy-cli analyze --format sarif --output results.sarif
          
      - name: Upload to Codacy
        run: |
          codacy-cli upload --sarif-path results.sarif \
            --commit-uuid ${{ github.sha }} \
            --project-token ${{ secrets.CODACY_PROJECT_TOKEN }}
            
      - name: Upload SARIF to GitHub
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: results.sarif
```

### GitLab CI Example
```yaml
codacy-analysis:
  stage: test
  image: ubuntu:latest
  before_script:
    - apt-get update && apt-get install -y curl
    - bash <(curl -Ls https://raw.githubusercontent.com/codacy/codacy-cli-v2/main/codacy-cli.sh)
  script:
    - codacy-cli init --api-token $CODACY_API_TOKEN --provider gl --organization $CI_PROJECT_NAMESPACE --repository $CI_PROJECT_NAME
    - codacy-cli install
    - codacy-cli analyze --format sarif --output results.sarif
    - codacy-cli upload --sarif-path results.sarif --commit-uuid $CI_COMMIT_SHA --project-token $CODACY_PROJECT_TOKEN
  artifacts:
    reports:
      sast: results.sarif
```

### Docker Integration
```dockerfile
FROM ubuntu:latest

# Install Codacy CLI
RUN apt-get update && apt-get install -y curl && \
    bash <(curl -Ls https://raw.githubusercontent.com/codacy/codacy-cli-v2/main/codacy-cli.sh)

WORKDIR /code
COPY . .

# Run analysis
RUN codacy-cli init && \
    codacy-cli install && \
    codacy-cli analyze --format sarif --output /results/codacy.sarif
```

## Troubleshooting

### Common Issues

#### 1. Configuration File Not Found
```
Error: No configuration file was found, execute init command first.
```
**Solution**: Run `codacy-cli init` to create the configuration.

#### 2. Tool Installation Failures
```
Error: Failed to install tool 'eslint'
```
**Solutions**:
- Check internet connectivity
- Verify disk space
- Try with custom registry: `codacy-cli install --registry <url>`
- Check logs in `.codacy/logs/codacy-cli.log`

#### 3. Runtime Version Conflicts
```
Error: Node.js version mismatch
```
**Solution**: The CLI manages its own runtimes. If conflicts occur:
- Delete `.codacy/runtimes/` directory
- Run `codacy-cli install` again

#### 4. Permission Issues (Linux/macOS)
```
Error: Permission denied
```
**Solutions**:
- Ensure write permissions to project directory
- Check `.codacy/` directory permissions
- Avoid running as root when possible

#### 5. WSL-Specific Issues
- Always use WSL terminal, not Windows Command Prompt
- Install required Linux tools: `sudo apt install curl tar`
- Ensure WSL 2 is being used

#### 6. Docker Credential Helper (macOS)
```
Error: docker-credential-osxkeychain not found
```
**Solution**: `brew install docker-credential-helper`

### Debugging

#### Enable Debug Logging
The CLI automatically logs to `.codacy/logs/codacy-cli.log`. Check this file for detailed error information.

#### Verbose Analysis
```bash
# Run with maximum verbosity
codacy-cli analyze --tool eslint 2>&1 | tee analysis.log
```

#### Configuration Validation
```bash
# Validate your configuration
codacy-cli config reset --dry-run  # (if this flag existed)

# Or manually check
cat .codacy/codacy.yaml
codacy-cli install  # Will show what needs to be installed
```

### Getting Help

1. **Check logs**: `.codacy/logs/codacy-cli.log`
2. **Validate config**: Ensure `.codacy/codacy.yaml` is valid YAML
3. **Test connectivity**: Verify internet access for tool downloads
4. **Check permissions**: Ensure write access to project directory
5. **Update CLI**: Run `codacy-cli update` for latest fixes

## Best Practices

### Project Setup
1. **Initialize early**: Run `codacy-cli init` when setting up new projects
2. **Use discovery**: Let `codacy-cli config discover` detect your languages
3. **Version control**: Commit `.codacy/` directory (except `logs/`)
4. **Pin versions**: Use specific tool versions for reproducibility

### Configuration Management
1. **Start simple**: Begin with default configurations, customize gradually
2. **Tool-specific configs**: Leverage existing ESLint, PyLint configs when available
3. **Language mapping**: Review `languages-config.yaml` for accuracy
4. **Regular updates**: Keep tool versions current for latest rules

### CI/CD Integration
1. **Cache installations**: Cache `.codacy/runtimes/` and `.codacy/tools/` in CI
2. **Parallel execution**: Run different tools in parallel jobs when possible
3. **Fail fast**: Configure CI to fail on critical security issues
4. **Upload results**: Always upload SARIF results for tracking trends

### Performance Optimization
1. **Selective analysis**: Use `--tool` flag for faster feedback loops
2. **File targeting**: Analyze specific paths instead of entire repository
3. **Incremental analysis**: Focus on changed files in CI
4. **Resource limits**: Monitor memory usage for large repositories

### Security Considerations
1. **Token management**: Use environment variables for API tokens
2. **Secret scanning**: Leverage Trivy for vulnerability detection
3. **Regular updates**: Keep security tools updated
4. **Review findings**: Don't auto-fix security issues without review

### Team Collaboration
1. **Shared configuration**: Use remote mode for consistent team settings
2. **Documentation**: Document custom configurations and exceptions
3. **Training**: Ensure team understands tool outputs and fixes
4. **Gradual adoption**: Introduce tools incrementally to avoid overwhelming

### Maintenance
1. **Regular updates**: Update CLI and tools monthly
2. **Log monitoring**: Periodically check logs for issues
3. **Configuration review**: Review and update configurations quarterly
4. **Performance monitoring**: Track analysis time and resource usage

---

## Quick Reference Card

### Essential Commands
```bash
# Setup
codacy-cli init
codacy-cli config discover .
codacy-cli install

# Analysis
codacy-cli analyze
codacy-cli analyze --tool eslint --fix
codacy-cli analyze --format sarif --output results.sarif

# Upload
codacy-cli upload -s results.sarif -c $(git rev-parse HEAD) -t $TOKEN

# Maintenance
codacy-cli update
codacy-cli config reset
```

### Key Files
- `.codacy/codacy.yaml` - Main configuration
- `.codacy/tools-configs/languages-config.yaml` - Language mapping
- `.codacy/logs/codacy-cli.log` - Debug logs
- Tool configs: `eslint.config.mjs`, `pylintrc`, `pmd.xml`, etc.

### Environment Variables
- `CODACY_API_TOKEN` - API token for cloud integration
- `CODACY_PROJECT_TOKEN` - Project-specific token
- `CODACY_CLI_V2_VERSION` - Pin CLI version
- `CODACY_CLI_V2_TMP_FOLDER` - Custom cache directory

This comprehensive guide covers all aspects of using Codacy CLI v2 effectively. For the latest updates and additional examples, visit the [official repository](https://github.com/codacy/codacy-cli-v2).

