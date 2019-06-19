# Question 9

## Avec Wait
```C++

#include <QThread>
#include <iostream>

QThread* threads[8];

class T1 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        std::cout << "Task 1" << std::endl;
    }
};

class T2 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        threads[0]->wait();
        std::cout << "Task 2" << std::endl;
    }
};

class T3 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        threads[0]->wait();
        std::cout << "Task 3" << std::endl;
    }
};


class T4 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        threads[0]->wait();
        std::cout << "Task 4" << std::endl;
    }
};

class T5 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        threads[0]->wait();
        std::cout << "Task 5" << std::endl;
    }
};

class T6 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        threads[1]->wait();
        threads[2]->wait();
        std::cout << "Task 6" << std::endl;
    }
};

class T7 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        threads[3]->wait();
        threads[4]->wait();
        std::cout << "Task 7" << std::endl;
    }
};

class T8 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        threads[5]->wait();
        threads[6]->wait();
        std::cout << "Task 8" << std::endl;

    }
};


int main(int ,char *[])
{
    // Le code suivant est un exemple, qui n'est pas forcément le bon
    threads[0] = new T1();
    threads[1] = new T2();
    threads[2] = new T3();
    threads[3] = new T4();
    threads[4] = new T5();
    threads[5] = new T6();
    threads[6] = new T7();
    threads[7] = new T8();

    for(int i=0; i< 8; i++)
        threads[i]->start();

    for(int i=0; i< 8; i++)
        threads[i]->wait();
}

```
## Avec Sémaphore
```C++
#include <QThread>
#include <iostream>
#include <QSemaphore>

static QSemaphore t1Done;
static QSemaphore t2Done;
static QSemaphore t3Done;
static QSemaphore t4Done;
static QSemaphore t5Done;
static QSemaphore t6Done;
static QSemaphore t7Done;

class T1 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        std::cout << "Task 1" << std::endl;
        t1Done.release();
        t1Done.release();
        t1Done.release();
        t1Done.release();
    }
};

class T2 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        t1Done.acquire();
        std::cout << "Task 2" << std::endl;
        t2Done.release();
    }
};

class T3 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        t1Done.acquire();
        std::cout << "Task 3" << std::endl;
        t3Done.release();
    }
};


class T4 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        t1Done.acquire();
        std::cout << "Task 4" << std::endl;
        t4Done.release();
    }
};

class T5 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        t1Done.acquire();
        std::cout << "Task 5" << std::endl;
        t5Done.release();
    }
};

class T6 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        t2Done.acquire();
        t3Done.acquire();
        std::cout << "Task 6" << std::endl;
        t6Done.release();
    }
};

class T7 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        t4Done.acquire();
        t5Done.acquire();
        std::cout << "Task 7" << std::endl;
        t7Done.release();
    }
};

class T8 : public QThread
{
    void run() Q_DECL_OVERRIDE
    {
        t6Done.acquire();
        t7Done.acquire();

        std::cout << "Task 8" << std::endl;

    }
};


int main(int ,char *[])
{
    // Le code suivant est un exemple, qui n'est pas forcément le bon
    QThread* threads[8];
    threads[0] = new T1();
    threads[1] = new T2();
    threads[2] = new T3();
    threads[3] = new T4();
    threads[4] = new T5();
    threads[5] = new T6();
    threads[6] = new T7();
    threads[7] = new T8();

    for(int i=0; i< 8; i++)
        threads[i]->start();
    for(int i=0; i< 8; i++)
        threads[i]->wait();

}
```
