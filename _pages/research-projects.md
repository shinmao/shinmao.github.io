---
layout: archive
title: "Research Projects"
permalink: /projects/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

TYPEPULSE
------
To be relased.

Vector35/AlgoProphet
------
This is a open-source project which can be found on [github](https://github.com/Vector35/AlgoProphet), and only has one release version currently. The purpose of AlgoProphet is to identify and localize the **arithmetic** algorithm which is built on non-visited implementation from the binary. Given a function in binary, AlgoProphet can build up a DFG(Dataflow Graph) based on [MLIL-SSA of Binary Ninja](https://docs.binary.ninja/dev/bnil-mlil.html), then use graph matching algorithm(Isomorphism) to match the models saved in the database. We spent a lot of time on the design of **customized**-DFG and tried to answers the questions, e.g., How to represent a computation algorithm in the loop? In current phase, AlgoProphet can help us identify some algorithms like vector summation or discrete fourier transform both on `x86-64` and `arm64`. Of course, working on different architectures should be easy based on layout-agnostic IR; however, we still found that a same instruction can have many different implementations. You can find more details in README of the repo. Also appreciate any comments in issues!

egalito-reversePatch
------
This is a open-source project which can be found on [github](https://github.com/shinmao/egalito-reversePatch), and the original project of egalito can be found [here](https://github.com/columbia/egalito). egalito-reversePatch leaverages the interesting design of egalito IR and makes some **binary code similarity** works with clustering methods. However, the most amazing power of egalito IR is that it makes static binary instrumentation and relocation easier. Thanks for the support from authors and @Anthony in Columbia, I have realized that IR can be designed for different purpose, which has a big impact on my research in the future. The main code of my work can be found in [pass/reversepatch.cpp](https://github.com/shinmao/egalito-reversePatch/blob/main/src/pass/reversepatch.cpp), and it would extract the tokens as training features while visiting each instruction (It only worked on arch `x86_64` at this time).