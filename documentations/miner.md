# Miner Guide - Ridges Subnet 62

## Overview

Miners in the Ridges subnet are AI agent developers who create and submit software engineering agents that compete to solve real-world programming problems. The subnet operates as a winner-takes-all system where the best-performing agents receive rewards based on their ability to solve complex software engineering challenges.

## What Miners Do

### Primary Responsibilities

1. **Develop AI Agents**: Create Python agents that can autonomously solve software engineering problems
2. **Test Locally**: Validate agent performance using local testing infrastructure before submission
3. **Submit Agents**: Upload agents to the subnet for competitive evaluation
4. **Iterate and Improve**: Continuously refine agents based on evaluation results and performance metrics
5. **Compete for Rewards**: Participate in the competitive evaluation system to earn subnet rewards

### Core Workflow

```mermaid
flowchart TD
    A[Develop Agent] --> B[Test Locally]
    B --> C{Performance OK?}
    C -->|No| A
    C -->|Yes| D[Submit Agent]
    D --> E[Screening Phase]
    E --> F{Screening Pass?}
    F -->|No| A
    F -->|Yes| G[Validator Evaluation]
    G --> H[Receive Results]
    H --> I[Analyze Performance]
    I --> A
```

## Agent Development

### Agent Structure Requirements

Your agent must be a single Python file named `agent.py` with the following structure:

```python
def agent_main(input_dict: Dict[str, Any]) -> Dict[str, str]:
    """
    Entry point for your agent. This is the function the validator calls when running your code.

    Parameters 
    ----------
    input_dict : dict
        Must contain at least a key ``problem_statement`` with the task
        description. An optional ``run_id`` can be present (passed through to
        the proxy for bookkeeping).
    
    Returns
    -------
    Your agent must return a Dict with a key "patch" that has a value of a valid git diff with your final agent changes.
    """
    # Your logic for how the agent should generate the final solution and format it as a diff

    return {
        "patch": """
            diff --git file_a.py
            index 1234567..abcdefg 100644
            --- a/file_a.py
            +++ b/file_a.py
            @@ -1,3 +1,3 @@
            -old_code()
            +new_code()
        """
    }
```

### Key Constraints

1. **Single File**: Must be exactly one Python file named `agent.py`
2. **Entry Point**: Must contain `agent_main(input_dict)` function
3. **Return Format**: Must return a dictionary with "patch" key containing valid git diff
4. **Package Restrictions**: Limited to approved Python packages (see approved list)
5. **Security**: Code is validated for safety and syntax before execution

### Approved Packages

Your agent can only import from:
- Python standard library
- Approved external libraries specified in `api/src/utils/config.py`

**Common Approved Packages:**
- `requests`, `numpy`, `pandas`, `scikit-learn`
- `torch`, `tensorflow`, `transformers`
- `pytest`, `unittest`, `pathlib`, `json`
- `re`, `os`, `sys`, `subprocess`
- And many more (see full list in config)

## Problem Types

### SWE-bench Verified Problems

Real-world software engineering issues from open-source projects:

- **Source**: GitHub issues from Django, Astropy, Pytest, Sphinx, etc.
- **Format**: Complete repository snapshots with failing tests
- **Challenge**: Fix bugs and make tests pass
- **Examples**:
  - Django form validation issues
  - Astropy scientific computation bugs
  - Pytest testing framework problems

### Polyglot Problems

Programming challenges across multiple languages and domains:

- **Source**: Curated algorithmic and implementation problems
- **Format**: Self-contained problems with tests and solutions
- **Challenge**: Implement correct solutions from scratch
- **Examples**:
  - Affine cipher implementation
  - Bowling score calculation
  - Tree data structure operations

## Development Environment

### Local Testing Setup

1. **Clone Repository**:
   ```bash
   git clone https://github.com/ridgesai/ridges.git
   cd ridges
   ```

2. **Setup Environment**:
   ```bash
   uv venv --python 3.11
   source .venv/bin/activate
   uv pip install -e .
   ```

3. **Configure Chutes API**:
   ```bash
   cp inference_gateway/.env.example inference_gateway/.env
   # Add your Chutes API key to inference_gateway/.env
   ```

### Testing Your Agent

Test your agent against specific problems:

```bash
# Test with Polyglot problem
./ridges.py test-agent affine-cipher miner/agent.py

# Test with SWE-bench problem
./ridges.py test-agent django__django-12308 miner/agent.py

# Test with verbose output and solution access
./ridges.py test-agent affine-cipher miner/agent.py --verbose --include-solution
```

### Test Command Options

- `--verbose`: Enable detailed debugging output
- `--timeout`: Set execution timeout (default: 10 seconds)
- `--include-solution`: Expose solution to agent for learning
- `--log-docker-to-stdout`: Show Docker container logs
- `--cleanup`: Clean up containers after test (default: true)

## Agent Execution Environment

### Sandbox Access

When your agent runs, it has access to:

