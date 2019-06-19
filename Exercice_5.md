# Question 5
```C++
#include <QThread>
#include <QSemaphore>
#include <iostream>

int NUM_THREADS = 2;
static QSemaphore* sem = new QSemaphore(0);


class MyThread : public QThread {
    int tid;

public :
    MyThread(int id) : tid (id) {}

    void run() {
        sem->acquire();
        std::cout << "Tache" << tid << "Execute" << std::endl;
    }
};


int main(void) {
    int i;
    QThread* threads[NUM_THREADS];

    for (i = 0; i < NUM_THREADS; i++) {
        threads[i] = new MyThread(i);
        threads[i]-> start();
    }

    MyThread::sleep(5);

    for (i = 0; i < NUM_THREADS; i++)
        sem->release();

    for (i = 0; i < NUM_THREADS; i++)
        threads[i]->wait();

    return EXIT_SUCCESS;
}
```
