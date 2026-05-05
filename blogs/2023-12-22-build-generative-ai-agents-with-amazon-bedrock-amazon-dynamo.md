---
title: "Build generative AI agents with Amazon Bedrock, Amazon DynamoDB, Amazon Kendra, Amazon Lex, and LangChain"
url: "https://aws.amazon.com/blogs/machine-learning/build-generative-ai-agents-with-amazon-bedrock-amazon-dynamodb-amazon-kendra-amazon-lex-and-langchain/"
date: "Fri, 22 Dec 2023 16:38:29 +0000"
author: "Kyle Blocksom"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-kendra/feed/"
---
<p><a href="https://aws.amazon.com/generative-ai/" rel="noopener" target="_blank">Generative AI</a> agents are capable of producing human-like responses and engaging in natural language conversations by orchestrating a chain of calls to foundation models (FMs) and other augmenting tools based on user input. Instead of only fulfilling predefined intents through a static decision tree, agents are autonomous within the context of their suite of available tools. <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> is a fully managed service that makes leading FMs from AI companies available through an API along with developer tooling to help build and scale generative AI applications.</p> 
<p>In this post, we demonstrate how to build a generative AI financial services agent powered by Amazon Bedrock. The agent can assist users with finding their account information, completing a loan application, or answering natural language questions while also citing sources for the provided answers. This solution is intended to act as a launchpad for developers to create their own personalized conversational agents for various applications, such as virtual workers and customer support systems. Solution code and deployment assets can be found in the <a href="https://github.com/aws-samples/generative-ai-amazon-bedrock-langchain-agent-example" rel="noopener" target="_blank">GitHub repository</a>.</p> 
<p><a href="https://aws.amazon.com/lex/" rel="noopener" target="_blank">Amazon Lex</a> supplies the natural language understanding (NLU) and natural language processing (NLP) interface for the open source <a href="https://github.com/langchain-ai/langchain/blob/master/libs/langchain/langchain/agents/conversational/base.py" rel="noopener" target="_blank">LangChain conversational agent</a> embedded within an <a href="https://aws.amazon.com/amplify/" rel="noopener" target="_blank">AWS Amplify</a> website. The agent is equipped with tools that include an Anthropic Claude 2.1 FM hosted on Amazon Bedrock and synthetic customer data stored on <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a> and <a href="https://aws.amazon.com/kendra/" rel="noopener" target="_blank">Amazon Kendra</a> to deliver the following capabilities:</p> 
<ul> 
 <li><strong>Provide personalized responses</strong> – Query DynamoDB for customer account information, such as mortgage summary details, due balance, and next payment date</li> 
 <li><strong>Access general knowledge </strong>– Harness the agent’s reasoning logic in tandem with the vast amounts of data used to pre-train the different FMs provided through Amazon Bedrock to produce replies for any customer prompt</li> 
 <li><strong>Curate opinionated answers </strong>– Inform agent responses using an Amazon Kendra index configured with authoritative data sources: customer documents stored in <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) and <a href="https://docs.aws.amazon.com/kendra/latest/dg/data-source-web-crawler.html" rel="noopener" target="_blank">Amazon Kendra Web Crawler</a> configured for the customer’s website</li> 
</ul> 
<h2>Solution overview</h2> 
<h3>Demo recording</h3> 
<p>The following demo recording highlights agent functionality and technical implementation details.</p> 
<p></p> 
<h3>Solution architecture</h3> 
<p>The following diagram illustrates the solution architecture.</p> 
<div class="wp-caption alignnone" id="attachment_68747" style="width: 2749px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-solution-overview.png"><img alt="Solution Architecture Overview" class="size-full wp-image-68747" height="1674" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-solution-overview.png" style="margin: 10px 0px 10px 0px;" width="2739" /></a>
 <p class="wp-caption-text" id="caption-attachment-68747">Diagram 1: Solution Architecture Overview</p>
