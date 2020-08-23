Компания Microsoft опубликовала исходные тексты библиотеки машинного обучения SPTAG (Space Partition Tree And Graph) с реализацией алгоритма приблизительного поиска ближайшего соседа. Библиотека разработана в исследовательском подразделении Microsoft Research и центре разработки поисковых технологий (Microsoft Search Technology Center). На практике SPTAG применяется в поисковой системе Bing для определения наиболее релевантных результатов с учётом контекста и задания поисковых запросов на естественном языке. Код написан на языке С++ и распространяется под лицензией MIT. Поддерживается сборка для Linux и Windows. Имеется обвязка для языка Python.
https://github.com/Microsoft/SPTAG


Ключевое отличие векторного поиска от поиска по ключевым словам заключается в том, что векторы учитывают смысл и сходство данных, а не только символьные совпадения. Векторы формируются на основе модели машинного обучения, которая учитывает также сопутствующую статистику, уточняющую связи и позволяющую более точно оценить суть запроса (например, учитываются связь запроса с последующими переходами в поисковой выдаче). Несмотря на то, что идеи применения векторных хранилищ в поисковых системах витают уже достаточно давно, на практике их внедрению мешает большая ресурсоёмкость операций с векторами и ограничения в масштабируемости.

Совмещение методов глубинного машинного обучения с алгоритмами приблизительного поиска ближайшего соседа позволило довести производительность и масштабируемость векторных систем до уровня, приемлемого для крупных поисковых систем. Например, в Bing для векторного индекса размером более 150 миллиардов векторов время выборки наиболее релевантных результатов укладывается в 8 мс.

В состав библиотеки включены средства для построения индекса и организации поиска векторов, а также набор инструментов для сопровождения распределённой системы online-поиска, охватывающей очень большие коллекции векторов. Предлагаются следующие модули: index builder для индексации, searcher для поиска с использованием индекса, распределённого в кластере из нескольких узлов, сервер для запуска обработчиков на узлах, Aggregator для объединения нескольких серверов в одно целое и клиент для отправки запросов. Поддерживается включение новых векторов в индекс и удаление векторов на лету.

