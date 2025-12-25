# Java Synchronization Works
The `synchronized` keyword will cause the thread that runs the block or method to acquire the object's lock before executing the block and release it when exiting.
When a thread calls `wait()` on the object, it will release the lock and block itself, place it into the wait-set of that object. When it is awakened by `notify()` or `notifyAll()`, it will reacquire the lock and continue after the wait.
When a thread calls `notify()` or `notifyAll()` on an object, one or all waiting threads on that object will be moved to the entry queue, and will be able to contend for the lock. This also means the notifying thread will not relinquish the lock immediately, and the notified threads will not acquire the lock immediately.
From this we can conclude that Java synchronization is based on **Mesa** model, which means, the signaling thread continues to run after signaling and the awakened thread must wait until the signaling thread quit the monitor.
# Semaphore
```
init(sem *s, int value) {
	s->value = value;
	s->lock = false;
}

Acquire(sem *s) {
	while (true) {
		while (test&set(&s->lock)); // acquire the lock protection
		if (s->value > 0) {
			s->value--;
			s->lock = false;
			break;
		} else {
			s->lock = false;
		}
	}
}

Release(sem *s) {
	while (test&set(&s->lock)); // acquire the lock protection
	s->value++;
	s->lock = false;
}
```
The key point is when we want to change the value of semaphore, we need to use an additional spin lock to protect this operation.