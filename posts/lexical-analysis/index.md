# Lexical Analysis


## The goal of lexical analysis
The goal of lexical analysis is to partition the input string into lexemes, identify the token of each lexeme then give to parser. On other hand, first part about lexemes is to partition input to units, and second part about token is to define the class of each units.  

1. Given a input string, such as `foo = 42`
2. Lexical-Analyzer would identify tokens as `<"id", "foo">`, `<Op, "=">`, and `<"Int", "42">`
3. Give to parser

There are several kinds of token classes, such as Identifier, Integer, Numbers, Keyword, Whitespace. Two interesting classes: One is identifier, which is string composed of letters or digits and start with letter, e.g. `A1`, `B17`. Another one is keyword, which is composed of something like `if`, `else`... etc. 

> It is also to see which value belongs to the token class. Therefore, the format of token consists of token name and attribute value.

There is another example for FORTRAN in the course: The difference between `DO 5 I = 1,25` and `DO 5 I = 1.25`. Whitespace in FORTRAN is insignificant, so we just need to ignore it. The first one represents `for-loop` while the second one represents simple assignment. In this case, `DO` in the first one would be keyword, but `DO` in the second one would be the substring of identifier. Now comes the problem, it makes sense that we scan the input from left-hand side to right-hand side; however, how do we identify whether it is keyword or identifier while we are still in left-hand side. We need the right-hand side, which is the difference between `1,25` and `1.25`; therefore, we need to **look-ahead**.  

1. The goal is to partition string. This can be implemented by scanning input from left to right, recognizing one token at a time.
2. **Look-ahead** may be required to decide where the next token starts.

## Lexical specification
To specify which set of strings belong to each token class, we could use **regular languages**. To define the regular languages, we use **regular expression**. Assume a meaning function `L`, which can map expression expression to set of strings by `L(E) -> set of strings`. It's said that meaning function can help map syntax to semantics.  

**Why is this mapping to semantics also important?** It can allow us to consider notation a separated issue. Same semantics can be mapped to by several syntaxs. Soemtimes our syntax is not the best solution for the problem, so we could find another solution based on same semantic.  

**How to check whether a string belongs to L(R) or not?**  
1. List all regular expressions of token classes
2. Union: `L(token_a) U L(token_b) U L(token_c) ...`
3. Given string `x`, check whether substring `sx` belongs to `L(R)` or not.
4. If true, remove `sx` from `x`. Put the rest of `x` to step 3 until `x` is totally empty.

**How if there are multiple token classes matched?** "Maximal mutch": choose the Longest Matched Prefix one. Given `sx`, it would belong to the token class with higher priority. For example, `if` could belong to identifier or keyword, and identifier has higher priority.

## Lexical implementation
We used regular expression to make lexical specification, but how do we implement it? We could use **Finite automata**. There is a brief introduction to automata on [geeksforgeeks](https://www.geeksforgeeks.org/introduction-of-finite-automata/). Therefore, we know that automata consists of an input alphabet, finite set of states S, a start state, a set of accepted states, and a set of transition.  

There are difference between Deterministic Finite Automata (DFA) and Nondeterministic Finite Automata (NFA). For DFA, one transition per input per state, and no epsilon moves are allowed. For NFA, multiple transition for per input given state, but epsilon moves are allowed. (epsilon move means transition to different states without assuming any input).  

Previously, we already know we can use regular expression to do lexical specification. In following steps, we would convert regular expressions to NFA, from NFA to DFA, then from DFA to Table-driven implementation of DFA at the end.

## From NFA to DFA
In fact, this section in course is very hard for me to understand. Personally I consider this step is for the purpose that NFA is too ambiguous to implement transition. Per input for NFA can transition to different states while per input for DFA can only transition to one state.

> I might still need some time to figure the meaning of this section out.

## Implementing Finite Automata
For the purpose of making iteration speedy, we would like to derive the table of DFA (row would be state, and col would be input symbol). The conversion between NFA to DFA is the trade-off between speed and space (conversion to DFA can reduce the col number).

## Reference
1. [Introduction of Finite Automata
](https://www.geeksforgeeks.org/introduction-of-finite-automata/)
