COMMON OPENCLAW MISTAKES & HOW TO FIX THEM

## 1. Inefficient Tool Use
Problem: Making too many separate API calls instead of batching
Solution: Combine related operations, use bulk operations when available

## 2. Poor Memory Patterns
Problem: Not writing important context to memory files
Solution: Use memory/YYYY-MM-DD.md for daily logs, MEMORY.md for long-term retention

## 3. Skill Design Problems
Problem: Creating skills that are too broad or too narrow
Solution: Keep skills focused on single, well-defined capabilities

## 4. Security Mistakes
Problem: Running destructive commands without confirmation
Solution: Always ask before rm, trash, or external actions

## 5. Cron Job Issues
Problem: Setting up too many cron jobs or incorrect schedules
Solution: Use heartbeat for flexible checks, cron for exact timing needs

## 6. Workflow Bottlenecks
Problem: Sequential execution when parallel is possible
Solution: Identify independent tasks and run concurrently

## 7. Ignoring Context Files
Problem: Not reading SOUL.md, USER.md at session start
Solution: Always load project context before taking action

## 8. Over-Automation
Problem: Automating things that need human judgment
Solution: Reserve automation for repetitive, rule-based tasks

## 9. Poor Error Handling
Problem: Not handling API failures gracefully
Solution: Implement retries with exponential backoff

## 10. Missing Security Gates
Problem: No confirmation for irreversible actions
Solution: Add confirmation steps for deletions, external sends
