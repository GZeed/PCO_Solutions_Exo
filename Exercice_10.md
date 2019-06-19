# Question 10

```cpp
#include <QSemaphore>
#include <iostream>

class PcoMutex {

private:
    QSemaphore mutex;
    QSemaphore realMutex;
    int nbLock;

public:
    PcoMutex()
        : mutex(1)
        , realMutex(1)
        , nbLock(0)
    {
    }

    void lock()
    {
        mutex.acquire();
        nbLock++;
        mutex.release();

        realMutex.acquire();
    }

    void unlock()
    {
        mutex.acquire();

        if (nbLock > 0) {
            nbLock--;
            realMutex.release();
        }

        mutex.release();
    }

    bool trylock()
    {
        mutex.acquire();

        if (nbLock > 0) {
            mutex.release();
            return false;
        }

        nbLock++;
        realMutex.acquire();
        mutex.release();

        return true;
    }
};
```
