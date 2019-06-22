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
