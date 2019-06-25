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

/** @file readerwriterclassab.h
* @brief Reader-writer with equal priority between two classes
*
* Implementation of a reader-writer resource manager with equal
* priority for both classes. Mutual exclusion between classes must be
* ensured
*
*
*
* @author Yann Thoma
* @date 15.05.2017
* @bug No known bug
*/


#include <QSemaphore>
#include <QMutex>
#include <iostream>
#include <QWaitCondition>
#include "abstractreaderwriter.h"
#include "hoaremonitor.h"


class ReaderWriterClassAB
{
protected:
QMutex mutex;
QWaitCondition condWaitingA;
QWaitCondition condWaitingB;
int nbA;
int nbB;
public:
ReaderWriterClassAB(): nbA(0), nbB(0) {
}



void lockA() {

mutex.lock();

while(nbB >= 1) {
condWaitingA.wait(&mutex);
}
nbA++;

mutex.unlock();
}

void unlockA() {
mutex.lock();
nbA--;

if(nbA <= 0){
nbA = 0;
condWaitingB.wakeAll();
}

mutex.unlock();
}

void lockB() {

mutex.lock();

while(nbA >= 1) {
condWaitingB.wait(&mutex);
}

nbB++;
mutex.unlock();

}

void unlockB() {
mutex.lock();
nbB--;

if(nbB <= 0){
nbB = 0;
condWaitingA.wakeAll();
}

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