</div> 
<p>The agent’s response workflow includes the following steps:</p> 
<ol> 
 <li>Users perform natural language dialog with the agent through their choice of web, SMS, or voice channels. The web channel includes an Amplify hosted website with an Amazon Lex embedded chatbot for a fictitious customer. SMS and voice channels can be optionally configured using <a href="https://aws.amazon.com/connect/" rel="noopener" target="_blank">Amazon Connect</a> and <a href="https://docs.aws.amazon.com/lexv2/latest/dg/deploying-messaging-platform.html" rel="noopener" target="_blank">messaging integrations</a> for Amazon Lex. Each user request is processed by Amazon Lex to determine user intent through a process called intent recognition, which involves analyzing and interpreting the user’s input (text or speech) to understand the user’s intended action or purpose.</li> 
 <li>Amazon Lex then invokes an <a href="http://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> handler for user intent fulfillment. The Lambda function associated with the Amazon Lex chatbot contains the logic and business rules required to process the user’s intent. Lambda performs specific actions or retrieves information based on the user’s input, making decisions and generating appropriate responses.</li> 
 <li>Lambda instruments the financial services agent logic as a LangChain conversational agent that can access customer-specific data stored on DynamoDB, curate opinionated responses using your documents and webpages indexed by Amazon Kendra, and provide general knowledge answers through the FM on Amazon Bedrock. Responses generated by Amazon Kendra include source attribution, demonstrating how you can provide additional contextual information to the agent through <a href="https://aws.amazon.com/what-is/retrieval-augmented-generation/" rel="noopener" target="_blank">Retrieval Augmented Generation</a> (RAG). RAG allows you to enhance your agent’s ability to generate more accurate and contextually relevant responses using your own data.</li> 
</ol> 
<h3>Agent architecture</h3> 
<p>The following diagram illustrates the agent architecture.</p> 
<div class="wp-caption alignnone" id="attachment_68759" style="width: 2564px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-agent.png"><img alt="LangChain Conversational Agent Architecture" class="wp-image-68759 size-full" height="1086" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-agent.png" style="margin: 10px 0px 10px 0px;" width="2554" /></a>
 <p class="wp-caption-text" id="caption-attachment-68759">Diagram 2: LangChain Conversational Agent Architecture</p>
</div> 
<p>The agent’s reasoning workflow includes the following steps:</p> 
<ol> 
 <li>The LangChain conversational agent incorporates conversation memory so it can respond to multiple queries with contextual generation. This memory allows the agent to provide responses that take into account the context of the ongoing conversation. This is achieved through contextual generation, where the agent generates responses that are relevant and contextually appropriate based on the information it has remembered from the conversation. In simpler terms, the agent remembers what was said earlier and uses that information to respond to multiple questions in a way that makes sense in the ongoing discussion. Our agent uses <a href="https://python.langchain.com/docs/modules/memory/integrations/dynamodb_chat_message_history" rel="noopener" target="_blank">LangChain’s DynamoDB chat message history class</a> as a conversation memory buffer so it can recall past interactions and enhance the user experience with more meaningful, context-aware responses.</li> 
 <li>The agent uses Anthropic Claude 2.1 on Amazon Bedrock to complete the desired task through a series of carefully self-generated text inputs known as <em>prompts</em>. The primary objective of prompt engineering is to elicit specific and accurate responses from the FM. Different prompt engineering techniques include: 
  <ul> 
   <li><strong>Zero-shot</strong> – A single question is presented to the model without any additional clues. The model is expected to generate a response based solely on the given question.</li> 
   <li><strong>Few-shot </strong>– A set of sample questions and their corresponding answers are included before the actual question. By exposing the model to these examples, it learns to respond in a similar manner.</li> 
   <li><strong>Chain-of-thought </strong>– A specific style of few-shot prompting where the prompt is designed to contain a series of intermediate reasoning steps, guiding the model through a logical thought process, ultimately leading to the desired answer.</li> 
  </ul> <p>Our agent utilizes chain-of-thought reasoning by running a set of actions upon receiving a request. Following each action, the agent enters the observation step, where it expresses a thought. If a final answer is not yet achieved, the agent iterates, selecting different actions to progress towards reaching the final answer. See the following example code:</p></li> 
</ol> 
<div style="background-color: #f2f2f2; padding: 10px; padding-left: 40px; margin-left: 20px; padding-bottom: 5px;">
 Thought: Do I need to use a tool? Yes
 <br /> Action: The action to take
 <br /> Action Input: The input to the action
 <br /> Observation: The result of the action
</div> 
<div style="background-color: #f2f2f2; padding: 10px; padding-left: 40px; margin-left: 20px; padding-bottom: 5px;">
 Thought: Do I need to use a tool? No
 <br /> FSI Agent: [answer and source documents]
</div> 
<ol start="3"> 
 <li>As part of the agent’s different reasoning paths and self-evaluating choices to decide the next course of action, it has the ability to access synthetic customer data sources through an <a href="https://python.langchain.com/docs/modules/data_connection/retrievers/integrations/amazon_kendra_retriever" rel="noopener" target="_blank">Amazon Kendra Index Retriever tool</a>. Using Amazon Kendra, the agent performs contextual search across a wide range of content types, including documents, FAQs, knowledge bases, manuals, and websites. For more details on supported data sources, refer to <a href="https://docs.aws.amazon.com/kendra/latest/dg/hiw-data-source.html" rel="noopener" target="_blank">Data sources</a>. The agent has the power to use this tool to provide opinionated responses to user prompts that should be answered using an authoritative, customer-provided knowledge library, instead of the more general knowledge corpus used to pretrain the Amazon Bedrock FM.</li> 
