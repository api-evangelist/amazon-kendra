---
title: "Develop a RAG-based application using Amazon Aurora with Amazon Kendra"
url: "https://aws.amazon.com/blogs/machine-learning/develop-a-rag-based-application-using-amazon-aurora-with-amazon-kendra/"
date: "Tue, 28 Jan 2025 17:42:39 +0000"
author: "Aravind Hariharaputran"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-kendra/feed/"
---
RAG retrieves data from a preexisting knowledge base (your data), combines it with the LLM’s knowledge, and generates responses with more human-like language. However, in order for generative AI to understand your data, some amount of data preparation is required, which involves a big learning curve. In this post, we walk you through how to convert your existing Aurora data into an index without needing data preparation for Amazon Kendra to perform data search and implement RAG that combines your data along with LLM knowledge to produce accurate responses.
