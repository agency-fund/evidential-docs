# Workers

Evidential does its background work in a single process, `xngin-worker`. It runs two kinds of work:

- **Experiment maintenance jobs**, which keep each experiment's snapshots and bandit outcomes up to date.
- **The task queue**, which delivers asynchronous tasks such as outbound webhooks.

You can start the worker locally via:

```shell
task start-worker
```

??? "Deployment Tips"

    In production, run `xngin-worker` as a long-running process alongside the API server, with `DATABASE_URL` pointing
    at the same database. Run `uv run xngin-worker --help` to see its settings, each of which can also be set by an
    `XNGIN_WORKER_*` environment variable. When deploying it:

    - **Restart it whenever it exits.** The worker exits after a set number of jobs (1000 by default), so that a fresh
        process replaces it. It also exits if its task queue fails.
    - **Give it time to stop.** On SIGTERM or SIGINT, the worker starts no new work and waits for the work in progress to
        finish. Allow it about two minutes before killing it.

## Experiment Maintenance Jobs

The worker runs three jobs. Each applies to committed experiments of particular types:

| Job       | Experiment types                                  | What it does                                                                                                                                                      |
| --------- | ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Snapshots | All                                               | Runs the experiment's analysis and records the result as a snapshot, every 3 hours from a day before the start date until a day after the end date.               |
| DWH pull  | `mab_online_dwh`                                  | Reads the outcomes of pending draws from the customer's data warehouse, every 4 hours while any draw has no outcome, until 8 days after the end date.             |
| Autofail  | `mab_online` and `cmab_online`, if autofail is on | Records a failure outcome for each draw that has received no outcome within the experiment's autofail window (24 hours by default). It runs as each draw expires. |

Frequentist experiments, which have no draws, only get snapshots.

The worker works on multiple experiments simultaneously, but works on at most one job per experiment at a time. Each
job's next run time is computed from the experiment.

## Task Queue

The Task Queue delivers outbound webhooks and facilitates integration with some 3rd parties. For example, we can notify
a user's infrastructure via HTTP POST when an experiment has been created. These requests may be delivered more than once.

??? "Implementation Details"

    The queue is implemented using the NOTIFY/LISTEN + SKIP LOCKED pattern.

    The following sequence diagram illustrates the flow of a task through the system:

    ```mermaid
    sequenceDiagram
        participant App as Application
        participant DB as PostgreSQL
        participant TQ as Task Queue
        participant EH as Handler
        App ->> DB: INSERT INTO tasks (task_type, payload, status=pending)
        Note over DB: Trigger fires
        DB ->> TQ: NOTIFY task_queue
        TQ ->> DB: UPDATE one task SET status='running' (FOR UPDATE SKIP LOCKED)
        DB ->> TQ: Return task
        TQ ->> TQ: Find handler for task_type
        TQ ->> EH: Call handler with task
        alt Success
            EH ->> TQ: Return success
            TQ ->> DB: UPDATE task status='success'
        else Failure
            EH ->> TQ: Throw exception
            TQ ->> DB: UPDATE task status='pending', retry_count++, embargo_until=NOW()+backoff
            Note over DB: Task will be retried later
        end

        loop Until max_retries
            Note over TQ: Wait for notification or poll interval
            alt New notification
                DB ->> TQ: Notification received
            else Poll interval elapsed
                TQ ->> TQ: Check for tasks
            end
            TQ ->> DB: UPDATE one task SET status='running' (FOR UPDATE SKIP LOCKED)
        end
    ```

    ### Task States

    Tasks in the queue can exist in one of several states, as illustrated in the following state diagram:

    ```mermaid
    stateDiagram-v2
        [*] --> pending: Task created
        pending --> running: Task selected for processing
        running --> success: Task completed successfully
        running --> pending: Task failed (retry)
        running --> dead: Max retries exceeded
        running --> running: Orphaned task reclaimed
        success --> [*]
        dead --> [*]

    ```

    ### Enqueuing a Task

    To enqueue a task, insert a row into the `tasks` table. Tasks are identified by their "type". The task queue uses the
    task's type to dispatch it to a handler. The value you store in the `payload` field must be comprehensible to the
    handler. Example:

    ```sql
    INSERT INTO tasks (id, task_type, payload)
    VALUES ('task_RANDOM',
            'webhook.outbound',
            '{"url": "https://example.com", "method": "POST"}')
    ```

    ### Implementing a Handler

    Handlers run inside the worker. To deploy a new handler, register it in
    [src/xngin/apiserver/worker/tq/task_queue_thread.py](https://github.com/agency-fund/evidential-be/blob/main/src/xngin/apiserver/worker/tq/task_queue_thread.py).
    Example:

    ```python
    def webhook_handler(task: Task):
        payload = json.loads(task.payload)
        response = httpx.post(payload["method"], payload["url"])
        response.raise_for_status()


    # register the handler before the queue's thread starts.
    queue.register_handler("webhook.outbound", webhook_handler)
    ```
