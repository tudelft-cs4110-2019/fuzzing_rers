# Fuzzing problems from the RERS challenge

We are going to fuzz programs from the RERS 2019 challenge, let me know if you're interested in joining my team to participate in this challenge!
The first step is to download and install AFL:

AFL - http://lcamtuf.coredump.cx/afl/

Simply run make and you are good to go.

Then download and unpack the RERS Reachability challenge programs:

http://rers-challenge.org/2019/problems/sequential/SeqReachabilityRers2019.zip

The archives contain highly obfuscated c and java code, e.g., :

In SeqReachabilityRers2019/Problem11/Problem11.c

```C++
...
	void errorCheck() {
	    if(((a1840022387 == 10 && a930233571 == 12) && a2135110469 == 10)){
	    cf = 0;
	    __VERIFIER_error(0);
	    }
	    if(((a1227434655 == 14 && a1074202851 == 11) && a2135110469 == 4)){
	    cf = 0;
	    __VERIFIER_error(1);
	    }
	    if(((a130053513 == 11 && a1644705049 == 10) && a2135110469 == 3)){
...
void calculate_outputm50(int input) {
    if(((a1644705049 == 10 && (a2135110469 == 3 && ((input == 6) &&  cf==1 ))) && a130053513 == 12)) {
    	cf = 0;
    	a344522688 = 13 ;
    	a1718050978 = 10 ;
    	a2135110469 = 8; 
    	 printf("%d\n", 23); fflush(stdout); 
    } if((a2135110469 == 3 && (((input == 8) && (a130053513 == 12 &&  cf==1 )) && a1644705049 == 10))) {
    	cf = 0;
    	a1228149849 = 3;
    	a2135110469 = 7;
    	a2128198108 = 12; 
    	 printf("%d\n", 17); fflush(stdout); 
    } 
}
 void calculate_outputm1(int input) {
    if(( cf==1  && a130053513 == 12)) {
    	calculate_outputm50(input);
    } 
}
...
 void calculate_output(int input) {
        cf = 1;

    if((a2135110469 == 3 &&  cf==1 )) {
    	if((a1644705049 == 10 &&  cf==1 )) {
    		calculate_outputm1(input);
    	} 
    	if((a1644705049 == 11 &&  cf==1 )) {
    		calculate_outputm2(input);
    	} 
    } 
    if((a2135110469 == 4 &&  cf==1 )) {
    	if(( cf==1  && a1074202851 == 13)) {
    		calculate_outputm9(input);
    	} 
    } 
    if(( cf==1  && a2135110469 == 5)) {
    	if(( cf==1  && a2128198108 == 8)) {
    		calculate_outputm11(input);
    	} 
    	if(( cf==1  && a2128198108 == 9)) {
    		calculate_outputm12(input);
    	} 
    	if(( cf==1  && a2128198108 == 10)) {
    		calculate_outputm13(input);
    	} 
    	if(( cf==1  && a2128198108 == 14)) {
    		calculate_outputm17(input);
    	} 
    } 
    if(( cf==1  && a2135110469 == 6)) {
    	if(( cf==1  && a261374011 == 11)) {
    		calculate_outputm20(input);
    	} 
    	if((a261374011 == 12 &&  cf==1 )) {
    		calculate_outputm21(input);
    	} 
    	if(( cf==1  && a261374011 == 14)) {
    		calculate_outputm22(input);
    	} 
    } 
    if((a2135110469 == 7 &&  cf==1 )) {
    	if(( cf==1  && a1228149849 == 3)) {
    		calculate_outputm24(input);
    	} 
    	if((a1228149849 == 7 &&  cf==1 )) {
    		calculate_outputm28(input);
    	} 
    } 
    if(( cf==1  && a2135110469 == 8)) {
    	if((a344522688 == 13 &&  cf==1 )) {
    		calculate_outputm34(input);
    	} 
    } 
    if(( cf==1  && a2135110469 == 9)) {
    	if((a1115144307 == 10 &&  cf==1 )) {
    		calculate_outputm36(input);
    	} 
    	if((a1115144307 == 13 &&  cf==1 )) {
    		calculate_outputm39(input);
    	} 
    } 
    if((a2135110469 == 10 &&  cf==1 )) {
    	if(( cf==1  && a930233571 == 5)) {
    		calculate_outputm41(input);
    	} 
    	if(( cf==1  && a930233571 == 9)) {
    		calculate_outputm45(input);
    	} 
    	if((a930233571 == 10 &&  cf==1 )) {
    		calculate_outputm46(input);
    	} 
    } 
    errorCheck();

    if( cf==1 ) 
    	fprintf(stderr, "Invalid input: %d\n", input); 
}

int main()
{
    // main i/o-loop
    while(1)
    {
        // read input
        int input;
        scanf("%d", &input);        
        // operate eca engine
        if((input != 0) && (input != 1) && (input != 2) && (input != 3) && (input != 4) && (input != 5) && (input != 6) && (input != 7) && (input != 8) && (input != 9))
          return -2;
        calculate_output(input);
    }
}
```

