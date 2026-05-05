---
title: "Develop a RAG-based application using Amazon Aurora with Amazon Kendra"
url: "https://aws.amazon.com/blogs/machine-learning/develop-a-rag-based-application-using-amazon-aurora-with-amazon-kendra/"
date: "Tue, 28 Jan 2025 17:42:39 +0000"
author: "Aravind Hariharaputran"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-kendra/feed/"
---
<div class="lb-alert lb-alert-info">
 April, 2026: Aurora Serverless v2 has been renamed Aurora serverless. No action required.
</div> 
<p><a href="https://aws.amazon.com/generative-ai/" rel="noopener" target="_blank">Generative AI</a> and large language models (LLMs) are revolutionizing organizations across diverse sectors to enhance customer experience, which traditionally would take years to make progress. Every organization has data stored in data stores, either on premises or in cloud providers.</p> 
<p>You can embrace generative AI and enhance customer experience by converting your existing data into an index on which generative AI can search. When you ask a question to an open source LLM, you get publicly available information as a response. Although this is helpful, generative AI can help you understand your data along with additional context from LLMs. This is achieved through Retrieval Augmented Generation (RAG).</p> 
<p>RAG retrieves data from a preexisting knowledge base (your data), combines it with the LLM’s knowledge, and generates responses with more human-like language. However, in order for generative AI to understand your data, some amount of data preparation is required, which involves a big learning curve.</p> 
<p><a href="https://aws.amazon.com/rds/aurora/" rel="noopener" target="_blank">Amazon Aurora</a> is a MySQL and PostgreSQL-compatible relational database built for the cloud. Aurora combines the performance and availability of traditional enterprise databases with the simplicity and cost-effectiveness of open source databases.</p> 
<p>In this post, we walk you through how to convert your existing Aurora data into an index without needing data preparation for <a href="https://aws.amazon.com/kendra/" rel="noopener" target="_blank">Amazon Kendra</a> to perform data search and implement RAG that combines your data along with LLM knowledge to produce accurate responses.</p> 
<h2>Solution overview</h2> 
<p>In this solution, use your existing data as a data source (Aurora), create an intelligent search service by connecting and syncing your data source to Amazon Kendra search, and perform generative AI data search, which uses RAG to produce accurate responses by combining your data along with the LLM’s knowledge. For this post, we use Anthropic’s Claude on <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> as our LLM.</p> 
<p>The following are the high-level steps for the solution:</p> 
<ul> 
 <li>Create an <a href="https://aws.amazon.com/rds/aurora/postgresql-features/" rel="noopener" target="_blank">Amazon Aurora PostgreSQL-Compatible Edition</a></li> 
 <li>Ingest data to Aurora PostgreSQL-Compatible.</li> 
 <li>Create an Amazon Kendra index.</li> 
 <li>Set up the Amazon Kendra Aurora PostgreSQL connector.</li> 
 <li>Invoke the RAG application.</li> 
</ul> 
<p>The following diagram illustrates the solution architecture.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454_solution_architecture.jpg" rel="noopener" target="_blank"><img alt="ML-16454_solution_architecture.jpg" class="alignnone size-full wp-image-97329" height="271" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454_solution_architecture.jpg" width="790" /></a></p> 
<h2>Prerequisites</h2> 
<p>To follow this post, the following prerequisites are required:</p> 
<ul> 
 <li>The <a href="https://aws.amazon.com/cli/" rel="noopener" target="_blank">AWS Command Line Interface</a> (AWS CLI) installed and configured</li> 
 <li>An <a href="https://portal.aws.amazon.com/billing/signup" rel="noopener" target="_blank">AWS account</a> and appropriate permissions to interact with resources in your AWS account</li> 
 <li>The AWS managed <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM) policy AmazonKendraReadOnlyAccess should be part of an <a href="https://aws.amazon.com/sagemaker/" rel="noopener" target="_blank">Amazon SageMaker</a> IAM role</li> 
 <li>An <a href="https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.CreateInstance.html" rel="noopener" target="_blank">Aurora DB cluster</a> where the current data is present</li> 
 <li>Your preferred interactive development environment (IDE) to run the Python script (such as SageMaker, or VS Code)</li> 
 <li>The <a href="https://www.pgadmin.org/download/" rel="noopener" target="_blank">pgAdmin</a> tool for data loading and validation</li> 
