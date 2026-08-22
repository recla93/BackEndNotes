---
topic: "NumPy — Calcolo Numerico Python"
tags: [python, numpy, array, scientific, numerical]
---
Riferimento ufficiale: [numpy.org/doc/stable](https://numpy.org/doc/stable/)

NumPy è la libreria fondamentale per il calcolo numerico in Python. Il suo cuore è `ndarray`: un array N-dimensionale omogeneo (tutti gli elementi dello stesso tipo) che permette operazioni vettoriali veloci grazie a implementazione in C.

A differenza delle liste Python, NumPy array occupano meno memoria e sono decine di volte più veloci per operazioni matematiche. NumPy è il fondamento di Pandas, SciPy, Scikit-learn, TensorFlow e PyTorch.

Vedi anche:
[[BE-NOTES/Python/Data/Pandas|Pandas]],
[[BE-NOTES/Python/Core Concepts/Liste Tuple Set Dict|Liste Tuple Set Dict]],
[[BE-NOTES/Python/Funzionale/itertools|itertools]].

## Creazione di array

```python
import numpy as np

# Da lista
arr = np.array([1, 2, 3, 4, 5])

# Array di zeri/uno
zeros = np.zeros((3, 4))      # 3 righe, 4 colonne
ones = np.ones((2, 2))

# Range
range_arr = np.arange(0, 10, 2)    # [0, 2, 4, 6, 8]
linspace = np.linspace(0, 1, 5)    # [0.0, 0.25, 0.5, 0.75, 1.0]

# Array casuale
random_arr = np.random.rand(3, 3)    # Uniforme [0, 1)
randn = np.random.randn(1000)        # Normale standard
```

`np.array()` è il costruttore base. `np.zeros()` e `np.ones()` sono utili per inizializzazione. `np.arange()` è simile a `range()` ma restituisce array. `np.linspace()` è ideale per intervalli equispaziati.

## Shape, dtype e reshaping

```python
arr = np.array([[1, 2, 3],
                [4, 5, 6]])

arr.shape       # (2, 3)  (righe, colonne)
arr.dtype       # dtype('int64')
arr.ndim        # 2 (numero dimensioni)
arr.size        # 6 (totale elementi)

# Reshape
arr_1d = arr.reshape(6)       # [1, 2, 3, 4, 5, 6]
arr_2d = arr_1d.reshape(2, 3) # Torna a (2, 3)
arr_T = arr.T                  # Trasposta: (3, 2)

# Appiattimento
arr_flat = arr.flatten()       # Copia
arr_rav = arr.ravel()          # Vista (se possibile)
```

`reshape()` restituisce una vista se possibile (condivide memoria). `flatten()` restituisce sempre una copia. `.T` è la trasposta per array 2D. `dtype` può essere specificato: `np.array([1, 2], dtype=np.float32)`.

## Operazioni vettoriali

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# Operazioni elemento per elemento
a + b           # [5, 7, 9]
a * b           # [4, 10, 18]
a ** 2          # [1, 4, 9]
np.sqrt(a)      # [1.0, 1.41, 1.73]
np.sin(a)       # [0.84, 0.91, 0.14]

# Operazioni di aggregazione
a.sum()         # 6
a.mean()        # 2.0
a.max()         # 3
a.std()         # 0.82 (deviazione standard)

# Broadcasting
a + 10          # [11, 12, 13] (10 viene "allargato")
```

Il broadcasting permette operazioni tra array di shape diversa. Le operazioni vettoriali sono 10-100x più veloci di loop Python equivalenti perché eseguite in C.

## Indicizzazione e slicing

```python
arr = np.array([[1, 2, 3, 4],
                [5, 6, 7, 8],
                [9, 10, 11, 12]])

arr[0, 1]           # 2 (riga 0, colonna 1)
arr[1]              # [5, 6, 7, 8] (riga 1)
arr[:, 2]           # [3, 7, 11] (colonna 2)
arr[0:2, 1:3]       # [[2, 3], [6, 7]] (sotto-matrice)

# Indicizzazione booleana
arr[arr > 5]        # [6, 7, 8, 9, 10, 11, 12]

# Indicizzazione con array di indici (fancy indexing)
indici = [0, 2]
arr[:, indici]      # [[1, 3], [5, 7], [9, 11]]
```

Lo slicing in NumPy segue la stessa sintassi delle liste Python ma su N dimensioni. L'indicizzazione booleana è potente per filtraggio. Il fancy indexing usa array di indici per selezionare elementi arbitrari.

## Algebra lineare

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# Moltiplicazione matrice
C = A @ B               # [[19, 22], [43, 50]] (Python 3.5+)
C = np.matmul(A, B)     # Equivalente

# Prodotto scalare
v = np.array([1, 2])
w = np.array([3, 4])
dot = np.dot(v, w)      # 11

# Trasposta, inversa, determinante
A_T = A.T
A_inv = np.linalg.inv(A)
det = np.linalg.det(A)

# Autovalori e autovettori
eigvals, eigvecs = np.linalg.eig(A)
```

`@` è l'operatore di moltiplicazione matriciale (Python 3.5+). `np.linalg` contiene tutte le funzioni di algebra lineare standard.

## Errori comuni

- **Liste Python vs array**: `a + b` per liste concatena, per array somma elemento per elemento.
- **Shape mismatch**: operazioni tra array di shape incompatibile. Broadcasting ha regole precise.
- **Copie vs viste**: `arr.reshape()` può condividere memoria. Modificare la vista modifica l'originale. Usa `.copy()` per sicurezza.
- **Dimenticare la tipizzazione**: `np.array([1, 2, 3])` è int64, non Python int. Mischiare tipi porta a conversioni implicite.
- **Loop su array**: `for x in arr:` è lentissimo. Usa sempre operazioni vettoriali.

## Best Practices & Conventions

- Usa `import numpy as np` come convenzione universale.
- Preferisci operazioni vettoriali ai loop Python. 10x-100x più veloci.
- Specifica `dtype` per controllo di memoria e precisione (`np.float32`, `np.int64`).
- Usa `np.newaxis` o `reshape(-1, 1)` per aggiungere dimensioni quando serve broadcasting.
- Per random, usa `np.random.default_rng()` (nuova API) invece di `np.random.rand()` (legacy).
- Libera memoria con `del arr` per array grandi che non servono più.
