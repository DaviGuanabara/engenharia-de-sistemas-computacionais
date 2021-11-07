```Python

def Maior(v):
    maior = v[0]

    for i in range(1, len(v)):
        if maior < v[i]:
            maior = v[i]    

    return maior
    
def T(n, k):

    if k >= n or k < 0:
       return 0

    if n == 1:
       return 1

    return T(n-1, k-1) + (n-1) * T(n-1, k)
    
    
import time
import math

nMax = 15


print("n, execTime(miliseconds)")           

for n in range(1, nMax + 1):

    start_time = time.time()

    for k in range(0, n-1):
        T(n,k)

    end_time=time.time()

    print(n, ",", "{:.3f}".format((end_time - start_time) * math.pow(10, 3)) )    

```
