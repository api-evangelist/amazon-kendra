---
title: "Introducing document-level sync reports: Enhanced data sync visibility in Amazon Kendra"
url: "https://aws.amazon.com/blogs/machine-learning/introducing-document-level-sync-reports-enhanced-data-sync-visibility-in-amazon-kendra/"
date: "Fri, 20 Sep 2024 19:36:16 +0000"
author: "Aneesh Mohan"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-kendra/feed/"
---
<p><a href="https://aws.amazon.com/kendra/" rel="noopener" target="_blank">Amazon Kendra</a>&nbsp;is an intelligent search service powered by machine learning (ML). Amazon Kendra helps you aggregate content from a variety of content repositories into a centralized index that lets you quickly search all your enterprise data and find the most accurate answer.</p> 
<p>Amazon Kendra securely connects to over 40 data sources. When using your data source, you might want better visibility into the document processing lifecycle during data source sync jobs. They could include knowing the status of each document you attempted to crawl and index, as well as being able to troubleshoot why certain documents were not returned with the expected answers. Additionally, you might need access to metadata, timestamps, and access control lists (ACLs) for the indexed documents.</p> 
<p>We are pleased to announce a new feature now available in Amazon Kendra that significantly improves visibility into data source sync operations. The latest release introduces a comprehensive document-level report incorporated into the sync history, providing administrators with granular indexing status, metadata, and ACL details for every document processed during a data source sync job. This enhancement to sync job observability enables administrators to quickly investigate and resolve ingestion or access issues encountered while setting up Amazon Kendra indexes. The detailed document reports are persisted in the new <code>SYNC_RUN_HISTORY_REPORT</code> log stream under the Amazon Kendra index log group, so critical sync job details are available on-demand when troubleshooting.</p> 
<p>In this post, we discuss the benefits of this new feature and how it offers enhanced data sync visibility in Amazon Kendra.</p> 
<h2><strong>Lifecycle of a document in a data source sync run job</strong></h2> 
<p>In this section, we examine the lifecycle of a document within a data source sync in Amazon Kendra. This provides valuable insight into the sync process. The data source sync comprises three key stages: crawling, syncing, and indexing. Crawling involves the connector connecting to the data source and extracting documents meeting the defined sync scope according to the data source configuration. These documents are then synced to the Amazon Kendra index during the syncing phase. Finally, indexing makes the synced documents searchable within the Amazon Kendra environment.</p> 
<p>The following diagram shows a flowchart of a sync run job.</p> 
<p><strong><img alt="" class="alignnone size-full wp-image-86537" height="7163" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/13/SYNC-RUN-Flowchart-v3-1.png" width="4088" /></strong></p> 
<h3><strong>Crawling stage</strong></h3> 
<p>The first stage is the crawling stage, where the connector crawls all documents and their metadata from the data source. During this stage, the connector also compares the checksum of the document against the Amazon Kendra index to determine if a particular document needs to be added, modified, or deleted from the index. This operation corresponds to the <code>CrawlAction</code> field in the sync run history report.</p> 
<p>If the document is unmodified, it’s marked as <code>UNMODIFIED</code> and skipped in the rest of the stages. If any document fails in the crawling stage, for example due to throttling errors, broken content, or if the document size is too big, that document is marked in the sync run history report with the <code>CrawlStatus</code> as <code>FAILED</code>. If the document was skipped due to any validation errors, its <code>CrawlStatus</code> is marked as <code>SKIPPED</code>. These documents are not sent to the next stage. All successful documents are marked as <code>SUCCESS</code> and are sent forward.</p> 
<p>We also capture the ACLs and metadata on each document in this stage to be able to add it to the sync run history report.</p> 
<h3><strong>Syncing stage</strong></h3> 
<p>During the syncing stage, the document is sent to Amazon Kendra ingestion service APIs like <code>BatchPutDocument</code> and <code>BatchDeleteDocument</code>. After a document is submitted to these APIs, Amazon Kendra runs validation checks on the submitted documents. If any document fails these checks, its <code>SyncStatus</code> is marked as <code>FAILED</code>. If there is an irrecoverable error for a particular document, it is marked as <code>SKIPPED</code> and other documents are sent forward.</p> 
<h3><strong>Indexing stage</strong></h3> 
<p>In this step, Amazon Kendra parses the document, processes it according to its content type, and persists it in the index. If the document fails to be persisted, its <code>IndexStatus</code> is marked as <code>FAILED</code>; otherwise, it is marked as <code>SUCCESS</code>.</p> 
<p>After the statuses of all the stages have been captured, we emit these statuses as an <a href="http://aws.amazon.com/cloudwatch" rel="noopener" target="_blank">Amazon CloudWatch</a> event to the customer’s AWS account.</p> 
<h2><strong>Key features and benefits of document-level reports</strong></h2> 
<p>The following are the key features and benefits of the new document-level report in Amazon Kendra indexes:</p> 
<ul> 
 <li><strong>Enhanced sync run history page</strong> – A new <strong>Actions</strong> column has been added to the sync run history page, providing access to the document-level report for each sync run.</li> 
