# Learn Compiler - Introduction


Once in college, I regreted not spending much time on learning compiler so hard. However, the opportunity comes to me again. Software analysis is very important for my research field in PhD: Automatic Exploitation Generation. Compiler is the necessary concept for Software analysis such for static analysis or dynamic analysis, I would grab this chance to learn compiler from basics again. This series include the learning notes for the Stanford course on edx, and would start from Introduction to each parts of Lexical Analysis, Parsing, Semantic Analysis, Optimization, and Code Generation.

## Brief introduction to components of compiler
First, what is the difference between Compiler and Interpreter?  
* Compiler would convert program into an executable. If we give data to executable as input, then we can get output we want.
* Interpreter would convert program and data into a output.  

Therefore, we can consider compiler off-line and interpreter online.  
We can separate compiler to five components:  
1. Lexical Analysis
2. Parsing
3. Semantic Analysis
4. Optimization
5. Code Generation  

1 and 2 can be concluded as the part of syntactic. So what is the difference between syntactic and semantic? Syntactic is similar to grammar for structure, or we can call it syntax. Semantic is the meaning behind the sentence, or even the conclusion. I would briefly introduce each parts in following sections.

## Lexical Analysis and parsing
Lexical analysis would divide text into "words" or "token" to recognize each words. Parsing would put the statement into the parsing tree with structure. For example,
```
if      x == y     then    z = 1;    else     z = 2;

   x   ==    y         z   =   1            z   =   2

    \   |    /          \      /            \       /
     \  |   /            \    /              \     /

    relation            assignment          assignment

        \                  |                    /
         \                 |                   /
          \                |                  /

                  if - then - else
```

## Semantic Analysis
Understanding the "meaning" of sentence is hard. Assume a sentence, Rafael left his assignment in home. With this sentence, how do we identify "his" as "Rafael"? Programming language defines strict rules to avoid ambiguity. For example, variable binding (assign variable to specific scope), type mismatching.

## Optimization
Replace long words with less words to reduce time complexity or space complexity. For an interesting example mentioned in the class, whether we can convert `X = Y * 0` to `X = 0`? The answer is no, we should specify the type of variable to be integers. For example, `NAN * 0` is still equal to `NAN`.

## Code Generation
Translation to another language, it is usually assembly language, but it can also be another type of high level programming language.

## Why we need a new programming language?
It is another interesting problem that why we always need new programming language? Isn't single language enough for our use? The answer is different programming language for different use. There is no universally accepted metric for language design. For example, if you need to do the work on scientific computing which needs float point or arrays operations, FORTRAN would be your choice. However, why can't we just change our old language for new use? The reason is that old language is difficult to change. It would cost less to just design a new language to move the entire community of programmers and existing system to acommodate new applications.

## Reference
* [StanfordOnline edX Compilers](https://learning.edx.org/course/course-v1:StanfordOnline+SOE.YCSCS1+3T2020/home)
