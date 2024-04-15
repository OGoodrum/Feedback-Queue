# Feedback-Queue

## Input
A C program (`feedbackq.c`) implementing a simulation of
round-robin CPU scheduling that also makes use of a multi-level
feedback queue and an anti-starvation task boosting mechanism.

feedbackq.c will accept a single argument
consisting of a text file with lines, the lines together containing
information about tasks for a particular simulation case.

For example,
to run the first test case the following would be entered at the
command line:

`./feedbackq cases/test1.txt`

Therefore an input file is a representation of a simulation case,
where each line represents one of three possible facts. For example,
the following line contained within such a file:

```
13,3,0
```

indicates that at tick 13, task 3 has been created (i.e. the 0 means creation). A line in the file such as:

```
14,3,6
```

indicates that at tick 14, task 3 initiates an action that will require 6 ticks of CPU time. There may be many such CPU-tick lines for a task contained in the simulation case.

Lastly, there will appear a line in the file such as:

```
21,3,-1
```

here indicating that at tick 21, task 3 will terminate once its
remaining burst’s CPU ticks are scheduled. (If no such CPU ticks are
remaining, then the task would be said to terminate immediately.)

## Output
Consider a line taken from an output smaple (i.e. sample
output from a simulation based on `cases/test1.txt`):

```
[00015] id=0003 req=6 used=2 queue=1
```

At tick 15, the simulator has scheduled task 3 which currently has a
most-recent CPU-tick requirement of 6 ticks, although 2 ticks of that
have been scheduled during some previous ticks. Also the task was
retrieved from queue 1 (i.e., the queue for which quantum q=2).

The very next line, however, shows an important change:

```
[00016] id=0003 req=6 used=3 queue=2
```

Although the amount of CPU ticks used (i.e. scheduled) has gone up,
the queue from which the task was scheduled has changed. That is,
after completing tick 15, task 3 had used up the whole CPU quantum
given to it, and as it still had more time left in its burst, and
according to the rules of the MLFQ, it had been enqueued after tick 15
to the next queue down (i.e. the queue for which quantum q=4).
 

Let us consider one more line from a sample output :

```
[00021] id=0003 EXIT wt=1 tat=7`
```

This indicates that at tick 21, task 3 exited the system having spent
one tick waiting in an MLFQ queue, and where the task had a
turn-around time of 7 ticks.

## Nature of the Scheduling

The program implements a multi-level feedback queue, or MLFQ. An MLFQ is
designed to ensure the tasks requiring quick responses (i.e. which are
often characterized as having short CPU bursts) are scheduled before
tasks which are more compute bound (i.e. which are characterized as
having long – or at least longer – CPU bursts).

My simulation is of an MLFQ with three different queues: one
with a quantum of 2, one with a quantum of 4, and one with a quantum
of 8.

* When there is a CPU scheduling event, the queue corresponding to q=2
is examined. If it is not empty, then the task at the front is
selected to run and given a quantum of 2. Otherwise the queue
corresponding to q=4 is examined, and if it is not empty, then the
task at the front of this queue selected to run and given a quantum of 4. Otherwise the queue corresponding to q=8 is examined, with the task
at the front of this queue selected to run and given a quantum of 8.

* If a task selected from either the q=2 or q=4 queues finishes its
current burst within quantum provided to it, then it is placed at
the back of the queue from which it was taken.

* However if a task selected from either the q=2 or q=4 queues has a
burst that exceeds its quantum, then it is interrupted (as would be
the case for any round-robin algorithm), and placed into the next
queue with a larger quantum. That is, a task taken from q=2 would be
placed at the end of the q=4 queue; a task taken from q=4 would be
placed at the end of the q=8 queue.

* Without some sort of mechanism for giving tasks a chance to leave
the q=8 queue, a task might starve - for example, a task on the q=8
queue might never get a chance to resume if it is pre-empted by a
series of new tasks on a queue with a higher priority. To prevent
this, the CPU must boost tasks from the q=8 queue up to the q=2 queue
periodically, to give them a chance to compete for CPU time with other
tasks.