</ul> 
<p><img alt="" class="alignnone size-full wp-image-86525" height="456" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/13/ML-17547-Image1.png" width="936" /></p> 
<ul> 
 <li><strong>Dedicated log stream</strong> – A new log stream named <code>SYNC_RUN_HISTORY_REPORT</code> has been created in the Amazon Kendra CloudWatch log group, containing the document-level report.</li> 
</ul> 
<p><img alt="" class="alignnone size-full wp-image-86527" height="440" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/13/ML-17547-Image2.png" width="936" /></p> 
<ul> 
 <li><strong>Comprehensive document information</strong> – The document-level report includes the following information for each document:</li> 
 <li><strong>Document ID</strong> – This is the document ID that is inherited directly from the data source or mapped by the customer in the data source field mappings.</li> 
 <li><strong>Document title</strong> – The title of the document is taken from the data source or mapped by the customer in the data source field mappings.</li> 
 <li><strong>Consolidated document status (SUCCESS, FAILED, or SKIPPED)</strong> – This is the final consolidated status of the document. It can have a value of <code>SUCCESS</code>, <code>FAILED</code>, or <code>SKIPPED</code>. If the document was successfully processed in all stages, then the value is <code>SUCCESS</code>. If the document failed or was skipped in any of the stages, then the value of this field will be <code>FAILED</code> or <code>SKIPPED</code>, respectively.</li> 
 <li><strong>Error message (if the document failed)</strong> – This field contains the error message with which a document failed. If a document was skipped due to throttling errors, or any internal errors, this will be shown in the error message field.</li> 
 <li><strong>Crawl status</strong> – This field denotes whether the document was crawled successfully from the data source. This status correlates to the syncing-crawling state in the data source sync.</li> 
 <li><strong>Sync status</strong> – This field denotes whether the document was sent for syncing successfully. This correlates to the syncing-indexing state in the data source sync.</li> 
 <li><strong>Index status</strong> – This field denotes whether the document was successfully persisted in the index.</li> 
 <li><strong>ACLs</strong> – This field contains a list of document-level permissions that were crawled from the data source. The details of each element in the list are: 
  <ul> 
   <li><strong>Global name </strong>– This is the email or user name of the user. This field is mapped across multiple data sources. For example, if a user has three datasources Confluence, SharePoint, and Gmail, with the local user ID as <code>confluence_user</code>, <code>sharepoint_user</code> and <code>gmail_user</code> respectively, and their email address user@email.com is the <code>globalName</code> in the ACL for all of them, then Amazon Kendra understands that all of these local user IDs map to the same global name.</li> 
   <li><strong>Name</strong> – This is the local unique ID of the user, which is assigned by the data source.</li> 
   <li><strong>Type</strong> – This field indicates the principal type. This can be either USER or GROUP.</li> 
   <li><strong>Is Federated</strong> – This is a boolean flag that indicates whether the group is of INDEX level (true) or DATASOURCE level (false).</li> 
   <li><strong>Access</strong> – This field indicates whether the user has access allowed or denied explicitly. Values can be either ALLOWED or DENIED.</li> 
   <li><strong>Data source ID</strong> – This is the data source ID. For federated groups (INDEX level), this field will be null.</li> 
  </ul> </li> 
 <li><strong>Metadata</strong> – This field contains the metadata fields (other than ACL) that were pulled from the data source. This list also includes the metadata fields mapped by the customer in the data source field mappings as well as extra metadata fields added by the connector.</li> 
 <li><strong>Hashed document ID (for troubleshooting assistance)</strong> – To safeguard your data privacy, we present a secure, one-way hash of the document identifier. This encrypted value enables the Amazon Kendra team to efficiently locate and analyze the specific document within our logs, should you encounter any issue that requires further investigation and resolution.</li> 
 <li><strong>Timestamp</strong> – The timestamp indicates when the document status was logged in CloudWatch.</li> 
