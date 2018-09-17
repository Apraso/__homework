#include <QCoreApplication>
#include <windows.h>
#include <time.h>
#include <random>
#include <chrono>
void* _nop_(void* i){
    while(1){
        Sleep(50);
    }
    return NULL;
}
pthread_mutex_t mutex_0, mutex_1, mutex_2, mutex_3;
void* _overflow_(void* arr){
    int* arr_ = (int*)arr;
    //srand(arr_[0]);
    int idx = 78;
    // Causes stack overflow
    for(int i = 2; i < idx; i++){
        pthread_mutex_lock(&mutex_0);
        arr_[i] = arr_[1]++; // arr_[1] changes
        pthread_mutex_unlock(&mutex_0);
    }
    printf("%d %d\n", arr_[0], arr_[10]);
    return arr;
}
void* _inneroverflow_(void* i){
    int u = *(int*)i;
    int x[10] = {0}, y[10] = {0};
    int idx = 10;  //Nothing seems wrong, why
    // Causes stack overflow
    for(int i = 0; i < idx; i++){
        y[i] = ++x[1]; // arr_[1] changes
    }
    y[u * 20] = x[1];
    y[u * 40] = x[2];
    y[u * 60] = x[3];
    y[u * 100] = x[4];
    y[u * 500] = x[5];
    /*for(int i = 0; i < 10; i++)
        printf("%d %d\n", x[i], y[i]);*/
    printf("%d\n", u);
    return i;
}
void* _deeprecursion_(void* arr){ //SIGSEGV
    return _deeprecursion_(arr);
}

const int numBatches = 20;
const int niters = 1000000;
const int nThread = 50;

void* _integral_log1prec_(void* x){
    pthread_mutex_lock(&mutex_1);
    int* _ptrx = (int*)x;
    int idx = *_ptrx;
    (*_ptrx)++;
    pthread_mutex_unlock(&mutex_0);
    unsigned seed = std::chrono::system_clock::now().time_since_epoch().count();
    std::default_random_engine gen (seed);
    unsigned seed2 = std::chrono::system_clock::now().time_since_epoch().count() + 12345;
    std::default_random_engine gen2 (seed2);
    double* result = new double[numBatches];
    double _x, _y, left, upper;
    int cnt = 0;
    for(int i = 1; i <= numBatches; i++){
        cnt = 0;
        left = (double)(idx * numBatches + i);
        upper = 1.0 / left;
        std::uniform_real_distribution<double> dis_x(left, left + 1.0);
        std::uniform_real_distribution<double> dis_y(0.0, upper);
        for(int j = 0; j < niters; j++){
             _x = dis_x(gen);
             _y = dis_y(gen2);
             if(_x * _y < 1.0) cnt++;
        }
        result[i - 1] = (double)cnt / niters * upper;
        pthread_mutex_lock(&mutex_2);
        int* fin = _ptrx + 1;
        (*fin)++;
        printf("Task %d for input %d: cnt=%d, prob=%f, log(1+1/%d)=%lf\n", *fin, idx * numBatches + i, cnt, (float)cnt / niters, idx * numBatches + i, result[i - 1]);
        pthread_mutex_unlock(&mutex_2);
    }
    return result;
}
int task1(int argc, char *argv[]){
    printf("%d\n", PTHREAD_THREADS_MAX); //2019
    pthread_t thr1 = 0;
    QCoreApplication a(argc, argv);
    int x = 1, i = 0;
    for(i = 0; ; i++){
        x = pthread_create(&thr1, NULL, _nop_, NULL);
        //printf("%d\n", thr1);
        if(x != 0) {
            printf("%d\n", x); // 11
            break;
        }
    }
    printf("%d\n", i);
    //printf("%d\n", thr1);
    return a.exec();
}
int task2(int argc, char *argv[]){
    pthread_t thr1, thr2;
    int _buffer_ [63] = {0};
    int arr[9] = {0};
    arr[0] = 0; arr[1] = 100;
    int arr2[9] = {0};
    arr2[0] = 1; arr2[1] = 200;
    pthread_mutex_init(&mutex_0, NULL);
    pthread_mutex_init(&mutex_1, NULL);
    QCoreApplication a(argc, argv);
    int x_ = 0, y_ = 1;
    /*pthread_create(&thr1, NULL, _overflow_, arr);
    pthread_create(&thr2, NULL, _overflow_, arr2);*/
    pthread_create(&thr1, NULL, _inneroverflow_, &x_);
    pthread_create(&thr2, NULL, _inneroverflow_, &y_);
    printf("%d %d", x_, y_);
    pthread_join(thr1, NULL); //crash when the buffer above cannot hold!
    pthread_join(thr2, NULL);
    printf("%d %d\n", x_, y_);
    int __ = a.exec();
    pthread_mutex_destroy(&mutex_0);
    pthread_mutex_destroy(&mutex_1);
    return __;
}
/**
 * Output in RUN mode:
 * 0 1
 * 0
 *
 */
int task3(int argc, char *argv[]){
    int cnt[2] = {0, 0};
    pthread_mutex_init(&mutex_0, NULL);
    pthread_mutex_init(&mutex_1, NULL);
    pthread_mutex_init(&mutex_2, NULL);
    //pthread_mutex_init(&mutex_3, NULL);
    pthread_t tid[nThread] = {0};
    QCoreApplication a(argc, argv);
    for(int i = 0; i < nThread; i++){
        pthread_mutex_lock(&mutex_0);
        pthread_create(&tid[i], NULL, _integral_log1prec_, &cnt);
        pthread_mutex_unlock(&mutex_1);
    }
    void* res[nThread];
    for(int i = 0; i < nThread; i++){
        pthread_join(tid[i], &res[i]);
    }
    pthread_mutex_destroy(&mutex_0);
    pthread_mutex_destroy(&mutex_1);
    pthread_mutex_destroy(&mutex_2);
    //pthread_mutex_destroy(&mutex_3);
    for(int i = 0; i < nThread; i++){
        for(int j = 0; j < numBatches; j++)
            printf("log(1+1/%d)=%lf\n", i * numBatches + j + 1, ((double*)res[i])[j]);
    }
    for(int i = 0; i < nThread; i++){
        delete [] res[i];
    }
    //printf("%d", cnt[0]);
    return a.exec();
}
int main(int argc, char *argv[])
{
    return task1(argc, argv);
}

