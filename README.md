# Job Scheduler with Priority and Concurrency Limitation

<br>

## Overview


**This Bash script implements a job scheduler that reads jobs from a text file, assigns priorities to each job, and executes them in a manner where the jobs are processed based on their priority. The script also controls the number of concurrent jobs that can run simultaneously. The jobs are sorted based on their priority and executed one by one while ensuring that the system does not exceed the specified number of concurrent jobs at any given time.**

<br>

## Scenario Example:

**Suppose you have thousands of nodes and repetitive daily tasks, like running certain commands on each node. This script can efficiently handle these tasks by automating job execution across nodes, prioritizing tasks as needed, and speeding up the process by running multiple jobs concurrently on all nodes. This saves significant time and effort compared to manual execution.**

*The scheduler uses a basic priority system where lower numerical values represent higher priority. It ensures that at most a specified number of jobs are executed concurrently. It also tracks the jobs being run and waits for them to complete before starting new ones.*

## Features

### 1. Job Priority Management:
*Jobs are executed based on their assigned priority (the lowest number is the highest priority).*

- Concurrency Limitation:

   - Limits the number of jobs that can run concurrently, preventing the system from being overloaded.

- Dynamic Job Management:
        - Continuously checks for running jobs, ensuring no more than the specified number of jobs run at once.

Job Execution:
The script supports any command passed to it, allowing flexibility in what jobs can be executed.

Use Case
This script is useful in scenarios where you need to execute multiple jobs (commands) in a controlled manner. It can be applied in various use cases, such as:

Running system commands or scripts that are resource-intensive.
Batch processing tasks where certain tasks must have higher priority.
Situations where you need to limit the number of concurrent processes to avoid overloading the system.
How It Works
Input Format
The script takes two arguments:

A file containing a list of jobs and their priorities.
A maximum number of concurrent jobs that can run simultaneously.
Jobs File (jobs.txt)
The jobs file consists of lines where each line contains:

Priority: An integer value representing the priority of the job. A smaller number indicates a higher priority.
Command: A command (or script) that will be executed.
Example jobs.txt file:

bash
Copy code
8 sleep 8
9 sleep 10
10 sleep 10
6 pwd
7 uname -r
1 sleep 3
2 sleep 3
3 sleep 3
4 ls
11 hostname
13 whoami
12 free
5 uptime
14 hostname -I
Execution Flow
Reading Input:
The script reads the provided file line by line, extracting the priority and command from each line.

Sorting:
The list of jobs is sorted by priority (ascending order).

Job Execution:

Jobs are executed in the sorted order.
Before executing a job, the script checks the number of concurrently running jobs.
If the maximum concurrent job limit is reached, the script waits for one or more jobs to finish before starting a new one.
Concurrency Management:
It uses the kill -0 command to check whether a job is still running. If a job finishes, its process ID is removed from the list of running jobs.

Completion:
The script waits for all jobs to complete using the wait command.

Script Breakdown
Bash Script:
bash
Copy code
#!/bin/bash

# Function to run jobs with priority and concurrency limitation
function runme() {
    local file=$1
    local max_concurrent_jobs=$2
    jobs_arr=()

    # Read the jobs and their priorities from the provided file
    while read -r line; do
        priority=$(echo "$line" | cut -d ' ' -f1)
        command=$(echo "$line" | cut -d ' ' -f2-)
        jobs_arr+=("$priority $command")
    done < "$file"

    # Sort jobs by priority (ascending)
    IFS=$'\n' sorted_jobs=($(sort -n -k1 <<<"${jobs_arr[*]}"))
    unset IFS

    pids=() # Array to store process IDs

    # Iterate through the sorted jobs and execute them
    for ((i = 0; i < ${#sorted_jobs[@]}; i++)); do
        # Wait for available slots if the max concurrent jobs are reached
        while [ ${#pids[@]} -ge $max_concurrent_jobs ]; do
            for pid in "${pids[@]}"; do
                if ! kill -0 "$pid" 2>/dev/null; then
                    pids=(${pids[@]/$pid}) # Remove finished jobs from the list
                    break
                fi
            done
        done

        # Extract job priority and command
        job_info=(${sorted_jobs[$i]})
        priority=${job_info[0]}
        command="${job_info[@]:1}"

        # Run the job
        echo "Running job with priority $priority: $command"
        eval "$command" & # Run command in the background
        pids+=("$!") # Store the process ID of the background job

        echo "Current running jobs: ${pids[@]}"
    done

    # Wait for all jobs to finish
    wait
}

# Check if enough arguments are provided
if [ "$#" -lt 2 ]; then
    echo "Usage: $0 <file_with_jobs_and_priorities> <max_concurrent_jobs>"
    exit 1
fi

# Execute the function and measure the time taken
time runme "$1" "$2"
How to Run
Prepare the Jobs File:
Create a jobs.txt file containing the jobs and their priorities. Each line should follow this format:

bash
Copy code
<priority> <command>
Execute the Script:
bash
Copy code
./script_name.sh jobs.txt <max_concurrent_jobs>
Replace script_name.sh with the actual name of the script file, jobs.txt with your file, and <max_concurrent_jobs> with the desired number of concurrent jobs.

Example Usage:
Create jobs.txt:
bash
Copy code
8 sleep 8
9 sleep 10
10 sleep 10
6 pwd
7 uname -r
1 sleep 3
2 sleep 3
3 sleep 3
4 ls
11 hostname
13 whoami
12 free
5 uptime
14 hostname -I
Run the Script:
bash
Copy code
chmod +x script_name.sh
./script_name.sh jobs.txt 3
This will run the jobs in jobs.txt, ensuring no more than 3 jobs are running at the same time.

Conclusion
This script provides a simple yet effective way to manage the execution of multiple jobs with different priorities and concurrency limitations. It can be used for system administration tasks, batch processing, or any scenario requiring controlled execution of jobs.

Let me know if further adjustments or details are needed
