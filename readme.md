# PiContFuncs carlos

Generation of continuous fractions identities.

## Installation

1) git lfs (large files storage) is used. Download and install it from [here](https://git-lfs.github.com/), and from the repository directory run:
```
git lfs install
```
if .gitattributes wasn't pulled from the repository for some reason, run:
```
git lfs track "hashtable_*.pkl"
```

2) Install everything on the requirements file:
```
pip install -r requirements.txt
```


## Running

To run the project, type:

```
python main.py
```

The algorithm loads the file config.ini, in which it expects additional parameters.
See configfile.py documentation for more details.
There are a few examples & sanitycheck configurations in tests\fixtures, e.g.:
```
python main.py apery_sanitycheck_config.ini
```

## General

This code is purposed for research, therefore it's still dynamic and experimental, and changes frequently. This file aims to give a brief introduction to the code and its structure. Due to the mentioned circumstances, it may be inaccurate.

### What does the code do?

We have continued fractions which are generated by polynomials. What does that mean? object of the type:
a(0)+b(1)/(a(1)+b(2)/...))
where a,b are polynomials with integer coefficients. This is the RHS.
 On the LHS we have a function of some constant, either rational or ULCD. A ULCD function is of the type
f(const) = (u/const + const/l + c)/d
while a rational function is the ratio of two polynomials.
 We seek for a match between some LHS function on a constant to a continued fraction.
To do this, we enumerate over the coefficients of a,b and those of the rational func on the LHS/the ulcd parameters.
They're all integers.
 We actually don't compare between the LHS to a continued fraction, but to a continued fraction after we applied some
function to it, e.g. contfrac^2, sqrt(contfrac), 1/contfrac etc.
 In order to enhance the algorithm's complexity, we first enumerate over the parameters of the RHS, saving the results
to a hashtable (a python 'dict'), and then we enumerate over the LHS and look for a match in the table. This is a TMTO
(time-memory tradeoff), it's called MITM (meet in the middle), you can look it up if it's not clear.
 We first find matches, then filter redundant results (discussed later on), filter only continued fractions the converge
fastly, find how fast are they converging, and then we know how many iterations (to what level of "nested fractions") we
should calculate to get the desired precision. We calculate them to that precision, and filter again.

 We wish to work with an arbitrary precision on the one hand, but want to compare only a few digits in the hashtable
 on the other hand. This was solved by implementing a customized hashtable over python's dict.
  We also wish to get rid of redundant results (results which are equivalent). There's a method for this too,
 which is far from perfect but reduces redundant results dramatically.
  At the end, we wish to save all the results, timestamped and reproducible.
 This is done by saving to a folder with a timestamped name, in which we save the configuration file that generated
 these results, csv (excel) file with the results and a PDF with the results in a more readable format.

### Code structure

The code is made up of these files (some of the names may not be very indicative, as these are results of an ongoing
process...):
* main - this file is combining everything together, you should begin with it to understand more or less what's going on.
* configfile - loads and parses the configuration file
* latex - generates a latex file for the results
* gen_consts - this module holds all the constants we're dealing with (pi, e etc).
* enum_params - does all the enumeration. Creates the hashtable, then looking for LHS matches (AKA clicks)
* enum_poly_params - generates the parameters for the polynomials, either for the RHS's a,b or for the LHS rationl
  function polynomials. Several types of polynomials enumeration that were found to be useful in different cases are
  supported,  such as "regular": a_0+a_1*x+...+a_n*x^n, "indexed": a_0*x^n+a_1*(x+1)^n+...+a_n*(x+n)^n etc.
* cont_fracs - generates continued fractions.
    * basic_representation_types - a more basic class which the ContFrac inherits from, for the case we'll have other 
      similar objects for the RHS in the future. a "virtual class".
* decimal_hashtable - implements the customized hashtable for our purposes
* postprocfuncs - contains all the function that we apply on the continued fraction before saving the results to the
  hashtable.
* lhs_evaluators - the LHS function is a rational function? ulcd? Maybe we want something else? This file
  implements these functions. As long as the interface is standard, any function can be implemented here and be
  used (almost) natively by the rest of the code. "Almost" because the class type should be defined by name, and
  added to the configfile to be supported and parsed correctly.
the hashtable
* utils - a few utilities that aren't specific to any of the above files, such as a time measuring class (similar to
  matlab's tic toc), gcd implemenation (for types that don't have built-in gcd implemenation), substitution in
  polynomials etc.
* tests directory - contains tests to many of the methods. They are written for pytest and can be ran (for example) by:
  py.test -vv tests

### Other

* You can ignore for now from: run_distributed_configs, join_hashtables, distribute_params. These are patchy (yet
working) approaches to distribution and parallelization.
* process_and_compare - a legacy file. If one needs to convert csv to pdfs, load old results from a csv to ContFrac
  objects etc, updating this file may be more effortless.
