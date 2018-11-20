---
layout:     post
title:      "tensorflow-serving"
subtitle:   "tensorflow模型直接提供服务"
date:       2018-11-20
author:     "Qian"
header-img: "img/in-post/post-bg-tensorflow-serving.svg"
tags:
    - docker
    - tensorflow
---

tensorflow-serving实现将模型作为client可访问的服务，client通过 restful api 或是 grpc 来访问serving。

可以同时提供多个模型或同一模型的多个版本。这种灵活性有助于引入新版本、非原子性地将客户端迁移到新模型或版本以及A/B测试实验性模型。

使用tensorflow serving最简单的方式是Docker images，[tensorflow/serving repo](https://hub.docker.com/r/tensorflow/serving/tags/)支持各种版本的tf-serving镜像，包括cpu、gpu等，简单运行如下：

```
# Port 8500 exposed for gRPC
# Port 8501 exposed for the REST API
# Optional environment variable MODEL_NAME (defaults to model)
# Optional environment variable MODEL_BASE_PATH (defaults to /models)
docker run -p 8501:8501 \
  --mount type=bind,source=/path/to/my_model/,target=/models/my_model \
  -e MODEL_NAME=my_model -t tensorflow/serving
```

如需使用gpu版本的tf-serving docker，需要额外安装NVIDIA drivers以及nvidia-docker，其余使用与cpu版本相似：

```
docker run --runtime=nvidia -p 8501:8501 \
  --mount type=bind,\
  source=/tmp/tfserving/serving/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_gpu,\
  target=/models/half_plus_two \
  -e MODEL_NAME=half_plus_two -t tensorflow/serving:latest-gpu &
```

使用tensorflow-serving的关键在于，训练好的模型必须保存为统一格式：

    $ ls /tmp/mnist/1
    saved_model.pb variables

pb是序列化的tensorflow::SavedModel，包括模型的gragh定义以及模型metadata，variables是保存图的序列化变量的文件。

将该格式的模型文件导入docker镜像后，即可启动对应的模型服务。[官方的模型保存要求](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/python/saved_model/README.md)

以mnist_saved_model.py中保存过程为例：

```
export_path_base = sys.argv[-1]
export_path = os.path.join(
      compat.as_bytes(export_path_base),
      compat.as_bytes(str(FLAGS.model_version)))
print 'Exporting trained model to', export_path
builder = tf.saved_model.builder.SavedModelBuilder(export_path)
builder.add_meta_graph_and_variables(
      sess, [tag_constants.SERVING],
      signature_def_map={
           'predict_images':
               prediction_signature,
           signature_constants.DEFAULT_SERVING_SIGNATURE_DEF_KEY:
               classification_signature,
      },
      main_op=main_op)
builder.save()
```

分享几个网上存在一些已经存放的模型的docker镜像：

## [tensorflow-serving-tutorial](https://gyang274.github.io/tensorflow-serving-tutorial/)  quick start


### Docker Run Image

```
$ docker run -p 80:80 -d gyang274/yg-tfs-slim:rest
# 也同步到我的dockerhub （liqian950624/tensorflow-serving-slim）
```

### REST API

- check usage: `GET /`

```
# endpoint: GET /
#  - returns: usage

$ curl -X GET 127.0.0.1:80
```

- main endpoint: `POST /`

```
# endpoint: POST /
#  - payload:
#    - host: optional, host for tensorflow serving model server, default "127.0.0.1",
#    - port: optional, port for tensorflow serving model server, default "9000",
#    - model_name: optional, tensorflow serving model name, default "slim_inception_resnet_v2",
#        all available models: slim_inception_resnet_v2 at port 9000, and slim_inception_v4 at port 9090',
#    - image_urls: required, image urls in list
#  - returns:
#    - classes: top 5 classes of each input image_urls, in shape `n x 5`
#    - scores: top 5 classes scores (probabilities) of each input image_urls, in shape `n x 5`,
#    - prelogits: a numeric vector of 1536 of each input image_urls, in shape `n x 1536`, 
#        this vector can be viewed as features of each input image_urls for transfer learning or etc.

$ curl -X POST 127.0.0.1:80 -d '{
    "image_urls": [
        "https://upload.wikimedia.org/wikipedia/commons/d/d9/First_Student_IC_school_bus_202076.jpg",
        "https://upload.wikimedia.org/wikipedia/commons/thumb/9/90/Labrador_Retriever_portrait.jpg/1200px-Labrador_Retriever_portrait.jpg",
        "https://upload.wikimedia.org/wikipedia/commons/f/fd/Qantas_a380_vh-oqa_takeoff_heathrow_arp.jpg"
    ]
}'
```

- note

```
# note: the rest api expected a valid json as payload, so it might need to remove line breaks and make the post data in 
# one line, since the terminal might interpret line breaks as `\n` and add it into payload which causes invalid json.

$ curl -X POST 127.0.0.1:80 -d '{"image_urls": ["https://upload.wikimedia.org/wikipedia/commons/d/d9/First_Student_IC_school_bus_202076.jpg","https://upload.wikimedia.org/wikipedia/commons/thumb/9/90/Labrador_Retriever_portrait.jpg/1200px-Labrador_Retriever_portrait.jpg","https://upload.wikimedia.org/wikipedia/commons/f/fd/Qantas_a380_vh-oqa_takeoff_heathrow_arp.jpg"]}'
```

### Test the REST API via Python

```
import requests

response = requests.post(
  url="http://127.0.0.1:80",
  json={
    "image_urls": [
      "https://upload.wikimedia.org/wikipedia/commons/d/d9/First_Student_IC_school_bus_202076.jpg",
      "https://upload.wikimedia.org/wikipedia/commons/thumb/9/90/Labrador_Retriever_portrait.jpg/1200px-Labrador_Retriever_portrait.jpg",
      "https://upload.wikimedia.org/wikipedia/commons/f/fd/Qantas_a380_vh-oqa_takeoff_heathrow_arp.jpg"
    ]
  }
)

print(response.json())
```

## [GraphPipe](https://gyang274.github.io/tensorflow-serving-tutorial/) 

![](/img/in-post/post-tensorflow/graphpipe.jpg)

```
docker run -it --rm \
    -e https_proxy=${https_proxy} \
    -p 9000:9000 \
    sleepsonthefloor/graphpipe-tf:cpu \
    --model=https://oracle.github.io/graphpipe/models/squeezenet.pb \
    --listen=0.0.0.0:9000
```

oracle开源的GraphPipe protocol采用一种基于flatbuffer的低开销数据格式，它实现了高效的序列化/反序列化操作。不同于tensorflow使用的protocol buffer。此外，GraphPipe提供了用go编写的参考模型服务器，以简化部署机器学习模型的过程。根据经验，将现有的模型转换成通用的格式可能会涉及很多缺陷。因此，GraphPipe提供了能够本地运行最常见的ML模型格式的模型服务器。

GraphPipe设计模型服务器的目标：
+ Excellent performance
+ Simple, documented build process
+ Flexible code that is easy to work with
+ Support for the most common ML Frameworks
+ Optimized cpu and gpu support

使用tensorflow模型，有以下两种docker镜像提供：

    docker pull sleepsonthefloor/graphpipe-tf:cpu
    docker pull sleepsonthefloor/graphpipe-tf:gpu

支持的tensorflow模型格式：
+ SavedModel format - the tensorflow-serving directory format
+ GraphDef (.pb) format. 这只是将graphdef序列化为原生buf。如果您试图生产Keras模型，这是最容易生成的格式。




