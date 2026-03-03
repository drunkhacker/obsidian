https://www.acmicpc.net/problem/9663

N-Queen , 백트래킹.

어쨌든 한줄에 한개를 놔야하는거니까 
NxN -> N-1xN -> N-2xN 순으로 문제 사이즈는 줄어드는듯.

NxN에서 
```
if (n = 0) {
  if (check) {
    return 1;
  }
  return 0;
}
s = 0
for col in 1..n 
  q[n] = col
  s += backtrack(n-1)
return s;

fun check:
  for r1 in 1..n
    for r2 in r1..n
      if q[r1] == q[r2] or |r2 - r1| == q[r1] - q[r2]
        return 0;
  return 1;
    
```


x q x x  
x x x q
q x x x
x x q x

나이브하게 짜니까 timeout난다. pruning을 해야할수도

```
#include <stdio.h>

int q[16] = {0,};
int N;

int backtrack(int n) {
    int r1, r2, c, s, ok;
    if (n < 0) {
        for (r1 = 0; r1 < N-1; r1++) {
            for (r2 = r1+1; r2 < N; r2++) {
                if (q[r1] == q[r2] || (r2 - r1 == q[r1] - q[r2]) || (r1 - r2 == q[r1] - q[r2])) {
                    return 0; // fail! queen attack each other
                }
            }
        }
        return 1;
    }

    s = 0;
    for (c = 0; c<N; c++) {
        ok = 1;
        // pruning..
        for (r1 = N-1; r1 > n; r1--) {
            if (q[r1] == c || (r1 - n == q[r1] - c) || (n - r1 == q[r1] - c)) {
                ok = 0;
                break;
            }
        }
        if (!ok) {
            continue;
        }
        q[n] = c;
        s += backtrack(n-1);
    }

    return s;
}

int main() {
    scanf("%d", &N);
    printf("%d\n", backtrack(N-1));
    return 0;
}

```


근데 굳이 n<0 일때 이미 다 가능성 체크하고 온 마당에 또 할 필요가 없다.
```
#include <stdio.h>

int q[16] = {0,};
int N;

int backtrack(int n) {
    int r1, r2, c, s, ok;
    if (n < 0) {
        return 1;
    }

    s = 0;
    for (c = 0; c<N; c++) {
        ok = 1;
        // pruning..
        for (r1 = N-1; r1 > n; r1--) {
            if (q[r1] == c || (r1 - n == q[r1] - c) || (n - r1 == q[r1] - c)) {
                ok = 0;
                break;
            }
        }
        if (!ok) {
            continue;
        }
        q[n] = c;
        s += backtrack(n-1);
    }

    return s;
}

int main() {
    scanf("%d", &N);
    printf("%d\n", backtrack(N-1));
    return 0;
}

```