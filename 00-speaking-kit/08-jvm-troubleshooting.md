# 08 · Java and JVM troubleshooting

5 modules · Day 15–16 · Long version: [05-java-jvm-performance](../05-java-jvm-performance/jvm-troubleshooting.md)

> **This whole subject is [Theory] for you.** Use the bridge sentence:
> "I haven't analysed heap or thread dumps in my project, because on AEM as a Cloud Service Adobe collects them, but I understand the process and I've practised it on a local AEM." (Only say the last part after you do the lab in the long module.)

---

## M1 · JVM memory and garbage collection

**In simple words**
Objects live on the heap. Garbage collection removes objects nobody uses. If it can't free enough, the app slows down or crashes.

**Say this**
> AEM runs on the Java Virtual Machine. Objects are stored in the heap, which is shared by all threads, while each thread has its own stack for method calls. The heap has a young generation for new objects and an old generation for long-lived ones. Garbage collection removes objects that are no longer referenced; it pauses the application for short times, and long or frequent pauses make all requests slow. If memory after garbage collection keeps rising over time, it is a memory leak; if it is high only under load and then recovers, it is memory pressure.

**Keywords:** heap (objects, shared) · stack (per thread) · young + old · GC removes unreferenced · pauses slow everything · leak = keeps rising · pressure = recovers

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| Explain JVM memory. | Sentences 1–3 |
| What is garbage collection? | Sentence 4 |
| Leak vs high memory usage? | Sentence 5 |

**Practice:** Heap vs stack · How do you know it's a memory leak?

---

## M2 · OutOfMemoryError and heap dumps

**In simple words**
When the heap is full, Java throws OutOfMemoryError. A heap dump is a snapshot that shows which objects fill memory.

**The flow, step by step**
1. Note the **type** of error (heap space, metaspace, native thread).
2. **Collect** a heap dump (automatically on OOM, or `jcmd <pid> GC.heap_dump`); restore service.
3. Open it in **Eclipse MAT**.
4. **Leak Suspects** report → **Dominator Tree** (biggest retained size).
5. **Path to GC roots**: who keeps these objects alive?
6. Map the class to the **code/bundle** → fix → check memory is stable.

**Diagram script**
> "First I note the type of error and collect a heap dump. I open it in Eclipse MAT, look at the leak suspects and the dominator tree to find what holds the most memory, then follow the path to GC roots to see what keeps it alive. That points to the code that needs fixing."

**Say this**
> An OutOfMemoryError means the JVM could not get memory, most often because the heap is full. I first check the type of error and collect a heap dump, which can be automatic on the error or taken with `jcmd`. I analyse it in Eclipse MAT: the leak suspects report and the dominator tree show which objects hold the most memory, and the path to GC roots shows what keeps them alive. That usually points to code, like a cache without a limit or resource resolvers that are not closed. After the fix, I watch that memory after garbage collection stays stable.

**Keywords:** OOM type · heap dump (auto / jcmd) · Eclipse MAT · leak suspects · dominator tree · path to GC roots · fix + watch

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you analyse a heap dump? | Sentence 3 |
| AEM crashed with OOM. What do you do? | All |
| Common causes of memory leaks in AEM? | "Unbounded caches, unclosed resource resolvers or sessions, large queries loaded into memory." |
| In the cloud? | "Adobe collects it; as a TSE I request it internally." |

**Practice:** Steps to analyse an OOM · What is retained heap?

---

## M3 · Thread dumps

**In simple words**
A thread dump is a photo of what every thread is doing. Take several to see what is stuck.

**Say this**
> A thread dump shows what every thread in the JVM is doing at one moment, with its state and stack trace. I take several dumps, about 5 to 10 seconds apart, using `jcmd <pid> Thread.print` or `jstack`, so I can see which threads stay on the same line. The main states are RUNNABLE, BLOCKED waiting for a lock, WAITING and TIMED_WAITING. In AEM, request threads are named with the URL, so I can see which page is slow and which code line it is stuck in. Many BLOCKED threads point to lock contention, and the JVM prints deadlocks at the end of the dump.