</ul> 
<h2>Create an Aurora PostgreSQL cluster</h2> 
<p>Run the following AWS CLI commands to create an Aurora PostgreSQL Serverless v2 cluster:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">aws rds create-db-cluster \
--engine aurora-postgresql \
--engine-version 15.4 \
--db-cluster-identifier genai-kendra-ragdb \
--master-username postgres \
--master-user-password XXXXX \
--db-subnet-group-name dbsubnet \
--vpc-security-group-ids "sg-XXXXX" \
--serverless-v2-scaling-configuration "MinCapacity=2,MaxCapacity=64" \
--enable-http-endpoint \
--region us-east-2

aws rds create-db-instance \
--db-cluster-identifier genai-kendra-ragdb \
--db-instance-identifier genai-kendra-ragdb-instance \
--db-instance-class db.serverless \
--engine aurora-postgresql</code></pre> 
</div> 
<p>The following screenshot shows the created instance.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-Aurora_instance.png" rel="noopener" target="_blank"><img alt="ML-16454-Aurora_instance" class="alignnone size-full wp-image-97337" height="102" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-Aurora_instance.png" width="1626" /></a></p> 
<h2>Ingest data to Aurora PostgreSQL-Compatible</h2> 
<p>Connect to the Aurora instance using the pgAdmin tool. Refer to <a href="https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToPostgreSQLInstance.html#USER_ConnectToPostgreSQLInstance.pgAdmin" rel="noopener" target="_blank">Connecting to a DB instance running the PostgreSQL database engine</a> for more information. To ingest your data, complete the following steps:</p> 
<ol> 
 <li>Run the following PostgreSQL statements in pgAdmin to create the database, schema, and table: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">CREATE DATABASE genai;
CREATE SCHEMA 'employees';

CREATE DATABASE genai;
SET SCHEMA 'employees';

CREATE TABLE employees.amazon_review(
pk int GENERATED ALWAYS AS IDENTITY NOT NULL,
id varchar(50) NOT NULL,
name varchar(300) NULL,
asins Text NULL,
brand Text NULL,
categories Text NULL,
keys Text NULL,
manufacturer Text NULL,
reviews_date Text NULL,
reviews_dateAdded Text NULL,
reviews_dateSeen Text NULL,
reviews_didPurchase Text NULL,
reviews_doRecommend varchar(100) NULL,
reviews_id varchar(150) NULL,
reviews_numHelpful varchar(150) NULL,
reviews_rating varchar(150) NULL,
reviews_sourceURLs Text NULL,
reviews_text Text NULL,
reviews_title Text NULL,
reviews_userCity varchar(100) NULL,
reviews_userProvince varchar(100) NULL,
reviews_username Text NULL,
PRIMARY KEY
(
pk
)
) ;</code></pre> 
  </div> </li> 
 <li>In your pgAdmin Aurora PostgreSQL connection, navigate to <strong>Databases</strong>, <strong>genai</strong>, <strong>Schemas</strong>, <strong>employees</strong>, <strong>Tables</strong>.</li> 
 <li>Choose (right-click) <strong>Tables </strong>and choose <strong>PSQL Tool</strong> to open a PSQL client connection.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454_PSQL_tool.png" rel="noopener" target="_blank"><img alt="ML-16454_psql_tool" class="alignnone size-full wp-image-97339" height="1158" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454_PSQL_tool.png" width="1772" /></a></li> 
 <li>Place the csv file under your pgAdmin location and run the following command: 
  <div class="hide-language"> 
   <pre><code class="lang-code">\copy employees.amazon_review (id, name, asins, brand, categories, keys, manufacturer, reviews_date, reviews_dateadded, reviews_dateseen, reviews_didpurchase, reviews_dorecommend, reviews_id, reviews_numhelpful, reviews_rating, reviews_sour
ceurls, reviews_text, reviews_title, reviews_usercity, reviews_userprovince, reviews_username) FROM 'C:\Program Files\pgAdmin 4\runtime\amazon_review.csv' DELIMITER ',' CSV HEADER ENCODING 'utf8';</code></pre> 
  </div> <p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454_copy_amazon_review_tbl.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-97890 size-full" height="219" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/23/copy_command_results.jpg" style="margin: 10px 0px 10px 0px;" width="1487" /></a></p></li> 
 <li>Run the following PSQL query to verify the number of records copied: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">Select count (*) from employees.amazon_review;</code></pre> 
  </div> <p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-count_amazon_review_tbl.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-97891 size-full" height="267" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/23/select_statement_image.jpg" style="margin: 10px 0px 10px 0px;" width="908" /></a></p></li> 