</ol> 
<h2>Deployment guide</h2> 
<p>In the following sections, we discuss the key steps to deploy the solution, including pre-deployment and post-deployment.</p> 
<h2>Pre-deployment</h2> 
<p>Before you deploy the solution, you need to create your own forked version of the solution repository with a token-secured webhook to automate continuous deployment of your Amplify website. The Amplify configuration points to a GitHub source repository from which our website’s frontend is built.</p> 
<h3>Fork and clone <a href="https://github.com/aws-samples/generative-ai-amazon-bedrock-langchain-agent-example.git" rel="noopener" target="_blank">generative-ai-amazon-bedrock-langchain-agent-example</a> repository</h3> 
<ol> 
 <li>To control the source code that builds your Amplify website, follow the instructions in <a href="https://docs.github.com/en/get-started/quickstart/fork-a-repo?tool=webui&amp;platform=mac" rel="noopener" target="_blank">Fork a repository</a> to fork the generative-ai-amazon-bedrock-langchain-agent-example repository. This creates a copy of the repository that is disconnected from the original code base, so you can make the appropriate modifications.</li> 
 <li>Please note of your forked repository URL to use to clone the repository in the next step and to configure the GITHUB_PAT environment variable used in the solution deployment automation script.</li> 
 <li>Clone your forked repository using the git clone command: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">git clone &lt;YOUR-FORKED-REPOSITORY-URL&gt;</code></pre> 
  </div> </li> 
</ol> 
<h3>Create a GitHub personal access token</h3> 
<p>The Amplify hosted website uses a <a href="https://docs.github.com/en/enterprise-server@3.6/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens" rel="noopener" target="_blank">GitHub personal access token (PAT)</a> as the OAuth token for third-party source control. The OAuth token is used to create a webhook and a read-only deploy key using SSH cloning.</p> 
<ol> 
 <li>To create your PAT, follow the instructions in <a href="https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic" rel="noopener" target="_blank">Creating a personal access token (classic)</a>. You may prefer to use a <a href="https://docs.github.com/en/apps/creating-github-apps/creating-github-apps/about-apps" rel="noopener" target="_blank">GitHub app</a> to access resources on behalf of an organization or for long-lived integrations.</li> 
 <li>Take note of your PAT before closing your browser—you will use it to configure the GITHUB_PAT environment variable used in the solution deployment automation script. The script will publish your PAT to <a href="https://aws.amazon.com/secrets-manager/" rel="noopener" target="_blank">AWS Secrets Manager</a> using <a href="http://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface</a> (AWS CLI) commands and the secret name will be used as the GitHubTokenSecretName <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html" rel="noopener" target="_blank">AWS CloudFormation</a> parameter.</li> 
</ol> 
<h2>Deployment</h2> 
<p>The solution deployment automation script uses the parameterized CloudFormation template, <a href="https://github.com/aws-samples/generative-ai-amazon-bedrock-langchain-agent-example/blob/main/cfn/GenAI-FSI-Agent.yml" rel="noopener" target="_blank">GenAI-FSI-Agent.yml</a>, to automate provisioning of following solution resources:</p> 
<ul> 
 <li>An Amplify website to simulate your front-end environment.</li> 
 <li>An Amazon Lex bot configured through a bot import deployment package.</li> 
 <li>Four DynamoDB tables: 
  <ul> 
   <li><strong>UserPendingAccountsTable</strong> – Records pending transactions (for example, loan applications).</li> 
   <li><strong>UserExistingAccountsTable</strong> – Contains user account information (for example, mortgage account summary).</li> 
   <li><strong>ConversationIndexTable</strong> – Tracks the conversation state.</li> 
   <li><strong>ConversationTable</strong> – Stores conversation history.</li> 
  </ul> </li> 
 <li>An S3 bucket that contains the Lambda agent handler, Lambda data loader, and Amazon Lex deployment packages, along with customer FAQ and mortgage application example documents.</li> 
 <li>Two Lambda functions: 
  <ul> 
   <li><strong>Agent handler</strong> – Contains the LangChain conversational agent logic that can intelligently employ a variety of tools based on user input.</li> 
   <li><strong>Data loader</strong> – Loads example customer account data into UserExistingAccountsTable and is invoked as a custom CloudFormation resource during stack creation.</li> 
  </ul> </li> 
 <li>A Lambda layer for Amazon Bedrock Boto3, LangChain, and pdfrw libraries. The layer supplies LangChain’s FM library with an Amazon Bedrock model as the underlying FM and provides pdfrw as an open source PDF library for creating and modifying PDF files.</li> 
 <li>An Amazon Kendra index that provides a searchable index of customer authoritative information, including documents, FAQs, knowledge bases, manuals, websites, and more.</li> 
 <li>Two Amazon Kendra data sources: 
  <ul> 
   <li><strong>Amazon S3</strong> – Hosts an <a href="https://github.com/aws-samples/generative-ai-amazon-bedrock-langchain-agent-example/blob/main/agent/assets/AnyCompany-FAQs.csv" rel="noopener" target="_blank">example customer FAQ document</a>.</li> 
   <li><strong>Amazon Kendra Web Crawler</strong> – Configured with a root domain that emulates the customer-specific website (for example, &lt;your-company&gt;.com).</li> 
  </ul> </li> 
 <li><a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM) permissions for the preceding resources.</li> 
