# Contributing to Inventory Management Accelerator

## Welcome Contributors!

We're excited that you're interested in contributing to the Inventory Management Accelerator! This document provides guidelines and information for contributors to help maintain code quality and project consistency.

## Code of Conduct

### Our Pledge
We pledge to make participation in our project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Our Standards
**Positive behaviors include:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

**Unacceptable behaviors include:**
- Trolling, insulting/derogatory comments, and personal attacks
- Public or private harassment
- Publishing private information without permission
- Other conduct which could reasonably be considered inappropriate

## Getting Started

### Prerequisites
- .NET 8.0 SDK or later
- Docker Desktop with Kubernetes enabled
- Git 2.30 or later
- Visual Studio 2022 or VS Code with C# extension
- PostgreSQL 15+ for local development
- Node.js 18+ for frontend development (if applicable)

### Development Environment Setup

1. **Fork and Clone Repository**
   ```bash
   git clone https://github.com/your-username/inventory_mgmt.git
   cd inventory_mgmt
   ```

2. **Set Up Local Development Environment**
   ```bash
   # Copy environment configuration
   cp .env.example .env.local
   
   # Edit configuration for local development
   nano .env.local
   ```

3. **Install Dependencies**
   ```bash
   # Backend dependencies
   dotnet restore
   
   # Frontend dependencies (if applicable)
   cd src/WebUI
   npm install
   ```

4. **Start Local Infrastructure**
   ```bash
   docker-compose -f docker-compose.dev.yml up -d
   ```

5. **Run Database Migrations**
   ```bash
   dotnet ef database update --project src/Infrastructure
   ```

6. **Start Development Services**
   ```bash
   # Start API
   dotnet run --project src/API
   
   # Start Web UI (separate terminal)
   cd src/WebUI && npm start
   ```

## Contribution Types

### 🐛 Bug Reports
Help us improve by reporting issues you encounter.

**Before submitting:**
- Search existing issues to avoid duplicates
- Test with the latest version
- Gather relevant information (logs, screenshots, environment details)

**Bug Report Template:**
```markdown
## Bug Description
Brief description of the issue

## Steps to Reproduce
1. Navigate to...
2. Click on...
3. See error...

## Expected Behavior
What should happen

## Actual Behavior  
What actually happens

## Environment
- OS: [e.g., Windows 11, Ubuntu 22.04]
- Browser: [e.g., Chrome 115.0]
- Version: [e.g., v1.2.3]

## Additional Context
Screenshots, logs, or other relevant information
```

### ✨ Feature Requests
Suggest new features or enhancements to existing functionality.

**Feature Request Template:**
```markdown
## Feature Summary
Brief description of the proposed feature

## Use Case
Describe the business scenario or user need

## Proposed Solution
Your suggested approach to implement this feature

## Alternative Solutions
Other approaches you've considered

## Additional Context
Mockups, references, or related information
```

### 🔧 Code Contributions
Contribute code improvements, new features, or bug fixes.

**Before coding:**
1. Check existing issues and discussions
2. Create an issue for significant changes
3. Fork the repository
4. Create a feature branch

### 📚 Documentation
Help improve documentation, examples, and tutorials.

## Development Guidelines

### Coding Standards

#### C# Code Style
- Follow Microsoft C# coding conventions
- Use PascalCase for class names, methods, and properties
- Use camelCase for private fields and local variables
- Use meaningful, descriptive names
- Add XML documentation for public APIs

```csharp
/// <summary>
/// Calculates inventory reorder point based on demand and lead time.
/// </summary>
/// <param name="averageDemand">Average daily demand for the item</param>
/// <param name="leadTimeDays">Supplier lead time in days</param>
/// <param name="safetyStock">Safety stock quantity</param>
/// <returns>Recommended reorder point quantity</returns>
public int CalculateReorderPoint(decimal averageDemand, int leadTimeDays, int safetyStock)
{
    if (averageDemand < 0) throw new ArgumentException(nameof(averageDemand));
    if (leadTimeDays < 0) throw new ArgumentException(nameof(leadTimeDays));
    
    var reorderPoint = (averageDemand * leadTimeDays) + safetyStock;
    return (int)Math.Ceiling(reorderPoint);
}
```

#### Architecture Principles
- **Domain-Driven Design**: Organize code around business domains
- **SOLID Principles**: Single responsibility, open/closed, etc.
- **Clean Architecture**: Separate concerns with clear layer boundaries
- **CQRS Pattern**: Separate command and query responsibilities
- **Event Sourcing**: Capture state changes as immutable events

#### Error Handling
- Use structured exception handling
- Implement proper logging with correlation IDs
- Return meaningful error messages
- Handle transient failures with retry policies