</ul> 
<p>In the following sections, we explore different use cases for the logging feature.</p> 
<h2><strong>Determine the optimal boosting duration for recent documents in using document-level reporting</strong></h2> 
<p>When it comes to generating accurate answers, you may want to fine-tune the way Amazon Kendra prioritizes its content. For instance, you may prefer to boost recent documents over older ones to make sure the most up-to-date passages are used to generate an answer. To achieve this, you can use the relevance tuning feature in Amazon Kendra to boost documents based on the last update date attribute, with a specified boosting duration. However, determining the optimal boosting period can be challenging when dealing with a large number of frequently changing documents.</p> 
<p>You can now use the per-document-level report to obtain the <code>_last_updated_at</code> metadata field information for your documents, which can help you determine the appropriate boosting period. For this, you use the following CloudWatch Logs Insights query to retrieve the <code>_last_updated_at</code> metadata attribute for machine learning documents from the <code>SYNC_RUN_HISTORY_REPORT</code> log stream.</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">filter @logStream like 'SYNC_RUN_HISTORY_REPORT/'
and Metadata like 'Machine Learning'
| parse Metadata '{"key":"_last_updated_at","value":{"dateValue":"*"}}' as @last_updated_at
| sort @last_updated_at desc, @timestamp desc
| dedup DocumentTitle</code></pre> 
</div> 
<p>With the preceding query, you can gain insights into the last updated timestamps of your documents, enabling you to make informed decisions about the optimal boosting period. This approach makes sure your chat responses are generated using the most recent and relevant information, enhancing the overall accuracy and effectiveness of your Amazon Kendra implementation.</p> 
<p>The following screenshot shows an example result.</p> 
<p><img alt="" class="alignnone size-full wp-image-86528" height="526" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/13/ML-17547-Image3.png" width="936" /></p> 
<h2><strong>Common document indexing observability and troubleshooting methods</strong></h2> 
<p>In this section, we explore some common admin tasks for observing and troubleshooting document indexing using the new document-level reporting feature.</p> 
<h3><strong>List all successfully indexed documents from a data source</strong></h3> 
<p>To retrieve a list of all documents that have been successfully indexed from a specific data source, you can use the following CloudWatch Logs Insights query:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">fields DocumentTitle, DocumentId, @timestamp
| filter @logStream like 'SYNC_RUN_HISTORY_REPORT/your-data-source-id/'
and ConnectorDocumentStatus.Status = "SUCCESS"
| sort @timestamp desc | dedup DocumentTitle, DocumentId</code></pre> 
</div> 
<p>The following screenshot shows an example result.</p> 
<p><img alt="" class="alignnone size-full wp-image-86532" height="504" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/13/ML-17547-Image5.png" width="936" /></p> 
<h3><strong>List all successfully indexed documents from a data source sync job</strong></h3> 
<p>To retrieve a list of all documents that have been successfully indexed during a specific sync job, you can use the following CloudWatch Logs Insights query:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">fields DocumentTitle, DocumentId, ConnectorDocumentStatus.Status AS IndexStatus, @timestamp
| filter @logStream like 'SYNC_RUN_HISTORY_REPORT/your-data-source-id/run-id'
and ConnectorDocumentStatus.Status = "SUCCESS"
| sort DocumentTitle</code></pre> 
</div> 
<p>The following screenshot shows an example result.</p> 
<p><img alt="" class="alignnone size-full wp-image-86530" height="534" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/13/ML-17547-Image6.png" width="936" /></p> 
<h3><strong>List all failed indexed documents from a data source sync job</strong></h3> 
<p>To retrieve a list of all documents that failed to index during a specific sync job, along with the error messages, you can use the following CloudWatch Logs Insights query:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">fields DocumentTitle, DocumentId, ConnectorDocumentStatus.Status AS IndexStatus, ErrorMsg, @timestamp
| filter @logStream like 'SYNC_RUN_HISTORY_REPORT/your-data-source-id/run-id'
and ConnectorDocumentStatus.Status = "FAILED"
| sort @timestamp desc</code></pre> 
</div> 
<p>The following screenshot shows an example result.</p> 
<p><img alt="" class="alignnone size-full wp-image-86529" height="356" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/13/ML-17547-Image7.png" width="936" /></p> 
<h3><strong>List all documents that contain a user’s ACL permission from an Amazon Kendra index</strong></h3> 
<p>To retrieve a list of documents that have a specific users ACL permission, you can use the following CloudWatch Logs Insights query:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">filter @logStream like 'SYNC_RUN_HISTORY_REPORT/'
and Acl like 'aneesh@mydemoaws.onmicrosoft.com'
| display DocumentTitle, SourceUri</code></pre> 
</div> 
<p>The following screenshot shows an example result.</p> 
<p><em> <img alt="" class="alignnone size-full wp-image-86523" height="498" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/13/ML-17547-Image8.png" width="936" /></em></p> 
<h3><strong>List the ACL of an indexed document from a data source sync job</strong></h3> 
<p>To retrieve the ACL information for a specific indexed document from a sync job, you can use the following CloudWatch Logs Insights query:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">filter @logStream like 'SYNC_RUN_HISTORY_REPORT/data-source-id/run-id'
and DocumentTitle = "your-document-title"
| display DocumentTitle, Acl</code></pre> 
</div> 
<p>The following screenshot shows an example result.</p> 
<p><img alt="" class="alignnone size-full wp-image-86524" height="398" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/13/ML-17547-Image9.png" width="936" /></p> 
<h3><strong>List metadata of an indexed document from a data source sync job</strong></h3> 
<p>To retrieve the metadata information for a specific indexed document from a sync job, you can use the following CloudWatch Logs Insights query:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">filter @logStream like 'SYNC_RUN_HISTORY_REPORT/data-source-id/run-id'
and DocumentTitle = "your-document-title"
| display DocumentTitle, Metadata</code></pre> 
</div> 
<p>The following screenshot shows an example result.</p> 
<p><strong> <img alt="" class="alignnone size-full wp-image-86524" height="398" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/13/ML-17547-Image9.png" width="936" /></strong></p> 
<h2><strong>Conclusion</strong></h2> 
<p>The newly introduced document-level report in Amazon Kendra provides enhanced visibility and observability into the document processing lifecycle during data source sync jobs. This feature addresses a critical need expressed by customers for better troubleshooting capabilities and access to detailed information about the indexing status, metadata, and ACLs of individual documents.</p> 
<p>The document-level report is stored in a log stream named <code>SYNC_RUN_HISTORY_REPORT</code> within the Amazon Kendra index CloudWatch log group. This report contains comprehensive information for each document, including the document ID, title, overall document sync status, error messages (if any), along with its ACLs and metadata information retrieved from the data sources. The data source sync run history page now includes an <strong>Actions</strong> column, providing access to the document-level report for each sync run. This feature significantly improves the ability to troubleshoot issues related to document ingestion and access control, and issues related to metadata relevance, and provides better visibility about the documents synced with an Amazon Kendra index.</p> 
<p>To get started with Amazon Kendra, explore the <a href="https://docs.aws.amazon.com/kendra/latest/dg/what-is-kendra.html" rel="noopener" target="_blank">Getting started</a> guide. To learn more about data source connectors and best practices, see <a href="https://docs.aws.amazon.com/kendra/latest/dg/data-source.html" rel="noopener" target="_blank">Creating a data source connector.</a></p> 
<hr /> 
<h3><strong>About the Authors</strong></h3> 
<p style="clear: both;"><img alt="" class="wp-image-82195 size-full alignleft" height="134" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/01/Aneesh.jpg" width="100" /><strong>Aneesh Mohan</strong> is a Senior Solutions Architect at Amazon Web Services (AWS), with over 20 years of experience in architecting and delivering high-impact solutions for mission-critical workloads. His expertise spans across the financial services industry, AI/ML, security, and data technologies. Driven by a deep passion for technology, Aneesh is dedicated to partnering with customers to design and implement well-architected, innovative solutions that address their unique business needs.</p> 
<p style="clear: both;"><img alt="" class="wp-image-82196 size-full alignleft" height="177" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/01/Ashwin.jpg" width="100" /><strong>Ashwin Shukla</strong> is a Software Development Engineer II on the Amazon Q for Business and Amazon Kendra engineering team, with 6 years of experience in developing enterprise software. In this role, he works on designing and developing foundational features for Amazon Q for Business.</p>
