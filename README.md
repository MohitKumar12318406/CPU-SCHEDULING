#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>

using namespace std;

struct Process {
    int id, arrivalTime, burstTime, priority, remainingTime, completionTime, turnaroundTime, waitingTime, responseTime, startTime;

    Process(int id, int at, int bt, int pr = 0) : id(id), arrivalTime(at), burstTime(bt), priority(pr) {
        remainingTime = bt; // Initialize remaining time to burst time
    }
};

// Function to print Gantt Chart
void printGanttChart(const vector<pair<int, int>>& ganttChart) {
    cout << "\nGantt Chart:\n";
    for (const auto& it : ganttChart) {
        cout << "| P" << it.first << " ";
    }
    cout << "|\n";
    for (const auto& it : ganttChart) {
        cout << it.second << "    ";
    }
    cout << "\n";
}

// First Come First Serve (FCFS)
void fcfs(vector<Process>& processes) {
    int currentTime = 0;
    vector<pair<int, int>> ganttChart;

    // Sort processes by arrival time
    sort(processes.begin(), processes.end(), [](Process a, Process b) {
        return a.arrivalTime < b.arrivalTime;
    });

    for (auto &p : processes) {
        if (currentTime < p.arrivalTime) currentTime = p.arrivalTime; // Wait for the process to arrive
        p.startTime = currentTime;
        p.completionTime = currentTime + p.burstTime;
        p.turnaroundTime = p.completionTime - p.arrivalTime;
        p.waitingTime = p.turnaroundTime - p.burstTime;
        p.responseTime = p.startTime - p.arrivalTime;
        ganttChart.push_back({p.id, currentTime});
        currentTime += p.burstTime; // Move current time forward
    }

    // Print results
    cout << "\nPID\tAT\tBT\tCT\tTAT\tWT\tRT\n";
    for (const auto& p : processes) {
        cout << p.id << "\t" << p.arrivalTime << "\t" << p.burstTime << "\t"
             << p.completionTime << "\t" << p.turnaroundTime << "\t"
             << p.waitingTime << "\t" << p.responseTime << "\n";
    }

    printGanttChart(ganttChart);
}

// Shortest Job First (SJF) - Non-Preemptive
void sjf_non_preemptive(vector<Process>& processes) {
    int n = processes.size();
    vector<pair<int, int>> ganttChart;
    int currentTime = 0, completed = 0;

    // Sort processes by arrival time
    sort(processes.begin(), processes.end(), [](Process a, Process b) {
        return a.arrivalTime < b.arrivalTime;
    });

    while (completed != n) {
        int minIndex = -1, minBurst = 1e9;

        for (int i = 0; i < n; i++) {
            if (processes[i].arrivalTime <= currentTime && processes[i].remainingTime > 0 && processes[i].burstTime < minBurst) {
                minBurst = processes[i].burstTime;
                minIndex = i;
            }
        }

        if (minIndex == -1) {
            currentTime++; // No process is ready, increment time
            continue;
        }

        processes[minIndex].startTime = currentTime;
        processes[minIndex].completionTime = currentTime + processes[minIndex].burstTime;
        processes[minIndex].turnaroundTime = processes[minIndex].completionTime - processes[minIndex].arrivalTime;
        processes[minIndex].waitingTime = processes[minIndex].turnaroundTime - processes[minIndex].burstTime;
        processes[minIndex].responseTime = processes[minIndex].startTime - processes[minIndex].arrivalTime;

        ganttChart.push_back({processes[minIndex].id, currentTime});
        currentTime += processes[minIndex].burstTime; // Move current time forward
        processes[minIndex].remainingTime = 0; // Mark process as completed
        completed++;
    }

    // Print results
    cout << "\nPID\tAT\tBT\tCT\tTAT\tWT\tRT\n";
    for (const auto& p : processes) {
        cout << p.id << "\t" << p.arrivalTime << "\t" << p.burstTime << "\t"
             << p.completionTime << "\t" << p.turnaroundTime << "\t"
             << p.waitingTime << "\t" << p.responseTime << "\n";
    }

    printGanttChart(ganttChart);
}