**Keywords:** snapshot of threads · several dumps 5–10 s apart · jcmd / jstack · RUNNABLE · BLOCKED · WAITING · thread name = URL · deadlock at end

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is a thread dump? | Sentence 1 |
| How do you find a slow request? | Sentences 2, 4 |
| Explain thread states. | Sentence 3 |
| What is a deadlock? | "Two threads each hold a lock the other needs, so both wait forever." |

**Practice:** Why take several dumps? · What does BLOCKED mean?

---

## M4 · Slow AEM and high CPU

**In simple words**
Find where the time goes: cache, page rendering, queries, external calls, or garbage collection.

**The flow, step by step**
1. **Which pages, since when**: all or some?
2. **Cache**: are requests hitting the CDN/Dispatcher cache?
3. **request.log**: which URLs take long?
4. **error.log**: query traversal warnings, timeouts.
5. **GC log**: long pauses? (memory problem)
6. **Thread dumps** during slowness: where threads wait.
7. **High CPU**: `top -H` → thread ID → hex → match in thread dump.

**Diagram script**
> "First I check if it's all pages or some. Then the cache, because a low hit rate overloads Publish. Then the request log for slow URLs, the error log for slow queries, and the GC log for pauses. During the slowness I take thread dumps to see where threads are waiting. For high CPU, I find the busy thread with top and match it in the thread dump."

**Say this**
> For a slow AEM, I first check whether it's all pages or only some, and since when. I check the cache, because a low cache hit rate sends too much traffic to Publish. Then I look at the request log for slow URLs, the error log for slow queries without indexes, and the GC log for long pauses. During the slowness I take thread dumps to see where request threads are waiting, for example on a query, a lock or an external call. For high CPU, I use `top -H` to find the busy thread and match its ID in the thread dump.

**My example** [Hands-on] (Story 4)
> "A page with too many containers was too slow to render and gave a 503. Fewer containers fixed it. With JVM tools, thread dumps would show those request threads busy in component includes."

**Keywords:** all or some · cache hit · request.log slow URLs · error.log queries · GC pauses · thread dumps · high CPU: top -H → hex → dump

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| AEM is slow. How do you investigate? | All |
| High CPU on AEM? | Last sentence |
| Performance issue you solved? | Story 4 |

**Practice:** Steps for a slow site · How do you match a CPU-heavy thread?

---

## M5 · Linux commands

**In simple words**
A small set of commands for processes, CPU, memory, disk, logs and network.

**Say this**
> On Linux, I find the AEM process with `ps -ef | grep java` or `jcmd -l`. For CPU and memory I use `top`, `top -H -p <pid>` for threads, `free -m`, and `jstat -gcutil` for garbage collection. For disk I use `df -h` and `du -sh`, because a full disk can stop AEM. For logs I use `tail -f`, `grep` and `less` on error.log and request.log. For network I use `curl -v`, `nc -zv host port` and `dig`, for example to check if AEM can reach an SMTP server.

**Commands to remember**
| Need | Command |
|------|---------|
| Process | `ps -ef \| grep java`, `jcmd -l` |
| CPU / threads | `top`, `top -H -p <pid>`, `printf '%x' <tid>` |
| Memory / GC | `free -m`, `jstat -gcutil <pid> 5000` |
| Dumps | `jcmd <pid> Thread.print`, `jcmd <pid> GC.heap_dump file.hprof` |
| Disk | `df -h`, `du -sh *` |
| Logs | `tail -f error.log`, `grep -i error error.log`, `grep "Caused by"` |
| Network | `curl -v URL`, `nc -zv host 587`, `dig domain` |

**My example** [Hands-on, Linux background]
> "I use Linux commands for logs and connectivity checks; for example, for an email issue I would check whether the mail server port is reachable."

**Keywords:** process (ps, jcmd) · CPU (top -H) · memory (free, jstat) · disk (df, du) · logs (tail, grep) · network (curl, nc, dig)

**Practice:** Which command for a full disk? · How do you check if a port is reachable?
