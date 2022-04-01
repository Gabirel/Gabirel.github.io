---
layout: post
lang: en
title: 0/1 Knapsack Problem Solver Code
categories: algorithm
tags: knapsack algorithm
date: 2019-10-05
---

<p class="intro"><span class="dropcap">T</span>his post only shows the correct source code which solving 0/1 knapsack prolem.</p>

I hope the source code could be helpful to you.

-----
Table of Contents

* TOC
{:toc}

-----

# Example

In this post, one example is listed:

| Maximum capacity | The number of items |
| :--------------: | :-----------------: |
|        10        |          3          |
|    **Weight**    |      **Value**      |
|        3         |          4          |
|        4         |          5          |
|        5         |          6          |

# Source Code

The source code is as follows:

```c++
#include <iostream>

using namespace std;

int main() {
  const int N = 3;              // the number of items
  const int Capacity = 10;      // the volumn of knapsack
  int weight[N + 1] = {-1, 3, 4, 5}; // the volumn of i-th item (index starting from 1)
  int value[N + 1] = {-1, 4, 5, 6};  // the value of i-th item
  int F[N + 1][Capacity + 1] = {0};  // status

  for (int i = 1; i <= N; i++) { // as for i-th item
    for (int j = 0; j <= Capacity; j++) {
      F[i][j] = F[i - 1][j]; // do not choose i-th item (default)
      if (j - weight[i] >= 0 &&
          F[i][j] < F[i - 1][j - weight[i]] + value[i]) { // if value is higher, then choose i-th item
        F[i][j] = F[i - 1][j - weight[i]] + value[i];
      }
    }
  }

  cout << "Max value: " << F[N][Capacity] << endl; // in this case: 11

  cout << "Status: " << endl;
  for (int i = 0; i <= N; ++i) {
    for (int j = 0; j <= Capacity; j++) {
        printf("%2d ", F[i][j]);
    }
    cout << endl;
  }

  return 0;
}
```



# Result

```shell
Max value: 11
Status:
 0  0  0  0  0  0  0  0  0  0  0
 0  0  0  4  4  4  4  4  4  4  4
 0  0  0  4  5  5  5  9  9  9  9
 0  0  0  4  5  6  6  9 10 11 11
```

<center><blockquote>The End</blockquote></center>