Библиотека подразумевает, что обрабатываемые и представленные в коллекции данные оформлены в виде связанных векторов, которые можно сравнивать на основе евклидовых (L2) или косинусных расстояний. При поисковом запросе возвращаются векторы, расстояние между которыми и исходным вектором минимально. В SPTAG предоставляется два метода организации векторного пространства: SPTAG-KDT (K-мерное дерево (kd-tree) и граф относительных окрестностей) и SPTAG-BKT (дерево k-средних (k-means tree и граф относительных окрестностей). Первый метод требует меньше ресурсов при работе с индексом, а второй демонстрирует более высокую точность результатов поиска при очень больших коллекциях векторов.

При этом векторный поиск не ограничивается текстом и может применяться к мультимедийной информации и изображениям, а также в системах автоматического формирования рекомендаций. Например, в одном из прототипов на базе фреймворка PyTorch была реализована векторная система для поиска с учётом сходства объектов на изображениях, построенная с использованием данных из нескольких эталонных коллекций с изображениями животных, кошек и собак, которые были преобразованы в наборы векторов. При поступлении входящего изображения для поиска оно преобразуется с использованием модели машинного обучения в вектор, на основе которого при помощи алгоритма SPTAG из индекса выбираются наиболее похожие векторы и как результат возвращаются связанные с ними изображения


# SPTAG: A library for fast approximate nearest neighbor search

[![MIT licensed](https://img.shields.io/badge/license-MIT-yellow.svg)](https://github.com/Microsoft/SPTAG/blob/master/LICENSE)
[![Build status](https://sysdnn.visualstudio.com/SPTAG/_apis/build/status/SPTAG-GITHUB)](https://sysdnn.visualstudio.com/SPTAG/_build/latest?definitionId=2)

## **SPTAG**
 SPTAG (Space Partition Tree And Graph) is a library for large scale vector approximate nearest neighbor search scenario released by [Microsoft Research (MSR)](https://www.msra.cn/) and [Microsoft Bing](http://bing.com). 

 <p align="center">
 <img src="docs/img/sptag.png" alt="architecture" width="500"/>
 </p>



## **Introduction**
 
This library assumes that the samples are represented as vectors and that the vectors can be compared by L2 distances or cosine distances. 
Vectors returned for a query vector are the vectors that have smallest L2 distance or cosine distances with the query vector. 

SPTAG provides two methods: kd-tree and relative neighborhood graph (SPTAG-KDT) 
and balanced k-means tree and relative neighborhood graph (SPTAG-BKT).
SPTAG-KDT is advantageous in index building cost, and SPTAG-BKT is advantageous in search accuracy in very high-dimensional data.



## **How it works**

SPTAG is inspired by the NGS approach [[WangL12](#References)]. It contains two basic modules: index builder and searcher. 
The RNG is built on the k-nearest neighborhood graph [[WangWZTG12](#References), [WangWJLZZH14](#References)] 
for boosting the connectivity. Balanced k-means trees are used to replace kd-trees to avoid the inaccurate distance bound estimation in kd-trees for very high-dimensional vectors.
The search begins with the search in the space partition trees for 
finding several seeds to start the search in the RNG. 
The searches in the trees and the graph are iteratively conducted. 

 ## **Highlights**
  * Fresh update: Support online vector deletion and insertion
  * Distributed serving: Search over multiple machines

 ## **Build**

### **Requirements**

* swig >= 3.0
* cmake >= 3.12.0
* boost >= 1.67.0

### **Install**

> For Linux:
```bash
mkdir build
cd build && cmake .. && make
```
It will generate a Release folder in the code directory which contains all the build targets.

> For Windows:
```bash
mkdir build
cd build && cmake -A x64 ..
```
It will generate a SPTAGLib.sln in the build directory. 
Compiling the ALL_BUILD project in the Visual Studio (at least 2019) will generate a Release directory which contains all the build targets.

For detailed instructions on installing Windows binaries, please see [here](docs/WindowsInstallation.md)

> Using Docker:
```bash
docker build -t sptag .
```
Will build a docker container with binaries in `/app/Release/`.

### **Verify** 

Run the test (or Test.exe) in the Release folder to verify all the tests have passed.

### **Usage**

The detailed usage can be found in [Get started](docs/GettingStart.md). There is also an end-to-end tutorial for building vector search online service using Python Wrapper in [Python Tutorial](docs/Tutorial.ipynb).
The detailed parameters tunning can be found in [Parameters](docs/Parameters.md).

## **References**
Please cite SPTAG in your publications if it helps your research:
```
@manual{ChenW18,
  author    = {Qi Chen and
               Haidong Wang and
               Mingqin Li and 
               Gang Ren and
               Scarlett Li and
               Jeffery Zhu and
               Jason Li and
               Chuanjie Liu and
               Lintao Zhang and
               Jingdong Wang},
  title     = {SPTAG: A library for fast approximate nearest neighbor search},
  url       = {https://github.com/Microsoft/SPTAG},
  year      = {2018}
}

@inproceedings{WangL12,
  author    = {Jingdong Wang and
               Shipeng Li},
  title     = {Query-driven iterated neighborhood graph search for large scale indexing},
  booktitle = {ACM Multimedia 2012},
  pages     = {179--188},
  year      = {2012}
}

@inproceedings{WangWZTGL12,
  author    = {Jing Wang and
               Jingdong Wang and
               Gang Zeng and
               Zhuowen Tu and
               Rui Gan and
               Shipeng Li},
  title     = {Scalable k-NN graph construction for visual descriptors},
  booktitle = {CVPR 2012},
  pages     = {1106--1113},
  year      = {2012}
}

@article{WangWJLZZH14,
  author    = {Jingdong Wang and
               Naiyan Wang and
               You Jia and
               Jian Li and
               Gang Zeng and
               Hongbin Zha and
               Xian{-}Sheng Hua},
  title     = {Trinary-Projection Trees for Approximate Nearest Neighbor Search},
  journal   = {{IEEE} Trans. Pattern Anal. Mach. Intell.},
  volume    = {36},
  number    = {2},
  pages     = {388--403},
  year      = {2014
}
```

## **Contribute**

This project welcomes contributions and suggestions from all the users.

We use [GitHub issues](https://github.com/Microsoft/SPTAG/issues) for tracking suggestions and bugs.

## **License**
The entire codebase is under [MIT license](https://github.com/Microsoft/SPTAG/blob/master/LICENSE)
