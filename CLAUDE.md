# CLAUDE.md - AI Assistant Guide

This document provides comprehensive guidance for AI assistants working with the DevOps Essentials Sample App repository.

## Repository Overview

**Project Name**: devops-essentials-sample-app
**Purpose**: Simple sample application for DevOps Essentials training labs
**Type**: Static HTML application
**Build System**: Gradle 4.6+
**CI/CD**: Jenkins Pipeline

This is an intentionally simple application designed for learning DevOps practices including version control, CI/CD pipelines, and deployment strategies.

## Project Structure

```
devops-essentials-sample-app/
├── .git/                    # Git repository data
├── .gitignore              # Git ignore patterns
├── README.md               # Basic project description
├── Jenkinsfile             # Jenkins pipeline configuration
├── build.gradle            # Gradle build configuration
├── settings.gradle         # Gradle project settings
├── gradlew                 # Gradle wrapper script (Unix)
├── gradlew.bat             # Gradle wrapper script (Windows)
├── gradle/                 # Gradle wrapper files
└── src/
    └── index.html          # Main application file
```

### Key Files

- **src/index.html**: The only application file - a simple HTML page displaying "DevOps is awesome!"
- **build.gradle**: Contains test and build tasks with lab grading logic
- **Jenkinsfile**: Three-stage pipeline (Build → DeployToStage → DeployToProd)
- **.gitignore**: Ignores `.DS_Store`, `.gradle`, and `src/*.state` files

## Build System

### Gradle Tasks

**Test Task** (`./gradlew test`):
- Validates that `src/index.html` exists
- Fails the build if the file is missing

**Build Task** (`./gradlew build`):
- Depends on test task (runs test first)
- Contains lab grading logic that checks index.html content
- Creates state files (`*.state`) in src/ directory to track lab progress:
  - `firstDeploy.state`: Created when content is "DevOps is awesome!" (first deploy)
  - `brokenDeploy.state`: Created when content is "DevOps is awesbfgdsfkjh!" (broken deploy)
  - `rollbackDeploy.state`: Created when rolling back to working version
  - `unknown.state`: Created for other content

### Running Builds Locally

```bash
# Run tests only
./gradlew test

# Run full build (includes test)
./gradlew build
```

## CI/CD Pipeline

### Jenkins Pipeline Stages

The Jenkinsfile defines a three-stage pipeline:

#### 1. Build Stage
- **Runs on**: All branches
- **Actions**:
  - Runs `./gradlew build`
  - Archives `src/index.html` as a build artifact
- **Purpose**: Validate that the application builds successfully

#### 2. DeployToStage Stage
- **Runs on**: `master` branch only
- **Actions**:
  - Deploys to staging server via SSH
  - Uses `sshPublisher` with credentials from Jenkins
  - Transfers all files from `src/` directory
- **Credentials**: Requires `cloud_user_pw` credential ID in Jenkins
- **Server**: Configured as 'staging' in Jenkins SSH configuration

#### 3. DeployToProd Stage
- **Runs on**: `master` branch only
- **Actions**:
  - Requires manual approval: "Does the staging environment look OK?"
  - Uses milestone to prevent concurrent deployments
  - Deploys to production server via SSH
- **Credentials**: Requires `cloud_user_pw` credential ID in Jenkins
- **Server**: Configured as 'production' in Jenkins SSH configuration

### Deployment Strategy

The pipeline implements a **staged deployment** with manual gates:
1. Automated deployment to staging (on master branch)
2. Manual verification step
3. Manual approval required for production deployment

## Git Workflow

### Branch Strategy

- **master**: Main branch, triggers staging and production deployments
- **Feature Branches**: Used for development (e.g., `new-feature`, `broken-feature`)
- **Pull Requests**: Features are merged to master via PRs

### Development Workflow

1. Create feature branch from master
2. Make changes to `src/index.html` or configuration files
3. Commit changes with descriptive messages
4. Push to remote repository
5. Create pull request to master
6. Review and merge PR
7. Jenkins pipeline automatically builds and deploys to staging
8. Manual approval triggers production deployment

### Git Conventions

Based on commit history:
- Commit messages are lowercase and concise
- PRs are used for merging features
- Branches are named descriptively (e.g., `new-feature`, `broken-feature`)

## Development Guidelines for AI Assistants

### Code Modifications

When modifying this repository:

1. **HTML Changes**: The main application file is `src/index.html`
   - Keep it simple - it's a demonstration app
   - The content "DevOps is awesome!" is the expected "working" state
   - Other content may trigger different lab states

