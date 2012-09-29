# Execution

An execution in Paws is a code block (like a lambda) that's pausable (like a
coroutine) and forkable (like fork(2)). There's two possible states:
*pausable* and *running*.

When an execution is paused there are two operations you can invoke:

* `resume(value)`: Resumes the execution by sending the `value`.
* `fork`: Forks an execution. Resuming in one of the executions do not affect
  the other execution.

When an execution is running there two operations, but they can only be
invoked by the execution itself:

* `pause`: Pauses the execution until another execution resumes it.

## Example programs

### call/return

```
square = execution {
  # Get caller
  caller = pause

  # Get argument
  value = pause

  # Return
  caller.resume(value ** 2)
}

caller = execution {
  # Fork it so we can re-use it
  s = square.fork

  # Pass in ourself
  s.resume(self)

  # Pass in the argument(s)
  s.resume(2)

  # Pause until we have an answer:
  res = pause
}
```

### Run in background

```
background = execution {
  # Get argument
  value = pause

  # Do stuff with value
}

caller = execution {
  # Fork it so we can re-use it
  b = background.fork

  # Pass in argument
  b.resume(123)

  # Carry on as usual...
}
```

### Continue multiple times

```
each = execution {
  # Get caller
  caller = pause

  # Get argument
  value = pause

  # Return
  for(thing in value)
    caller.fork.resume(thing)
}

caller = execution {
  e = each.fork
  e.resume(self)
  e.resume([1, 2, 3])

  value = pause
}
```

