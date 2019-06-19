# Question 12

## Avec sémaphores

```c++
#include <QSemaphore>

#define CARWEIGHT 1
#define TRUCKWEIGHT 10

/*
Commencer par abstraire par véhicule.

On doit implémenter une file d'attente sur le pont quand on atteint
le poids maximum.

Logique de base vehicleAccess: s'il y a trop de poids sur le pont,
on attend.

Logique de base vehicleRelease: on soustrait le poids et on libère.

Seulement, le problème c'est qu'on peut pas simplement libérer la thread
de véhicule, il faut encore vérifier son poids. Donc il faut une boucle
while avec la condition. Et donc nous avons besoin de deux méchanismes
d'attente: un pour le "parking", et un pour la "rampe". Une fois sur la
rampe, le véhicule sera accepté sur le pont seulement quand le poids
maximum sera respecté. Il faut par exemple dix voitures qui sortent pour
pouvoir faire entrer un camion à capacité maximale. Une fois que le véhicule
sort de la rampe, il laisse sa place au suivant dans la file du parking.
*/
class BridgeManager
{
    QSemaphore parkingFifo;
    QSemaphore waitingRamp;
    QSemaphore mutex;
    unsigned int currentWeight;
    unsigned int maxWeight;
    bool someoneOnRamp;

public:
    BridgeManager(unsigned int maxWeight):
        parkingFifo(1),
        mutex(1),
        currentWeight(0),
        maxWeight(maxWeight),
        someoneOnRamp(false)
    {
    }

    void vehicleAccess(unsigned int vehicleWeight) {
        parkingFifo.acquire();
        mutex.acquire();
        while(currentWeight + vehicleWeight > maxWeight) {
            someoneOnRamp = true;
            mutex.release();
            waitingRamp.acquire();
            mutex.acquire();
            someoneOnRamp = false;
        }
        parkingFifo.release();
        currentWeight += vehicleWeight;
        mutex.release();
    }

    void vehicleRelease(unsigned int vehicleWeight) {
        mutex.acquire();
        currentWeight -= vehicleWeight;
        if (someoneOnRamp) {
            waitingRamp.release();
        }
        mutex.release();
    }

    void carAccess()
    {
        vehicleAccess(CARWEIGHT);
    }

    void truckAccess()
    {
        vehicleAccess(TRUCKWEIGHT);
    }

    void carLeave()
    {
        vehicleRelease(CARWEIGHT);
    }

    void truckLeave()
    {
        vehicleRelease(TRUCKWEIGHT);
    }
};
```

## Avec Variable de Condition
```cpp
#include <QSemaphore>
#include <QWaitCondition>
#include <QMutex>

#define CARWEIGHT 1
#define TRUCKWEIGHT 10


class BridgeManager
{
private:
    unsigned maxWeight, currentWeight;
    QWaitCondition cond;
    QWaitCondition fifo;
    bool accesParking;
    QMutex mutex;
public:
    BridgeManager(unsigned int maxWeight): maxWeight(maxWeight), currentWeight(0), accesParking(true)
    {

    }

    ~BridgeManager()
    {

    }

    void vehiculeAccess(unsigned weightVehicule)
    {
        mutex.lock();

        while(!accesParking) {
            fifo.wait(&mutex);
        }

        accesParking = false;

        while(currentWeight + weightVehicule > maxWeight) {
            cond.wait(&mutex);
        }

        currentWeight += weightVehicule;
        accesParking = true;
        fifo.wakeOne();
        mutex.unlock();
    }

    void vehiculeLeave(unsigned weightVehicule)
    {
        mutex.lock();

        currentWeight -= weightVehicule;
        cond.wakeOne();

        mutex.unlock();
    }

    void carAccess()
    {
        vehiculeAccess(CARWEIGHT);
    }

    void truckAccess()
    {
        vehiculeAccess(TRUCKWEIGHT);
    }

    void carLeave()
    {
        vehiculeLeave(CARWEIGHT);
    }

    void truckLeave()
    {
        vehiculeLeave(TRUCKWEIGHT);
    }
};
```