</ul> 
<p>AWS CloudFormation prepopulates stack parameters with the default values provided in the template. To provide alternative input values, you can specify parameters as environment variables that are referenced in the `ParameterKey=&lt;ParameterKey&gt;,ParameterValue=&lt;Value&gt;` pairs in the following shell script’s `aws cloudformation create-stack` command.</p> 
<ol> 
 <li>Before you run the shell script, navigate to your forked version of the generative-ai-amazon-bedrock-langchain-agent-example repository as your working directory and modify the shell script permissions to executable: 
  <div class="hide-language"> 
   <pre><code class="lang-bash"># If not already forked, fork the remote repository (https://github.com/aws-samples/generative-ai-amazon-bedrock-langchain-agent-example) and change working directory to shell folder:
cd generative-ai-amazon-bedrock-langchain-agent-example/shell/
chmod u+x create-stack.sh</code></pre> 
  </div> </li> 
 <li>Set your Amplify repository and GitHub PAT environment variables created during the pre-deployment steps: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">export AMPLIFY_REPOSITORY=&lt;YOUR-FORKED-REPOSITORY-URL&gt; # Forked repository URL from Pre-Deployment (Exclude '.git' from repository URL)
export GITHUB_PAT=&lt;YOUR-GITHUB-PAT&gt; # GitHub PAT copied from Pre-Deployment
export STACK_NAME=&lt;YOUR-STACK-NAME&gt; # Stack name must be lower case for S3 bucket naming convention
export KENDRA_WEBCRAWLER_URL=&lt;YOUR-WEBSITE-ROOT-DOMAIN&gt; # Public or internal HTTPS website for Kendra to index via Web Crawler (e.g., https://www.&lt;your-company&gt;.com) - Please see https://docs.aws.amazon.com/kendra/latest/dg/data-source-web-crawler.html</code></pre> 
  </div> </li> 
 <li>Finally, run the solution deployment automation script to deploy the solution’s resources, including the <a href="https://github.com/aws-samples/generative-ai-amazon-bedrock-langchain-agent-example/blob/main/cfn/GenAI-FSI-Agent.yml" rel="noopener" target="_blank">GenAI-FSI-Agent.yml</a> CloudFormation stack: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">source ./create-stack.sh</code></pre> 
  </div> </li> 
</ol> 
<h3>Solution Deployment Automation Script</h3> 
<p>The preceding <code>source ./create-stack.sh shell</code> command runs the following AWS CLI commands to deploy the solution stack:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">export ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export S3_ARTIFACT_BUCKET_NAME=$STACK_NAME-$ACCOUNT_ID
export DATA_LOADER_S3_KEY="agent/lambda/data-loader/loader_deployment_package.zip"
export LAMBDA_HANDLER_S3_KEY="agent/lambda/agent-handler/agent_deployment_package.zip"
export LEX_BOT_S3_KEY="agent/bot/lex.zip"

aws s3 mb s3://${S3_ARTIFACT_BUCKET_NAME} --region us-east-1
aws s3 cp ../agent/ s3://${S3_ARTIFACT_BUCKET_NAME}/agent/ --recursive --exclude ".DS_Store"

export BEDROCK_LANGCHAIN_LAYER_ARN=$(aws lambda publish-layer-version \
    --layer-name bedrock-langchain-pdfrw \
    --description "Bedrock LangChain pdfrw layer" \
    --license-info "MIT" \
    --content S3Bucket=${S3_ARTIFACT_BUCKET_NAME},S3Key=agent/lambda-layers/bedrock-langchain-pdfrw.zip \
    --compatible-runtimes python3.11 \
    --query LayerVersionArn --output text)

