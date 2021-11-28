---
title: Ray Datasets 사용하기
date: 2021-11-26 21:18:00
tags: [Python, Distributed, Parallel, Ray]
categories: [Software, Disbritubed Computing]
keywords:
- Ray
- Error
coverImage: cover.jpeg
thumbnailImagePosition: left
thumbnailImage: cover.jpeg
autoThumbnailImage: yes
photos: https://images.unsplash.com/photo-1480843669328-3f7e37d196ae?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1470&q=80
---

이번 포스팅은 Ray에서 제공하는 Dataset에 대해서 알아봅니다.


<!-- excerpt -->


<!--toc-->

글쓰기에 앞서 본 포스팅은 제가 Ray Datasets를 이해하고자 쓰는 글입니다. 
글의 구성은 1). Ray Datasets에 대한 공식문서 번역, 2). parquet 파일을 Ray Dataset으로 활용하는 방법에 대해서 소개하려고 합니다.

## Ray Datasets

[Ray Dataset](https://docs.ray.io/en/latest/data/dataset.html)은 Ray 라이브러리와 애플리케이션에서 데이터를 불러오고 교환하는데 사용하는 기본적인 방법입니다. Datasets는 `map.filter`, `repartition`과 같은 분산 데이터 변환을 기본적으로 제공하고, 다양한 데이터 포맷, 데이터 소스, 분산 프레임워크와 호환됩니다.

![datasets](1.dataset.svg)

### 개념
Ray Datasets는 [Distributed Arrow](https://arrow.apache.org/)의 구현체입니다. Dataset은 `block`을 참조하고있는 Ray object의 list로 구성되어있습니다. 각 block은 Python list나 [Arrow Table](https://arrow.apache.org/docs/python/data.html#tables)을 가지고있습니다. Dataset에 다수의 block이 있다면, 데이터 병렬 변환 및 수집이 가능합니다.

다음은 3개의 Arrow table block를 갖는 Dataset을 시각화한 그림입니다. 여기서 각 block은 1,000개의 row를 가지고있다고 가정했습니다.

![dataset-arch](2.dataset-arch.svg)

Ray Dataset은 Ray object reference들을 모아놓은 list일 뿐이므로, Ray tasks, actor, 라이브러리 간에 자유롭게 전달할 수 있습니다. 이러한 유연성은 Ray Dataset의 고유한 특징입니다.

Spark RDDs나 Dask Bags과 비교했을 때, Datasets은 조금 더 기본적인 feature의 집합을 제공하고 단순함을 위해 작업(operation)을 즉시 실행(eagerly)합니다. 이는 사용자가 Datasets를 다른 dataframe type(`ds.to_dask()`)으로 casting할 수 있도록 의도한 것입니다. 다른 dataframe type으로 cating하게되면, 특정 dataframe type에서만 사용할 수 있는 고급 operation을 사용할 수 있게됩니다.

호환되는 Datasource 리스트는 [링크](https://docs.ray.io/en/latest/data/dataset.html#datasource-compatibility-matrices)에서 확인할 수 있습니다.

### Datasets 만들기

`ray.data.range()`와 `ray.data.from_items()`를 이용해서 생성된 데이터로 Ray Datasets를 만들어볼 수 있습니다. 이 때, Datasets는 Plain Python object나 Arrow records를 갖게됩니다.

```python
import ray

# Create a Dataset of Python objects.
ds = ray.data.range(10000)
# -> Dataset(num_blocks=200, num_rows=10000, schema=<class 'int'>)

ds.take(5)
# -> [0, 1, 2, 3, 4]

ds.count()
# -> 10000

# Create a Dataset of Arrow records.
ds = ray.data.from_items([{"col1": i, "col2": str(i)} for i in range(10000)])
# -> Dataset(num_blocks=200, num_rows=10000, schema={col1: int64, col2: string})

ds.show(5)
# -> ArrowRow({'col1': 0, 'col2': '0'})
# -> ArrowRow({'col1': 1, 'col2': '1'})
# -> ArrowRow({'col1': 2, 'col2': '2'})
# -> ArrowRow({'col1': 3, 'col2': '3'})
# -> ArrowRow({'col1': 4, 'col2': '4'})

ds.schema()
# -> col1: int64
# -> col2: string
```

Datasets는 local disk의 파일이나 S3와 같이 원격 datasource로부터 만들 수도 있습니다. pyarrow가 지원하는 filesystem이라면 특정한 파일 위치를 사용하면 됩니다.

```python
# Read a directory of files in remote storage.
ds = ray.data.read_csv("s3://bucket/path")

# Read multiple local files.
ds = ray.data.read_csv(["/path/to/file1", "/path/to/file2"])

# Read multiple directories.
ds = ray.data.read_csv(["s3://bucket/path1", "s3://bucket/path2"])
```

마지막으로, Ray object store에 있거나 Ray와 호환되는 Distributed DataFrame에 있는 데이터로부터 Dataset을 만들 수 있습니다. 아래 예제는 `pandas`의 `DataFrame`과 `dask`의 `dataframe`을 Ray Dataset으로 변환하는 예제입니다.
```python
import pandas as pd
import dask.dataframe as dd

# Create a Dataset from a list of Pandas DataFrame objects.
pdf = pd.DataFrame({"one": [1, 2, 3], "two": ["a", "b", "c"]})
ds = ray.data.from_pandas([pdf])

# Create a Dataset from a Dask-on-Ray DataFrame.
dask_df = dd.from_pandas(pdf, npartitions=10)
ds = ray.data.from_dask(dask_df)
```

### Datasets 저장하기
Datasets는 `.write_csv()`, `.write_json()`, `.write_parquet()` API를 통해 local이나 remote storage에 저장할 수 있습니다.
```python
# Write to csv files in /tmp/output.
ray.data.range(10000).write_csv("/tmp/output")
# -> /tmp/output/data0.csv, /tmp/output/data1.csv, ...

# Use repartition to control the number of output files:
ray.data.range(10000).repartition(1).write_csv("/tmp/output2")
# -> /tmp/output2/data0.csv
```

또한 Dataset을 Ray와 호환되는 Distributed DataFrames로 변환할 수 있습니다.
```python
# Convert a Ray Dataset into a Dask-on-Ray DataFrame.
dask_df = ds.to_dask()
```

### Dataset 변환
Datasets는 `.map()`을 사용하면 병렬적으로 변환작업을 수행할 수 있습니다. 변환(Transformation)은 즉시(eagerly) 실행되며 작업(operation)이 끝날 때까지 blocking됩니다. Datasets는 `.filter()`, `.flat_map()`을 지원합니다.
```python
ds = ray.data.range(10000)
ds = ds.map(lambda x: x * 2)
# -> Map Progress: 100%|████████████████████| 200/200 [00:00<00:00, 1123.54it/s]
# -> Dataset(num_blocks=200, num_rows=10000, schema=<class 'int'>)
ds.take(5)
# -> [0, 2, 4, 6, 8]

ds.filter(lambda x: x > 5).take(5)
# -> Map Progress: 100%|████████████████████| 200/200 [00:00<00:00, 1859.63it/s]
# -> [6, 8, 10, 12, 14]

ds.flat_map(lambda x: [x, -x]).take(5)
# -> Map Progress: 100%|████████████████████| 200/200 [00:00<00:00, 1568.10it/s]
# -> [0, 0, 2, -2, 4]
```

벡터화 함수(vectorized function)의 장점을 취하고 싶을 때에는 `.map_batches()`를 사용할 수 있습니다.

> `filter`, `flat_map`을 `.map_batches()`를 통해 구현할 수 있습니다. 이 때, map function은 특정 크기를 갖는 batch 출력을 반환해야합니다.

```python
ds = ray.data.range_arrow(10000)
ds = ds.map_batches(lambda df: df.applymap(lambda x: x * 2), batch_format="pandas")
# -> Map Progress: 100%|████████████████████| 200/200 [00:00<00:00, 1927.62it/s]
ds.take(5)
# -> [ArrowRow({'value': 0}), ArrowRow({'value': 2}), ...]
```

변환은 기본적으로 Ray tasks를 사용해서 실행됩니다. 설정이 필요한 변환의 경우 `compute="actors"`를 지정합니다. `compute="actors"`를 설정하면, Ray는 autoscaling actor pool을 사용하여 변환작업을 실행합니다.

```python
# Example of GPU batch inference on an ImageNet model.
def preprocess(image: bytes) -> bytes:
    return image

class BatchInferModel:
    def __init__(self):
        self.model = ImageNetModel()
    def __call__(self, batch: pd.DataFrame) -> pd.DataFrame:
        return self.model(batch)

ds = ray.data.read_binary_files("s3://bucket/image-dir")

# Preprocess the data.
ds = ds.map(preprocess)
# -> Map Progress: 100%|████████████████████| 200/200 [00:00<00:00, 1123.54it/s]

# Apply GPU batch inference with actors, and assign each actor a GPU using
# ``num_gpus=1`` (any Ray remote decorator argument can be used here).
ds = ds.map_batches(BatchInferModel, compute="actors", batch_size=256, num_gpus=1)
# -> Map Progress (16 actors 4 pending): 100%|██████| 200/200 [00:07, 27.60it/s]

# Save the results.
ds.repartition(1).write_json("s3://bucket/inference-results")
```

### Datasets의 교환
Datasets는 Ray tasks나 actor에 전달할 수 있고 `.iter_batches()`나 `.iter_rows()`를 통해 읽어들일 수 있습니다. 이 때, 읽기 작업은 복사를 수행하는 것이 아니라 block들의 reference가 담긴 Ray objects로 전달합니다.
```python
@ray.remote
def consume(data: Dataset[int]) -> int:
    num_batches = 0
    for batch in data.iter_batches():
        num_batches += 1
    return num_batches

ds = ray.data.range(10000)
ray.get(consume.remote(ds))
# -> 200
```

Datasets는 sub-datasets로 분리할 수 있습니다. 원하는 분할 개수와 actor의 handle을 `split()`함수에 전달하면 Locality-aware 분리를 수행할 수 있습니다.
```python
@ray.remote(num_gpus=1)
class Worker:
    def __init__(self, rank: int):
        pass

    def train(self, shard: ray.data.Dataset[int]) -> int:
        for batch in shard.iter_batches(batch_size=256):
            pass
        return shard.count()

workers = [Worker.remote(i) for i in range(16)]
# -> [Actor(Worker, ...), Actor(Worker, ...), ...]

ds = ray.data.range(10000)
# -> Dataset(num_blocks=200, num_rows=10000, schema=<class 'int'>)

shards = ds.split(n=16, locality_hints=workers)
# -> [Dataset(num_blocks=13, num_rows=650, schema=<class 'int'>),
#     Dataset(num_blocks=13, num_rows=650, schema=<class 'int'>), ...]

ray.get([w.train.remote(s) for s in shards])
# -> [650, 650, ...]
```

### Custom Datasources
Datasets는 python에서 정의된 [custom datasource](https://docs.ray.io/en/latest/data/package-ref.html#custom-datasource-api)을 사용해서 병렬적으로 read/write작업을 수행할 수 있습니다.

```python
# Read from a custom datasource.
ds = ray.data.read_datasource(YourCustomDatasource(), **read_args)

# Write to a custom datasource.
ds.write_datasource(YourCustomDatasource(), **write_args)
```

## 문제상황
용량이 매우 큰 parquet 파일들을 읽어들여 Ray tasks에 전달하여 작업해야한다고 했을 때, 이를 Datasets로 처리할 수 있습니다.

### 예제 데이터를 parquet 파일로 변환하기
json data를 pandas의 DataFrame으로 읽어들이고 이를 `sample.parq` 이름으로 저장합니다.
```python
import json
import pandas as pd

json_data = {"col1":{"row1":1,"row2":2,"row3":3},"col2":{"row1":"x","row2":"y","row3":"z"}}
df = pd.DataFrame(json_data)

df.to_parquet("./sample/sample.parq")
```

### Ray Dataset을 사용하는 actor를 정의하기
Ray의 Dataset을 batch로 처리하는 actor class를 정의합니다.

```python
import ray

@ray.remote
class Worker():
    def __init__(self, rank: int):
        pass

    def load_df_from_ds(self, shard: ray.data.Dataset):
        l = []
        for batch in shard.iter_batches():
            df = batch.to_pandas()
            l.append(df)
            # ... pandas processing
        
        return l
```

### Ray Datasets를 생성하기

이전에 저장했던 `sample.parq`을 Ray Dataset으로 만듭니다. 

```python
from pathlib import Path
import ray

# 절대경로 필수
data_path = Path("./sample").absolute()

parq_list = list(map(str, data_path.glob("*.parq")))
ds = ray.data.read_parquet(parq_list)
```

### Ray Datasets Split하기
locality-aware을 사용해서 Datasets을 split해줍니다.
```python
workers = [Worker.remote(i) for i in range(10)]
shards = ds.split(n=10, locality_hints=workers)
```

### Split한 데이터를 처리하기

아래 코드를 실행하여 split한 데이터를 처리합니다.

```python
ray.init()
print(ray.get([workers[rank].load_df_from_ds.remote(s) for rank, s in enumerate(shards)]))
ray.shutdown()
```

### 전체코드
```python
import json
from pathlib import Path

import pandas as pd
import ray

# Ray 실행
ray.init()

# Ray actor class 정의
@ray.remote
class Worker():
    def __init__(self, rank: int):
        pass

    def load_df_from_ds(self, shard: ray.data.Dataset):
        l = []
        for batch in shard.iter_batches():
            df = batch.to_pandas()
            l.append(df)
            # ... pandas processing
        
        return l

# 샘플 데이터 저장
json_data = {"col1":{"row1":1,"row2":2,"row3":3},"col2":{"row1":"x","row2":"y","row3":"z"}}
df = pd.DataFrame(json_data)

df.to_parquet("./sample/sample.parq")

data_path = Path("./sample").absolute()

# 데이터셋 로드
parq_list = list(map(str, data_path.glob("*.parq")))
ds = ray.data.read_parquet(parq_list)

# Ray actor 생성 및 Ray Datasets을 split
workers = [Worker.remote(i) for i in range(10)]
shards = ds.split(n=10, locality_hints=workers)

# Ray 코드 실행
print(ray.get([workers[rank].load_df_from_ds.remote(s) for rank, s in enumerate(shards)]))

# Ray 중지
ray.shutdown()
```


## Reference

1. [Datasets: Flexible Distributed Data Loading](https://docs.ray.io/en/latest/data/dataset.html)


