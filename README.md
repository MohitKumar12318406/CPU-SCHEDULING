# CPU Scheduler Simulator
A web-based CPU scheduling algorithm simulator that demonstrates various scheduling algorithms including:
- First Come First Serve (FCFS)
- Shortest Job First (SJF)
- Round Robin (RR)

## Features
- Interactive web interface
- Real-time visualization of process execution
- Multiple scheduling algorithms
- Calculation of important metrics (turnaround time, waiting time)
- Visual timeline of process execution

## Setup and Running
1. Install the required dependencies:
   ```
   pip install -r requirements.txt
   ```

2. Run the application:
   ```
   python app.py
   ```

3. Open your web browser and navigate to `http://localhost:5000`

## Usage
1. Select a scheduling algorithm from the dropdown
2. Add processes by clicking the "Add Process" button
3. For each process, specify:
   - Arrival Time: When the process arrives
   - Burst Time: How long the process needs to execute
4. For Round Robin, specify the time quantum
5. Click "Simulate" to see the results

## Metrics Explained
- Turnaround Time: Time taken from process arrival to completion
- Waiting Time: Time spent waiting in the ready queue
- Timeline: Visual representation of process execution order
