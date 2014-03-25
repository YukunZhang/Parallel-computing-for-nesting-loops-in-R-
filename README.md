Parallel-computing-for-nesting-loops-in-R-
==========================================
 R is widely used around the world by researchers. I am working on my master's thesis and using R to do simulation. It took 3 days to run 50 times simulations using my code. I was tired of waiting several days until getting my updated results which still need to be improved. Thus I am trying to embed parallel computing in the code. 
   Nowadays, we have more than one cores in our computers and we also have servers in our research institutes. Why don't we let more cores work on our task simultaneously?  Parallel computing is the form of computation that can divide our large problems to smaller ones which are then solved in parallel in different cores/processors. There are several useful R packages available to help us saving computation time: "multicore", "snow", "parallel","doSNOW", "foreach",etc..
Here are some useful links regarding to these packages:
http://homepage.stat.uiowa.edu/~luke/talks/uiowa03.pdf
http://www.sfu.ca/~sblay/R/snow.html
http://www.uio.no/english/services/it/research/hpc/courses/r/2013-04_parallel_r.pdf
http://www.imbi.uni-freiburg.de/parallel/docs/Reisensburg2009_TutParallelComputing_Knaus_Porzelius.pdf

   Here I want to discuss about the parallel computing for loops. There is few online information about this, I hope my example can help you a little bit. My method may not be the only way and I am open to comments.

   When we do simulation study, loops are inevitable although sometime we can use matrices instead of loops. It consumes more time when we have nesting loops. I use package "foreach" and "doSNOW" to achieve parallel computing for loops. The following is a simple example that I want to implement:

S=matrix(0,5,1)
A=matrix(0,5,1)
Q=matrix(0,3,3)
for (i in 1:3)
{
r=i+3
for (m in 1:5)    
  {
S[m,]=m^2
A[m,]=m+r
   }
l=i^2
s=apply(S,2,mean)
a=apply(A,2,mean)
Q[i,]=c(s,a,l)
}

After running this code, we can get Q as:
     [,1] [,2] [,3]
[1,]   11    7    1
[2,]   11    8    4
[3,]   11    9    9

Using parallel computing, the corresponding code is:

library(doSNOW)
library(foreach)
S=matrix(0,5,1)
A=matrix(0,5,1)
P=matrix(0,5,2)
K=matrix(0,3,2)
Q=matrix(0,3,3)

cl<-makeCluster(2)  #change the 2 to your number of CPU cores
registerDoSNOW(cl)

Q=foreach (i=1:3,.combine='rbind',.packages=c("doSNOW","foreach")) %dopar%{
r=i+3
P=foreach(m=1:5,.combine='rbind') %dopar% {
S[m,]=m^2
A[m,]=m+r
c(S[m,],A[m,])
}
l=i^2
K[i,]=apply(P,2,mean)
Q[i,]=c(K[i,],l)
}
stopCluster(cl)
 To do parallel computing, first we need to initialize the slave processes by "cl<-makeCluster(2)", the slave number depends on your system configuration. Then I use "registerDoSNOW" function to register the SNOW parallel backend with the “foreach” package. At the end of the code, we need to shut down the cluster and clean up any remaining connections between machines by using "stopCluster(cl)".


After these parallel computing basic settings, let us see how to use foreach instead of for loop.
You may notice that we need to claim the packages that are used inside the loop. For example here we need to use option called .packages to tell the foreach package that the R expression needs to have the "foreach" and "doSNOW" package loaded in order to execute successfully.

Begin from the inner loop, I use "foreach(m=1:5,.combine='rbind') %dopar%" to substitute "for(m in 1:5)",  %dopar% means evaluating the expression parallel. An important difference between “foreach” and “for” is that foreach loop cannot save the variables automatically. That is, when we call S outside the loop, we can only get a zero matrix. So I define P as c(S[m,],A[m,]) for the inner loop. In this way I can save the useful variables to P. The same thing for outer loop. The command that save variables must stay at the last line of the loop. "foreach" loop tends to assign the last command to P and ignore all the variables above it. “foreach” loop will also ignore the output command such as "cat()" and "print ()".

   At the end, I can get Q:
          [,1]  [,2]  [,3]
result.1   11    7    1
result.2   11    8    4
result.3   11    9    9
It is coincident with "for" loop.


Note: "foreach" only works for independent loops, if the iteration is depend on the last step, using parallel computing may cause problems.

Reference:
1. http://cran.r-project.org/web/packages/foreach/foreach.pdf
2. http://cran.r-project.org/web/packages/foreach/vignettes/foreach.pdf
3. http://cran.r-project.org/web/packages/foreach/vignettes/nested.pdf
