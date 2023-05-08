Download Link: https://assignmentchef.com/product/solved-programming-assignment-2-lab-2-scheduler-dispatcher
<br>
You are to implement a Scheduler in C, C++ (only) and submit the <strong>source </strong>code through NYU classes, which we will compile and run. The submission must include a <em>Makefile</em> for building your code with additional instructions if necessary wrt specific compiler versions etc. Please do not submit inputs and outputs (we already got them).




In this lab we explore the implementation and effects of different scheduling policies discussed in class on a set of processes/threads executing on a system. The system is to be implemented using Discrete Event Simulation (DES) <strong>(http://en.wikipedia.org/wiki/Discrete_event_simulation)</strong>. In discrete-event simulation, the operation of a system is represented as a chronological sequence of events. Each event occurs at an instant in time and marks a change of state in the system. This implies that the system progresses in time through defining and executing the events (state transitions) and by progressing time discretely between the events as opposed to incrementing time continuously (<strong>e.g. don’t do <em>“sim_time++”</em></strong>). Events are removed from the event queue in chronological order, processed and new events might be created during the simulation. Note that DES has nothing to do with OS, it is just an awesome generic way to step through time and simulating system behavior that you can utilize in many system simulation scenarios.




A process behavior is characterized by 4 parameters:

Arrival Time (AT), Total CPU Time (TC), CPU Burst (CB) and IO Burst (IO).




Each process follows the following state diagram that should be used to guide you in defining the events (state transitions):

<img decoding="async" data-recalc-dims="1" data-src="https://i0.wp.com/www.ankitcodinghub.com/wp-content/uploads/2018/06/954.png?w=980&amp;ssl=1" class="aligncenter lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" class="aligncenter" src="https://i0.wp.com/www.ankitcodinghub.com/wp-content/uploads/2018/06/954.png?w=980&amp;ssl=1" data-recalc-dims="1">

 </noscript>

Initially when a process arrives at the system it is put into CREATED state. The processes’ CPU and the IO bursts are statistically defined. When a process is scheduled (becomes RUNNING (transition 2)) the CPU burst is defined as a random number between [ 1 .. CB ]. If the remaining execution time is smaller than the random number chosen, reduce the random integer number to the remaining execution time. When a process finishes its current CPU burst (assuming it has not yet reached its total CPU time), it enters into a period of IO (aka BLOCKED) (transition 3) at which point the IO burst is defined as a random number between [ 1 .. IO ]. If the previous CPU burst was not yet exhausted due to preemption (transition 5), then no new CPU burst shall be computed yet in transition 2.




Note, you are not implementing this as a multi-program or multi-threaded application. By using DES, a process is simply the PCB object that goes through discrete state transitions. In the PCB object you maintain the state and statistics of the process as any OS would do. In reality the OS doesn’t get involved during the execution of the program (other than system calls) but only at the scheduling events.




We make a few simplifications:

<ul>

 <li>all time is based on integers not floats, hence nothing will happen or has to be simulated between integer numbers;</li>

 <li>to enforce a uniform repeatable behavior, a file with random numbers is provided (see NYU classes attachment) that your program must read in and use (note the first line defines the count of random numbers in the file) a random number is then created by using (don’t make assumptions about the number of random numbers):</li>

</ul>

“<em>int myrandom(int burst) { return 1 + (randvals[ofs] % burst); }”</em>      // yes you can copy the code




You should increase ofs with each invocation and <strong><em>wrap around</em></strong> when you run out of numbers in the file/array. It is therefore important that you call the random function only when you have to, namely for transitions 2 and 3 (with noted exceptions) and the initial assignment of the static priority.

<ul>

 <li>IOs are independent from each other, i.e. they can commensurate concurrently without affecting each other’s IO burst time.</li>

</ul>




The input file provides a separate process specification (see above) in each line: AT TC CB IO

You can make the assumption that the input file is well formed and that the ATs are not decreasing. So no fancy parsing is required. It is possible that multiple processes have the same arrival times. Then the order at which they are presented to the system is based on the order they appear in the file. Simply, create a process start event for all processes in the input file and enter these events into the event queue. Then and only then start the event simulation. Naturally, when the event queue is empty the simulation is completed.




The scheduling algorithms to be simulated are:

FCFS, LCFS, SJF, RR (RoundRobin), and PRIO (PriorityScheduler). In RR &amp; PRIO your program should accept the <em>time quantum</em> as an input (see below “<em>Execution and Invocation Format</em>”). We will test with multiple <em>time quantum,</em> so do not make any assumption that it is a fixed number. The context switching overhead is “0”.




You have to implement the scheduler as “objects” without replicating the event simulation infrastructure for each case, i.e. you define one interface to the scheduler (e.g. <em>add_process()</em>, <em>get_next_process()</em>) and implement the schedulers using object oriented programming (inheritance). The proper “scheduler object” is selected at program starttime based on the “-s” parameter. The rest of the simulation must stay the same (e.g. event handling mechanism and <em>Simulation()</em>). The code must be properly documented. When reading in the process specification, assign a static_priority to the process using myrandom(4) (see above) which will select a priority between 1..4. A process’s dynamic priority is defined between [ 0 .. (static_priority-1) ]. With every quantum expiration the dynamic priority decreases by one. When “-1” is reached the prio is reset to (static_priority-1). Please do this for all schedulers though it only has implications for the PRIO scheduler as all other schedulers do not take priority into account. However uniformly calculating this will enable a simpler and scheduler independent state transition implementation.




<strong>A few things you need to pay attention to: </strong>

Round Robin: you should only regenerate a new CPU burst, when the current one has expired.

SJF: schedule is based on the shortest remaining execution time, not shortest CPU burst.

PRIO: same as Round Robin plus: the scheduler has exactly 4 priority levels [0..3], 3 the highest. Please use the concept of an active and an expired array/queue and utilize independent process lists at each prio level as discussed in class. When “-1” is reached the process’s dynamic priority is reset to (static_priority-1) and it is enqueued into the expired queue. When the active queue is empty, active and expired are switched.

Remember runqueue under PRIO is the combination of active and expired.

All: When a process returns from I/O its dynamic priority is reset (to (static_priority-1).




<strong>Output: </strong>

<strong> </strong>

At the end of the program you should print the following information and the example outputs provide the proper expected formatting (including precision); this is necessary to automate the results checking; all required output should go to the console ( stdout / cout ).

<ol>

 <li>Scheduler information (which scheduler algorithm and if RR the quantum)</li>

 <li>Per process information (see below):</li>

</ol>

for each process (assume processes start with pid=0), the correct desired format is shown below: pid: AT   TC   CB   IO  PRIO   |      FT    TT   IT   CW

FT: Finishing time

TT: Turnaround time ( finishing time  –   AT )

IT:   I/O Time ( time in blocked state)

PRIO:  static priority assigned to the process ( note this only has meaning in PRIO case )

<ol>

 <li>CW: CPU Waiting time ( time in Ready state )</li>

 <li>Summary Information – Finally print a summary for the simulation:</li>

</ol>

Finishing time of the last event (i.e. the last process finished execution)

CPU utilization (i.e. percentage (0.0 – 100.0) of time at least one process is running

IO utilization     (i.e. percentage (0.0 – 100.0) of time at least one process is performing IO

Average turnaround time among processes

Average cpu waiting time among processes

Throughput of number processes per 100 time units




CPU / IO utilizations and throughput are computed from time=0 till the finishing time.

Example:

FCFS

0000:      0  100   10   10 0 |  223  223  123    0

0001:    500  100   20   10 0 |  638  138   38    0 SUM:    638 31.35 25.24 180.50 0.00 0.313




You must <strong><u>strictly </u></strong>adhere to this format. The programs results will be graded by a testing harness that uses “diff –b”. In particular you must pay attention to separate the tokens and to the rounding. In the past we have noticed that different runtimes (C vs. C++) use different rounding.  For instance 1/3 was rounded to 0.334 in one environment vs. 0.333 in the other  ( similar 0.666 should be rounded to 0.667 ). <strong>Always use double</strong> (instead of float) variables where non-integer computation occurs. See <em>outformat.c</em> in assignment file.

In C++ you must specify the precision and the rounding behavior.




If in doubt, here is a small C program (gcc) to test your behavior (you can transfer to C++ and verify):

#include &lt;stdio.h&gt; main() {

double a,b;     a = 1.0/3.0;     b = 2.0/3.0;

printf(“%.2lf %.2lf
”,  a, b);     printf(“%.3lf %.3lf
”,  a, b);

}

Should produce the following output

0.33 0.67

0.333 0.667




Use the following printf’s (or design your equivalents for C++) to print out the per-process and summary report.

See C++ examples in ~frankeh/Public/ProgExamples.tz  ( Format subdirectory for C and C++).

printf(“%04d: %4d %4d %4d %4d %1d | %5d %5d %5d %5d
”,  printf(“SUM: %d %.2lf %.2lf %.2lf %.2lf %.3lf
”,

<strong>note </strong>“ %4d %4d” is not equivalent to “%5d%5d” .. this is often a source of problems.




<strong>Deterministic Behavior: </strong>




There will be scenarios where events will have the same time stamp and you <strong><u>must</u></strong> follow these rules to break the ties in order to create consistent behavior:

<ul>

 <li>Processes with the same arrival time should be entered into the run queue in the order of their occurrence in the input file.</li>

 <li>On the same process: termination takes precedence over scheduling the next IO burst over preempting the process on quantum expiration.</li>

 <li>Events with the same time stamp (e.g. IO completing at time X for process 1 and cpu burst expiring at time X for process 2) should be processed in the order they were generated, i.e. if the IO start event (process 1 blocked event) occurred before the event that made process 2 running (naturally has to be) then the IO event should be processed first.</li>

</ul>

If two IO bursts expire at the same time, then first process the one that was generated earlier.

<ul>

 <li>You must process all events at a given time stamp before invoking the scheduler/dispatcher (See <em>Simulation() </em>at end).</li>

</ul>




Not following these rules implies that fetching the next random number will be out of order and a different event sequence will be generated. The net is that such situations are very difficult to debug (see for relieve further down).




<strong><u>ALSO:</u> </strong>

Do not keep events in separate queues and then every time stamp figure which of the events might have fired. E.g. keeping different queues for when various I/O will complete vs a queue for when new processes will arrive etc. will result in incorrect behavior. There should be effectively two logical queues:

<ol>

 <li>An event queue that drives the simulation and</li>

 <li>the run queue/ready queue(s) [same thing] which are implemented inside the scheduler object classes.</li>

</ol>

<strong>These queues are independent from each other</strong>. In reality there can be at most one event pending per process and a process cannot be simultaneously referred to by an event in the event queue and be referred to in a runqueue (I leave this for you to think about why that is the case). Be aware of C++ build in container classes, which often pass arguments by value. When you use lists or similar containers from C++ for process object lists, the object will most likely be passed by value and hence you will create a new object. As a result you will get wrong accounting and that is just plain wrong. There should only be one process object per process in the system. To avoid this make lists of process pointers (  list&lt;Process*&gt; ).

<strong>Execution and Invocation Format: </strong>

Your program should follow the following invocation:

&lt;program&gt;  [-v]  [-s&lt;schedspec&gt;]  inputfile   randfile




Options should be able to be passed in any order. This is the way a good programmer will do that.  http://www.gnu.org/software/libc/manual/html_node/Example-of-Getopt.html




Test input files and the sample file with random numbers are NYU classes attachment.

The scheduler specification is “–s [ FLS | R&lt;num&gt; | P&lt;num&gt; ]”, where F=FCFS, L=LCFS, S=SJF and R10 and P10 are RR and PRIO with quantum 10. (e.g.   <em>“./sched –sR10”</em>). Supporting this parameter is required and the quantum is a positive number.




The –v option stands for verbose and should print out some tracing information that allows one to follow the state transition. Though this is <strong><u>not</u></strong> mandatory, it is highly suggested you build this into your program to allow you to follow the state transition and to verify the program. I include samples from my tracing for some inputs (not all). Matching my format will allow you to run diffs and identify why results and where the results don’t match up. You can always use /home/frankeh/Public/sched to create your own detailed output for not provided samples.




Two scripts “runit.sh” and “diffit.sh” are provided that will allow you to simulate the grading process. “runit.sh” will generate the entire output files and “diffit.sh” will compare with the outputs supplied. SEE &lt;README.txt&gt;




Please ensure the following:

<ul>

 <li>The input and randfile must accept any path and should not assume a specific location relative to the code or executable.</li>

 <li>All output must go to the console (due to the harness testing)</li>

 <li><strong>All code/grading will be executed on machine &lt;courses2/3.cims.nyu.edu&gt;</strong> to which you can log in using “ssh &lt;userid&gt;@courses2/3.cims.nyu.edu”. You should have an account by default, but you might have to tunnel through access.cims.nyu.edu.</li>

</ul>




As always, if you detect errors in the sample inputs and outputs, let me know immediately so I can verify and correct if necessary. Please refer the input/output file number and the line number.




<strong>Reference Program: </strong>

<strong> </strong>

The reference program used for grading is accessible on my CIMS account under <em>/home/frankeh/Public/sched </em>and you can run inputs against it to determine whether your output matches or not if you want to go beyond the provided inputs/outputs.




<strong>Scoring and deductions: </strong>

<strong> </strong>

We score this lab as 100pts. You will receive 40 pts for a submission that attempts to solve the problem. The rest you get

60/N points for each successful test that passes the “<em>diff</em>”. Due to the difference of complexity, F,R,S scheduler are 15% each, RR is 20% and Prio is 35% (of the 60 points). In order to institute a certain software engineering discipline, i.e. following a specification and avoiding unintended releases of code and data in real life, we account for the following additional deductions:




<table width="686">

 <tbody>

  <tr>

   <td width="253"><strong>Reason </strong></td>

   <td width="78"><strong>Deduction </strong></td>

   <td width="355"><strong>How to avoid </strong></td>

  </tr>

  <tr>

   <td width="253">Makefile not working on CIMS or missing.</td>

   <td width="78">2pts</td>

   <td width="355">Just follow instructions above or see lab1.</td>

  </tr>

  <tr>

   <td width="253">Late submission</td>

   <td width="78">2pts/day</td>

   <td width="355">Upto 7 days. After which reach out to me or TA but work on next lab (don’t fall behind).</td>

  </tr>

  <tr>

   <td width="253">Inputs/Outputs or *.o files in the submission</td>

   <td width="78">1pt</td>

   <td width="355">Go through your intended submission and clean it up.</td>

  </tr>

  <tr>

   <td width="253">Output not going to the screen but to a file ( you will have to fix this )</td>

   <td width="78">1pt</td>

   <td width="355">We utilize the output to &lt;stdout&gt; during the runit.sh and gradeit.sh so just use printf or cout.</td>

  </tr>

  <tr>

   <td width="253">Replicating Event and Simulation per scheduler</td>

   <td width="78">6 pts</td>

   <td width="355">Use object oriented coding style and code fragments at the end for the simulation.</td>

  </tr>

 </tbody>

</table>




<strong><u>Explanation of the verbose output:</u> </strong>




Two examples of an event in my trace




<strong><u>Example 1:</u> </strong>




57 0 12: BLOCK -&gt; READY




At timestamp 57 process 0 is going from BLOCKED into READY state. The process has been in its current state for 12 units (hence it must have been BLOCKED at time 45).







<strong><u>Example 2:</u> </strong>




42 2 7: RUNNG -&gt; BLOCK  ib=3 rem=77




At time unit 42 the transition for process 2 to BLOCKED state is executed.

It was in RUNNING state since time timeunit 35.

The IO burst created is ib=3 and there remains 77 time units (rem=77) left for executing this job.




By providing this extended output you will be able to create a detailed trace for your execution and compare it to the reference and identify where you start to differ. At a point of difference you should see which rule potentially was deployed that choose a different job/event in the reference vs. your program.




<strong><u>Some hints on structuring your program.</u></strong>




The structure / modules of your program should look something like the following.




<img decoding="async" data-recalc-dims="1" data-src="https://i0.wp.com/www.ankitcodinghub.com/wp-content/uploads/2018/06/740.png?w=980&amp;ssl=1" class="aligncenter lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" class="aligncenter" src="https://i0.wp.com/www.ankitcodinghub.com/wp-content/uploads/2018/06/740.png?w=980&amp;ssl=1" data-recalc-dims="1">

 </noscript>Start by reading in the input file and creating Process objects.

Then program a generic DES Layer, which basically means you need to be able to create events that take the timestamp when it is supposed to fire, a pointer to the Process (don’t pass by value, as there can only be ONE object related a process, otherwise your accounting will be incorrect) and the state you want to transitions to (see diagram). Make sure when you enter the event it is inserted based on the prior description. Don’t use sort() functions as they are inefficient in this case, often don’t fit the problem (know your stable vs instable sort behavior otherwise) and are simply overkill (e.g. use insert_sort()).




Next implement ONE scheduler (suggest you start with FIFO as we might not have covered all schedulers by the time of handing out the problem). Implement the schedulers as a class hierarchy (C++ or for C see <em>~frankeh/Public/ProgExamples.tz</em>  subdir VirtFunC). Note from the code fragments below, the Simulation() should not know any details about the specific scheduler itself, so all has to be accomplished through <em>virtual </em>functions. One trick to deal with schedulers is to treat nonpreemptive scheduler as preemptive with very large quantum (10K is good for our simulation) that will never fire. This way the <em>TRANS_TO_RUN  </em>transition is implemented generically.




After you have created the process objects and after you have put initial events for processes’s arrival into the event queue, the simulation can start. The simulation code structure will look something like below (very sketchy, after all you are supposed to write the code). Note (again) that runqueue/readyqueue has nothing to do with the event queue, they are completely different entities.




Also based on the sketchy code, note that the simulation knows no details about the run/ready queue or other details from the scheduler. It simply adds process to the runqueue (transitions 1,4,5) or ask the scheduler for the next process to run (there might not be one, at which point the scheduler returns NULL).




void Simulation() {          EVENT* evt;

while( (evt = get_event()) ) {

Process *proc = evt-&gt;evtProcess;    // this is the process the event works on                  CURRENT_TIME = evt-&gt;evtTimeStamp;

evt-&gt;evtProcess-&gt;timeInPrevState = CURRENT_TIME – proc-&gt;state_ts;

switch(evt-&gt;transition) {  // which state to transition to?                  case TRANS_TO_READY:

// must come from BLOCKED or from PREEMPTION

// must add to run queue

CALL_SCHEDULER = true; // conditional on whether something is run                          break;                  case TRANS_TO_RUN:

// create event for either preemption or blocking                          break;                  case TRANS_TO_BLOCK:

//create an event for when process becomes READY again

CALL_SCHEDULER = true;                          break;                  case TRANS_TO_PREEMPT:

// add to runqueue (no event is generated)

CALL_SCHEDULER = true;                          break;

}

//remove current event object from Memory                  delete evt;                  evt = nullptr;




if(CALL_SCHEDULER) {

if (get_next_event_time() == CURRENT_TIME) {

continue;//process next event from Event queue

}

CALL_SCHEDULER = false;

if (CURRENT_RUNNING_PROCESS == nullptr) {

CURRENT_RUNNING_PROCESS = THE_SCHEDULER-&gt;get_next_process();                                  if (CURRENT_RUNNING_PROCESS == nullptr) {                                          continue;

}

}

// create event to make process runnable for same time.

}

} }







Some other useful suggestions ( read this how things are done in real programming to be readable and efficient ), since learning proper programming is a side goal/direct.




    Do not represent process states or transitions as strings or integers. Use enumerations. Let the compiler do the hard work. Why?  Assume you want to represent states using strings (e.g. “STATE_RUNNING”, “STATE_BLOCKED”). First to store the current state in a process requires memory to hold the string which above would be 16 bytes or if you are clever could be stored in a pointer (last one not to bad). At some point you have to check a processes state and you have to code something like

<em>if (proc-&gt;state == “STATE_RUNNING”)</em>

which has two problems. (a) you compiler might convert that into a string comparison OR a pointer comparison .. do you know the difference ?  à debugging nightmare.  (b) string compares can/are be inefficient




Integers are unreadable and code can’t be maintained. Have mercy on those that potentially have to deal with your code in the real world.

<em>            if (proc-&gt;state == 1)   </em>

What does that mean?




Instead you would have in C/C





typedef enum { STATE_RUNNING , STATE_BLOCKED }   process_state_t;              if (proc-&gt;state == STATE_RUNNING)




This is efficient and readable. An enumeration takes at most an integer worth (4 bytes) and the compiler converts names to integers starting with 0.




<ul>

 <li>Don’t use vectors/arrays for runqueue/readyqueue. You can not efficiently add/remove at different locations, use lists or queues. (exceptions is the priority levels which should be implemented as vectors/arrays of lists/queues).</li>

</ul>




<ul>

 <li>Don’t create lists of processes. List&lt;Process&gt; implies that when you add a Process to a list you will make a full copy of a process. This is incorrect and will <span style="text-decoration: line-through;">potentially</span> lead to wrong implementations. There can only be ONE process object per running process. It also is inefficient, in a real OS the process object is ~4K. Instead create a process object while you are reading the input and then create List&lt;Process*&gt; for the ready/runqueue and passing pointers to that object. As a result you always point to that one process which is important for accouting and efficiency, let alone for correctness.</li>

</ul>




<ul>

 <li>The scheduler classes really have to provide only two functions:</li>

</ul>




void add_to_qeueue(Process *p);

Process* get_next_process();




<ul>

 <li>To make the while loop uniform, you can pretend that non-preemptive schedulers have a very large quantum (e.g. 10000) which essentially means that no preemption will ever occur for those schedulers.</li>

</ul>