</ol> 
<h2>Create an Amazon Kendra index</h2> 
<p>The Amazon Kendra index holds the contents of your documents and is structured in a way to make the documents searchable. It has three index types:</p> 
<ul> 
 <li><strong>Generative AI Enterprise Edition index</strong> – Offers the highest accuracy for the Retrieve API operation and for RAG use cases (recommended)</li> 
 <li><strong>Enterprise Edition index</strong> – Provides semantic search capabilities and offers a high-availability service that is suitable for production workloads</li> 
 <li><strong>Developer Edition index</strong> – Provides semantic search capabilities for you to test your use cases</li> 
</ul> 
<p>To create an Amazon Kendra index, complete the following steps:</p> 
<ol> 
 <li>On the Amazon Kendra console, choose <strong>Indexes</strong> in the navigation pane.</li> 
 <li>Choose <strong>Create an index</strong>.</li> 
 <li>On the <strong>Specify index details</strong> page, provide the following information: 
  <ul> 
   <li>For <strong>Index name</strong>, enter a name (for example, <code>genai-kendra-index</code>).</li> 
   <li>For <strong>IAM role</strong>, choose <strong>Create a new role (Recommended)</strong>.</li> 
   <li>For <strong>Role name</strong>, enter an IAM role name (for example, <code>genai-kendra</code>). Your role name will be prefixed with<code> AmazonKendra-&lt;region&gt;-</code> (for example, <code>AmazonKendra-us-east-2-genai-kendra</code>).</li> 
  </ul> </li> 
 <li>Choose <strong>Next</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-specify-index-details.png" rel="noopener" target="_blank"><img alt="ML-16454-specify-index-details" class="alignnone size-full wp-image-97341" height="938" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-specify-index-details.png" width="993" /></a></li> 
 <li>On the <strong>Add additional capacity</strong> page, select <strong>Developer edition </strong>(for this demo) and choose <strong>Next</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-additional-capacity.png" rel="noopener" target="_blank"><img alt="ML-16454-additional-capacity" class="alignnone size-full wp-image-97338" height="667" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-additional-capacity.png" width="1154" /></a></li> 
 <li>On the <strong>Configure user access control</strong> page, provide the following information: 
  <ul> 
   <li>Under <strong>Access control settings</strong>¸ select <strong>No</strong>.</li> 
   <li>Under <strong>User-group expansion</strong>, select <strong>None</strong>.</li> 
  </ul> </li> 
 <li>Choose <strong>Next</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-configure-user-access-control.png" rel="noopener" target="_blank"><img alt="ML-16454-configure-user-access-control" class="alignnone size-full wp-image-97335" height="630" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-configure-user-access-control.png" width="1006" /></a></li> 
 <li>On the <strong>Review and create </strong>page, verify the details and choose <strong>Create</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-review-and-create.png" rel="noopener" target="_blank"><img alt="ML-16454-review-and-create" class="alignnone size-full wp-image-97344" height="741" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-review-and-create.png" width="1051" /></a></li> 