export GITHUB_TOKEN_SECRET_NAME=$(aws secretsmanager create-secret --name $STACK_NAME-git-pat \
--secret-string $GITHUB_PAT --query Name --output text)

aws cloudformation create-stack \
--stack-name ${STACK_NAME} \
--template-body file://../cfn/GenAI-FSI-Agent.yml \
--parameters \
ParameterKey=S3ArtifactBucket,ParameterValue=${S3_ARTIFACT_BUCKET_NAME} \
ParameterKey=DataLoaderS3Key,ParameterValue=${DATA_LOADER_S3_KEY} \
ParameterKey=LambdaHandlerS3Key,ParameterValue=${LAMBDA_HANDLER_S3_KEY} \
ParameterKey=LexBotS3Key,ParameterValue=${LEX_BOT_S3_KEY} \
ParameterKey=GitHubTokenSecretName,ParameterValue=${GITHUB_TOKEN_SECRET_NAME} \
ParameterKey=KendraWebCrawlerUrl,ParameterValue=${KENDRA_WEBCRAWLER_URL} \
ParameterKey=BedrockLangChainPyPDFLayerArn,ParameterValue=${BEDROCK_LANGCHAIN_LAYER_ARN} \
ParameterKey=AmplifyRepository,ParameterValue=${AMPLIFY_REPOSITORY} \
--capabilities CAPABILITY_NAMED_IAM

aws cloudformation describe-stacks --stack-name $STACK_NAME --query "Stacks[0].StackStatus"
aws cloudformation wait stack-create-complete --stack-name $STACK_NAME

export LEX_BOT_ID=$(aws cloudformation describe-stacks \
    --stack-name $STACK_NAME \
    --query 'Stacks[0].Outputs[?OutputKey==`LexBotID`].OutputValue' --output text)

export LAMBDA_ARN=$(aws cloudformation describe-stacks \
    --stack-name $STACK_NAME \
    --query 'Stacks[0].Outputs[?OutputKey==`LambdaARN`].OutputValue' --output text)

aws lexv2-models update-bot-alias --bot-alias-id 'TSTALIASID' --bot-alias-name 'TestBotAlias' --bot-id $LEX_BOT_ID --bot-version 'DRAFT' --bot-alias-locale-settings "{\"en_US\":{\"enabled\":true,\"codeHookSpecification\":{\"lambdaCodeHook\":{\"codeHookInterfaceVersion\":\"1.0\",\"lambdaARN\":\"${LAMBDA_ARN}\"}}}}"

aws lexv2-models build-bot-locale --bot-id $LEX_BOT_ID --bot-version "DRAFT" --locale-id "en_US"

export KENDRA_INDEX_ID=$(aws cloudformation describe-stacks \
    --stack-name $STACK_NAME \
    --query 'Stacks[0].Outputs[?OutputKey==`KendraIndexID`].OutputValue' --output text)

export KENDRA_S3_DATA_SOURCE_ID=$(aws cloudformation describe-stacks \
    --stack-name $STACK_NAME \
    --query 'Stacks[0].Outputs[?OutputKey==`KendraS3DataSourceID`].OutputValue' --output text)

export KENDRA_WEBCRAWLER_DATA_SOURCE_ID=$(aws cloudformation describe-stacks \
    --stack-name $STACK_NAME \
    --query 'Stacks[0].Outputs[?OutputKey==`KendraWebCrawlerDataSourceID`].OutputValue' --output text)

aws kendra start-data-source-sync-job --id $KENDRA_S3_DATA_SOURCE_ID --index-id $KENDRA_INDEX_ID

aws kendra start-data-source-sync-job --id $KENDRA_WEBCRAWLER_DATA_SOURCE_ID --index-id $KENDRA_INDEX_ID

export AMPLIFY_APP_ID=$(aws cloudformation describe-stacks \
    --stack-name $STACK_NAME \
    --query 'Stacks[0].Outputs[?OutputKey==`AmplifyAppID`].OutputValue' --output text)

export AMPLIFY_BRANCH=$(aws cloudformation describe-stacks \
    --stack-name $STACK_NAME \
    --query 'Stacks[0].Outputs[?OutputKey==`AmplifyBranch`].OutputValue' --output text)