Good luck understanding the logic underlying this code... All we know is that it is a simple state machine, which has been obfuscated to make analysis harder. In case you are interested in obfuscation: check out Tigress: http://tigress.cs.arizona.edu. We of course use automated tools in order to try to understand it!

The goal of the RERS challenge is to discover exactly which VERIFIER_error(s) are reachable, some of the if statements in errorCheck() can never occur, while other can occur but only after reasing a very long input. The input to the program is simply a long list of integers, seperated by spaces or line ends. After reading an input, the program returns an integer, or gives an error.

The c code will not compile as given, we need to make a few changes, and some more to run AFL on it.

First, replace:

```C++
    extern void __VERIFIER_error(int);
(line 6)
```

with:
   
```C++
void __VERIFIER_error(int i) {
    fprintf(stderr, "error_%d ", i);
    assert(0);
}
```

The assert causes a crash, which is what we want to find using AFL.

Then, replace:

```C++
int main()
{
    // main i/o-loop
    while(1)
    {
        // read input
        int input;
        scanf("%d", &input);        
        // operate eca engine
        if((input != 0) && (input != 1) && (input != 2) && (input != 3) && (input != 4) && (input != 5) && (input != 6) && (input != 7) && (input != 8) && (input != 9))
          return -2;
        calculate_output(input);
    }
}
```

with:

```C++
int main()
{
	// main i/o-loop
	while (1) {
		// read input
		int input = 0;
		int ret = scanf("%d", &input);
		if (ret != 1) return 0;
        // operate eca engine
        if((input != 0) && (input != 1) && (input != 2) && (input != 3) && (input != 4) && (input != 5) && (input != 6) && (input != 7) && (input != 8) && (input != 9))
          return -2;
        calculate_output(input);
    }
}
```

This avoids a hang in the scanf function created when the input ends.

Now all we need to run afl is to create two directories: tests and findings. In tests you have to provide AFL with some example inputs, makes sure that you give an integer with a space or new line. For instance, I have the file 1.txt in the test directory, containing:

```C++
"1
"
```

so a 1 with a new line.

Make sure that you create files with all possible input symbols (otherwise AFL has to discover these which might take a long time).

Now we are ready to fuzz.

First compile the file you edited (for example Problem10.c) using the afl-gcc compiler or in my case afl-clang:

path_to_afl/afl-clang Problem11.c -o Problem11

Then run afl on the obtained binary:

path_to_afl/afl-fuzz -i path_to_test_dir -o path_to_findings_dir path_to_binary/Problem11

You should see AFL starting, perhaps with some warnings due to useless input files, you can simply ignore these.

Keep on fuzzing.

Then after some time, you can investigate the findings directory. You will find for instance the crashes found in the crashes directory, and interesting test cases (which it uses to trigger new paths) in queue. You can run the crashes found in the fuzzed program in order to answer the reachability problems:

cd findings/crashes
cat id:000000,sig:06,src:000001,op:havoc,rep:4 | ../../Problem11

Gives in my case:

```
16
18
error_51 Assertion failed: (0), function __VERIFIER_error, file Problem11.c, line 8.
Abort trap: 6
```

Implying that error 6 is reachable.

In this way, and some extra tricks, we obtained third place in the reachability category of the RERS 2016 challenge: http://rers-challenge.org/2016/index.php?page=results. We are team Radboud (together with PhD students from Radboud university), for some reason they still did not update the names. We wrote a paper describing our approach (including learning which you will learn later in this course), which is available at https://arxiv.org/pdf/1611.02429.pdf. Please take a look if you are interested.
AFL can reach quite some errors, but to compete in the 2019 challenge we expect combinations will be required with learning, mutation, tainting, and concolic execution.