</ol> 
<p>It might take some time for the index to create. Check the list of indexes to watch the progress of creating your index. When the status of the index is <strong>ACTIVE</strong>, your index is ready to use.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-genai-kendra-index.png" rel="noopener" target="_blank"><img alt="ML-16454-genai-kendra-index" class="alignnone size-full wp-image-97348" height="755" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-genai-kendra-index.png" width="1640" /></a></p> 
<h2>Set up the Amazon Kendra Aurora PostgreSQL connector</h2> 
<p>Complete the following steps to set up your data source connector:</p> 
<ol> 
 <li>On the Amazon Kendra console, choose <strong>Data sources</strong> in the navigation pane.</li> 
 <li>Choose <strong>Add data source</strong>.</li> 
 <li>Choose <strong>Aurora PostgreSQL connector</strong> as the data source type.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-postgresql-connector.png" rel="noopener" target="_blank"><img alt="ML-16454-postgresql-connector" class="alignnone size-full wp-image-97347" height="702" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-postgresql-connector.png" width="1470" /></a></li> 
 <li>On the <strong>Specify data source details</strong> page, provide the following information: 
  <ul> 
   <li>For <strong>Data source name</strong>, enter a name (for example, <code>data_source_genai_kendra_postgresql</code>).</li> 
   <li>For <strong>Default language</strong>¸ choose <strong>English (en)</strong>.</li> 
   <li>Choose <strong>Next</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_specify-data-source-details.jpg"><img alt="" class="alignnone size-full wp-image-97948" height="813" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_specify-data-source-details.jpg" width="2560" /></a></li> 
  </ul> </li> 
 <li>On the <strong>Define access and security </strong>page, under <strong>Source</strong>, provide the following information: 
  <ul> 
   <li>For <strong>Host</strong>, enter the host name of the PostgreSQL instance (<code>cvgupdj47zsh.us-east-2.rds.amazonaws.com</code>).</li> 
   <li>For <strong>Port</strong>, enter the port number of the PostgreSQL instance (<code>5432</code>).</li> 
   <li>For <strong>Instance</strong>, enter the database name of the PostgreSQL instance (<code>genai</code>).<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_define_access_and_security.jpg"><img alt="" class="alignnone size-full wp-image-97947" height="1200" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_define_access_and_security.jpg" width="2560" /></a></li> 
  </ul> </li> 
 <li>Under <strong>Authentication</strong>, if you already have credentials stored in <a href="https://aws.amazon.com/secrets-manager/" rel="noopener" target="_blank">AWS Secrets Manager</a>, choose it on the dropdown Otherwise, choose <strong>Create and add new secret</strong>.</li> 
 <li>In the <strong>Create an AWS Secrets Manager secret </strong>pop-up window, provide the following information: 
  <ul> 
   <li>For <strong>Secret name</strong>, enter a name (for example, <code>AmazonKendra-Aurora-PostgreSQL-genai-kendra-secret</code>).</li> 
   <li>For <strong>Data base user name</strong>, enter the name of your database user.</li> 
   <li>For <strong>Password</strong>¸ enter the user password.</li> 
  </ul> </li> 
 <li>Choose <strong>Add Secret</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-create-aws-secrets-manager.png" rel="noopener" target="_blank"><img alt="ML-16454-create-aws-secrets-manager" class="alignnone size-full wp-image-97332" height="443" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-create-aws-secrets-manager.png" width="675" /></a></li> 
 <li>Under <strong>Configure VPC and security group</strong>, provide the following information: 
  <ul> 
   <li>For <strong>Virtual Private Cloud</strong>, choose your virtual private cloud (VPC).</li> 
   <li>For <strong>Subnet</strong>, choose your subnet.</li> 
   <li>For <strong>VPC security groups</strong>, choose the VPC security group to allow access to your data source.</li> 
  </ul> </li> 
 <li>Under <strong>IAM role</strong>¸ if you have an existing role, choose it on the dropdown menu. Otherwise, choose <strong>Create a new role</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-create_a_new_IAM_role.png" rel="noopener" target="_blank"><img alt="ML-16454-create_a_new_IAM_role" class="alignnone size-full wp-image-97333" height="534" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-create_a_new_IAM_role.png" width="803" /></a></li> 
 <li>On the <strong>Configure sync settings</strong> page, under <strong>Sync scope</strong>, provide the following information: 
  <ul> 
   <li>For <strong>SQL query</strong>, enter the SQL query and column values as follows: <code>select * from employees.amazon_review</code>.</li> 
   <li>For <strong>Primary key</strong>, enter the primary key column (<code>pk</code>).</li> 
   <li>For <strong>Title</strong>, enter the title column that provides the name of the document title within your database table (<code>reviews_title</code>).</li> 
   <li>For <strong>Body</strong>, enter the body column on which your Amazon Kendra search will happen (<code>reviews_text</code>).</li> 
  </ul> </li> 
 <li>Under <strong>Sync node</strong>, select <strong>Full sync</strong> to convert the entire table data into a searchable index.</li> 