aws amplify start-job --app-id $AMPLIFY_APP_ID --branch-name $AMPLIFY_BRANCH --job-type 'RELEASE'
</code></pre> 
</div> 
<h2>Post-deployment</h2> 
<p>In this section, we discuss the post-deployment steps for launching a frontend application that is intended to emulate the customer’s Production application. The financial services agent will operate as an embedded assistant within the example web UI.</p> 
<h3>Launch a web UI for your chatbot</h3> 
<p>To set up your web application, follow the post-deployment steps outlined in the <a href="https://github.com/aws-samples/generative-ai-amazon-bedrock-langchain-agent-example/blob/main/documentation/deployment-guide.md#launch-a-web-ui-for-your-chatbot" rel="noopener" target="_blank">Launch a web UI for your chatbot</a> section of the sample repository. Amplify provides an automated build and release pipeline that triggers based on new commits to your forked repository and publishes the new version of your website to your Amplify domain. You can view the deployment status on the Amplify console.</p> 
<div class="wp-caption alignnone" id="attachment_68758" style="width: 1613px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-amplify-deployment.png"><img alt="AWS Amplify Pipeline Status" class="size-full wp-image-68758" height="685" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-amplify-deployment.png" style="margin: 10px 0px 10px 0px;" width="1603" /></a>
 <p class="wp-caption-text" id="caption-attachment-68758">Figure 4: AWS Amplify Pipeline Status</p>
</div> 
<h3>Access the Amplify website</h3> 
<p>With your Amazon Lex web UI JavaScript plugin in place, you are now ready to launch your Amplify demo website.</p> 
<ol> 
 <li>To access your website’s domain, navigate to the CloudFormation stack’s <strong>Outputs</strong> tab and locate the Amplify domain URL. Alternatively, use the following command: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">aws cloudformation describe-stacks \
    --stack-name $STACK_NAME \
    --query 'Stacks[0].Outputs[?OutputKey==`AmplifyDemoWebsite`].OutputValue' --output text</code></pre> 
  </div> </li> 
 <li>After you access your Amplify domain URL, you can proceed with testing and validation.</li> 
</ol> 
<div class="wp-caption alignnone" id="attachment_68757" style="width: 1440px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-amplify-website.png"><img alt="AWS Amplify Frontend" class="size-full wp-image-68757" height="870" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-amplify-website.png" style="margin: 10px 0px 10px 0px;" width="1430" /></a>
 <p class="wp-caption-text" id="caption-attachment-68757">Figure 5: AWS Amplify Frontend</p>
</div> 
<h2>Testing and validation</h2> 
<p>The following testing procedure aims to verify that the agent correctly identifies and understands user intents for accessing customer data (such as account information), fulfilling business workflows through predefined intents (such as completing a loan application), and answering general queries, such as the following sample prompts:</p> 
<ol> 
 <li>Why should I use &lt;your-company&gt;?</li> 
 <li>How competitive are their rates?</li> 
 <li>Which type of mortgage should I use?</li> 
 <li>What are current mortgage trends?</li> 
 <li>How much do I need saved for a down payment?</li> 
 <li>What other costs will I pay at closing?</li> 
</ol> 
<p>Response accuracy is determined by evaluating the relevancy, coherency, and human-like nature of the answers generated by the Amazon Bedrock provided Anthropic Claude 2.1 FM. The source links provided with each response (for example, &lt;your-company&gt;.com based on the Amazon Kendra Web Crawler configuration) should also be confirmed as credible.</p> 
<h3>Provide personalized responses</h3> 
<p>Verify the agent successfully accesses and utilizes relevant customer information in DynamoDB to tailor user-specific responses.</p> 
<div class="wp-caption alignnone" id="attachment_68754" style="width: 493px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-customer-data.png"><img alt="Personalized Response" class="wp-image-68754" height="799" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-customer-data.png" style="margin: 10px 0px 10px 0px;" width="483" /></a>
 <p class="wp-caption-text" id="caption-attachment-68754">Figure 6: Personalized Response</p>
</div> 
<p>Note that the use of PIN authentication within the agent is for demonstration purposes only and should not be used in any production implementation.</p> 
<h3>Curate opinionated answers</h3> 
<p>Validate that opinionated questions are met with credible answers by the agent correctly sourcing replies based on authoritative customer documents and webpages indexed by Amazon Kendra.</p> 
<div class="wp-caption alignnone" id="attachment_68749" style="width: 493px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-opinionated.png"><img alt="Opinionated Response" class="wp-image-68749" height="1184" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-opinionated.png" style="margin: 10px 0px 10px 0px;" width="483" /></a>
 <p class="wp-caption-text" id="caption-attachment-68749">Figure 7: Opinionated RAG Response</p>
