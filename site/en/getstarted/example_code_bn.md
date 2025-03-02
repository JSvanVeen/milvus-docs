---
id: example_code_bn.md
---

# পাইথন ব্যবহার করে মিলভাস চালান

এই টপিকটি বর্ণনা করে কিভাবে পাইথন ব্যবহার করে মিলভাস চালানো যায়।

## 1. PyMilvus ইনস্টল করুন

```Python
pip3 install pymilvus=={{var.milvus_python_sdk_version}}
```
<div class="alert note">
পাইথন 3.6 বা তার পরের ভার্সন প্রয়োজন। দেখুন  <a href="https://wiki.python.org/moin/BeginnersGuide/Download">পাইথন ডাউনলোড করা হচ্ছে
</a> আরও তথ্যের জন্য.
</div>

## 2. নমুনা কোড ডাউনলোড করুন

```Python
$ wget https://raw.githubusercontent.com/milvus-io/pymilvus/v{{var.milvus_python_sdk_version}}/examples/hello_milvus.py
```

## 3. নমুনাটি স্ক্যান করুন
নমুনা কোডটি নিম্নলিখিত ধাপগুলি সম্পাদন করে।

- একটি PyMilvus প্যাকেজ আমদানি করে:
```Python
from pymilvus import connections, FieldSchema, CollectionSchema, DataType, Collection
```

- একটি সার্ভারের সাথে সংযোগ স্থাপন করে:
```Python
connections.connect(host='localhost', port='19530')
```

- একটি কালেকশন তৈরি করে:
```Python
dim = 128
default_fields = [
    FieldSchema(name="count", dtype=DataType.INT64, is_primary=True),
    FieldSchema(name="random_value", dtype=DataType.DOUBLE),
    FieldSchema(name="float_vector", dtype=DataType.FLOAT_VECTOR, dim=dim)
]
default_schema = CollectionSchema(fields=default_fields, description="পরীক্ষামূলক সংগ্রহ")

print(f"\nCreate collection...")
collection = Collection(name="hello_milvus", schema=default_schema)
```

- কাল্কেশনে ভেক্টর প্রবেশ করে:
```Python
import random
nb = 3000
vectors = [[random.random() for _ in range(dim)] for _ in range(nb)]
collection.insert(
    [
        [i for i in range(nb)],
        [float(random.randrange(-20,-10)) for _ in range(nb)],
        vectors
    ]
)
```

- সূচী তৈরি করে এবং কালেকশনটি লোড করে:
```Python
default_index = {"index_type": "IVF_FLAT", "params": {"nlist": 128}, "metric_type": "L2"}
collection.create_index(field_name="float_vector", index_params=default_index)
collection.load()
```

- ভেক্টর সাদৃশ্য অনুসন্ধান করে:
```Python
topK = 5
search_params = {"metric_type": "L2", "params": {"nprobe": 10}}
# define output_fields of search result
res = collection.search(
    vectors[-2:], "float_vector", search_params, topK,
    "count > 100", output_fields=["count", "random_value"]
)
```

আইডি এবং দূরত্ব অনুসারে অনুসন্ধান ফলাফলগুলি মুদ্রণ করতে, নিম্নলিখিত কমান্ডটি চালান।
```Python
for raw_result in res:
    for result in raw_result:
        id = result.id  # result id
        distance = result.distance
        print(id, distance)
```
আরও তথ্য পেতে [API Reference](/api-reference/pymilvus/v{{var.milvus_python_sdk_version}}/results.html) দেখুন। 

- একটি হাইব্রিড অনুসন্ধান করে
<div class="alert note">
    নিম্নলিখিত উদাহরণটি <code>film_id</code> এর সাহায্যে [২,৪,৬,৮] এর পরিসরে এন্টিটিগুলোর উপর আনুমানিক অনুসন্ধান করে
    </div>

```Python
from pymilvus import connections, Collection, FieldSchema, CollectionSchema, DataType
>>> import random
>>> connections.connect()
>>> schema = CollectionSchema([
...     FieldSchema("film_id", DataType.INT64, is_primary=True),
...     FieldSchema("films", dtype=DataType.FLOAT_VECTOR, dim=2)
... ])
>>> collection = Collection("test_collection_search", schema)
>>> # insert
>>> data = [
...     [i for i in range(10)],
...     [[random.random() for _ in range(2)] for _ in range(10)],
... ]
>>> collection.insert(data)
>>> collection.num_entities
10
>>> collection.load()
>>> # search
>>> search_param = {
...     "data": [[1.0, 1.0]],
...     "anns_field": "films",
...     "param": {"metric_type": "L2"},
...     "limit": 2,
...     "expr": "film_id in [2,4,6,8]",
... }
>>> res = collection.search(**search_param)
>>> assert len(res) == 1
>>> hits = res[0]
>>> assert len(hits) == 2
>>> print(f"- Total hits: {len(hits)}, hits ids: {hits.ids} ")
- Total hits: 2, hits ids: [2, 4]
>>> print(f"- Top1 hit id: {hits[0].id}, distance: {hits[0].distance}, score: {hits[0].score} ")
- Top1 hit id: 2, distance: 0.10143111646175385, score: 0.101431116461

```

## 4. নমুনাটি রান করুন 
```Python
$ python3 hello_milvus.py
```

*ফিরে আসা ফলাফল এবং ক্যোয়ারী এর বিলম্ব নিম্নে দেয়া হলো:*

<div class='result-bock'>
<p>Search...</p>
<p>(distance: 0.0, id: 2998) -20.0</p>
<p>(distance: 13.2614107131958, id: 989) -11.0</p>
<p>(distance: 14.489648818969727, id: 1763) -19.0</p>
<p>(distance: 15.295698165893555, id: 968) -20.0</p>
<p>(distance: 15.34445571899414, id: 2049) -19.0</p>
<p>(distance: 0.0, id: 2999) -12.0</p>
<p>(distance: 14.63361930847168, id: 1259) -13.0</p>
<p>(distance: 15.421361923217773, id: 2530) -15.0</p>
<p>(distance: 15.427900314331055, id: 600) -14.0</p>
<p>(distance: 15.538337707519531, id: 637) -19.0</p>
<p>search latency = 0.0549s</p>
</div>


<br/>


*অভিনন্দন! আপনি মিলভাস স্ট্যান্ড-অ্যালোনভাবে শুরু করেছেন এবং আপনার প্রথম ভেক্টর সাদৃশ্য অনুসন্ধান করেছেন।*

