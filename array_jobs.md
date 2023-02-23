
# Array jobs (probably you should be using them)
### by Bailey Harrington
### St Patrick's Day, 2020 — Day 2 of coronavirus-induced isolation

In this tutorial, I will be covering how to write an array job script for the SunGridEngine (SGE) scheduler (much, if not all of this, seems to be the same on UnivaGridEngine schedulers), as well as one that takes in a parameter file.

Array jobs allow for funning nearly-identical jobs in parallel, rather than in sequence, and as such are often good alternatives to using a `for` loop inside a job script. Use of a parameter file in an array job allows for easily changing several small details of the job, such as the input data file and arguments to be passed to a command, while all of the actual operations of the job remain unchanged.

Another particularly useful result of using a parameter file is the possibility of giving output files more informative names that relate to the specific data for that task.

I will give two examples of array jobs. The first will use as the input for the tasks, files that contain data for the chromosomes. These, having natural numerical designations, fit quite readily into the array job framework. For the second example, the tasks will relate to a different of the data: proteins. These, unlike the chromosomes, do not have natural numeric values, so associating their names with the output files will require some extra steps.

# An array job, with chromosomes
#### *In humans, the non-sex chromosomes are numbered from 1-22.

This first example will hopefully prevent you from doing the following in the future:


```bash
# Using >> ensures new output will be appended to the destination file, not overwrite it.

for chrom in {1..22}; do
  grep -nw ind5000 chromosome_${chrom}_genotypes.tsv >> ind5000_genotypes.tsv
done
```

This, of course, works. But it is liable to take a long time to go through all of the chromosomes sequentially. Using an array job, the same thing can be accomplished in a fraction of the time by essentially running each pass of the `for` loop independently and consecutively.

We begin with the setup of the jobscript. This includes a line designating the interpreter with which to read the file and then lines designating the SGE options to use. There are many that are useful, but these are the most essential for an array job.

`-N` designates the name of the job. If omitted, the name will be derived from the jobscript name, itself.