</div> 
<h3>Deliver contextual generation</h3> 
<p>Determine the agent’s ability to provide contextually relevant responses based on previous chat history.</p> 
<div class="wp-caption alignnone" id="attachment_68755" style="width: 493px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-contextual.png"><img alt="Contextual Generation Response" class="size-full wp-image-68755" height="1147" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-contextual.png" style="margin: 10px 0px 10px 0px;" width="483" /></a>
 <p class="wp-caption-text" id="caption-attachment-68755">Figure 8: Contextual Generation Response</p>
</div> 
<h3>Access general knowledge</h3> 
<p>Confirm the agent’s access to general knowledge information for non-customer-specific, non-opinionated queries that require accurate and coherent responses based on Amazon Bedrock FM training data and RAG.</p> 
<div class="wp-caption alignnone" id="attachment_68753" style="width: 2967px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-general.png"><img alt="General Knowledge Response" class="size-full wp-image-68753" height="2316" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-general.png" style="margin: 10px 0px 10px 0px;" width="2957" /></a>
 <p class="wp-caption-text" id="caption-attachment-68753">Figure 9: General Knowledge Response</p>
</div> 
<h3>Run predefined intents</h3> 
<p>Ensure the agent correctly interprets and conversationally fulfills user prompts that are intended to be routed to predefined intents, such as completing a loan application as part of a business workflow.</p> 
<div class="wp-caption alignnone" id="attachment_68748" style="width: 3945px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-pre-defined.png"><img alt="Pre-Defined Intent Response" class="size-full wp-image-68748" height="2081" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-pre-defined.png" style="margin: 10px 0px 10px 0px;" width="3935" /></a>
 <p class="wp-caption-text" id="caption-attachment-68748">Figure 10: Pre-Defined Intent Response</p>
</div> 
<p>The following is the resultant loan application document completed through the conversational flow.</p> 
<div class="wp-caption alignnone" id="attachment_68750" style="width: 946px;">
 <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-mortgage-app.png"><img alt="Resultant Loan Application" class="wp-image-68750 size-full" height="954" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/19/ML-15256-mortgage-app.png" style="margin: 10px 0px 10px 0px;" width="936" /></a>
 <p class="wp-caption-text" id="caption-attachment-68750">Figure 11: Resultant Loan Application</p>
</div> 
<p>The multi-channel support functionality can be tested in conjunction with the preceding assessment measures across web, SMS, and voice channels. For more information about integrating the chatbot with other services, refer to <a href="https://docs.aws.amazon.com/lexv2/latest/dg/deploy-twilio-sms.html" rel="noopener" target="_blank">Integrating an Amazon Lex V2 bot with Twilio SMS</a> and <a href="https://docs.aws.amazon.com/connect/latest/adminguide/amazon-lex.html" rel="noopener" target="_blank">Add an Amazon Lex bot to Amazon Connect</a>.</p> 
<h2>Clean up</h2> 
<p>To avoid charges in your AWS account, clean up the solution’s provisioned resources.</p> 
<ol> 
 <li>Revoke the GitHub personal access token. GitHub PATs are configured with an expiration value. If you want to ensure that your PAT can’t be used for programmatic access to your forked Amplify GitHub repository before it reaches its expiry, you can revoke the PAT by following the <a href="https://docs.github.com/en/organizations/managing-programmatic-access-to-your-organization/reviewing-and-revoking-personal-access-tokens-in-your-organization" rel="noopener" target="_blank">GitHub repo’s instructions</a>.</li> 
 <li>Delete the GenAI-FSI-Agent.yml CloudFormation stack and other solution resources using the solution deletion automation script. The following commands use the default stack name. If you customized the stack name, adjust the commands accordingly.<code># export STACK_NAME=&lt;YOUR-STACK-NAME&gt;</code><br /> <code>./delete-stack.sh</code></li> 
</ol> 
<h3>Solution Deletion Automation Script</h3> 
<p>The <code>delete-stack.sh shell</code> script deletes the resources that were originally provisioned using the solution deployment automation script, including the GenAI-FSI-Agent.yml CloudFormation stack.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"># cd generative-ai-amazon-bedrock-langchain-agent-example/shell/
	# chmod u+x delete-stack.sh
	# ./delete-stack.sh

	echo "Deleting Kendra Data Source: $KENDRA_WEBCRAWLER_DATA_SOURCE_ID"

	aws kendra delete-data-source --id $KENDRA_WEBCRAWLER_DATA_SOURCE_ID --index-id $KENDRA_INDEX_ID

	echo "Emptying and Deleting S3 Bucket: $S3_ARTIFACT_BUCKET_NAME"

	aws s3 rm s3://${S3_ARTIFACT_BUCKET_NAME} --recursive
	aws s3 rb s3://${S3_ARTIFACT_BUCKET_NAME}

	echo "Deleting CloudFormation Stack: $STACK_NAME"

	aws cloudformation delete-stack --stack-name $STACK_NAME
	aws cloudformation wait stack-delete-complete --stack-name $STACK_NAME

	echo "Deleting Secrets Manager Secret: $GITHUB_TOKEN_SECRET_NAME"

	aws secretsmanager delete-secret --secret-id $GITHUB_TOKEN_SECRET_NAME</code></pre> 
