---
title: ELK-日志系统代码调试
date: '2023-08-23 22:42:25'
updated: '2023-08-29 16:53:19'
tags:
  - ELK
permalink: /post/elklog-system-code-debugging-go-java-z2340ln.html
comments: true
toc: true
---
# ELK-日志系统代码调试（Go+Java）

调试思路：利用Go后端模拟文档数据请求数据调试ES，在Kibana上显示

## 第一版（非官方）

中间件：`"github.com/olivere/elastic/v7"`​

优化：添加的索引记录不会立刻查询到，也就是本代码执行后，搜索文档可能查不到新增的索引以及数据

代码：

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"log"

	"github.com/olivere/elastic/v7"
)

// Article 创建一个文档索引记录内容（文档标题、文章内容）
type Article struct {
	Title   string `json:"title"`
	Content string `json:"content"`
}

func main() {
	// 创建 Elasticsearch 客户端
	client, err := elastic.NewClient(
		elastic.SetURL("http://localhost:9200"),
		elastic.SetBasicAuth("elastic", "123456"),
		elastic.SetSniff(false),
	)
	if err != nil {
		log.Fatal("Error creating Elasticsearch client: ", err)
	}
	fmt.Println("Elasticsearch client created")

	// 创建索引
	indexName := "articles"
	createIndex(client, indexName)

	// 添加文档
	article := Article{Title: "Hello, World! Hello, Bobo!", Content: "This is a sample article about Go and Elasticsearch."}
	addDocument(client, indexName, article)

	// 搜索文档
	searchTerm := "Go"
	search(client, indexName, searchTerm)
}

func createIndex(client *elastic.Client, indexName string) {
	ctx := context.Background()
	exists, err := client.IndexExists(indexName).Do(ctx)

	if err != nil {
		log.Fatalf("Error checking if index [%s] exists: %v", indexName, err)
	}

	if !exists {
		createIndex, err := client.CreateIndex(indexName).Do(ctx)

		if err != nil {
			log.Fatalf("Error creating index: %v", err)
		}
		if createIndex.Acknowledged {
			fmt.Printf("Index [%s] created\n", indexName)
		}
	}
}

func addDocument(client *elastic.Client, indexName string, article Article) {
	ctx := context.Background()

	// 将文章对象序列化为 JSON 格式
	jsonArticle, err := json.Marshal(article)
	if err != nil {
		log.Fatalf("Error marshaling article: %v", err)
	}

	// 执行文档添加操作
	_, err = client.Index().
		Index(indexName).
		BodyJson(string(jsonArticle)).
		Do(ctx)

	if err != nil {
		log.Fatalf("Error adding document: %v", err)
	}

	fmt.Println("Document added successfully")
}

func search(client *elastic.Client, indexName string, searchTerm string) {
	ctx := context.Background()
	query := elastic.NewMultiMatchQuery(searchTerm, "title", "content").Type("phrase")

	searchResult, err := client.Search().
		Index(indexName).
		Query(query).
		From(0).Size(10).
		Pretty(true).
		Do(ctx)

	if err != nil {
		log.Fatalf("Error executing search: %v", err)
	}

	if searchResult.TotalHits() > 0 {
		fmt.Printf("Found %d articles:\n", searchResult.TotalHits())

		for _, hit := range searchResult.Hits.Hits {
			var article Article
			err := json.Unmarshal(hit.Source, &article)
			if err != nil {
				log.Fatalf("Error parsing document: %v", err)
			}
			fmt.Printf(" * %s\n", article.Title)
		}
	} else {
		fmt.Println("No articles found")
	}
}

```

‍

请求：http://localhost:9200/articles 可查看到新添加的articles索引

确保索引处于可搜索状态：http://localhost:9200/articles/_settings

全文搜索：http://localhost:9200/articles/_search?q=Go

即可搜索到代码中添加的：`This is a sample article about Go and Elasticsearch.`​

‍

## 第二版（官方）

这个是目前比较权威的实现方式。

目前的测试服的ELK版本是8.6.2

​![image](http://127.0.0.1:6806/assets/image-20230827152037-s46cxwx.png)​

ES的官方Go版本客户端：[github.com/elastic/go-el...](https://github.com/elastic/go-elasticsearch)

> // go.mod  
> github.com/elastic/go-elasticsearch/v7 v7.17.0  
> github.com/elastic/go-elasticsearch/v8 v8.0.0
>
> // main.go  
> import (  
>   elasticsearch7 "github.com/elastic/go-elasticsearch/v7"  
>   elasticsearch8 "github.com/elastic/go-elasticsearch/v8"  
> )  
> // ...  
> es7, _ := elasticsearch7.NewDefaultClient()  
> es8, _ := elasticsearch8.NewDefaultClient()

```go
package main