`-t` designates the number of tasks to run, and which number to start them on (it doesn't have to start with 1).


```bash
#!/bin/bash

# Options

#$ -N job_name
#$ -t 1-22
```

Now we can get to the meat of the jobscript. When we want to refer to the task number, which in this case is a chromosome, we will use `bash`'s variable syntax and the SGE task id, like this: `${SGE_TASK_ID}`.

As an aside, I, personally, like to rename variables to more informative things to make my code more self-commenting; however, if you don't want to, you can omit this next line, and replace future instances of `${chrom}` with `${SGE_TASK_ID}`.


```bash
chrom=${SGE_TASK_ID}
```

Using this syntax to help us reference the correct input file(s), we can now run simple `bash` commands. Here I extract all of the genotype information for individual 5000 in a cohort for the chromosome corresponding to the current task, writing this output to a file:


```bash
# Remember, the \ in the following code snippet is a continuation symbol.
# It tells the interpreter the command continues onto the next line.

grep -nw ind5000 chromosome_${chrom}_genotypes.tsv > \
         ind5000_chromosome_${chrom}_genotypes.tsv
```

We can also use the variable syntax and task id to submit things into command-line software, such as `plink`. However, to do this, we need to first add the module space and load `plink`.


```bash
#add the module space
. /etc/profile.d/modules.sh
#load the plink module
module load igmm/apps/plink/1.90b4

plink --bfile input_chromosome_${chrom} \
      --recode vcf \
      --out output_chromosome_${chrom}
```

Now you've seen how to structure a simple array job, hopefully it will be clear how to write one that actually does something useful, and when you should be using array jobs!

Next, we'll look at...

# An array job with proteins
#### * The 'proteins' used herein are fictitious. Any resemblance to actual proteins, human or otherwise, is entirely coincidental.
###### ** I don't work with proteins. This will become rather apparent, rather quickly.

For this to work, we need to create a separate parameter file (obviously). It is just a text file. (Note, you would actually need to save this somewhere.)

    Protein    Chain1    Chain2  
    FB2B       M         Y  
    SVO3       L         A  
    BISV       K         U  
    NCIZ       C         C  
    MLKJ5      D         Q  
    HDGY       T         V  
    ZOI        R         A  
    DJB8       Z         N  
    SNUI       X         Y  

Now that we have our parameter file, we can create the job script. For this version, each line of the parameter file (not including the header line) will correspond to one task. It'll start mostly the same, but the number of tasks will be different. You'll also notice I start the tasks at 2, rather than 1. This means the header line won't be read as input.


```bash
#!/bin/bash

# Options

#$ -N job_name
#$ -t 2-10

chrom=${SGE_TASK_ID}
parameter_file=sample_parameter_file.txt     # or whatever you've called the file
```

Now for the fun part. Bear with me. It'll all be explained down below.


```bash
#reading lines from the parameter file
line=`sed -n -e "${SGE_TASK_ID} p" ${parameter_file}`
```

##### Breaking that down:
This line pulls the line corresponding to the number of the current task from the file. NB: this command is essentially 1-indexed.

  -  The `\`` (backticks) surrounding the expression tell `bash` to evaluate it, then set the result to line.

  -  `sed` is a stream-editor for the command line. It does a lot, but here it just prints the line it is told to do.

  -  `-n` suppresses the normal output from sed

  -  `-e` tells `sed` to evaluate the expression that follows

  -  `"${SGE_TASK_ID} p"` is the expression

  -  `${SGE_TASK_ID}` is the task number (and the line number we want)

  -  the `p` tells sed to print the line

  -  `${parameter_file}` is the file `sed` is operating on

So,...what can we do with that? Well, we've now read in a line of the parameter file, and now we want to get the names of the protein and its chains.


```bash
# This line saves the contents of the line as an array called parameters.
parameters=($line)

#These lines set the different elements of the array to variables.
name=${parameters[0]}
chain1=${parameters[1]}
chain2=${parameters[2]}
```

Now you can reference the different column values using the variable names, passing them as parameters to a program, or whatever. I don't really know what people do with these, so this will be another obviously made-up example.


```bash
protein_software --protein ${name} --chains ${chain1} ${chain2} \
    --out_file ${name}_protein_software.log
```

If you need to have a variable number of chains, for instance, this can be modified:


```bash
parameters=($line)

chains=${parameters[@]:1}
protein_software --protein ${name} --chains ${chains} \
    --out_file ${name}_protein_software.log
```

The line above takes the parameter list (here the chains are not assigned to variables), and it saves everything after the name of the protein to the `chains` variable, which then passes its value to the `--chains` flag.

It will be important to pay attention to which separators are appropriate for inputting multiple things in a flag. If you need to change the separator, you would have to do something like this, which changes it to a comma:


```bash
parameters=($line)
chains=$(echo ${parameters[@]:1} | tr ' ' ',')
protein_software --protein ${name} --chains ${chains} \
    --out_file ${name}_protein_software.log
```

All of the examples that have been used here have been made up of a small number of tasks; however, it is very likely that you will find yourself in a situation where you need to run an array job made up of hundreds, if not thousands, of tasks.

If you are the only user on the computer system where the array job will run, you may not need to worry about whether your array job will hog all of the available compute resources. If, however, you are on a shared system, it is best to pre-emptively prevent your job from flooding the system. If nothing else, it may keep you from getting scolding emails from the sysadmin, or other annoyed users, about how no one else can run anything.

# Curtailing your array jobs' concurrent tasks
#### * a.k.a. Being polite.

I'm now going to introduce the `-tc` option. I believe this stands for 'tasks, concurrent', but even if it doesn't officially stand for this, it summarises the meaning well.

`-tc` designates the number of tasks that can be running on the system **at the same time**.

It is, in essence, a rate-limiting parameter, that can save you from needing to monitor how many things you have running manually.


```bash
#!/bin/bash

# Options

#$ -N job_name
#$ -t 1-2000
#$ -tc 100         #this line will ensure that no more than 100 of my tasks will run at one time
```

This ability to control the number of tasks running at one time is another reason to submit jobs as an array, rather than using a `for` loop.

You might think the only differences between using a `for` loop and an array job to submit lots of nearly-identical jobs disappear once submission occurs, but...

# There are lots of good reasons to prefer array jobs to `for` loop submissions

With a for loop, there is no unifying characteristic that ties the jobs together; they each get their own job id, and are seen as independent of each other from the scheduler's perspective. This has several implications for you, as the user:

  - With an array job, all of the tasks have the same job id, but have unique task ids, allowing them to be referenced altogether, or individually, and allowing the scheduler to recognise that they are linked.
      - Individual tasks, or entire array jobs, can be placed on hold, deleted, examined, paused, or restarted, by referencing a single number.
      - You can use the `-tc` flag and the scheduler will monitor the number of running jobs for you, continuing to schedule new jobs as old ones finish.
      
  - With a `for` loop submission strategy, each job has its own id, and is independent of all other jobs.
      - To put on hold, delete, examine, pause, or restart several jobs, you must refer to each job's id individually.
      - The only way to limit how many jobs run concurrently is to check frequently and submit new jobs when an old one finishes.

That's not all there is to it, or even anywhere close. But it should help you recognise when an array job is appropriate, and figure out how to implement it!
