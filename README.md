#include <iostream>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

using namespace std;

#define BUFFER_SIZE 5
#define TOTAL_ITEMS 4

int buffer[BUFFER_SIZE];
int in = 0;
int out = 0;

sem_t empty;
sem_t full;
sem_t mutex;

void* producer(void* arg) {
    for (int item = 1; item <= TOTAL_ITEMS; item++) {
        sem_wait(&empty);
        sem_wait(&mutex);

        buffer[in] = item;
        cout << "Producer produced item " << item << endl;
        in = (in + 1) % BUFFER_SIZE;

        sem_post(&mutex);
        sem_post(&full);

        sleep(1);
    }
    return nullptr;
}

void* consumer(void* arg) {
    for (int i = 1; i <= TOTAL_ITEMS; i++) {
        sem_wait(&full);
        sem_wait(&mutex);

        int item = buffer[out];
        cout << "Consumer consumed item " << item << endl;
        out = (out + 1) % BUFFER_SIZE;

        sem_post(&mutex);
        sem_post(&empty);

        sleep(1);
    }
    return nullptr;
}

int main() {
    pthread_t prodThread, consThread;

    sem_init(&empty, 0, BUFFER_SIZE);
    sem_init(&full, 0, 0);
    sem_init(&mutex, 0, 1);

    pthread_create(&prodThread, nullptr, producer, nullptr);
    pthread_create(&consThread, nullptr, consumer, nullptr);

    pthread_join(prodThread, nullptr);
    pthread_join(consThread, nullptr);

    sem_destroy(&empty);
    sem_destroy(&full);
    sem_destroy(&mutex);

    return 0;
}



g++ task.cpp -pthread -o task.out
./task.out