// Priority Scheduling - Non-Preemptive
void priority_non_preemptive(vector<Process>& processes) {
    int n = processes.size();
    vector<pair<int, int>> ganttChart;
    int currentTime = 0, completed = 0;

    // Sort processes by arrival time
    sort(processes.begin(), processes.end(), [](Process a, Process b) {
        return a.arrivalTime < b.arrivalTime;
    });

    while (completed != n) {
        int highestPriority = 1e9, minIndex = -1;

        for (int i = 0; i < n; i++) {
            if (processes[i].arrivalTime <= currentTime && processes[i].remainingTime > 0 && processes[i].priority < highestPriority) {
                highestPriority = processes[i].priority;
                minIndex = i;
            }
        }

        if (minIndex == -1) {
            currentTime++; // No process is ready, increment time
            continue;
        }

        processes[minIndex].startTime = currentTime;
        processes[minIndex].completionTime = currentTime + processes[minIndex].burstTime;
        processes[minIndex].turnaroundTime = processes[minIndex].completionTime - processes[minIndex].arrivalTime;
        processes[minIndex].waitingTime = processes[minIndex].turnaroundTime - processes[minIndex].burstTime;
        processes[minIndex].responseTime = processes[minIndex].startTime - processes[minIndex].arrivalTime;

        ganttChart.push_back({processes[minIndex].id, currentTime});
        currentTime += processes[minIndex].burstTime; // Move current time forward
        processes[minIndex].remainingTime = 0; // Mark process as completed
        completed++;
    }

    // Print results
    cout << "\nPID\tAT\tBT\tPriority\tCT\tTAT\tWT\tRT\n";
    for (const auto& p : processes) {
        cout << p.id << "\t" << p.arrivalTime << "\t" << p.burstTime << "\t"
             << p.priority << "\t" << p.completionTime << "\t"
             << p.turnaroundTime << "\t" << p.waitingTime << "\t" << p.responseTime << "\n";
    }

    printGanttChart(ganttChart);
}

// Round Robin Scheduling
void roundRobin(vector<Process>& processes, int timeQuantum) {
    queue<int> readyQueue;
    int currentTime = 0, completed = 0;
    vector<pair<int, int>> ganttChart;

    // Initialize remaining time for each process
    for (auto& p : processes) {
        p.remainingTime = p.burstTime;
    }

    // Push all processes into the ready queue
    for (int i = 0; i < processes.size(); i++)
        readyQueue.push(i);

    while (completed != processes.size()) {
        int index = readyQueue.front();
        readyQueue.pop();

        // If the process has just arrived
        if (processes[index].remainingTime == processes[index].burstTime)
            processes[index].startTime = max(currentTime, processes[index].arrivalTime);

        // Execute the process for the time quantum or remaining time
        int executionTime = min(timeQuantum, processes[index].remainingTime);
        ganttChart.push_back({processes[index].id, currentTime});
        processes[index].remainingTime -= executionTime;
        currentTime += executionTime;

        // If the process is completed
        if (processes[index].remainingTime == 0) {
            processes[index].completionTime = currentTime;
            processes[index].turnaroundTime = processes[index].completionTime - processes[index].arrivalTime;
            processes[index].waitingTime = processes[index].turnaroundTime - processes[index].burstTime;
            processes[index].responseTime = processes[index].startTime - processes[index].arrivalTime;
            completed++;
        } else {
            readyQueue.push(index); // Re-add process to the queue
        }
    }

    // Print results
    cout << "\nPID\tAT\tBT\tCT\tTAT\tWT\tRT\n";
    for (const auto& p : processes) {
        cout << p.id << "\t" << p.arrivalTime << "\t" << p.burstTime << "\t"
             << p.completionTime << "\t" << p.turnaroundTime << "\t"
             << p.waitingTime << "\t" << p.responseTime << "\n";
    }

    printGanttChart(ganttChart);
}

int main() {
    int n, choice, timeQuantum;

    cout << "Enter number of processes: ";
    cin >> n;

    vector<Process> processes;
    for (int i = 0; i < n; i++) {
        int at, bt, pr;
        cout << "Enter Arrival Time, Burst Time, and Priority (0 for FCFS/SJF/RR): ";
        cin >> at >> bt >> pr;
        processes.emplace_back(i + 1, at, bt, pr);
    }

    cout << "Choose Scheduling Algorithm:\n1. FCFS\n2. SJF\n3. Priority\n4. Round Robin\n";
    cin >> choice;

    switch (choice) {
        case 1:
            fcfs(processes);
            break;
        case 2:
            sjf_non_preemptive(processes);
            break;
        case 3:
            priority_non_preemptive(processes);
            break;
        case 4:
            cout << "Enter Time Quantum: ";
            cin >> timeQuantum;
            roundRobin(processes, timeQuantum);
            break;
        default:
            cout << "Invalid choice!" << endl;
            break;
    }

    return 0;
}
