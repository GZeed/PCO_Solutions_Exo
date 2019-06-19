# Question 11
## Avec Sémaphores
```C++
#include <QSemaphore>

class PcoBarrier
{
private:
    QSemaphore barriere;
    QSemaphore mutex;
    unsigned compteur;
    unsigned nbToWait;
public:
    PcoBarrier(unsigned int nbToWait): mutex(1), compteur(0), nbToWait(nbToWait)
    {
    }

    ~PcoBarrier()
    {
    }

    void wait()
    {
        mutex.acquire();
        if(++compteur < nbToWait) {
            mutex.release();
            barriere.acquire();
        }
        else {
            compteur = 0;
            for(unsigned i = 0; i < nbToWait; i++) {
                barriere.release();
            }
            mutex.release();
        }
    }
};
```

## Avec Variable de Condition
```C++
#include <QWaitCondition>
#include <QMutex>

class PcoBarrier
{
private:
    QWaitCondition cond;
    QMutex mutex;
    unsigned nbToWait, compteur;
public:
    PcoBarrier(unsigned int nbToWait): nbToWait(nbToWait), compteur(0)
    {
    }

    ~PcoBarrier()
    {
    }

    void wait()
    {
        mutex.lock();
        compteur++;

        while(compteur < nbToWait) {
            cond.wait(&mutex);
        }

        cond.wakeOne();
        mutex.unlock();
    }
};
```

## Avec Moniteur de Hoare
```C++
#include <QWaitCondition>
#include <QMutex>
#include "hoaremonitor.h"

/** Implémentation du moniteur de Hoare sur
 * les slides 19-21 du pdf PCO_7_moniteurs.pdf
**/

class PcoBarrier : HoareMonitor
{
private:
    unsigned nbToWait, compteur;
    Condition cond;
public:
    PcoBarrier(unsigned int nbToWait): nbToWait(nbToWait), compteur(0)
    {
    }

    ~PcoBarrier()
    {
    }

    void wait()
    {
        monitorIn();
        compteur++;

        if(compteur < nbToWait) {
            HoareMonitor::wait(cond);
        }

        signal(cond);
        monitorOut();
    }
};
```
