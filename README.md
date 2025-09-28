# Divide-and-Conquer Algorithms — Report

Кратко
-----
Реализованы MergeSort, QuickSort (robust), Deterministic Select (MoM, group=5), Closest Pair (2D). Собраны метрики: время, макс. глубина рекурсии, сравнения, аллокации. Генерируется CSV для построения графиков.

Архитектура
----------
- `metrics.Metrics` — счётчики и трекинг глубины рекурсии (ThreadLocal + AtomicLong для maxDepth).
- `sort` — MergeSort и QuickSort. MergeSort использует reusable buffer и cutoff на 32 элементов. QuickSort: случайный pivot + recurse-on-smaller + insertion cutoff.
- `select` — deterministic median-of-medians (5-элементные группы), in-place partition.
- `closest` — классический D&C по x-координате, strip scan по y с константой ~7.
- `Main` — CLI для генерации измерений и сохранения CSV.

Рекуррентные соотношения и выводы
---------------------------------
- MergeSort: T(n)=2T(n/2)+Θ(n) => Θ(n log n) (Master case 2).
- QuickSort (рандомизированный): ожидаемое Θ(n log n), худшее Θ(n^2); recurse-on-smaller и рандомизация дают ожидаемую глубину O(log n).
- Deterministic Select (MoM): T(n)=T(n/5)+T(7n/10)+Θ(n) => Θ(n) (Akra–Bazzi / CLRS result).
- Closest Pair (2D D&C): T(n)=2T(n/2)+Θ(n) => Θ(n log n).

Как запускать
-------------
1. Сборка и тесты:
```
mvn clean test
```
2. Собрать jar:
```
mvn -DskipTests package
```
3. Пример запуска CLI (генерирует/дописывает в out.csv):
```
java -jar target/divide-and-conquer-algorithms-1.0-SNAPSHOT.jar mergesort 100000 5 out.csv
```

Построение графиков
-------------------
Пример Python-скрипта `plot.py` в корне:
```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('out.csv', names=['algo','n','time_ms','depth','comparisons','allocations'])
for algo in df['algo'].unique():
    sub = df[df['algo']==algo]
    mean = sub.groupby('n').mean().reset_index()
    plt.plot(mean['n'], mean['time_ms'], label=algo)
plt.xlabel('n')
plt.ylabel('time (ms)')
plt.legend()
plt.xscale('log')
plt.yscale('log')
plt.savefig('time_vs_n.png')
```

Git workflow
------------
Пример коммитов и веток описан ниже:
```
git init
git add .
git commit -m "init: maven, junit5, ci, readme"
git checkout -b feature/metrics
... (далее по README)
```
