# Minimal Code Interpreter - Implementation Plan

## One Line Summary
A minimal code interpreter/advanced data analysis tool using llama.cpp (local LLM) with sandboxed Python code execution.

---

## Project Overview

### Core Features
- **LLM Backend**: llama.cpp running locally, accessed via OpenAI SDK
- **Sandboxed Code Execution**: REPL-style Python execution (no containers/VMs)
- **Error Recovery**: Automatic code correction when LLM generates errors
- **CLI Interface**: Interactive command-line mode (no frontend)
- **Data Analysis**: CSV file processing with natural language responses

---

## Architecture

```
main.py (CLI Entry Point)
    ├── config.py (LLM endpoint config)
    ├── llm_client.py (OpenAI SDK wrapper for llama.cpp)
    ├── code_executor.py (Sandboxed Python execution)
    ├── code_generator.py (Prompt engineering + code generation)
    ├── error_handler.py (Error parsing + retry logic)
    ├── result_formatter.py (Natural language summarization)
    └── data_processor.py (CSV loading + validation)
```

---

## 1. Assumptions

| # | Assumption |
|---|------------|
| 1 | llama.cpp server is running locally (endpoint to be discovered) |
| 2 | Python 3.8+ with `openai`, `pandas`, `numpy` libraries available |
| 3 | User has basic CLI knowledge to run the script |
| 4 | Code execution will be single-threaded, no multiprocessing |
| 5 | CSV files are well-formed with headers |
| 6 | Generated code will be Python 3 compatible |
| 7 | Max execution time per code: 30 seconds |
| 8 | Max retries before failure: 3 attempts |

---

## 2. Unknowns

| # | Unknown | Impact |
|---|---------|--------|
| 1 | Exact llama.cpp endpoint structure | Requires flexibility in config |
| 2 | Model context window size | May need prompt chunking for complex queries |
| 3 | How well model handles code errors | May require prompt engineering tweaks |
| 4 | Execution environment limitations | Unknown system calls/libraries available |
| 5 | Network access from sandbox | Should restrict by default |

---

## 3. What Must Work (Critical Path)

✅ **Essential Components:**
- CSV file loading and validation
- LLM connection to llama.cpp endpoint
- Code generation from natural language questions
- Safe code execution with timeout
- Error parsing and retry mechanism
- Natural language result summarization

---

## 4. What Must NOT Happen (Safety)

❌ **Forbidden Operations:**
- Network requests from executed code
- File system writes outside temp directory
- Infinite loops (enforce timeout)
- Access to sensitive system files
- Process spawning or subprocess calls
- Import of dangerous libraries (`os.system`, `subprocess`, `socket`, `requests`, etc.)

---

## 5. Success Criteria

✅ **Definition of Done:**
- [ ] Script runs without errors on test_data.csv
- [ ] Can answer at least 5 different question types:
  - Summary statistics (mean, median, max, min)
  - Group by aggregations (avg salary by department)
  - Filtering queries (employees earning > 80k)
  - Date-based queries (hire date analysis)
  - Custom calculations (bonus calculations)
- [ ] Error recovery works (invalid code auto-corrected)
- [ ] Results are in plain English (no code snippets)
- [ ] Execution timeout works (30s limit)
- [ ] All test cases pass

---

## 6. Failure Criteria

❌ **Failure Conditions:**
- LLM connection fails after 3 retries
- Code execution crashes sandbox (segfault)
- Timeout exceeded without result
- Invalid CSV format (no recovery)
- LLM returns non-code response (parsing error)
- Circular error correction (same error 3x)

---

## 7. Failure Modes & Recovery

| Mode | Trigger | Recovery |
|------|---------|----------|
| ConnectionError | llama.cpp offline | Retry 3x with exponential backoff |
| TimeoutError | Code hangs | Kill process, report timeout |
| SyntaxError | Bad code generation | Send error + context, regenerate |
| ImportError | Banned library used | Remove import, regenerate |
| KeyError | Column not found | Suggest valid columns, retry |
| InfiniteLoop | User query causes loop | Timeout kill, suggest simpler query |

---

## 8. Hardest Technical Component

**🎯 Code Error Recovery Loop**

**Why:** The LLM must understand its own mistakes, which requires:
- Precise error message parsing
- Context preservation between retries
- Prompt engineering to guide corrections
- Detection of infinite error loops

**Mitigation Strategy:**
- Store conversation history
- Provide explicit error context in prompts
- Add "common mistakes" guidance in system prompt
- Track error patterns to detect loops

---

## 9. Why It May Fail

1. **LLM Limitations**: Model may not understand pandas/numpy well
2. **Prompt Drift**: Multi-turn conversation loses context
3. **Sandbox Escape**: Code finds way to bypass restrictions
4. **Memory Limits**: Large CSV files crash executor
5. **Timeout Precision**: Python timeouts hard to enforce accurately
6. **Error Ambiguity**: Python errors may be unclear for LLM to fix

---

## 10. Testing Strategy

### Test Dataset
- **Primary**: `test_data.csv` (10 employees, 5 columns: id, name, department, salary, hire_date)

### Test Cases

