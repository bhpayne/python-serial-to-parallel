Ben Payne
2015-01-16

# serial 
```bash
$ python serial_print_to_file.py
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100]
```
this Python script reads from "parameters.input" to determine how many integers to write to a file and to stdout
The integers get printed as plain text in output.dat and as Pickle binary in output.pkl

# multiple independent workers, sequential 
```bash
$ ./launch_workers.sh 6 print_to_file_worker.py
```
This bash script launches the "print_to_file_worker.py" script which generates two sets of output files,
`output_<N>.dat` and `output_<N>.pkl`
where <N> is the index for the number of files (here 6, so files are numbered 1 to 6)
Since 6 does not divide 100, the last file (output_6) contains more lines than the others
```bash
$ wc -l output_*.dat
      16 output_1.dat
      16 output_2.dat
      16 output_3.dat
      16 output_4.dat
      16 output_5.dat
      20 output_6.dat
     100 total
```
Each worker includes a "sleep" delay to replicate the behavior of a computational task.
Since the set of independent workers is sequential, the total time is a sum of each worker delay

To join these files, 
```bash
$ python join_worker_output.py 6
```
checking that this produced the right output,
```bash
$ wc -l output_from_joiner.dat 
     100 output_from_joiner.dat
```
## math of dividing indices among workers ====

Suppose I have 1:13 and 3 computation cores
```
indices:   1 2 3 4 5 6 7 8 9 10 11 12 13
CPU cores: <- 1 -> <- 2 -> <-    3    ->
```
let num_items  = 13
let num_cores  =  3

then chunk_size = floor(13/3) = 4

let core_indx = 1:num_cores = 1:3
then we want
```
core_indx | start | end
----------------------
     1    |   1   |  4
     2    |   5   |  8
     3    |   9   | 13
```
To capture this, let
start = (core_indx-1)*chunk_size+1
if core_indx = num_cores, then end = num_items
else end = core_indx*chunk_size



# multiple independent workers, parallel 

The launch_workers bash script can either run the "print_to_file_worker" scripts sequentially or, using subshells, in parallel

sequential:      
python $2 $core_indx $1

parallel: 
( python $2 $core_indx $1 ) &

When run in parallel, the total delay is the time for one worker to run