</div> 
<h2>Considerations</h2> 
<p>Although the solution in this post showcases the capabilities of a generative AI financial services agent powered by Amazon Bedrock, it is essential to recognize that this solution is not production-ready. Rather, it serves as an illustrative example for developers aiming to create personalized conversational agents for diverse applications like virtual workers and customer support systems. A developer’s path to production would iterate on this sample solution with the following considerations.</p> 
<h3>Security and privacy</h3> 
<p>Ensure data security and user privacy throughout the implementation process. Implement appropriate access controls and encryption mechanisms to protect sensitive information. Solutions like the generative AI financial services agent will benefit from data that isn’t yet available to the underlying FM, which often means you will want to use your own private data for the biggest jump in capability. Consider the following best practices:</p> 
<ol> 
 <li><strong>Keep it secret, keep it safe</strong> – You will want this data to stay completely protected, secure, and private during the generative process, and want control over how this data is shared and used.</li> 
 <li><strong>Establish usage guardrails</strong> – Understand how data is used by a service before making it available to your teams. Create and distribute the rules for what data can be used with what service. Make these clear to your teams so they can move quickly and prototype safely.</li> 
 <li><strong>Involve Legal, sooner rather than later</strong> – Have your Legal teams review the terms and conditions and service cards of the services you plan to use before you start running any sensitive data through them. Your Legal partners have never been more important than they are today.</li> 
</ol> 
<p>As an example of how we are thinking about this at AWS with Amazon Bedrock: All data is encrypted and does not leave your VPC, and Amazon Bedrock makes a separate copy of the base FM that is accessible only to the customer, and fine tunes or trains this private copy of the model.</p> 
<h3>User acceptance testing</h3> 
<p>Conduct user acceptance testing (UAT) with real users to evaluate the performance, usability, and satisfaction of the generative AI financial services agent. Gather feedback and make necessary improvements based on user input.</p> 
<h3>Deployment and monitoring</h3> 
<p>Deploy the fully tested agent on AWS, and implement monitoring and logging to track its performance, identify issues, and optimize the system as needed. <a href="https://docs.aws.amazon.com/lambda/latest/dg/lambda-monitoring.html" rel="noopener" target="_blank">Lambda monitoring and troubleshooting features</a> are enabled by default for the agent’s Lambda handler.</p> 
<h3>Maintenance and updates</h3> 
<p>Regularly update the agent with the latest FM versions and data to enhance its accuracy and effectiveness. Monitor customer-specific data in DynamoDB and synchronize your Amazon Kendra data source indexing as needed.</p> 
<h2>Conclusion</h2> 
<p>In this post, we delved into the exciting world of generative AI agents and their ability to facilitate human-like interactions through the orchestration of calls to FMs and other complementary tools. By following this guide, you can use Bedrock, LangChain, and existing customer resources to successfully implement, test, and validate a reliable agent that provides users with accurate and personalized financial assistance through natural language conversations. To learn how the same functionality can be delivered through AWS-managed services using <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html" rel="noopener" target="_blank">Amazon Bedrock Agents</a>&nbsp;and <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html" rel="noopener" target="_blank">Amazon Bedrock Knowledge Bases</a>, refer to <a href="https://aws.amazon.com/blogs/machine-learning/automate-the-insurance-claim-lifecycle-using-agents-and-knowledge-bases-for-amazon-bedrock/" rel="noopener" target="_blank">Automate the insurance claim lifecycle using Amazon Bedrock Agents and Knowledge Bases</a>. This alternative approach offers intelligent automation and data search capabilities through personalized agents that transform the way users interact with your applications, making interactions more natural, efficient, and effective.</p> 
<hr /> 
<h3>About the author</h3> 
<p><strong><img alt="" class="alignleft size-full wp-image-68873" height="105" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/12/20/kyle-blocksom-100.jpg" width="100" />Kyle T. Blocksom</strong> is a Sr. Solutions Architect with AWS based in Southern California. Kyle’s passion is to bring people together and leverage technology to deliver solutions that customers love. Outside of work, he enjoys surfing, eating, wrestling with his dog, and spoiling his niece and nephew.</p>
