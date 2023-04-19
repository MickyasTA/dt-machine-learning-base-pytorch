#!/usr/bin/env python3
import argparse
import datetime
import multiprocessing
import time

import torch

# To run this script: dts --debug devel run -H ROBOT_NAME --build -t ente -M -s -f -L test-stresstest -A stress_time=30

# This is a safety variable. Do not change!
MAX_DURATION = 300  # Maximum allow to run 300 seconds (5 minutes)


def spawn_gpu():
    end_time = time.time() + MAX_DURATION
    time_str = datetime.datetime.fromtimestamp(end_time).strftime("%H:%M:%S.%f")
    print(f"GPU Thread Max Execution till: {time_str}")
    try:
        x = torch.linspace(0, 4, 16 * 1024 ** 2).cuda()
        while time.time() < end_time:
            x = x * (1.0 - x)
        print(f"GPU Thread shutdown due to safety trigger!")
    except KeyboardInterrupt:
        print("External Shutdown Received!")


def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True


def find_next_prime(n):
    while not is_prime(n):
        n += 1
    return n


# A function used for stress testing CPU per thread
def prime_runner(thread_id):
    end_time = time.time() + MAX_DURATION
    time_str = datetime.datetime.fromtimestamp(end_time).strftime("%H:%M:%S.%f")
    print(f"CPU Thread {thread_id} Max Execution till: {time_str}")
    try:
        n = 2
        iterations = 0
        while time.time() < end_time:
            n = find_next_prime(n + 1)
            iterations += 1
        print(f"CPU Thread {thread_id} shutdown due to safety trigger!")
    except KeyboardInterrupt:
        print("External Shutdown Received!")


def run():
    # Get the number of CPU threads
    num_threads = multiprocessing.cpu_count()
    print(f"Total stressing CPU counts: {num_threads}")

    # Set up a list of worker processes
    workers = []

    # GPU Worker
    worker_gpu = multiprocessing.Process(target=spawn_gpu)
    workers.append(worker_gpu)
    worker_gpu.start()

    # CPU Worker
    for threads in range(num_threads - 1):
        worker = multiprocessing.Process(target=prime_runner, args=(threads,))
        workers.append(worker)
        worker.start()

    # Time Manager
    start = time.time()
    required_shutdown = start + DESIRED_BENCH_DURATION
    try:
        while time.time() < required_shutdown:
            time_str = datetime.datetime.fromtimestamp(time.time()).strftime("%H:%M:%S.%f")
            print(f"Now: {time_str}, Elapsed: {time.time() - start:.2f} seconds")
            time.sleep(1)
        print("Desired Time reached. Shutting down...")
    except KeyboardInterrupt:
        print("Pre-mature Termination call recieved. moving to shutdown")

    # Shutdowner
    for task in workers:
        task.terminate()


if __name__ == "__main__":
    # This varaiable should be modified by launcher
    parser = argparse.ArgumentParser()
    parser.add_argument("-t", "--stress_time", help="The total seconds you want to stress test your board.", default=30,
                        type=int)
    args = parser.parse_args()
    DESIRED_BENCH_DURATION = args.stress_time

    print(f"Safety trigger set at: {MAX_DURATION} seconds!")
    if DESIRED_BENCH_DURATION > MAX_DURATION:
        print(f"Request a stress test time larger than safety threshold! Will only run 300 seconds!")
    else:
        print(f"System starting stress test. Might be unresponsive!")
        print(f"Benchmarking for: {DESIRED_BENCH_DURATION} seconds!")
    print("=====================================================")
    time.sleep(5)  # So that user will see the log above
    run()
    exit(0)
