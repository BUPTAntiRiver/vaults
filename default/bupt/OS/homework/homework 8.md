# Read-Write lock
```c
typedef struct {
	pthread_mutex_t mutex;
	pthread_cond_t cond;
	int reader_count;
	int writer_active;
	int waiting_writers;
} rwlock_t;

void read_lock(rwlock_t* r) {
	pthread_mutex_lock(&r->mutex);
	while (r->writer_active || r->waiting_writers > 0) {
		// give priority to writers
		pthread_cond_wait(&r->cond, &r->mutex);
	}
	r->reader_count++;
    pthread_mutex_unlock(&r->mutex);
}

void read_unlock(rwlock_t* r) {
	pthread_mutex_lock(&r->mutex);
	r->reader_count--;
	if (r->reader_count == 0) {
		// no more reader then wake
		pthread_cond_signal(&r->cond);
	}
    pthread_mutex_unlock(&r->mutex);
}

void write_lock(rwlock_t* r) {
	pthread_mutex_lock(&r->mutex);
	while (r->reader_coutn > 0 || r->writer_active) {
		// if there is any reader or writer
		pthread_cond_wait(&r->cond, &r->mutex);
	}
	r->waiting_writers--;
	r->writer_active = 1;
    pthread_mutex_unlock(&r->mutex);
}

void write_unlock(rwlock_t* r) {
	pthread_mutex_lock(&r->mutex);
	// wake all readers or a writer
	pthread_cond_broadcast(&r->cond);
	r->writer_active = 0;
    pthread_mutex_unlock(&r->mutex);
}
```
# Banker's Algorithm
The simulation and algorithm code are in the file, here is an example:
```terminal
========== Initial State ==========

Checking initial safety...
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 

========== Step 1 ==========
Process P7 requests resources: 1 2 1 2 0 3 3 1 3 0 
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 
Request GRANTED to process P7

========== Step 2 ==========
Process P1 requests resources: 2 3 0 1 0 2 1 3 2 2 
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 
Request GRANTED to process P1

========== Step 3 ==========
Process P0 requests resources: 0 0 1 1 1 0 0 3 2 3 
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 
Request GRANTED to process P0

========== Step 4 ==========
Process P9 requests resources: 2 0 0 2 0 3 1 0 3 2 
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 
Request GRANTED to process P9

========== Step 5 ==========
Process P2 requests resources: 1 2 2 0 1 3 3 0 0 3 
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 
Request GRANTED to process P2

========== Step 6 ==========
Process P1 requests resources: 1 0 1 2 5 1 1 0 3 1 
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 
Request GRANTED to process P1

========== Step 7 ==========
Process P0 requests resources: 5 5 0 1 2 4 2 0 0 0 
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 
Request GRANTED to process P0

========== Step 8 ==========
Process P3 requests resources: 2 0 3 3 2 0 0 2 3 2 
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 
Request GRANTED to process P3

========== Step 9 ==========
Process P2 requests resources: 2 0 2 1 0 2 2 4 0 2 
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 
Request GRANTED to process P2

========== Step 10 ==========
Process P8 requests resources: 3 1 1 0 2 3 2 2 0 0 
System is in SAFE state. Safe sequence: P0 P1 P2 P3 P4 P5 P6 P7 P8 P9 
Request GRANTED to process P8
```