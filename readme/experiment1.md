[« Back](../README.md)

# Experiment 1: Geocoding Technique Comparison

## 1. Geohash

1. Clone repository from [https://github.com/ponlawat-w/uji_mt-geohash_evaluation_test](https://github.com/ponlawat-w/uji_mt-geohash_evaluation_test).

```
git clone https://github.com/ponlawat-w/uji_mt-geohash_evaluation_test.git
```

2. Browse to directory `geohash`

```
cd ./geohash
```

3. Execute `geohash.js`

```
node ./geohash.js
```

4. The raw results will be located at `./out/stats.csv`. This file is to be used in Excel or other softwares to make a report.

---

## 2. S2

1. Clone repository from [https://github.com/ponlawat-w/uji_mt-s2encoding](https://github.com/ponlawat-w/uji_mt-s2encoding)

```
git clone https://github.com/ponlawat-w/uji_mt-s2encoding.git
```

2. Run GoLang script to cover polygons into S2 cells

```
go run s2encoding
```

3. Run JavaScript to compress the S2 cells

```
node .\make-tree-test.js
```

4. The raw output files will be located at `./out/stats.csv` and `./out/stats-b64tree.csv`. These files can be used in Excel or other softwares to make a report.

---
