# OrchestrateOS Docker Installation Test Suite

## What This Does

Automated end-to-end testing for Docker container installations. Turns a 3-hour manual debugging session into a 2-minute automated test run.

## Quick Start

### 1. Run Full Test Suite
```bash
cd /path/to/orchestrate-beta-sandbox/tests
python3 orchestrate_docker_test.py
```

### 2. Run Only Critical Tests (Quick Mode)
```bash
python3 orchestrate_docker_test.py --quick
```

### 3. Use Custom Test File
```bash
python3 orchestrate_docker_test.py --tests-file my_tests.json
```

## Autonomous Testing with Claude

This is designed to be run autonomously via `claude_assistant`. Just assign the task:

```json
{
  "tool_name": "claude_assistant",
  "action": "assign_task",
  "params": {
    "task_id": "validate_docker_install",
    "description": "Run the Docker installation test suite located at /opt/orchestrate-core-runtime/tests/orchestrate_docker_test.py and report results"
  }
}
```

Claude will:
1. Execute all tests
2. Generate comprehensive report
3. Return pass/fail status
4. Highlight critical failures

## What Gets Tested

### Critical Tests (Must Pass)
- ✅ Docker container health check
- ✅ System settings accessible
- ✅ Claude task queue exists
- ✅ Execution context loaded
- ✅ System settings NDJSON valid

### Non-Critical Tests (Nice to Have)
- JSON manager (create, read, update)
- File manager (create, read, check)
- Outline editor (search)
- Tool unlock status

## Test Configuration

Edit `orchestrate_docker_tests.json` to add/modify tests:

```json
{
  "name": "test_name",
  "description": "What this test does",
  "endpoint": "/execute_task",
  "payload": {
    "tool_name": "tool_name",
    "action": "action_name",
    "params": {}
  },
  "expected_status": "success",
  "critical": true
}
```

**Fields:**
- `name`: Unique test identifier
- `description`: Human-readable description
- `endpoint`: API endpoint to call
- `payload`: Request payload
- `expected_status`: Expected response status
- `expected_contains`: (Optional) String that must be in response
- `critical`: If true, failure indicates broken installation
- `skip_if_missing_credentials`: (Optional) Skip if credentials not configured

## Output Example

```
OrchestrateOS Docker Installation Test Suite
Started: 2025-11-08 16:00:00

Testing endpoint: https://supposedly-faithful-termite.ngrok-free.app
Total tests: 13

▶ Running: health_check
  Verify Docker container is running and accessible
  ✓ PASS (0.23s)

▶ Running: list_available_tools
  Get all available tools from system_settings
  ✓ PASS (0.18s)

...

============================================================
TEST RESULTS SUMMARY
============================================================

Total Tests:    13
Passed:         12
Failed:         1
Critical Fails: 0
Duration:       4.56s

============================================================
✅ ALL TESTS PASSED - INSTALLATION VERIFIED
============================================================
```

## Integration with Installer

Add to your installer script:

```bash
# After Docker installation completes
echo "Running validation tests..."
cd /opt/orchestrate-core-runtime/tests
python3 orchestrate_docker_test.py --quick

if [ $? -eq 0 ]; then
  echo "✅ Installation validated successfully"
else
  echo "❌ Installation validation failed - check logs"
  exit 1
fi
```

## Autonomous Task Assignment

Instead of manual testing, assign it to Claude:

```bash
# Via container API
curl -X POST http://localhost:8000/execute_task \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "claude_assistant",
    "action": "assign_task",
    "params": {
      "task_id": "test_installation",
      "description": "Execute /opt/orchestrate-core-runtime/tests/orchestrate_docker_test.py and report if all critical tests pass"
    }
  }'
```

Claude will run the tests and report back with results.

## Why This Matters

**Before:**
- 3 hours of manual debugging
- Human intervention required
- Inconsistent testing
- Hard to reproduce issues
- "It works on my machine" syndrome

**After:**
- 2 minutes automated testing
- Zero human intervention
- Consistent, repeatable
- Clear pass/fail criteria
- Catches issues immediately

## Extending Tests

### Add a New Test

Edit `orchestrate_docker_tests.json`:

```json
{
  "name": "my_new_test",
  "description": "Test my new feature",
  "endpoint": "/execute_task",
  "payload": {
    "tool_name": "my_tool",
    "action": "my_action",
    "params": {
      "param1": "value1"
    }
  },
  "expected_status": "success",
  "critical": false
}
```

No code changes needed - just update the JSON.

## Troubleshooting

### Test fails: "Connection refused"
- Docker container not running
- Check: `docker ps | grep orchestrate`
- Start: `docker start <container_id>`

### Test fails: "Invalid ngrok URL"
- Update `ngrok_url` in `orchestrate_docker_tests.json`
- Check ngrok: `curl https://your-url.ngrok-free.app/health`

### All tests fail immediately
- Check if tests file exists and is valid JSON
- Run: `python3 -m json.tool orchestrate_docker_tests.json`

## Files

- `orchestrate_docker_test.py` - Main test runner
- `orchestrate_docker_tests.json` - Test configuration
- `README.md` - This file

---

**TL;DR**: Run `python3 orchestrate_docker_test.py` to validate your Docker installation in 2 minutes instead of debugging for 3 hours.