| # | Question Type | Expected Output | Validation |
|---|---------------|-----------------|------------|
| 1 | "What is the average salary?" | Numeric value ~83,200 | Check calculation |
| 2 | "How many employees in Engineering?" | Count = 3 | Verify count |
| 3 | "Show me employees earning > 85k" | List names | Check filter logic |
| 4 | "Average salary by department" | Dict with 4 depts | Validate groups |
| 5 | "Who was hired most recently?" | Name + date | Verify date logic |
| 6 | Invalid file path | Clear error message | Check error handling |
| 7 | Nonsense question | Helpful suggestion | Check fallback |
| 8 | Timeout test (infinite loop) | Timeout error | Verify 30s limit |

---

## 11. Implementation Phases

### Phase 1: Core Infrastructure (Day 1)
- [ ] Set up project structure
- [ ] Implement `config.py` (LLM endpoint config)
- [ ] Implement `llm_client.py` (OpenAI SDK wrapper)
- [ ] Implement `data_processor.py` (CSV loading)
- [ ] Create basic CLI in `main.py`

### Phase 2: Code Generation & Execution (Day 1-2)
- [ ] Implement `code_generator.py` (prompt templates)
- [ ] Implement `code_executor.py` (sandbox with timeouts)
- [ ] Add banned library detection
- [ ] Implement result formatting

### Phase 3: Error Recovery (Day 2)
- [ ] Implement `error_handler.py` (parsing + retry)
- [ ] Add conversation history management
- [ ] Implement infinite loop detection
- [ ] Add logging system

### Phase 4: Testing & Polish (Day 2-3)
- [ ] Write comprehensive test suite
- [ ] Create additional test CSVs
- [ ] Add logging and debugging
- [ ] Performance optimization
- [ ] Documentation

---

## 12. Safety Implementation Details

```python
# Safety restrictions for code executor:
BANNED_MODULES = ['os', 'subprocess', 'socket', 'requests', 'urllib', 
                  'sys', 'ctypes', 'pickle', 'eval', 'exec', 'compile']
ALLOWED_MODULES = ['pandas', 'numpy', 'math', 'statistics', 'datetime', 'collections']
MAX_MEMORY_MB = 512
EXECUTION_TIMEOUT = 30  # seconds
MAX_RETRIES = 3
```

---

## 13. Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| LLM generates dangerous code | Medium | High | Static analysis + banned modules |
| Infinite loops in generated code | High | Medium | Hard timeout enforcement |
| Memory exhaustion | Low | High | Memory limits + pandas chunking |
| llama.cpp endpoint instability | Medium | Medium | Retry logic + graceful degradation |
| Poor LLM code quality | High | Medium | Multiple retry attempts + prompt tuning |

---

## 14. Dependencies

```
openai>=1.0.0
pandas>=2.0.0
numpy>=1.24.0
```

---

## 15. Configuration File Structure

```python
# config.py
CONFIG = {
    "llm_endpoint": "http://localhost:8080/v1",
    "api_key": "not-needed-for-llama-cpp",  # or empty string
    "model_name": "llama.cpp",
    "max_retries": 3,
    "execution_timeout": 30,
    "max_memory_mb": 512,
    "banned_modules": ["os", "subprocess", "socket", "requests", "urllib", 
                       "sys", "ctypes", "pickle", "eval", "exec", "compile"],
    "allowed_modules": ["pandas", "numpy", "math", "statistics", "datetime", "collections"]
}
```

---

## 16. Prompt Engineering Strategy

### System Prompt
```
You are a Python data analysis expert. Your task is to generate Python code that:
1. Uses pandas for data manipulation
2. Provides clear, commented code
3. Returns results in a variable called 'result'
4. Handles errors gracefully
5. Does NOT use any banned modules

Format your response as:

```python
# Your code here
import pandas as pd
# ... code ...
result = analysis_output
```
```

### Error Recovery Prompt
```
The previous code had an error:

{error_message}

Please fix the code and try again. Common issues:
- Check column names match the CSV
- Use correct pandas syntax
- Ensure proper indentation
```

---

## 17. User Flow

```
1. User runs: python main.py
2. Prompt: "Please enter the path to your CSV file:"
3. User provides: test_data.csv
4. Prompt: "What would you like to know about this data?"
5. User asks: "What is the average salary by department?"
6. System:
   - Loads CSV
   - Generates code via LLM
   - Executes code safely
   - Formats result in natural language
7. Output: "The average salaries by department are:..."
8. Prompt: "Anything else you'd like to know?"
```

---

## 18. Next Steps

### Before Implementation
1. ✅ Confirm llama.cpp endpoint (test locally)
2. ✅ Verify test environment has required dependencies
3. ✅ Test with existing test_data.csv

### Immediate Actions
1. Create project structure
2. Implement config.py with endpoint detection
3. Build llm_client.py with connection testing
4. Create code_executor.py with safety checks
5. Write main.py CLI flow
6. Test end-to-end with sample questions

---

## 19. Success Metrics

- **Functional**: 80%+ success rate on test questions
- **Performance**: < 5 seconds per question (excluding LLM wait time)
- **Safety**: 0 successful sandbox escapes in testing
- **Usability**: Clear error messages, no cryptic stack traces

---

## 20. Future Enhancements (Post-MVP)

- [ ] Multiple file upload support
- [ ] Chart/visualization generation
- [ ] Query history/saved questions
- [ ] Export results to CSV/Excel
- [ ] Support for JSON/Excel file formats
- [ ] Web UI interface
- [ ] Batch processing mode
- [ ] Custom function definitions

---

*This plan was generated based on the requirements in Idea.txt and refined with best practices for data analysis tools.*
