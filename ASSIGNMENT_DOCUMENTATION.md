# Assignment 3 - Complete Documentation

**Student Name**: [Naif Abdulaziz Alnahdi]  
**Student ID**: [445050281]  
**Date Submitted**: [30 April]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)
Entry 1 - [April 29, 6:00 PM]

What I implemented:
I started analyzing the given code and identified shared resources like counters and execution log.

Challenges encountered:
I noticed that multiple threads were updating variables like contextSwitchCount and totalWaitingTime at the same time which caused incorrect values.

How I solved it:
I decided to use ReentrantLock to protect these shared variables.

Testing approach:
Ran the program multiple times and compared results.

Time spent:
1 hour

Entry 2 - [April 30, 7:00 PM]

What I implemented:
Added locks to critical sections such as incrementing counters and updating execution log.

Challenges encountered:
I was confused about where exactly to place the lock, especially inside methods like incrementContextSwitch() and logExecution().

How I solved it:
I wrapped only the critical code inside lock and used try-finally to ensure unlock always happens.

Testing approach:
Printed values before and after adding locks to confirm correctness.

Time spent:
1.5 hours

Entry 3 - [April 30, 8:30 PM]

What I implemented:
Implemented Semaphore to control CPU access (only one thread runs at a time).

Challenges encountered:
At first, threads were still interfering because I forgot to release the semaphore.

How I solved it:
Used acquire() before execution and release() inside finally block.

Testing approach:
Checked that threads execute one by one and no overlap happens.

Time spent:
1 hour
## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:
First race condition is in shared counters like contextSwitchCount and totalWaitingTime. More than one thread can update them at the same time. This makes the value wrong because increment is not safe. Two threads

Second race condition is in executionLog. The code uses ArrayList which is not thread-safe. If multiple threads add at the same time, logs can mix or some messages disappear. This makes the output not correct
---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:
ReentrantLock is used to protect critical sections so only one thread accesses shared data at a time. Semaphore is used to control how many threads can access a resource.

In my code, I used Lock to protect counters and execution log. I used Semaphore to allow only one thread to use the CPU at a time.

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:
Deadlock is when threads wait for each other and never continue.

One prevention technique is using try-finally to always release locks. Another is avoiding multiple locks to prevent circular waiting. In my code, I used try-finally and avoided nested locks.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:
I used one lock for all shared variables. This is called coarse-grained locking. It is easier and safer because everything is protected. It reduces mistakes in synchronization.

But the problem is it reduces performance because threads wait even if they use different variables. Fine-grained locking is faster but more complicated. I chose this way because it is simpler and works correctly.

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: contextSwitchCount, completedProcessCount, totalWaitingTime

**Why they need protection**: These variables are shared between multiple threads. Each thread updates them during execution. Without protection, values can become incorrect (race condition).

**Synchronization mechanism used**: ReentrantLock

**Code snippet**:
```java
 public static void incrementContextSwitch() {
        contextSwitchLock.lock();
        try {
            contextSwitchCount++;
        } finally {
            contextSwitchLock.unlock();
        }
    }
```

**Justification**: 
I used a lock to make sure only one thread updates the variables at a time. This prevents race conditions and keeps the values correct.
---

### Critical Section #2: Execution Log

**What resource**: executionLog (ArrayList)

**Why it needs protection**: ArrayList is not thread-safe. If multiple threads add messages at the same time, the log can be corrupted or some messages may be lost.

**Synchronization mechanism used**: ReentrantLock

**Code snippet**:
```java
public static void logExecution(String message) {
        logLock.lock();
        try {
            executionLog.add(message);
        } finally {
            logLock.unlock();
        }
    }
```

**Justification**: 
Using a lock ensures that log messages are added in a safe and ordered way. This avoids mixed or missing outputs.
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To control how many threads can access the CPU at the same time

**Number of permits and why**: 1 permit to simulate a single CPU (only one thread executes at a time).

**Where implemented**: 
Inside the run() method in the Process class.

**Code snippet**:
```java


SharedResources.cpuSemaphore.acquire();
try {
    // process execution
} finally {
    SharedResources.cpuSemaphore.release();
}
```

**Effect on program behavior**: 
It makes sure that threads execute one by one instead of running together. This reduces interference and keeps execution correct.
---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
I ran the program 5 times using the same inputs. Each time I observed the final values such as total waiting time and completed processes.

**Results**: 
The results were consistent in all runs. The values of counters and logs were correct and did not change randomly.

**Why synchronization is necessary**: 
Without synchronization, race conditions may happen in shared counters and execution log. This can lead to incorrect values or mixed outputs. Since multiple threads access the same variables, protection is required.

**Conclusion**: Synchronization ensured stable and correct results across multiple runs

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: I ran the program with multiple threads updating the execution log at the same time.

**Results**: No exceptions occurred during execution. The program ran smoothly without errors.

**What this proves**: This shows that synchronization on executionLog is working correctly and prevents concurrent modification issues.

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: Total completed processes should equal number of processes. Waiting time should be calculated correctly.

**Actual values**: The output matched the expected values in all runs. Counters were accurate and consistent.

**Analysis**: This confirms that synchronization using locks works correctly and prevents incorrect updates.

---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

**Purpose**: To check how the scheduler behaves under different conditions.

**Results**: The program handled different values correctly without errors. The scheduling behavior changed as expected.

**What I learned**: I learned that synchronization still works correctly even when program conditions change.

---

## Part 5: Reflection and Learning

### What I learned about synchronization:
I learned that synchronization is very important when working with threads. Without it, shared variables can give wrong results because of race conditions. I understood that locks help protect critical sections so only one thread can access them at a time. I also learned how semaphores can control access to resources like CPU. Using fine-grained locks helped improve performance because threads can work in parallel. I faced some confusion at first about where to place locks, but testing helped me understand better. I also learned how deadlocks can happen and how to avoid them. Overall, synchronization made the program more correct and stable.

---

### Real-world applications:


**Example 1**: Banking systems where multiple users update account balance at the same time.

**Example 2**: Operating systems where multiple processes share CPU and resources.

---

### How I would explain synchronization to others:
Synchronization is like organizing access to a shared resource. If many people want to use the same thing, they must take turns. Without it, everything becomes messy and incorrect. Using locks is like giving one person access at a time. Using semaphore is like allowing limited number of people to enter. This makes the system work correctly and without problems.

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/Naif-Alnahdi-281/OS-Assignment3-Naif-Alnahdi.git

**Number of commits**: 

**Commit messages**: 
1. 
2. 
3. 
4. 

---

## Summary

**Total time spent on assignment**: 

**Key takeaways**: 
1. 
2. 
3. 

**Most challenging aspect**: 

**What I'm most proud of**: 

---

**End of Documentation**
