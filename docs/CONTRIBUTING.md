# MPAS-A Contributors Guide

Welcome to the Model for Prediction Across Scales - Atmosphere (MPAS-A) project! **This guide is designed to help you contribute effectively to the MPAS-A codebase,** whether you're adding new physics schemes, modifying the dynamical core, or integrating new analysis tools. 

If you're trying to build and run the existing MPAS-A code, then the [MPAS-A User guide](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide.html) is probably a better resource for you. Thank you!

## Table of Contents

### Quick Start
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Codebase Structure](#codebase-structure)

### Code Standards
- [Fortran Guidelines](#fortran-guidelines)
- [Things Not to Do](#things-not-to-do)
- [Testing](#testing)
- [Adding Documentation](#adding-documentation)

### Development Tasks
- [Starting a New Feature](#starting-a-new-feature)
- [Fixing a Bug](#fixing-a-bug)
- [Optimizing Modules](#optimizing-modules)
- [Adding GPU Acceleration](#adding-gpu-acceleration)

### Collaboration
- [Commit Messages](#commit-messages)
- [Pull Requests](#pull-requests)
- [Code Review](#code-review)
- [Getting Help](#getting-help)

### Resources
- [Official Resources](#official-resources)
- [Development Resources](#development-resources)
- [Code of Conduct](#code-of-conduct)

---

## Getting Started

### What is MPAS-A?

The Model for Prediction Across Scales - Atmosphere (MPAS-A) is a non-hydrostatic atmosphere model and a key component within the broader MPAS family of Earth-system models. All MPAS models share a common characteristic: they use centroidal Voronoi tessellations for their horizontal meshes, which underpins the development of a shared software framework.

### The MPAS Framework

The MPAS framework provides essential infrastructure for Earth-system model development, including:
- High-level data types
- Communication routines
- Input/Output (I/O) routines
- A high-level driver program
- Infrastructure for parallel execution
- Time management and block decomposition

### Collaborative Development

MPAS is a collaborative project with primary development partners being:
- **Los Alamos National Laboratory (LANL)**: Ocean, land-ice, and sea-ice models
- **National Center for Atmospheric Research (NCAR)**: Atmospheric model (MPAS-A)

### Licensing

The software is open source and copyrighted under a BSD license, making it freely available for use and redistribution under specific conditions.

---

## Development Workflow

### Prerequisites

Before contributing, ensure you have:

- **Version Control**: Git and GitHub familiarity
- **Fortran Programming**: Understanding of modern Fortran (2003/2008 standards)
- **MPI**: Required for parallel execution (MPICH, OpenMPI, MVAPICH2, MPT)
- **I/O Libraries**:
  - NetCDF (version 4.4.x or later)
  - Parallel-NetCDF (version 1.8.x or later)
  - PIO (Parallel I/O library, latest PIO 2.x)
- **METIS** (Optional): For graph partitioning

### Environment Setup and Building

For detailed instructions on setting up your development environment and building MPAS-A, please refer to the [MPAS Atmosphere User's Guide](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide.html). The User's Guide provides comprehensive information on:

![MPAS-A Build Process](assets/mpas-build-process.png)
*Typical MPAS-A build workflow showing compilation steps and dependencies*

- **Quick Start Guide**: High-level description of building and running MPAS-A
- **Building MPAS**: Complete setup instructions including prerequisites and compilation
- **Preparing Meshes**: Steps for preparing SCVT meshes for use in MPAS-Atmosphere
- **Running MPAS-A**: Detailed instructions for creating initial conditions and running the model

### Fork and Clone

1. **Fork the Repository**: Create a fork of the official [MPAS-Model repository](https://github.com/MPAS-Dev/MPAS-Model)
2. **Clone Your Fork**: `git clone https://github.com/yourusername/MPAS-Model.git`

---

## Codebase Structure

### High-Level Directory Structure

```
MPAS-Model/
├── src/
│   ├── driver/        # Main driver for MPAS in stand-alone mode
│   ├── external/      # External software for MPAS
│   ├── framework/     # MPAS Framework (shared routines)
│   ├── operators/     # MPAS Operators (shared)
│   ├── tools/         # Registry parser and input generation
│   └── core_*/        # Individual model cores
├── testing_and_setup/ # Configuration and test case tools
└── default_inputs/    # Default stream and namelist files
```

### MPAS-A Specific Structure (src/core_atmosphere)

```
core_atmosphere/
├── mpas_atm_core.F           # Main atmosphere core module
├── mpas_atm_core_interface.F # Interface for the atmosphere core
├── diagnostics/              # Diagnostic field modules
├── dynamics/                 # Atmospheric dynamics modules
└── physics/                  # Physics parameterization modules
```

### Key Components

- **Model Component**: Atmospheric dynamics and physics (`atmosphere_model`)
- **Initialization Component**: Generates initial conditions (`init_atmosphere_model`)

---

## Starting a New Feature

### 1. Create a Feature Branch

```bash
git checkout develop
git pull origin develop
git checkout -b feature/your-feature-name
```

### 2. Plan Your Feature

- **Identify the Scope**: Determine which components need modification
- **Consider Dependencies**: Check if your feature affects other modules
- **Design the Interface**: Plan how your feature integrates with existing code

### 3. Implementation Guidelines

- **Follow MPAS Patterns**: Use existing code as templates
- **Maintain Compatibility**: Ensure backward compatibility
- **Add Documentation**: Include comprehensive docstrings
- **Consider Performance**: Think about parallel execution and memory usage

### 4. Testing Strategy

- **Unit Tests**: Test individual functions/subroutines
- **Integration Tests**: Test feature within the larger system
- **Performance Tests**: Ensure no significant performance regression

### 5. Documentation Updates

- Update relevant documentation
- Add examples if applicable
- Update user guides if user-facing

---

## Fixing a Bug

### 1. Reproduce the Bug

- **Create a Minimal Test Case**: Isolate the issue
- **Document Steps**: Write clear reproduction steps
- **Identify Root Cause**: Understand why the bug occurs

### 2. Create a Bug Fix Branch

```bash
git checkout develop
git pull origin develop
git checkout -b bugfix/describe-the-bug
```

### 3. Implement the Fix

- **Minimal Changes**: Make only necessary modifications
- **Preserve Functionality**: Ensure fix doesn't break other features
- **Add Tests**: Include tests to prevent regression

### 4. Verify the Fix

- **Test the Fix**: Ensure bug is resolved
- **Run Existing Tests**: Verify no regressions
- **Test Edge Cases**: Check boundary conditions

---

## Optimizing Modules

### 1. Profile the Code

- **Identify Bottlenecks**: Use profiling tools to find slow sections
- **Measure Performance**: Establish baseline performance metrics
- **Focus on Hotspots**: Prioritize optimization efforts

### 2. Optimization Strategies

#### Memory Optimization
- **Reduce Memory Allocations**: Minimize dynamic allocations
- **Improve Data Locality**: Optimize memory access patterns
- **Use Appropriate Data Types**: Choose efficient data representations

#### Computational Optimization
- **Vectorization**: Enable compiler vectorization
- **Loop Optimization**: Unroll loops, reduce dependencies
- **Algorithm Improvements**: Use more efficient algorithms

#### Parallel Optimization
- **MPI Optimization**: Optimize communication patterns
- **OpenMP Optimization**: Improve shared-memory parallelism
- **Load Balancing**: Ensure even work distribution

### 3. Testing Optimizations

- **Performance Tests**: Measure speedup/throughput improvements
- **Correctness Tests**: Ensure results remain unchanged
- **Scalability Tests**: Test across different problem sizes

---

## Adding GPU Acceleration

### 1. Understand GPU Programming

- **OpenACC**: Directive-based approach for portability
- **CUDA Fortran**: NVIDIA-specific approach for explicit control
- **Data Management**: Understand host-device memory transfers

### 2. Identify Target Code

- **Compute-Intensive Loops**: Focus on loops with significant computation
- **Data-Parallel Operations**: Look for operations that can be parallelized
- **Memory Access Patterns**: Consider data locality on GPU

### 3. OpenACC Implementation

```fortran
!$acc parallel loop
do i = 1, n
    ! computation
end do
!$acc end parallel loop
```

### 4. Data Management

```fortran
!$acc enter data copyin(array)
! computation
!$acc exit data copyout(array)
```

### 5. Testing and Optimization

- **Correctness**: Verify results match CPU implementation
- **Performance**: Measure speedup and identify bottlenecks
- **Memory Usage**: Monitor GPU memory consumption

---

## Fortran Guidelines

### Modern Fortran Standards

- **Use Fortran 2003/2008**: Leverage modern language features
- **Avoid Legacy Code**: Refactor Fortran 77/90 code when possible
- **Follow Standards**: Ensure compiler portability

### Naming Conventions

- **Lowercase for Constructs**: Use lowercase for `do`, `subroutine`, `module`, etc.
- **Mathematical Notation**: Use short notation for math variables (e.g., `Ylm`, `Gamma`)
- **Clear Names**: Use descriptive names for subroutines and variables

### Code Structure

```fortran
module example_module
    implicit none
    private
    public :: example_subroutine
    
contains
    
    subroutine example_subroutine(input, output)
        ! Brief description
        ! Author: Your Name
        ! Date: YYYY-MM-DD
        ! Details: Detailed explanation
        
        implicit none
        real, intent(in) :: input
        real, intent(out) :: output
        
        ! Implementation
        
    end subroutine example_subroutine
    
end module example_module
```

### Best Practices

- **Use Modules**: Organize code into logical modules
- **Explicit Interfaces**: Use `implicit none` and explicit interfaces
- **Documentation**: Include comprehensive docstrings
- **Error Handling**: Implement proper error checking

---

## Commit Messages

### Why Commit Messages Matter

People on this project read your commit messages! You are not shouting into a void.

When you're in the middle of coding, the process may seem incredibly memorable and vivid, but those details can slip away like a dream. Your commit message is the best time to capture what you did while it's fresh. 

- **You're about to forget**: The debugging process fades quickly from memory
- **Nobody else knows your process**: Your colleagues weren't there during your debugging journey
- **Future you will thank you**: When you return to this code in 6 months, you'll need those details
- **It's greppable history**: Well-structured messages make searching git history powerful

### The 80-Character Rule

The first 80 characters of your commit message are crucial for greppable logs. This line should contain the most important identifying information:

- **Variable names** that were changed
- **Module names** that were modified
- **Specific compiler/version** information
- **Key functionality** affected
- **Critical debugging details**

### Commit Message Structure

```
Short summary (max 80 characters)
│
├─ Variable names: temperature, pressure
├─ Module: mpas_atm_core, mpas_physics
├─ Compiler: gfortran-11.3.0, ifort-2021.5.0
└─ Key info: GPU acceleration, memory optimization

Detailed explanation of the work done:
- What was the problem you were solving?
- What debugging steps did you take?
- What compiler flags or environment variables were critical?
- What performance improvements were achieved?
- What edge cases did you discover?
- What would you do differently next time?
```

### Examples of Good Commit Messages

#### Example 1: Bug Fix
```
Fix memory leak in mpas_atm_core temperature calculation (gfortran-11.3.0)

The temperature array in mpas_atm_core was not being properly deallocated
in the cleanup routine, causing memory leaks during long runs. This was
discovered when running 72-hour simulations on Cheyenne with 1000+ cores.

Debugging process:
1. Used valgrind to identify the leak location
2. Found that deallocate() was called but array was already deallocated
3. Root cause: double deallocation due to shared pointer logic
4. Solution: Added null check before deallocation

Performance impact: Eliminated 2GB/hour memory leak
Compiler flags used: -g -O2 -fcheck=all
Test case: CONUS_12km_72h simulation
```

#### Example 2: Performance Optimization
```
Optimize OpenACC data transfer in mpas_physics microphysics (nvfortran-22.7)

Reduced GPU data transfer overhead by 40% in Thompson microphysics scheme
by implementing asynchronous data movement and kernel fusion.

Key changes:
- Fused separate kernels for rain/snow/graupel calculations
- Added async data transfers with proper synchronization
- Optimized memory layout for GPU access patterns

Performance results:
- Data transfer time: 2.3s → 1.4s (40% reduction)
- Total kernel time: 8.7s → 7.1s (18% reduction)
- Memory bandwidth: 180 GB/s → 220 GB/s

Compiler flags: -acc -gpu=cc80 -Minfo=accel
Hardware: NVIDIA A100, 8 GPUs, 32 MPI ranks
Test case: Hurricane simulation, 3km resolution
```

#### Example 3: Feature Addition
```
Add GPU-accelerated advection scheme to mpas_operators (ifort-2021.5.0)

Implemented OpenACC version of monotonic advection scheme for better
performance on GPU architectures while maintaining bit-for-bit accuracy.

Implementation details:
- Ported mpas_tracer_advection_mono.F to use OpenACC directives
- Added data management for tracer arrays and mesh variables
- Implemented proper boundary condition handling for GPU kernels
- Added fallback to CPU version for unsupported configurations

Validation:
- Bit-for-bit accuracy compared to CPU version
- Performance: 3.2x speedup on V100 GPUs
- Memory usage: 15% increase due to device arrays

Dependencies: OpenACC 3.0+, PGI/NVIDIA compilers
Test cases: All standard MPAS test suite passes
```

### What to Include in Detailed Explanations

#### Problem Context
- What was the original issue or requirement?
- What symptoms were you seeing?

#### Debugging Journey
- What tools did you use? (gdb, valgrind, profilers, etc.)
- What compiler flags were critical?
- What environment variables mattered?
- What test cases revealed the issue?

#### Technical Details
- Specific variable names and their roles
- Module dependencies and interactions
- Memory layout considerations
- Performance characteristics
- Edge cases discovered

#### Lessons Learned
- What would you do differently next time?
- What patterns should others follow?
- What gotchas should be avoided?
- What documentation is missing?

### Anti-Patterns to Avoid

#### ❌ Vague Messages
```bash
# Bad
git commit -m "Fix bug"
git commit -m "Update code"
git commit -m "Changes"
```

#### ❌ Missing Context
```bash
# Bad - no debugging info
git commit -m "Fix temperature calculation"
```

#### ❌ No Technical Details
```bash
# Bad - no compiler/environment info
git commit -m "Add GPU support"
```


### Grep-Friendly Patterns

Structure your messages to be easily searchable:

```bash
# Search for specific variables
git log --grep="temperature"

# Search for specific modules
git log --grep="mpas_atm_core"

# Search for compiler issues
git log --grep="gfortran"

# Search for performance improvements
git log --grep="optimize\|performance\|speedup"

# Search for GPU-related changes
git log --grep="GPU\|OpenACC\|CUDA"
```

### Remember: You're Writing for Future You

When you write a commit message, imagine yourself 6 months from now:
- Will you understand what you did?
- Will you remember why you made those specific choices?
- Will you know what to avoid if you encounter similar issues?
- Will your colleagues understand the context and reasoning?

The few extra minutes you spend writing a detailed commit message will save hours of debugging and re-discovery in the future.

---

## Adding Documentation

### FORD Documentation Style

Use the FORD tool for generating documentation:

```fortran
!***********************************************************************
!
! routine subroutine_name
!
!> \brief Brief description of the routine
!> \author Author Name
!> \date Date of creation/last update
!> \details Detailed explanation of the routine's purpose and functionality
!
!-----------------------------------------------------------------------
```

### Documentation Requirements

- **Public Entities**: Document all public subroutines and functions
- **Parameters**: Document all input/output parameters
- **Examples**: Include usage examples when helpful
- **References**: Cite relevant papers or documentation

### Documentation Updates

- **Keep Current**: Update documentation with code changes
- **Be Clear**: Write for your future self and other developers
- **Include Context**: Explain why, not just what

---

## Testing

### Test Types

#### Unit Tests
- Test individual functions/subroutines
- Mock dependencies when necessary
- Test edge cases and error conditions

#### Integration Tests
- Test module interactions
- Verify data flow between components
- Test with realistic data

#### Performance Tests
- Measure execution time
- Monitor memory usage
- Test scalability

### Test Structure

```fortran
program test_example
    use example_module
    implicit none
    
    call test_basic_functionality()
    call test_edge_cases()
    call test_error_conditions()
    
contains
    
    subroutine test_basic_functionality()
        ! Test normal operation
    end subroutine test_basic_functionality
    
end program test_example
```

### Test Integration

- **Automated Testing**: Integrate with CI/CD pipeline
- **Test Coverage**: Aim for comprehensive coverage
- **Regression Testing**: Prevent reintroduction of bugs

---

## Pull Requests

### 1. Prepare Your Changes

- **Complete Implementation**: Ensure feature is fully implemented
- **Add Tests**: Include appropriate tests
- **Update Documentation**: Update relevant documentation
- **Code Review**: Self-review your changes

### 2. Create Pull Request

- **Clear Title**: Use descriptive, concise title
- **Detailed Description**: Explain what, why, and how
- **Link Issues**: Reference related issues
- **Screenshots**: Include visual aids if applicable

### 3. Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Performance improvement
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Performance tests pass

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] Tests added/updated
```

### 4. Review Process

- **Address Feedback**: Respond to reviewer comments
- **Update PR**: Make requested changes
- **Be Patient**: Allow time for thorough review

---

## Code Review

### Review Checklist

#### Code Quality
- [ ] Code follows style guidelines
- [ ] Logic is correct and efficient
- [ ] Error handling is appropriate
- [ ] Documentation is adequate

#### Testing
- [ ] Tests are comprehensive
- [ ] Tests actually test the code
- [ ] Edge cases are covered
- [ ] Performance implications considered

#### Integration
- [ ] Changes integrate well with existing code
- [ ] No breaking changes without justification
- [ ] Backward compatibility maintained
- [ ] Dependencies properly managed

### Review Guidelines

- **Be Constructive**: Provide helpful feedback
- **Be Specific**: Point to exact lines and issues
- **Be Respectful**: Maintain professional tone
- **Be Thorough**: Check all aspects of the code

---

## Getting Help

### Before Reporting

1. **Search Existing Issues**: Check if issue already exists
2. **Verify the Issue**: Ensure it's reproducible
3. **Gather Information**: Collect relevant details

### Issue Template

```markdown
## Bug Report / Feature Request

### Description
Clear description of the issue or feature request

### Steps to Reproduce (for bugs)
1. Step one
2. Step two
3. Step three

### Expected Behavior
What should happen

### Actual Behavior
What actually happens

### Environment
- OS: [e.g., Linux, macOS, Windows]
- Compiler: [e.g., gfortran, ifort]
- Version: [e.g., MPAS-A v8.2.0]

### Additional Context
Any other relevant information
```

### Issue Guidelines

- **Be Clear**: Provide clear, detailed descriptions
- **Be Specific**: Include version numbers and environment details
- **Be Patient**: Allow time for response and resolution

---

## Things Not to Do

### Code Quality Issues

#### ❌ Don't: Add unnecessary comments on every line
```fortran
! Bad example
real :: temperature  ! This is the temperature variable
temperature = 25.0   ! Set temperature to 25 degrees
if (temperature > 30.0) then  ! Check if temperature is hot
    ! Do something
end if
```

**Why not**: Over-commenting clutters the code and makes it harder to read. Comments should explain *why*, not *what*.

#### ❌ Don't: Add "Modified by" comments with initials
```fortran
! Bad example
! Modified by J.S. on 2024-01-15
! Modified by A.B. on 2024-01-20
```

**Why not**: Git already tracks who modified what and when. These comments become outdated and create noise in the code.

#### ❌ Don't: Use inconsistent indentation
```fortran
! Bad example
if (condition) then
  call subroutine1()
    call subroutine2()
  call subroutine3()
end if
```

**Why not**: Inconsistent formatting makes code harder to read and maintain. Use consistent indentation (typically 2-4 spaces).

#### ❌ Don't: Leave trailing whitespace
```fortran
! Bad example
real :: variable    
integer :: counter  
```

**Why not**: Trailing whitespace makes git diffs harder to read and can cause issues with some tools.

#### ❌ Don't: Use magic numbers
```fortran
! Bad example
if (temperature > 273.15) then
    ! Do something
end if
```

**Why not**: Magic numbers make code hard to understand and maintain. Use named constants instead.

#### ❌ Don't: Ignore compiler warnings
```fortran
! Bad example
real :: unused_variable
! Code that doesn't use unused_variable
```

**Why not**: Compiler warnings often indicate real issues. Fix warnings rather than ignoring them.

### Git and Version Control Issues

#### ❌ Don't: Commit directly to main/develop branches
```bash
# Bad example
git checkout develop
git add .
git commit -m "Fix bug"
git push origin develop
```

**Why not**: Direct commits to main branches bypass the review process and can introduce bugs.

#### ❌ Don't: Write vague commit messages
```bash
# Bad example
git commit -m "Fix stuff"
git commit -m "Update"
git commit -m "Changes"
```

**Why not**: Vague messages make it hard to understand what changed and why. Use descriptive commit messages.

#### ❌ Don't: Force push to shared branches
```bash
# Bad example
git push --force origin develop
```

**Why not**: Force pushing rewrites history and can cause problems for other developers.

### Performance and Optimization Issues

#### ❌ Don't: Optimize prematurely
```fortran
! Bad example - premature optimization
!$acc parallel loop
do i = 1, 3
    result = a(i) + b(i)
end do
!$acc end parallel loop
```

**Why not**: Premature optimization can make code more complex without providing benefits. Profile first, then optimize.

#### ❌ Don't: Ignore memory leaks
```fortran
! Bad example
allocate(array(1000))
! Use array
! Forget to deallocate
```

**Why not**: Memory leaks can cause programs to run out of memory, especially in long-running simulations.

#### ❌ Don't: Use inefficient algorithms
```fortran
! Bad example - O(n²) when O(n) is possible
do i = 1, n
    do j = 1, n
        if (i == j) then
            result(i) = array(i)
        end if
    end do
end do
```

**Why not**: Inefficient algorithms can significantly impact performance, especially in scientific computing.

### Documentation Issues

#### ❌ Don't: Leave outdated documentation
```fortran
! Bad example
! This subroutine calculates temperature
! Author: Old Developer
! Date: 2020-01-01
! Note: This is the old implementation
subroutine new_temperature_calculation()
    ! New implementation
end subroutine
```

**Why not**: Outdated documentation is worse than no documentation because it provides incorrect information.

#### ❌ Don't: Document obvious things
```fortran
! Bad example
! This function adds two numbers
function add(a, b)
    real, intent(in) :: a, b
    add = a + b
end function
```

**Why not**: Obvious documentation adds noise. Focus on explaining complex logic and business rules.

### Testing Issues

#### ❌ Don't: Skip testing
```fortran
! Bad example - no tests for critical function
function critical_calculation(input)
    ! Complex calculation
end function
```

**Why not**: Untested code is unreliable. Critical functions especially need comprehensive testing.

#### ❌ Don't: Write tests that always pass
```fortran
! Bad example
subroutine test_always_passes()
    logical :: result
    result = .true.
    call assert_true(result)
end subroutine
```

**Why not**: Tests that always pass don't verify correctness. Write meaningful tests that can fail.

#### ❌ Don't: Ignore test failures
```bash
# Bad example
make test
# Some tests fail, but continue anyway
```

**Why not**: Test failures indicate real problems. Fix failing tests before proceeding.

### Communication Issues

#### ❌ Don't: Submit incomplete pull requests
```markdown
# Bad example - PR description
"Here's my code"
```

**Why not**: Incomplete PRs waste reviewers' time. Provide context, description, and testing information.

#### ❌ Don't: Ignore feedback
```markdown
# Bad example - response to review
"Thanks for the feedback" # But doesn't address any issues
```

**Why not**: Ignoring feedback prevents improvement and can lead to PR rejection.

#### ❌ Don't: Be defensive about criticism
```markdown
# Bad example - defensive response
"That's not a bug, it's a feature!"
```

**Why not**: Defensive responses prevent learning and improvement. Consider feedback objectively.

---

## Code of Conduct

### Our Commitment

We are committed to providing a welcoming and inclusive environment for all contributors, regardless of background, experience level, gender, gender identity, sexual orientation, disability, personal appearance, race, ethnicity, age, religion, or nationality.

### Expected Behavior

- **Be Respectful**: Treat everyone with respect and dignity
- **Be Inclusive**: Welcome newcomers and help them learn
- **Be Constructive**: Provide helpful, constructive feedback
- **Be Patient**: Remember that everyone has different experience levels
- **Be Professional**: Maintain professional communication

### Unacceptable Behavior

- Harassment, discrimination, or intimidation
- Personal attacks or trolling
- Inappropriate language or imagery
- Spam or off-topic discussions
- Violation of privacy or confidentiality

### Reporting Issues

If you experience or witness unacceptable behavior, please report it to the project maintainers. All reports will be handled confidentially and investigated promptly.

---

## Resources

### Official Resources

- **GitHub Repository**: [MPAS-Dev/MPAS-Model](https://github.com/MPAS-Dev/MPAS-Model)
- **User's Guide**: [MPAS Atmosphere User's Guide](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide.html) - Complete setup, building, and running instructions
- **Documentation**: [MPAS Documentation](https://mpas-dev.github.io/)
- **Support Forum**: [MPAS-Atmosphere Support](http://forum.mmm.ucar.edu/)

### Development Resources

- **FORD Documentation**: [FORD Wiki](https://github.com/Fortran-FOSS-Programmers/ford/wiki)
- **Modern Fortran**: [Fortran-lang.org](https://fortran-lang.org/)
- **MPI Documentation**: [MPI Forum](https://www.mpi-forum.org/)
- **OpenACC Documentation**: [OpenACC.org](https://www.openacc.org/)

### Getting Help

1. **Check Documentation**: Review relevant guides and documentation
2. **Search Issues**: Look for similar issues in the GitHub repository
3. **Ask Questions**: Use the support forum or GitHub discussions
4. **Contribute Back**: Share your solutions with the community

### Acknowledgments

We thank all contributors who have helped make MPAS-A what it is today. Your contributions, whether code, documentation, testing, or feedback, are invaluable to the project's success.

---

*This guide is a living document. If you have suggestions for improvements, please open an issue or submit a pull request.*
