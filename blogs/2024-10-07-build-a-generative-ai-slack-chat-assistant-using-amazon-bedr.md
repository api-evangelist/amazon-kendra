---
title: "Build a generative AI Slack chat assistant using Amazon Bedrock and Amazon Kendra"
url: "https://aws.amazon.com/blogs/machine-learning/build-a-generative-ai-slack-chat-assistant-using-amazon-bedrock-and-amazon-kendra/"
date: "Mon, 07 Oct 2024 20:48:43 +0000"
author: "Kruthi Jayasimha Rao"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-kendra/feed/"
---
<p>Despite the proliferation of information and data in business environments, employees and stakeholders often find themselves searching for information and struggling to get their questions answered quickly and efficiently. This can lead to productivity losses, frustration, and delays in decision-making.</p> 
<p>A <a href="https://aws.amazon.com/generative-ai/" rel="noopener" target="_blank">generative AI</a> Slack chat assistant can help address these challenges by providing a readily available, intelligent interface for users to interact with and obtain the information they need. By using the natural language processing and generation capabilities of generative AI, the chat assistant can understand user queries, retrieve relevant information from various data sources, and provide tailored, contextual responses.</p> 
<p>By harnessing the power of generative AI and <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services</a> (AWS) services <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a>, <a href="https://aws.amazon.com/kendra/" rel="noopener" target="_blank">Amazon Kendra</a>, and <a href="https://aws.amazon.com/lex/" rel="noopener" target="_blank">Amazon Lex</a>, this solution provides a sample architecture to build an intelligent Slack chat assistant that can streamline information access, enhance user experiences, and drive productivity and efficiency within organizations.</p> 
<h2>Why use Amazon Kendra for building a RAG application?</h2> 
<p>Amazon Kendra is a fully managed service that provides out-of-the-box semantic search capabilities for state-of-the-art ranking of documents and passages. You can use Amazon Kendra to <a href="https://aws.amazon.com/blogs/machine-learning/quickly-build-high-accuracy-generative-ai-applications-on-enterprise-data-using-amazon-kendra-langchain-and-large-language-models/" rel="noopener" target="_blank">quickly build high-accuracy generative AI applications on enterprise data</a> and source the most relevant content and documents to maximize the quality of your Retrieval Augmented Generation (RAG) payload, yielding better large language model (LLM) responses than using conventional or keyword-based search solutions. Amazon Kendra offers simple-to-use deep learning search models that are pre-trained on 14 domains and don’t require machine learning (ML) expertise. Amazon Kendra can index content from a wide range of sources, including databases, content management systems, file shares, and web pages.</p> 
<p>Further, the FAQ feature in Amazon Kendra complements the broader retrieval capabilities of the service, allowing the RAG system to seamlessly switch between providing prewritten FAQ responses and dynamically generating responses by querying the larger knowledge base. This makes it well-suited for powering the retrieval component of a RAG system, allowing the model to access a broad knowledge base when generating responses. By integrating the <a href="https://docs.aws.amazon.com/kendra/latest/dg/in-creating-faq.html" rel="noopener" target="_blank">FAQ capabilities of Amazon Kendra</a> into a RAG system, the model can use a curated set of high-quality, authoritative answers for commonly asked questions. This can improve the overall response quality and user experience, while also reducing the burden on the language model to generate these basic responses from scratch.</p> 
<p>This solution balances retaining customizations in terms of model selection, prompt engineering, and adding FAQs with not having to deal with word embeddings, document chunking, and other lower-level complexities typically required for RAG implementations.</p> 
<h2>Solution overview</h2> 
<p>The chat assistant is designed to assist users by answering their questions and providing information on a variety of topics. The purpose of the chat assistant is to be an internal-facing Slack tool that can help employees and stakeholders find the information they need.</p> 
<p>The architecture uses Amazon Lex for intent recognition, <a href="https://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> for processing queries, Amazon Kendra for searching through FAQs and web content, and Amazon Bedrock for generating contextual responses powered by LLMs. By combining these services, the chat assistant can understand natural language queries, retrieve relevant information from multiple data sources, and provide humanlike responses tailored to the user’s needs. The solution showcases the power of generative AI in creating intelligent virtual assistants that can streamline workflows and enhance user experiences based on model choices, FAQs, and modifying system prompts and inference parameters.</p> 
<h3>Architecture diagram</h3> 
<p>The following diagram illustrates a RAG approach where the user sends a query through the Slack application and receives a generated response based on the data indexed in Amazon Kendra. In this post, we use Amazon Kendra Web Crawler as the data source and include FAQs stored on <a href="https://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3). See <a href="https://docs.aws.amazon.com/kendra/latest/dg/data-sources.html" rel="noopener" target="_blank">Data source connectors</a> for a list of supported data source connectors for Amazon Kendra.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-arch-diag.png" rel="noopener" target="_blank"><img alt="ML-16837-arch-diag" class="alignnone size-full wp-image-88111" height="407" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-arch-diag.png" width="911" /></a></p> 
<p>The step-by-step workflow for the architecture is the following:</p> 
<ol> 
 <li>The user sends a query such as <code>What is the AWS Well-Architected Framework?</code> through the Slack app.</li> 
 <li>The query goes to Amazon Lex, which identifies the intent.</li> 
 <li>Currently two intents are configured in Amazon Lex (<code>Welcome</code> and <code>FallbackIntent</code>).</li> 
 <li>The welcome intent is configured to respond with a greeting when a user enters a greeting such as “hi” or “hello.” The assistant responds with “Hello! I can help you with queries based on the documents provided. Ask me a question.”</li> 
 <li>The fallback intent is fulfilled with a Lambda function. 
  <ol type="a"> 
   <li>The Lambda function searches Amazon Kendra FAQs through the <code>search_Kendra_FAQ</code> method by taking the user query and Amazon Kendra index ID as inputs. If there’s a match with a high confidence score, the answer from the FAQ is returned to the user. 
    <div class="hide-language"> 
     <pre><code class="lang-python">def search_Kendra_FAQ(question, kendra_index_id):
    """
    This function takes in the question from the user, and checks if the question exists in the Kendra FAQs.
    :param question: The question the user is asking that was asked via the frontend input text box.
    :param kendra_index_id: The kendra index containing the documents and FAQs
    :return: If found in FAQs, returns the answer along with any relevant links. If not, returns False and then calls kendra_retrieve_document function.
    """
    kendra_client = boto3.client('kendra')
    response = kendra_client.query(IndexId=kendra_index_id, QueryText=question, QueryResultTypeFilter='QUESTION_ANSWER')
    for item in response['ResultItems']:
        score_confidence = item['ScoreAttributes']['ScoreConfidence']
        # Taking answers from FAQs that have a very high confidence score only
        if score_confidence == 'VERY_HIGH' and len(item['AdditionalAttributes']) &gt; 1:
            text = item['AdditionalAttributes'][1]['Value']['TextWithHighlightsValue']['Text']
            url = "None"
            if item['DocumentURI'] != '':
                url = item['DocumentURI']
            return (text, url)
    return (False, False)</code></pre> 
    </div> </li> 
   <li>If there isn’t a match with a high enough confidence score, relevant documents from Amazon Kendra with a high confidence score are retrieved through the <code>kendra_retrieve_document</code> method and sent to Amazon Bedrock to generate a response as the context. 
    <div class="hide-language"> 
     <pre><code class="lang-python">def kendra_retrieve_document(question, kendra_index_id):
    """
    This function takes in the question from the user, and retrieves relevant passages based on default PageSize of 10.
    :param question: The question the user is asking that was asked via the frontend input text box.
    :param kendra_index_id: The kendra index containing the documents and FAQs
    :return: Returns the context to be sent to the LLM and document URIs to be returned as relevant data sources.
    """
    kendra_client = boto3.client('kendra')
    documents = kendra_client.retrieve(IndexId=kendra_index_id, QueryText=question)
    text = ""
    uris = set()
    if len(documents['ResultItems']) &gt; 0:
        for i in range(len(documents['ResultItems'])):
            score_confidence = documents['ResultItems'][i]['ScoreAttributes']['ScoreConfidence']
            if score_confidence == 'VERY_HIGH' or score_confidence == 'HIGH':
                text += documents['ResultItems'][i]['Content'] + "\n"
                uris.add(documents['ResultItems'][i]['DocumentURI'])
    return (text, uris)</code></pre> 
    </div> </li> 
   <li>The response is generated from Amazon Bedrock with the <code>invokeLLM</code> method. The following is a snippet of the <code>invokeLLM</code> method within the fulfillment function. Read more on <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/inference-parameters.html" rel="noopener" target="_blank">inference parameters</a> and <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-anthropic-claude-messages.html" rel="noopener" target="_blank">system prompts</a> to modify parameters that are passed into the Amazon Bedrock invoke model request. 
    <div class="hide-language"> 
     <pre><code class="lang-python">def invokeLLM(question, context, modelId):
    """
    This function takes in the question from the user, along with the Kendra responses as context to generate an answer
    for the user on the frontend.
    :param question: The question the user is asking that was asked via the frontend input text box.
    :param documents: The response from the Kendra document retrieve query, used as context to generate a better
    answer.
    :return: Returns the final answer that will be provided to the end-user of the application who asked the original
    question.
    """
    # Setup Bedrock client
    bedrock = boto3.client('bedrock-runtime')
    # configure model specifics such as specific model
    modelId = modelId

    # body of data with parameters that is passed into the bedrock invoke model request
    body = json.dumps({"max_tokens": 350,
            "system": "You are a truthful AI assistant. Your goal is to provide informative and substantive responses to queries based on the documents provided. If you do not know the answer to a question, you truthfully say you do not know.",
            "messages": [{"role": "user", "content": "Answer this user query:" + question + "with the following context:" + context}],
            "anthropic_version": "bedrock-2023-05-31",
                "temperature":0,
            "top_k":250,
            "top_p":0.999})

    # Invoking the bedrock model with your specifications
    response = bedrock.invoke_model(body=body,
                                    modelId=modelId)
    # the body of the response that was generated
    response_body = json.loads(response.get('body').read())
    # retrieving the specific completion field, where you answer will be
    answer = response_body.get('content')
    # returning the answer as a final result, which ultimately gets returned to the end user
    return answer</code></pre> 
    </div> </li> 
   <li>Finally, the response generated from Amazon Bedrock along with the relevant referenced URLs are returned to the end user.</li> 
  </ol> <p>When selecting websites to index, adhere to the <a href="https://aws.amazon.com/aup/" rel="noopener" target="_blank">AWS Acceptable Use Policy</a> and other AWS terms. Remember that you can only use Amazon Kendra Web Crawler to index your own web pages or web pages that you have authorization to index. Visit the <a href="https://docs.aws.amazon.com/kendra/latest/dg/data-source-web-crawler.html" rel="noopener" target="_blank">Amazon Kendra Web Crawler</a> data source guide to learn more about using the web crawler as a data source. Using Amazon Kendra Web Crawler to aggressively crawl websites or web pages you don’t own is <strong>not</strong> considered acceptable use.</p> <h3>Supported features</h3> <p>The chat assistant supports the following features:</p> 
  <ol> 
   <li>Support for the following Anthropic’s models on Amazon Bedrock: 
    <ul> 
     <li><code>claude-v2</code></li> 
     <li><code>claude-3-haiku-20240307-v1:0</code></li> 
     <li><code>claude-instant-v1</code></li> 
     <li><code>claude-3-sonnet-20240229-v1:0</code></li> 
    </ul> </li> 
   <li>Support for FAQs and the Amazon Kendra Web Crawler data source</li> 
   <li>Returns FAQ answers only if the confidence score is <code>VERY_HIGH</code></li> 
   <li>Retrieves only documents from Amazon Kendra that have a <code>HIGH</code> or <code>VERY_HIGH</code> confidence score</li> 
   <li>If documents with a high confidence score aren’t found, the chat assistant returns “No relevant documents found”</li> 
  </ol> <h2>Prerequisites</h2> <p>To perform the solution, you need to have following prerequisites:</p> 
  <ul> 
   <li>Basic knowledge of AWS</li> 
   <li>An <a href="https://signin.aws.amazon.com/signin?redirect_uri=https://portal.aws.amazon.com/billing/signup/resume&amp;client_id=signup" rel="noopener" target="_blank">AWS account</a> with access to Amazon S3 and Amazon Kendra</li> 
   <li>An S3 bucket to store your documents. For more information, see <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html#creating-bucket" rel="noopener" target="_blank">Step 1: Create your first S3 bucket</a> and the <a href="https://docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html" rel="noopener" target="_blank">Amazon S3 User Guide</a>.</li> 
   <li>A Slack workspace to integrate the chat assistant</li> 
   <li>Permission to install Slack apps in your Slack workspace</li> 
   <li>Seed URLs for the Amazon Kendra Web Crawler data source 
    <ul> 
     <li>You’ll need authorization to crawl and index any websites provided</li> 
    </ul> </li> 
   <li><a href="https://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> for deploying the solution resources</li> 
  </ul> <h2>Build a generative AI Slack chat assistant</h2> <p>To build a Slack application, use the following steps:</p> 
  <ol> 
   <li><a href="https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html" rel="noopener" target="_blank">Request model access on Amazon Bedrock</a> for all Anthropic models</li> 
   <li><a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html" rel="noopener" target="_blank">Create an S3 bucket</a> in the <code>us-east-1</code> (N. Virginia) AWS Region.</li> 
   <li><a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/uploading-an-object-bucket.html" rel="noopener" target="_blank">Upload</a> the <a href="https://aws-blogs-artifacts-public.s3.amazonaws.com/ML-16837/AIBot-LexJson.zip" rel="noopener" target="_blank">AIBot-LexJson.zip</a> and <a href="https://aws-blogs-artifacts-public.s3.amazonaws.com/ML-16837/SampleFAQ.csv" rel="noopener" target="_blank">SampleFAQ.csv</a> files to the S3 bucket</li> 
   <li>Launch the CloudFormation stack in the <code>us-east-1</code> (N. Virginia) AWS Region.<a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=slack-chat-assistant-demo&amp;templateURL=https://aws-blogs-artifacts-public.s3.amazonaws.com/ML-16837/aibot.yaml" rel="noopener" target="_blank"><img alt="Launch Stack to create solution resources" class="alignnone wp-image-84212 size-full" height="27" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/20/ML-16991-cloudformation-launch-stack-producttableandapi.png" width="144" /></a></li> 
   <li>Enter a <strong>Stack name</strong> of your choice</li> 
   <li>For <strong>S3BucketName</strong>, enter the name of the S3 bucket created in Step 2</li> 
   <li>For <strong>S3KendraFAQKey</strong>, enter the name of the <code>SampleFAQs</code> uploaded to the S3 bucket in Step 3</li> 
   <li>For <strong>S3LexBotKey</strong>, enter the name of the Amazon Lex .zip file uploaded to the S3 bucket in Step 3</li> 
   <li>For <strong>SeedUrls</strong>, enter up to 10 URLs for the web crawler as a comma delimited list. In the example in this post, we give the publicly available Amazon Bedrock service page as the seed URL</li> 
   <li>Leave the rest as defaults. Choose <strong>Next</strong>. Choose <strong>Next</strong> again on the <strong>Configure stack options</strong></li> 
   <li>Acknowledge by selecting the box and choose <strong>Submit</strong>, as shown in the following screenshot<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-cfn-checkbox.jpg" rel="noopener" target="_blank"><img alt="ML-16837-cfn-checkbox" class="alignnone size-full wp-image-88112" height="303" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-cfn-checkbox.jpg" style="margin: 10px 0px 10px 0px;" width="1029" /></a></li> 
   <li>Wait for the stack creation to complete</li> 
   <li>Verify all resources are created</li> 
   <li>Test on the AWS Management Console for Amazon Lex 
    <ol type="a"> 
     <li>On the Amazon Lex console, choose your chat assistant <code>${<em>YourStackName</em>}-AIBot</code></li> 
     <li>Choose <strong>Intents</strong></li> 
     <li>Choose <strong>Version 1</strong> and choose <strong>Test</strong>, as shown in the following screenshot<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-lex-version1.jpg" rel="noopener" target="_blank"><img alt="ML-16837-lex-version1" class="alignnone size-full wp-image-88113" height="477" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-lex-version1.jpg" style="margin: 10px 0px 10px 0px;" width="1289" /></a></li> 
     <li>Select the <strong>AIBotProdAlias</strong> and choose <strong>Confirm</strong>, as shown in the following screenshot. If you want to make changes to the chat assistant, you can use the draft version, publish a new version, and assign the new version to the <code>AIBotProdAlias</code>. Learn more about <a href="https://docs.aws.amazon.com/lex/latest/dg/versioning-aliases.html" rel="noopener" target="_blank">Versioning and Aliases</a>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-aibotprodalias.jpg" rel="noopener" target="_blank"><img alt="" class="wp-image-88312 size-full alignnone" height="318" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/10/01/ML-16837-aibotprodalias.jpg" style="margin: 10px 0px 10px 0px;" width="500" /></a></li> 
     <li>Test the chat assistant with questions such as, “Which AWS service has 11 nines of durability?” and “What is the AWS Well-Architected Framework?” and verify the responses. The following table shows that there are three FAQs in the sample .csv file.<br /> 
      <table border="1px" cellpadding="10px"> 
       <thead> 
        <tr style="background-color: #000000;"> 
         <td width="106"><span style="color: #ffffff;"><strong>_question</strong></span></td> 
         <td width="239"><span style="color: #ffffff;"><strong>_answer</strong></span></td> 
         <td width="279"><span style="color: #ffffff;"><strong>_source_uri</strong></span></td> 
        </tr> 
       </thead> 
       <tbody> 
        <tr> 
         <td width="106">Which AWS service has 11 nines of durability?</td> 
         <td width="239">Amazon S3</td> 
         <td width="279">https://aws.amazon.com/s3/</td> 
        </tr> 
        <tr> 
         <td width="106">What is the AWS Well-Architected Framework?</td> 
         <td width="239">The AWS Well-Architected Framework enables customers and partners to review their architectures using a consistent approach and provides guidance to improve designs over time.</td> 
         <td width="279">https://aws.amazon.com/architecture/well-architected/</td> 
        </tr> 
        <tr> 
         <td width="106">In what Regions is Amazon Kendra available?</td> 
         <td width="239">Amazon Kendra is currently available in the following AWS Regions: Northern Virginia, Oregon, and Ireland</td> 
         <td width="279">https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/</td> 
        </tr> 
       </tbody> 
      </table> </li> 
     <li>The following screenshot shows the question <code>“Which AWS service has 11 nines of durability?”</code> and its response. You can observe that the response is the same as in the FAQ file and includes a link.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q1inLex.png" rel="noopener" target="_blank"><img alt="ML-16837-Q1inLex" class="alignnone size-full wp-image-88114" height="1320" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q1inLex.png" style="margin: 10px 0px 10px 0px;" width="2774" /></a></li> 
     <li>Based on the pages you have crawled, ask a question in the chat. For this example, the publicly available Amazon Bedrock page was crawled and indexed. The following screenshot shows the question, <code>“What are agents in Amazon Bedrock?”</code> and and a generated response that includes relevant links.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q2inLex.png" rel="noopener" target="_blank"><img alt="ML-16837-Q2inLex" class="alignnone size-full wp-image-88116" height="838" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q2inLex.png" style="margin: 10px 0px 10px 0px;" width="1864" /></a></li> 
    </ol> </li> 
  </ol> 
  <ol start="15"> 
   <li>For integration of the Amazon Lex chat assistant with Slack, see <a href="https://docs.aws.amazon.com/lexv2/latest/dg/deploy-slack.html" rel="noopener" target="_blank">Integrating an Amazon Lex V2 bot with Slack</a>. Choose the <strong>AIBotProdAlias </strong>under <strong>Alias</strong> in the <strong>Channel Integrations</strong></li> 
  </ol> <h2>Run sample queries to test the solution</h2> 
  <ol> 
   <li>In Slack, go to the <strong>Apps</strong> section. In the dropdown menu, choose <strong>Manage</strong> and select <strong>Browse apps</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-slackBrowseApps.jpg" rel="noopener" target="_blank"><img alt="ML-16837-slackBrowseApps" class="alignnone size-full wp-image-88120" height="364" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-slackBrowseApps.jpg" style="margin: 10px 0px 10px 0px;" width="1108" /></a></li> 
   <li>Search for <code>${AIBot}</code> in <strong>App Directory</strong> and choose the chat assistant. This will add the chat assistant to the Apps section in Slack. You can now start asking questions in the chat. The following screenshot shows the question <code>“Which AWS service has 11 nines of durability?”</code> and its response. You can observe that the response is the same as in the FAQ file and includes a link.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q1slack.png" rel="noopener" target="_blank"><img alt="ML-16837-Q1slack" class="alignnone size-full wp-image-88115" height="262" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q1slack.png" style="margin: 10px 0px 10px 0px;" width="983" /></a></li> 
   <li>The following screenshot shows the question, <code>“What is the AWS Well-Architected Framework?”</code> and its response.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q2slack.png" rel="noopener" target="_blank"><img alt="ML-16837-Q2slack" class="alignnone size-full wp-image-88117" height="255" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q2slack.png" style="margin: 10px 0px 10px 0px;" width="1276" /></a></li> 
   <li>Based on the pages you have crawled, ask a question in the chat. For this example, the publicly available Amazon Bedrock page was crawled and indexed. The following screenshot shows the question, <code>“What are agents in Amazon Bedrock?”</code> and and a generated response that includes relevant links.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q3slack.jpg" rel="noopener" target="_blank"><img alt="ML-16837-Q3slack" class="alignnone size-full wp-image-88118" height="736" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q3slack.jpg" style="margin: 10px 0px 10px 0px;" width="1289" /></a></li> 
   <li>The following screenshot shows the question, <code>“What is amazon polly?”</code> Because there is no Amazon Polly documentation indexed, the chat assistant responds with “No relevant documents found,” as expected.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q4slack.jpg" rel="noopener" target="_blank"><img alt="ML-16837-Q4slack" class="alignnone size-full wp-image-88119" height="176" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/ML-16837-Q4slack.jpg" style="margin: 10px 0px 10px 0px;" width="1726" /></a></li> 
  </ol> <p>These examples show how the chat assistant retrieves documents from Amazon Kendra and provides answers based on the documents retrieved. If no relevant documents are found, the chat assistant responds with “No relevant documents found.”</p> <h2>Clean up</h2> <p>To clean up the resources created by this solution:</p> 
  <ol> 
   <li>Delete the CloudFormation stack by navigating to the CloudFormation console</li> 
   <li>Select the stack you created for this solution and choose <strong>Delete</strong></li> 
   <li>Confirm the deletion by entering the stack name in the provided field. This will remove all the resources created by the CloudFormation template, including the Amazon Kendra index, Amazon Lex chat assistant, Lambda function, and other related resources.</li> 
  </ol> <h2>Conclusion</h2> <p>This post describes the development of a generative AI Slack application powered by Amazon Bedrock and Amazon Kendra. This is designed to be an internal-facing Slack chat assistant that helps answer questions related to the indexed content. The solution architecture includes Amazon Lex for intent identification, a Lambda function for fulfilling the fallback intent, Amazon Kendra for FAQ searches and indexing crawled web pages, and Amazon Bedrock for generating responses. The post walks through the deployment of the solution using a CloudFormation template, provides instructions for running sample queries, and discusses the steps for cleaning up the resources. Overall, this post demonstrates how to use various AWS services to build a powerful generative AI–powered chat assistant application.</p> <p>This solution demonstrates the power of generative AI in building intelligent chat assistants and search assistants. Explore the generative AI Slack chat assistant: Invite your teams to a Slack workspace and start getting answers to your indexed content and FAQs. Experiment with different use cases and see how you can harness the capabilities of services like Amazon Bedrock and Amazon Kendra to enhance your business operations. For more information about using Amazon Bedrock with Slack, refer to <a href="https://aws.amazon.com/blogs/machine-learning/deploy-a-slack-gateway-for-amazon-bedrock/" rel="noopener" target="_blank">Deploy a Slack gateway for Amazon Bedrock</a>.</p> 
  <hr /> <h3>About the authors</h3> <p style="clear: both;"><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/02/09/ml-15900-image037-kruthira.jpg" rel="noopener" target="_blank"><img alt="" class="size-full wp-image-70951 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/02/09/ml-15900-image037-kruthira.jpg" width="100" /></a><strong>Kruthi Jayasimha Rao</strong> is a Partner Solutions Architect with a focus on AI and ML. She provides technical guidance to AWS Partners in following best practices to build secure, resilient, and highly available solutions in the AWS Cloud.</p> <p style="clear: both;"><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/mhamuzn.jpg" rel="noopener" target="_blank"><img alt="" class="size-full wp-image-88125 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/30/mhamuzn.jpg" width="100" /></a><strong>Mohamed Mohamud</strong> is a Partner Solutions Architect with a focus on Data Analytics. He specializes in streaming analytics, helping partners build real-time data pipelines and analytics solutions on AWS. With expertise in services like Amazon Kinesis, Amazon MSK, and Amazon EMR, Mohamed enables data-driven decision-making through streaming analytics.</p> </li> 
</ol>
