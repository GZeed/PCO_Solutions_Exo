# Question 16
## Avec sémaphores (solution non-validée)
```cpp
#ifndef BUFFER2CONSO_H
#define BUFFER2CONSO_H

#include "abstractbuffer.h"
#include <QSemaphore>
#include <QMutex>

template<typename T> class Buffer2ConsoSemaphoreGeneral : public AbstractBuffer<T> {
protected:
    QMutex mutex;
    QSemaphore jetonPourLire;
    QSemaphore jetonPourEcrire;
    T* buffer;
    int writerPointer1;
    int readerPointer1;
    int readerPointer2;
    int bufferSize;
    int nbLecture;



public:
    Buffer2ConsoSemaphoreGeneral(unsigned int bufferSize) : bufferSize(bufferSize), jetonPourEcrire(1), jetonPourLire(0){
        buffer = new T[bufferSize];
        if(buffer == 0){
            return;
        }
    }

    virtual ~Buffer2ConsoSemaphoreGeneral() {
        delete buffer;
    }

    virtual void put(T item) {
        jetonPourEcrire.acquire();
        mutex.lock();
        buffer[writerPointer1++] = item;
        writerPointer1 %= bufferSize;
        nbLecture = 0;
        jetonPourLire.release();
        jetonPourLire.release();
        mutex.unlock();
    }

    virtual T get(void) {
        T item;
        jetonPourLire.acquire();
        mutex.lock();
        nbLecture++;
        if(nbLecture == 1){
            item = buffer[readerPointer1++];
            readerPointer1 %= bufferSize;
        }
        else{
            item = buffer[readerPointer2++];
            readerPointer2 %= bufferSize;
            jetonPourEcrire.release();
        }
        mutex.unlock();
        return item;
    }
};

#endif // BUFFER2CONSO_H
```