```csharp
try 
{
    var result = await inventoryService.UpdateStockAsync(itemId, quantity);
    logger.LogInformation("Stock updated for {ItemId}: {Quantity}", itemId, quantity);
    return Ok(result);
}
catch (ItemNotFoundException ex)
{
    logger.LogWarning("Item not found: {ItemId}", itemId);
    return NotFound($"Item {itemId} not found");
}
catch (InsufficientStockException ex)
{
    logger.LogWarning("Insufficient stock for {ItemId}: requested {Requested}, available {Available}", 
        itemId, quantity, ex.AvailableQuantity);
    return BadRequest("Insufficient stock available");
}
catch (Exception ex)
{
    logger.LogError(ex, "Failed to update stock for {ItemId}", itemId);
    return StatusCode(500, "An error occurred while updating stock");
}
```

### Testing Requirements

#### Unit Tests
- Write unit tests for all business logic
- Achieve minimum 80% code coverage
- Use meaningful test names that describe the scenario
- Follow Arrange-Act-Assert (AAA) pattern

```csharp
[Test]
public void CalculateReorderPoint_WithValidInputs_ReturnsCorrectQuantity()
{
    // Arrange
    var calculator = new ReorderPointCalculator();
    var averageDemand = 10.5m;
    var leadTimeDays = 7;
    var safetyStock = 15;
    var expectedReorderPoint = 89; // (10.5 * 7) + 15 = 88.5, rounded up to 89
    
    // Act
    var result = calculator.CalculateReorderPoint(averageDemand, leadTimeDays, safetyStock);
    
    // Assert
    Assert.AreEqual(expectedReorderPoint, result);
}
```

#### Integration Tests
- Test API endpoints with realistic scenarios
- Validate database interactions
- Test external service integrations with mocks
- Ensure proper error handling

#### Performance Tests
- Load test critical endpoints
- Validate database performance under load
- Test memory usage and garbage collection
- Monitor resource utilization

### Git Workflow

#### Branch Naming Convention
- `feature/ISSUE-123-add-inventory-tracking`
- `bugfix/ISSUE-456-fix-stock-calculation`
- `hotfix/critical-security-update`
- `docs/update-api-documentation`

#### Commit Message Format
Use [Conventional Commits](https://www.conventionalcommits.org/) format:

```
type(scope): description

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Build system or tool changes

**Examples:**
```
feat(inventory): add support for serial number tracking

Implement serial number tracking for high-value items including:
- Database schema updates
- API endpoints for serial number management
- Unit tests and integration tests

Closes #123
```

### Pull Request Process

#### Before Submitting
1. **Update your branch** with latest main
   ```bash
   git checkout main
   git pull upstream main
   git checkout feature/your-branch
   git rebase main
   ```

2. **Run tests** and ensure they pass
   ```bash
   dotnet test
   ```

3. **Run linting** and code formatting
   ```bash
   dotnet format
   ```

4. **Update documentation** if needed

#### Pull Request Template
```markdown
## Description
Brief description of changes and motivation

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)  
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Related Issues
Closes #123
Related to #456

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] Performance impact assessed

## Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Comments added for hard-to-understand areas
- [ ] Documentation updated
- [ ] No new warnings introduced

## Screenshots/Demo
(If applicable, add screenshots or demo links)
```

#### Review Process
1. **Automated Checks**: All CI/CD checks must pass
2. **Peer Review**: At least one maintainer approval required
3. **Architecture Review**: Required for significant changes
4. **Security Review**: Required for security-related changes
5. **Performance Review**: Required for performance-critical changes

## Community Guidelines

### Communication Channels
- **GitHub Issues**: Bug reports, feature requests, project discussions
- **GitHub Discussions**: General questions, ideas, and community chat  
- **Slack Channel**: Real-time development coordination (invite-only)
- **Monthly Video Calls**: Community meetings and architecture discussions

### Recognition
We appreciate all contributions! Contributors will be:
- Listed in the project README
- Included in release notes for significant contributions
- Eligible for contributor swag and recognition
- Invited to special contributor events

## Release Process

### Version Numbering
We follow [Semantic Versioning](https://semver.org/):
- **Major (X.0.0)**: Breaking changes
- **Minor (X.Y.0)**: New features, backwards compatible
- **Patch (X.Y.Z)**: Bug fixes, backwards compatible

### Release Cycle
- **Major releases**: Quarterly
- **Minor releases**: Monthly
- **Patch releases**: As needed for critical fixes
- **Pre-releases**: Weekly development builds

## Getting Help

### Documentation Resources
- **Architecture Documentation**: `/architecture` folder
- **API Documentation**: Available at `/api/docs` endpoint  
- **User Guides**: `/docs` folder
- **Video Tutorials**: Project YouTube channel

### Support Channels
- **GitHub Issues**: Technical questions and bug reports
- **Stack Overflow**: Tag questions with `inventory-mgmt-accelerator`
- **Community Slack**: Real-time help and discussions
- **Office Hours**: Weekly virtual office hours with maintainers

### Mentorship Program
New contributors can request mentorship for:
- Understanding the codebase
- Guidance on first contributions
- Architecture and design reviews
- Career development in open source

## License

By contributing to this project, you agree that your contributions will be licensed under the same license as the project (MIT License).

## Thank You!

Thank you for your interest in contributing to the Inventory Management Accelerator. Your contributions help make this project better for everyone in the community!

For questions about contributing, please reach out to the maintainer team through GitHub issues or our community channels.