2. **Build Configuration**: Avoid modifying `build.gradle` unless explicitly requested
   - The lab grading logic is intentional
   - Changes may break the training lab experience

3. **Pipeline Changes**: Be cautious with Jenkinsfile modifications
   - The three-stage pipeline is a learning example
   - Credentials and server configurations are environment-specific

### Testing Changes

Always test changes locally before committing:

```bash
# Test that index.html exists
./gradlew test

# Run full build including state file generation
./gradlew build

# Check what state was created
ls -la src/*.state
```

### Commit Guidelines

1. **Write clear commit messages**: Describe what changed and why
2. **Test before committing**: Run `./gradlew build` to validate
3. **Keep changes focused**: One logical change per commit
4. **Check git status**: Ensure you're not committing generated files

Excluded from commits (per .gitignore):
- `.gradle/` directory
- `src/*.state` files (generated by build)
- `.DS_Store` files

### Branch Management

When working on this repository:

1. **Always work on feature branches** (never commit directly to master)
2. **Branch naming**: Use descriptive names (e.g., `fix-styling`, `update-content`)
3. **Claude branches**: When working as Claude, use branches starting with `claude/`
4. **Push after changes**: Always push your branch after making commits

### Common Tasks

**Updating the application content**:
```bash
# Edit the HTML file
vim src/index.html

# Test the change
./gradlew build

# Commit and push
git add src/index.html
git commit -m "update application content"
git push -u origin <branch-name>
```

**Fixing a broken deployment**:
The lab includes scenarios where content is intentionally broken:
- Look for typos or incorrect content in index.html
- Restore to "DevOps is awesome!" for the working state
- Check that build completes successfully

## Environment Requirements

### Local Development
- **Java**: Required for Gradle wrapper
- **Git**: Version control
- **Text Editor**: For editing HTML and configuration files

### CI/CD Environment
- **Jenkins**: CI/CD server with:
  - SSH Publisher plugin
  - Configured staging and production servers
  - Credential: `cloud_user_pw` for SSH authentication

## Troubleshooting

### Build Failures

**Test fails**: `src/index.html` doesn't exist
- Solution: Ensure the file exists and path is correct

**Build fails**: Syntax error in build.gradle
- Solution: Validate Groovy syntax, check for typos

### Deployment Issues

**Stage/Prod deployment skipped**:
- Verify you're on the master branch
- Check Jenkins logs for conditional evaluation

**SSH deployment fails**:
- Verify SSH credentials are configured in Jenkins
- Check that staging/production server configs exist
- Ensure network connectivity to deployment servers

### Git Issues

**Push rejected**:
- Pull latest changes first: `git pull origin <branch>`
- Resolve any merge conflicts
- Push again

**Wrong branch**:
- Check current branch: `git branch`
- Switch branches: `git checkout <branch-name>`

## Lab Context

This repository is designed for learning:

1. **Version Control**: Git branching, merging, pull requests
2. **Build Automation**: Gradle builds, testing, artifacts
3. **CI/CD Pipelines**: Multi-stage Jenkins pipeline
4. **Deployment Strategies**: Staging → Manual approval → Production
5. **Rollbacks**: Reverting to previous working versions

The application intentionally includes scenarios for:
- Successful deployments
- Broken deployments (typos in content)
- Rollbacks to previous versions

State files track progress through these scenarios for lab grading purposes.

## Best Practices for AI Assistants

1. **Understand the context**: This is a training lab, not a production application
2. **Preserve lab functionality**: Don't "improve" the simplicity away
3. **Test changes**: Always run `./gradlew build` before committing
4. **Follow git workflow**: Use feature branches and PRs
5. **Document changes**: Update this file if you make structural changes
6. **Be explicit**: When making changes, explain what and why
7. **Respect .gitignore**: Don't commit generated files or IDE configurations
8. **Consider the learning objective**: Changes should support DevOps learning

## Questions to Ask Users

Before making changes, consider asking:

- "Are you working through a specific lab exercise?" (to understand context)
- "Should I preserve the lab grading logic?" (before modifying build.gradle)
- "What branch should I work on?" (to ensure proper workflow)
- "Do you want me to create a PR after changes?" (to follow git workflow)
- "Should I test the deployment pipeline?" (if Jenkins is available)

## Additional Notes

- The master branch name is referenced in Jenkinsfile (not main)
- SSH deployments require pre-configured Jenkins credentials
- The application is intentionally minimal for educational purposes
- State files are temporary and should not be committed to git

---

**Last Updated**: 2025-11-21
**Repository**: nycjay01/devops-essentials-sample-app
**Maintained for**: DevOps Essentials Training Labs
