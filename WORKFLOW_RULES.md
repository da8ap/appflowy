# Mandatory Workflow Rules - ENFORCED

## CRITICAL: These rules are MANDATORY, not optional

### Rule 1: Speed is NOT a priority
- **NEVER prioritize speed over correctness**
- Take as long as needed - hours, days, whatever it takes
- User does NOT care about speed - only correctness matters
- Quality over speed ALWAYS

### Rule 2: Research First - Verify, Don't Assume
- **ALWAYS research before implementing**
- Check official documentation
- Verify paths, commands, and structures exist
- Show findings before coding
- NEVER assume - always verify

### Rule 3: Test Every Build Before Committing
- **ALWAYS run `docker build` on every Dockerfile before committing**
- Test each service independently
- Fix build errors before moving on
- NEVER commit untested code
- Show proof of successful builds

### Rule 4: Verify Compatibility and Structure
- **ALWAYS verify before using**
- Confirm file/directory paths exist
- Check dependency version compatibility
- Verify build commands work for the specific project
- Test in isolation before integrating

### Rule 5: Show Proof of Success
- **ALWAYS provide evidence**
- Successful build logs as proof
- Verify outputs exist (binaries, build artifacts)
- Test full stack before considering done
- Show working endpoints/functionality

### Rule 6: Never Assume - Always Verify
- **NEVER assume anything**
- Don't assume a project supports a platform without checking
- Don't assume paths or commands without verification
- Don't assume dependencies work together without testing
- Verify each assumption with proof

### Rule 7: Diagnose Before Fixing
- **ALWAYS find root cause first**
- Add debug output to see what's happening
- Capture actual error messages (not just exit codes)
- Verify each step works before moving on
- Don't try solutions until you know the problem

### Rule 8: Follow the Process - No Shortcuts
1. Research → show findings
2. Diagnose → identify root cause
3. Test → show proof it works
4. Verify → confirm each component
5. Commit → only after proof of success

## Enforcement
- These rules are MANDATORY for ALL work
- No exceptions
- No shortcuts
- No "quick fixes"
- Quality and correctness ONLY

## Consequences of Not Following Rules
- Wasted time
- Broken deployments
- User frustration
- Loss of trust
- Need to redo work correctly

## Reminder
**SPEED DOES NOT MATTER. CORRECTNESS IS EVERYTHING.**