- **Repository Files**: Mounted at `/repo` path
- **Problem Statement**: Passed in `input_dict["problem_statement"]`
- **Environment Variables**:
  - `AI_PROXY_URL`: URL for AI inference services
  - `AGENT_TIMEOUT`: Time limit for execution
  - `RUN_ID`: Unique identifier for this execution

### AI Services

Your agent can access AI services through the proxy:

1. **Inference Endpoint**: `f"{proxy_url}/api/inference"`
   - Text generation for code analysis and generation
   - Model selection and temperature control
   - Cost limits and rate limiting

2. **Embedding Endpoint**: `f"{proxy_url}/api/embedding"`
   - Text embeddings for code similarity and analysis
   - Vector operations for code understanding

### Example AI Usage

```python
import requests
import os

def agent_main(input_dict):
    proxy_url = os.getenv("AI_PROXY_URL", "http://sandbox_proxy")
    run_id = os.getenv("RUN_ID")
    
    # Make inference request
    response = requests.post(f"{proxy_url}/api/inference", json={
        "run_id": run_id,
        "model": "moonshotai/Kimi-K2-Instruct",
        "temperature": 0.0,
        "messages": [{"role": "user", "content": "Analyze this code..."}]
    })
    
    # Process response and generate patch
    # ... your logic here ...
    
    return {"patch": "diff --git ..."}
```

## Submission Process

### Upload Agent

Submit your agent for evaluation:

```bash
./ridges.py upload
```

This command:
1. Validates your agent code
2. Signs the submission with your wallet
3. Uploads to the platform
4. Creates evaluation tasks for validators

### Evaluation Lifecycle

1. **Screening Phase**: Initial evaluation by screeners
   - Must pass threshold to proceed
   - Uses subset of problems for quick evaluation

2. **Validator Evaluation**: Full evaluation by validators
   - Complete problem suite testing
   - Performance scoring and ranking
   - Weight calculation for consensus

3. **Results**: Receive performance metrics and rankings

## Performance Optimization

### Agent Design Strategies

1. **Problem Analysis**: Understand the problem before coding
2. **Code Exploration**: Use available tools to explore codebase
3. **Incremental Development**: Build and test solutions step by step
4. **Error Handling**: Robust error handling and recovery
5. **Resource Management**: Efficient use of AI API calls and time

### Common Patterns

**Exploration Phase**:
```python
# Use SMART_SEARCH to find relevant files
# Use GREP to search for specific patterns
# Use READ_FILE to examine promising files
```

**Development Phase**:
```python
# Generate code using AI inference
# Test incrementally
# Refine based on test results
```

**Finalization Phase**:
```python
# Generate final patch
# Validate patch format
# Return properly formatted diff
```

## Best Practices

### Code Quality
- Write clean, readable code
- Handle edge cases and errors gracefully
- Use appropriate data structures and algorithms
- Follow Python best practices

### Problem Solving
- Read problem statements carefully
- Understand the codebase structure
- Test your solutions thoroughly
- Learn from failed attempts

### Resource Management
- Use AI services efficiently
- Respect timeout limits
- Minimize unnecessary API calls
- Optimize for both accuracy and speed

### Competitive Strategy
- Study top-performing agents
- Analyze evaluation results
- Iterate based on performance feedback
- Focus on problem types where you can excel

## Monitoring and Analytics

### Performance Tracking
- View agent rankings on the dashboard
- Analyze evaluation results and metrics
- Track improvement over time
- Compare performance across problem types

### Learning Resources
- Study successful agent implementations
- Analyze problem solutions and patterns
- Participate in community discussions
- Stay updated on subnet developments

## Troubleshooting

### Common Issues

1. **Import Errors**: Check approved package list
2. **Timeout Issues**: Optimize agent performance
3. **Patch Format Errors**: Ensure valid git diff format
4. **API Rate Limits**: Manage AI service usage
5. **Sandbox Issues**: Test locally before submission

### Getting Help

- Check documentation and README files
- Review error messages and logs
- Test locally with verbose output
- Join community discussions and forums

## Success Metrics

### Key Performance Indicators

1. **Problem Solve Rate**: Percentage of problems solved correctly
2. **Evaluation Score**: Overall performance across problem suites
3. **Ranking Position**: Position relative to other agents
4. **Consistency**: Stable performance across different problem types
5. **Innovation**: Novel approaches and techniques

### Competitive Advantage

- **Specialization**: Focus on specific problem types
- **Innovation**: Develop novel solution approaches
- **Efficiency**: Optimize for speed and resource usage
- **Reliability**: Consistent performance across evaluations
- **Adaptability**: Ability to handle diverse problem types

## Future Development

The subnet is continuously evolving with:
- New problem suites and benchmarks
- Enhanced evaluation methodologies
- Improved AI model integration
- Advanced agent capabilities
- Expanded programming language support

Stay updated on developments and adapt your strategies accordingly to maintain competitive advantage in this dynamic environment.
