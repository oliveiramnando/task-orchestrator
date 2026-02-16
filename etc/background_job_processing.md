
# Background Job Processing Architecture

Background jobs can be executed without requiring user interaction. The application can start the job and then continue to process interactive request from users. This can help to minimize the load on the appplicaiton UI, which can improve availability and reduce interactive response times

When you consider whether to implement a task as a background job, the main criterion is whether the task can run without user interaction and without the UI needing to wait for the job to be completed. Tasks that require the user or the UI to wait while they are completed might not be appropriate as background jobs.

## Types of Background Jobs

Background jobs typically include one or more of the followoing types of jobs:

- CPI-intensive jobs, such as mathematical caclulations or structural model analysis.
- I/O-intensive jobes, such as executing a series of storage transactions or indexing files.
- Batch jobs, such as nightly data updates or scheduled processing.
- Long-running workflows, such as order fulfullment, or provisioning services and systems.
- Sensitive-data processing where the task is handed off to a more secure location for processing. For example, you might not want to process sensitive data within a web app.

## Triggers

Background jobs can be initieated in serveral different ways. They fall into one of the following categories:

- *Event-driven triggers:* The task is started in response to an event, typically an action taken by a user or a step in a workflow.
- *Schdeule-driven triggers:* The task is invoked on a schedule based on a timer. this might be a recurring schedule or a one-off invocation that is specified for a later time.

#### Event-driven triggers

Event-driven invocation uses a trigger to start the background task, examples of using event-driven triggers include:

- The UI or another job places a message in a queue. The message contains data about an action that has taken place, such as the user placing an order. The background task listens on this queue and detects the arrival of a new message. It reads the message and uses the data in it as the input to the background job. This pattern is known as asynchronous message-based communication.
- The UI or another job saves or updates a value in storage. The background task monitors the storage and detects changes. It reads the data and uses it as the input to the background job.
- The UI or another job makes a reuqest to anendpoint, such as an HTTPS URI, or an API that is exposed as a web service. It passes the data that is required to complete the background task as part of the request. The endpoint or web service invokes the background task, which uses the data as its input.

Typical examples of tasks that are suited to event-driven invocation include image processing, workflows, sending information to remote services, sending email messages, and provisioning new users in multitenant applications.

#### Schedule-driven triggers

Schedule-driven invocation uses a timer to start the background task. Examples using schedule-drive triggers include:

- A timer that is running locally within the application or as part of the application's operating system invokes a background task on a regular basis.
- A timer that is running in a different application, such as Azure Logic Apps, sends a request to an API or web service on a regular basis. The API or web service invokes the background task.
- A separate process or application starts a timer that causes the background task to be invoked once after a specified time delay, or at a specific time.

Typical examples of tasks that are suited to schedule-driven invocation include batch-processing routines (such as updating related-products lists for users based on their recent behavior), routine data processing tasks (such as updating indexes or generating accumulated results), data analysis for daily reports, data retention cleanup, and data consistency checks.

## Returning Results

Background jobs execute asynchronously in a separate process, or even in a separate location, from the UI or the process that invoked the background task. Ideally, background tasks are "fire and forget" operations, and their execution progress has no impact on the UI or the calling process. This means that the calling process doesn't wait for completion of the tasks. Therefore, it cannot automatically detect when the task ends.

If you require a background task to communicate with the calling task to indicate profress or completion, you must implement a mechanism for this. Some examples are"

- Write a status indicator value to storage that is accessible to the UI or caller task, which can monitor or check this value when required. other data that the background task must return to the caller can be placed into the same storage.
- Establish a reply queue that the UI or caller listens on. The background task can send messages to the queue that indicate status and completion. Data that the backround task must return to the caller can be placed into the messages. 
- Expose an API or endpoint from the background task that the UI or caller can access to ontain status information. Data that the background task must reutnr to the caller can be included in the response.
- Have the background task call back to the UI or caller through an API to indicate status at predefined points or on completion. This might be through events raised locally or through a publish-and-subscribe mechanism. Data that the background task must return to the caller can be included in the request or event payload.

## Partitioning 

If you decide to include background tasks within an exisiting compute instance, you must consider how this affects the quality attributes of the compute instance and the background task itself. These factors help you to decide whether to colocate the tasks with the existing compute instance or separate them out into a separate compute instance:

- *Availability:* Background tasks might not need to have the same level of availability as other parts of the application, in particular the UI and other parts that are directly involved in user interaction. Background tasks might be more tolerant of latency, retried connection failures, and other factors that affect availability because the operations can be queued. However, there must be sufficient capacity to prevent the backup of requests that could block queues and affect the application as a whole.
- *Scalability:* Background tasks are likely to have a different scalability requirement than the UI and the interactive parts of the application. Scaling the UI might be necessary to meet peaks in demand, while outstanding background tasks might be completed during less busy times by fewer compute instances.
- *Resiliency:* Failure of a compute instance that just hosts background tasks might not fatally affect the application as a whole if the requests for these tasks can be queued or postponed until the task is available again. If the compute instance or tasks can be restarted within an appropriate interval, users of the application might not be affected.
- *Security:* Background tasks might have different security requirements or restrictions that the UI or other parts of the application. By using a separate compute instance, you can specify a different security environment for the tasks.
- *Performance:* You can choose the type of compute instance for the background tasks to specifically match the performance requirements of the tasks. This might mean using a less expensive compute option if the tasks do not require the same processing capabilities as the UI, or a larger instance if they require additional capacity and resources.
- *Manageability:* Background tasks might have a different development and deployment rhythm from the main application code or the UI. Deploying them to a separate compute instance can simplify updates and versioning.
- *Cost:* Adding compute instances to execute background tasks increases hosting costs. You should carefully consider the trade-off between additional capacity and these extra costs.

## Conflicts

If you have multiple instances of a background job, they might compete for access to resources and services, such as databases and storage. This concurrent access can result in resource contention, which might cause conflicts in availability of the services and in the integrity of data in storage. You can resolve resource contention by using a pessimistic locking approach. This prevents competing instances of a task from concurrently accessing a service or corrupting data.

Another approach to resolve conflicts is to define background tasks as a singleton, so that there's only ever one instance running. But this eliminates the reliability and performance benefits that a multiple instance configuration can provide. This is especially true if the UI can supply sufficient work to keep more than one background task busy.

It is vital to ensure that the background task can automatically restart and that it has sufficient capacity to cope with peaks in demand. You can achieve this by allocating a compute instance with sufficient resources, by implementing a queueing mechanism that can store requests for late execution when demand decreases, or by using a combination of these techniques.

## Coordination
The 