</ol> 
<p>After the sync completes successfully, your Amazon Kendra index will contain the data from the specified Aurora PostgreSQL table. You can then use this index for intelligent search and RAG applications.</p> 
<ol start="13"> 
 <li>Under <strong>Sync run schedule</strong>, choose <strong>Run on demand</strong>.</li> 
 <li>Choose <strong>Next</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_configure_sync_setting.jpg"><img alt="" class="alignnone size-full wp-image-97950" height="1243" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_configure_sync_setting.jpg" width="2560" /></a></li> 
 <li>On the <strong>Set field mappings </strong>page, leave the default settings and choose <strong>Next</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_set-field-mapping.jpg"><img alt="" class="alignnone size-full wp-image-97949" height="653" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_set-field-mapping.jpg" width="2560" /></a></li> 
 <li>Review your settings and choose <strong>Add data source</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_review_create_11.jpg"><img alt="" class="alignnone size-full wp-image-97952" height="1539" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_review_create_11.jpg" width="1485" /></a><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_review_create_21.jpg"><img alt="" class="alignnone size-full wp-image-97953" height="1256" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/24/AK_ML-16454_review_create_21.jpg" width="1485" /></a></li> 
</ol> 
<p>Your data source will appear on the <strong>Data sources</strong> page after the data source has been created successfully.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-data-source-creation-success.png" rel="noopener" target="_blank"><img alt="ML-16454-data-source-creation-success" class="alignnone size-full wp-image-97331" height="1023" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-data-source-creation-success.png" width="1603" /></a></p> 
<h2>Invoke the RAG application</h2> 
<p>The Amazon Kendra index sync can take minutes to hours depending on the volume of your data. When the sync completes without error, you are ready to develop your RAG solution in your preferred IDE. Complete the following steps:</p> 
<ol> 
 <li>Configure your AWS credentials to allow Boto3 to interact with AWS services. You can do this by setting the <code>AWS_ACCESS_KEY_ID</code> and <code>AWS_SECRET_ACCESS_KEY</code> environment variables or by using the <code>~/.aws/credentials</code> file: 
  <div class="hide-language"> 
   <pre><code class="lang-python">import boto3
&nbsp;&nbsp;pip install langchain

# Create a Boto3 session

session = boto3.Session(
&nbsp;&nbsp; aws_access_key_id='YOUR_AWS_ACCESS_KEY_ID',
&nbsp;&nbsp; aws_secret_access_key='YOUR_AWS_SECRET_ACCESS_KEY',
&nbsp;&nbsp; region_name='YOUR_AWS_REGION'
)</code></pre> 
  </div> </li> 
 <li>Import LangChain and the necessary components: 
  <div class="hide-language"> 
   <pre><code class="lang-python">from langchain_community.llms import Bedrock
from langchain_community.retrievers import AmazonKendraRetriever
from langchain.chains import RetrievalQA</code></pre> 
  </div> </li> 
 <li>Create an instance of the LLM (Anthropic’s Claude): 
  <div class="hide-language"> 
   <pre><code class="lang-python">llm = Bedrock(
region_name = "bedrock_region_name",
model_kwargs = {
"max_tokens_to_sample":300,
"temperature":1,
"top_k":250,
"top_p":0.999,
"anthropic_version":"bedrock-2023-05-31"
},
model_id = "anthropic.claude-v2"
)</code></pre> 
  </div> </li> 
 <li>Create your prompt template, which provides instructions for the LLM: 
  <div class="hide-language"> 
   <pre><code class="lang-code">from langchain_core.prompts import PromptTemplate

