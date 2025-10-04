# Project Implementation Guidelines

## Agent Communication

You can use various development tools and agents for command execution in terminal, such as:
- Forge
- ./acli rovodev run
- qodo -m=claude-4-sonnet

## Key Documentation

**Important:** Read the Key Takeaways.md file for comprehensive details about the company's standards and practices. These should be consistently applied across the entire organization.

## Analysis Requirements

When tasks require detailed analysis:
1. Create dedicated analysis files
2. Document findings comprehensively
3. Use appropriate agents for specialized work (e.g., Claude 4 Opus for complex analysis)

## Development Rules

### Core Principles

1. **Use 'project_rules' as the Knowledge Base**
   - Always refer to 'project_rules' for project context
   - Do not code outside the provided context
   - This folder contains fundamental rules and guidelines
   - Check this folder before proceeding with any coding

2. **Verify Information**
   - Always verify information from context before presenting
   - Do not make assumptions or speculate without clear evidence

3. **Follow 'implementation-plan.mdc' for Feature Development**
   - Strictly follow steps outlined in implementation-plan.mdc
   - Complete each step in sequence
   - Update with "Done" and two-line summary after each step
   - Maintain clear work log for transparency

### Coding Standards

4. **File-by-File Changes**
   - Make changes incrementally
   - Allow review opportunities between changes

5. **Communication Style**
   - No apologies in responses
   - No understanding feedback in comments/documentation
   - No whitespace-only suggestions
   - No unnecessary summaries unless requested
   - No inventions beyond explicit requests
   - No unnecessary confirmations for provided information

6. **Code Preservation**
   - Preserve existing code and functionalities
   - Pay attention to existing structures
   - Don't remove unrelated code

7. **Edit Practices**
   - Provide all edits in single chunks
   - No multi-step instructions for same file
   - No implementation verification requests for visible context
   - Provide automated tests instead of manual verification requests

8. **Documentation**
   - Provide real file links, not context-generated ones
   - Check context-generated files for current contents
   - Don't discuss current implementation unless requested

### Code Quality Standards

9. **Naming Conventions**
   - Use explicit, descriptive variable names
   - Avoid short, ambiguous names

10. **Consistency**
    - Follow existing project coding style
    - Maintain consistency across codebase

11. **Performance**
    - Prioritize performance in changes
    - Consider optimization opportunities

12. **Security**
    - Apply security-first approach
    - Consider security implications for all changes

13. **Testing**
    - Include appropriate unit tests
    - Ensure adequate test coverage

14. **Error Handling**
    - Implement robust error handling
    - Include proper logging mechanisms

15. **Design Principles**
    - Follow modular design principles
    - Improve maintainability and reusability

16. **Compatibility**
    - Ensure version compatibility
    - Suggest alternatives for conflicts

17. **Code Clarity**
    - Avoid magic numbers
    - Use named constants
    - Consider edge cases
    - Include assertions to validate assumptions

## Project Context

The Origin Alpha Management System is designed as a comprehensive printing business management solution. All implementations should align with:

- Printing industry workflows
- Multi-tenant SaaS architecture
- Scalability requirements
- User experience priorities

## Implementation Workflow

1. **Planning Phase**
   - Review requirements
   - Check existing implementations
   - Plan approach based on project rules

2. **Development Phase**
   - Implement features incrementally
   - Follow coding standards
   - Document progress in implementation-plan.mdc

3. **Testing Phase**
   - Write comprehensive tests
   - Handle edge cases
   - Validate security

4. **Documentation Phase**
   - Update relevant documentation
   - Create user guides if needed
   - Document API changes

## Notes

- If project_rules folder doesn't exist, create it comprehensively
- Maintain detailed logs of all implementation steps
- Focus on business-specific requirements for printing operations
