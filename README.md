#include <iostream>
#include <vector>
#include <thread>
#include <chrono>
#include <mutex>
#include <condition_variable>

using namespace std;

// Класс для представления процесса
class Process {
private:
    int id; // Идентификатор процесса
    string name; // Имя процесса
    bool running; // Флаг запуска процесса
    mutex mutex; // Мьютекс для синхронизации доступа к данным процесса
    condition_variable cv; // Условие для ожидания завершения процесса

public:
    // Конструктор
    Process(int id, const string& name) : id(id), name(name), running(false) {}

    // Метод для запуска процесса
    void start() {
        // Защита доступа к данным процесса
        lock_guard<mutex> lock(mutex);

        // Установка флага запуска
        running = true;

        // Создание и запуск потока для процесса
        thread t([this]() {
            cout << "Процесс " << name << " запущен.\n";
            // Выполнение процесса
            // ...
            // Завершение процесса
            cout << "Процесс " << name << " завершен.\n";
            // Уведомление о завершении процесса
            cv.notify_one();
        });
        t.detach(); // Отсоединение потока
    }

    // Метод для ожидания завершения процесса
    void wait() {
        // Защита доступа к данным процесса
        unique_lock<mutex> lock(mutex);
        // Ожидание сигнала об успешном завершении процесса
        cv.wait(lock, [this]() { return !running; });
    }
};

int main() {
    // Создание списка процессов
    vector<Process> processes;

    // Создание и запуск процессов
    processes.push_back(Process(1, "Процесс 1"));
    processes.push_back(Process(2, "Процесс 2"));
    processes.push_back(Process(3, "Процесс 3"));
    for (auto& process : processes) {
        process.start();
    }

    // Ожидание завершения процессов
    for (auto& process : processes) {
        process.wait();
    }

    // Вывод сообщения о завершении всех процессов
    cout << "Все процессы завершены.\n";

    return 0;
}