import (
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"log"
	"strings"

	"github.com/elastic/go-elasticsearch/v8"
	"github.com/elastic/go-elasticsearch/v8/esapi"
)

// Article 结构体，用于创建一个文档索引记录内容（文档标题、文章内容）
type Article struct {
	Title   string `json:"title"`
	Content string `json:"content"`
}

func main() {
	// 创建 Elasticsearch 客户端
	cfg := elasticsearch.Config{
		Addresses: []string{
			"http://localhost:9200",
		},
		Username: "elastic",
		Password: "123456",
	}
	es, err := elasticsearch.NewClient(cfg)
	if err != nil {
		log.Fatalf("创建Elasticsearch客户端时出错: %s", err)
	}

	// 创建索引
	indexName := "articles"
	createIndex(es, indexName)

	// 添加文档
	article := Article{Title: "Hello, World! Hello, Bobo!", Content: "This is a sample article about Go and Elasticsearch."}
	addDocument(es, indexName, article)

	// 搜索文档
	searchTerm := "Go"
	search(es, indexName, searchTerm)
}

// 创建索引的函数
func createIndex(es *elasticsearch.Client, indexName string) {
	// 检查索引是否存在
	res, err := es.Indices.Exists([]string{indexName})
	if err != nil {
		log.Fatalf("检查索引[%s]是否存在时出错: %s", indexName, err)
	}
	// 如果索引不存在，则创建索引
	if res.StatusCode == 404 {
		res, err := es.Indices.Create(indexName)
		if err != nil {
			log.Fatalf("创建索引时出错: %s", err)
		}
		if res.IsError() {
			log.Fatalf("创建索引时出错: %s", res.String())
		}
		fmt.Printf("索引[%s]创建成功\n", indexName)
	}
}

// 添加文档的函数
func addDocument(es *elasticsearch.Client, indexName string, article Article) {
	// 将文章对象序列化为 JSON 格式
	jsonArticle, err := json.Marshal(article)
	if err != nil {
		log.Fatalf("序列化文章时出错: %s", err)
	}

	// 执行文档添加操作
	req := esapi.IndexRequest{
		Index:      indexName,
		DocumentID: "1",
		Body:       strings.NewReader(string(jsonArticle)),
		Refresh:    "true",
	}

	res, err := req.Do(context.Background(), es)
	if err != nil {
		log.Fatalf("添加文档时出错: %s", err)
	}
	defer res.Body.Close()

	if res.IsError() {
		log.Fatalf("添加文档时出错: %s", res.String())
	}

	fmt.Println("文档添加成功")
}

// 搜索文档的函数
func search(es *elasticsearch.Client, indexName string, searchTerm string) {
	var buf bytes.Buffer
	query := map[string]interface{}{
		"query": map[string]interface{}{
			"match": map[string]interface{}{
				"title": searchTerm,
			},
		},
	}
	if err := json.NewEncoder(&buf).Encode(query); err != nil {
		log.Fatalf("编码搜索查询时出错: %s", err)
	}

	res, err := es.Search(
		es.Search.WithContext(context.Background()),
		es.Search.WithIndex(indexName),
		es.Search.WithBody(&buf),
		es.Search.WithTrackTotalHits(true),
		es.Search.WithPretty(),
	)
	if err != nil {
		log.Fatalf("执行搜索时出错: %s", err)
	}
	defer res.Body.Close()

	if res.IsError() {
		log.Fatalf("搜索返回错误: %s", res.String())
	}

	var r map[string]interface{}
	if err := json.NewDecoder(res.Body).Decode(&r); err != nil {
		log.Fatalf("解析搜索结果时出错: %s", err)
	}

	for _, hit := range r["hits"].(map[string]interface{})["hits"].([]interface{}) {
		doc := hit.(map[string]interface{})
		source, _ := json.MarshalIndent(doc["_source"], "", "  ")
		fmt.Printf(" * %s\n", source)
	}
}
```
