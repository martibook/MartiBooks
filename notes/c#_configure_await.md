ConfigureAwait(**continueOnCapturedContext**: bool)

Should the rest of code run in the original thread?


&nbsp;


...run in thread A...

await some_task.ConfigureAwait(**true**) // this some_task run in thread B

...the rest of code run in **thread A**...


&nbsp;


...run in thread A...

await some_task.ConfigureAwait(**false**) // this some_task run in thread B

...the rest of code run in **thread B**...


&nbsp;

[Introduction Video](https://www.youtube.com/watch?v=F9_8MJbsnzg)