prompt_template = """
You are a &lt;persona&gt;Product Review Specialist&lt;/persona&gt;, and you provide detail product review insights.
You have access to the product reviews in the &lt;context&gt; XML tags below and nothing else.

&lt;context&gt;
{context}
&lt;/context&gt;

&lt;question&gt;
{question}
&lt;/question&gt;
"""

prompt = PromptTemplate(template=prompt_template, input_variables=["context", "question"])</code></pre> 
  </div> </li> 
 <li>Initialize the <code>KendraRetriever</code> with your Amazon Kendra index ID by replacing the <code>Kendra_index_id</code> that you created earlier and the Amazon Kendra client: 
  <div class="hide-language"> 
   <pre><code class="lang-python">session = boto3.Session(region_name='Kendra_region_name')
kendra_client = session.client('kendra')
# Create an instance of AmazonKendraRetriever
kendra_retriever = AmazonKendraRetriever(
kendra_client=kendra_client,
index_id="Kendra_Index_ID"
)</code></pre> 
  </div> </li> 
 <li>Combine Anthropic’s Claude and the Amazon Kendra retriever into a RetrievalQA chain: 
  <div class="hide-language"> 
   <pre><code class="lang-code">qa = RetrievalQA.from_chain_type(
llm=llm,
chain_type="stuff",
retriever=kendra_retriever,
return_source_documents=True,
chain_type_kwargs={"prompt": prompt},
)</code></pre> 
  </div> </li> 
 <li>Invoke the chain with your own query: 
  <div class="hide-language"> 
   <pre><code class="lang-code">query = "What are some products that has bad quality reviews, summarize the reviews"
result_ = qa.invoke(
query
)
result_</code></pre> 
  </div> <p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-RAG-output.png" rel="noopener" target="_blank"><img alt="ML-16454-RAG-output" class="alignnone size-full wp-image-97346" height="301" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ML-16454-RAG-output.png" width="1844" /></a></p></li> 
</ol> 
<h2>Clean up</h2> 
<p>To avoid incurring future charges, delete the resources you created as part of this post:</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_DeleteCluster.html" rel="noopener" target="_blank">Delete the Aurora DB cluster and DB instance</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/kendra/latest/dg/delete-index.html" rel="noopener" target="_blank">Delete the Amazon Kendra index</a>.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this post, we discussed how to convert your existing Aurora data into an Amazon Kendra index and implement a RAG-based solution for the data search. This solution drastically reduces the data preparation need for Amazon Kendra search. It also increases the speed of generative AI application development by reducing the learning curve behind data preparation.</p> 
<p>Try out the solution, and if you have any comments or questions, leave them in the comments section.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/Aravind_blog_image.jpg" rel="noopener" target="_blank"><img alt="" class="wp-image-97701 size-full alignleft" height="134" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/22/Aravind_blog_image-1.jpg" width="100" /></a><strong>Aravind Hariharaputran</strong> is a Data Consultant with the Professional Services team at Amazon Web Services. He is passionate about Data and AIML in general with extensive experience managing Database technologies .He helps customers transform legacy database and applications to Modern data platforms and generative AI applications. He enjoys spending time with family and playing cricket.</p> 
<p style="clear: both;"><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/17/ivan_cui.png" rel="noopener" target="_blank"><img alt="" class="wp-image-97700 size-full alignleft" height="110" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/01/22/ivan_cui-1.png" width="100" /></a><strong>Ivan Cui</strong> is a Data Science Lead with AWS Professional Services, where he helps customers build and deploy solutions using ML and generative AI on AWS. He has worked with customers across diverse industries, including software, finance, pharmaceutical, healthcare, IoT, and entertainment and media. In his free time, he enjoys reading, spending time with his family, and traveling.</p>
