# Question 21
## Priorité égale, sémaphore

```c++
#ifndef READERWRITERCLASSAB_H
#define READERWRITERCLASSAB_H

#include <QSemaphore>
#include <QMutex>
#include <iostream>
#include "abstractreaderwriter.h"
#include "hoaremonitor.h"


class ReaderWriterClassAB
{
protected:
    int nbA;
    int nbB;
    QSemaphore fifo;
    QMutex mutex ;
    QSemaphore jeton;


public:
  ReaderWriterClassAB() : fifo(1), jeton(1){
      this->nbA = 0;
      this->nbB = 0;
  }

  void lockA() {
      fifo.acquire();
      mutex.lock();
      if(nbA++ == 0){
          mutex.unlock();
          jeton.acquire();
      } else{
          mutex.unlock();
      }
      // Accès à la ressource
      fifo.release();
  }

  void unlockA() {
      mutex.lock();
      if(nbA-- == 1){
          jeton.release();
      }
      mutex.unlock();
  }

  void lockB() {
      fifo.acquire();
      mutex.lock();
      if(nbB++ == 0){
          mutex.unlock();
          jeton.acquire();
      } else{
          mutex.unlock();
      }
      // Accès à la ressource
      fifo.release();
  }

  void unlockB() {
      mutex.lock();
      if(nbB-- == 1){
          jeton.release();
      }
      mutex.unlock();
  }
};
#endif // READERWRITERCLASSAB_H
```

## Avec variable de condition
```c++
#ifndef READERWRITERCLASSAB_H
#define READERWRITERCLASSAB_H

#include <QSemaphore>
#include <QMutex>
#include <QWaitCondition>

#include "abstractreaderwriter.h"
#include "hoaremonitor.h"


class ReaderWriterClassAB
{
protected:
    QMutex mutex;
    QSemaphore fifo;
    QWaitCondition cond;
    int nbReadersA;
    int nbReadersB;
public:
  ReaderWriterClassAB(): fifo(1), nbReadersA(0), nbReadersB(0) {
  }

  void lockA() {
      fifo.acquire();

      mutex.lock();
      nbReadersA++;

      while(nbReadersB >= 1) {
          cond.wait(&mutex);
      }

      mutex.unlock();
      fifo.release();
  }

  void unlockA() {
      mutex.lock();
      nbReadersA--;

      if(nbReadersA == 0)
        cond.wakeOne();

      mutex.unlock();
  }

  void lockB() {
      fifo.acquire();

      mutex.lock();
      nbReadersB++;

      while(nbReadersA >= 1) {
          cond.wait(&mutex);
      }

      mutex.unlock();
      fifo.release();
  }

  void unlockB() {
      mutex.lock();
      nbReadersB--;

      if(nbReadersB == 0)
        cond.wakeOne();

      mutex.unlock();
  }
};
#endif // READERWRITERCLASSAB_H
```

## Avec Moniteur de Hoare
```C++
#ifndef READERWRITERCLASSAB_H
#define READERWRITERCLASSAB_H

#include <QSemaphore>
#include <QMutex>

#include "abstractreaderwriter.h"
#include "hoaremonitor.h"

class ReaderWriterClassAB : HoareMonitor
{
protected:
    QSemaphore fifo;
    Condition cond;
    int nbReadersA;
    int nbReadersB;
public:
  ReaderWriterClassAB(): fifo(1), nbReadersA(0), nbReadersB(0) {
  }

  void lockA() {
      fifo.acquire();
      monitorIn();

      nbReadersA++;

      if(nbReadersB >= 1)
          wait(cond);

      monitorOut();
      fifo.release();
  }

  void unlockA() {
      monitorIn();
      nbReadersA--;
      if(nbReadersA == 0)
          signal(cond);
      monitorOut();
  }

  void lockB() {
      fifo.acquire();
      monitorIn();

      nbReadersB++;

      if(nbReadersA >= 1)
          wait(cond);

      monitorOut();
      fifo.release();
  }

  void unlockB() {
      monitorIn();
      nbReadersB--;
      if(nbReadersB == 0)
          signal(cond);
      monitorOut();
  }
};
#endif // READERWRITERCLASSAB_H
```
