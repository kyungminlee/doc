### Julia

```julia
A = [(i,j) for i in ['A','B'] for j in 1:3]
B = [(i,j) for i in ['A','B'], j in 1:3]

C = []
for i in ['A','B']
  for j in 1:3
    push!(C, (i,j))
  end
end

D = []
for i in ['A','B'], j in 1:3
  push!(D, (i,j))
end
```

```julia
A = Tuple{Char,Int64}[('A', 1), ('A', 2), ('A', 3), ('B', 1), ('B', 2), ('B', 3)]
B = Tuple{Char,Int64}[('A', 1) ('A', 2) ('A', 3); ('B', 1) ('B', 2) ('B', 3)]
C = Any[('A', 1), ('A', 2), ('A', 3), ('B', 1), ('B', 2), ('B', 3)]
D = Any[('A', 1), ('A', 2), ('A', 3), ('B', 1), ('B', 2), ('B', 3)]
reshape(A, :) = Tuple{Char,Int64}[('A', 1), ('A', 2), ('A', 3), ('B', 1), ('B', 2), ('B', 3)]
reshape(B, :) = Tuple{Char,Int64}[('A', 1), ('B', 1), ('A', 2), ('B', 2), ('A', 3), ('B', 3)]
reshape(C, :) = Any[('A', 1), ('A', 2), ('A', 3), ('B', 1), ('B', 2), ('B', 3)]
reshape(D, :) = Any[('A', 1), ('A', 2), ('A', 3), ('B', 1), ('B', 2), ('B', 3)]
```

### Python

```python
import itertools

A = [(i,j) for i in ['A','B'] for j in [1,2,3]]

C = []
for i in ['A','B']:
    for j in [1,2,3]:
        C.append((i,j))

D = []
for i, j in itertools.product(['A', 'B'], [1,2,3]):
    D.append((i,j))
```


```python
A = [('A', 1), ('A', 2), ('A', 3), ('B', 1), ('B', 2), ('B', 3)]
C = [('A', 1), ('A', 2), ('A', 3), ('B', 1), ('B', 2), ('B', 3)]
D = [('A', 1), ('A', 2), ('A', 3), ('B', 1), ('B', 2), ('B', 3)]
```
