---
title: "Enhance your media search experience using Amazon Q Business and Amazon Transcribe"
url: "https://aws.amazon.com/blogs/machine-learning/enhance-your-media-search-experience-using-amazon-q-business-and-amazon-transcribe/"
date: "Tue, 30 Jul 2024 17:25:40 +0000"
author: "Roshan Thomas"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-kendra/feed/"
---
<p>In today’s digital landscape, the demand for audio and video content is skyrocketing. Organizations are increasingly using media to engage with their audiences in innovative ways. From product documentation in video format to podcasts replacing traditional blog posts, content creators are exploring diverse channels to reach a wider audience. The rise of virtual workplaces has also led to a surge in content captured through recorded meetings, calls, and voicemails. Additionally, contact centers generate a wealth of media content, including support calls, screen-share recordings, and post-call surveys.</p> 
<p>We are excited to introduce Mediasearch Q Business, an open source solution powered by <a href="https://aws.amazon.com/q/business/" rel="noopener" target="_blank">Amazon Q Business</a> and <a href="https://aws.amazon.com/transcribe/" rel="noopener" target="_blank">Amazon Transcribe</a>. Mediasearch Q Business builds on the <a href="https://aws.amazon.com/blogs/machine-learning/make-your-audio-and-video-files-searchable-using-amazon-transcribe-and-amazon-kendra/" rel="noopener" target="_blank">Mediasearch solution powered by Amazon Kendra</a> and enhances the search experience using Amazon Q Business. Mediasearch Q Business supercharges the way you consume media files by using them as part of the knowledge base used by Amazon Q Business to generate reliable answers to user questions. The solution also features an enhanced Amazon Q Business query application that allows users to play the relevant section of the original media files or YouTube videos directly from the search results page, providing a seamless and intuitive user experience.</p> 
<h2>Solution overview</h2> 
<p>Mediasearch Q Business is straightforward to install and try out.</p> 
<div class="wp-video" style="width: 640px;">
 <video class="wp-video-shortcode" controls="controls" height="360" id="video-80395-1" preload="metadata" width="640">
  <source src="https://d2908q01vomqb2.cloudfront.net/artifacts/DBSBlogs/ML-16519/Converted+MediasearchQBusiness.mp4?_=1" type="video/mp4" />
 </video>
</div> 
<p>The solution has two components, as illustrated in the following diagram:</p> 
<ul> 
 <li>A Mediasearch indexer that transcribes media files (audio and video) on an <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) bucket or media from a YouTube playlist and ingests the transcriptions into either an Amazon Q Business native index (configured as part of the Amazon Q Business application) or an <a href="https://aws.amazon.com/kendra/" rel="noopener" target="_blank">Amazon Kendra</a></li> 
 <li>A Mediasearch finder, which provides a UI and makes API calls to the Amazon Q Business service APIs on behalf of the logged-in user. The response from API calls are displayed to the end-user.</li> 
</ul> 
<p><img alt="" class="alignnone wp-image-80602 " height="453" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/mediasearch-amazonq.drawio5.png" width="883" /></p> 
<p>The Mediasearch indexer finds and transcribes audio and video files stored in an S3 bucket. The indexer can also index YouTube videos from a YouTube playlist as audio files and transcribe these audio files. It prepares the transcriptions by embedding time markers at the start of each sentence, and it indexes each prepared transcription in an <a href="https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/select-retriever.html#native-retriever" rel="noopener" target="_blank">Amazon Q Business native retriever</a> or an <a href="https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/select-retriever.html#add-kendra-retriever" rel="noopener" target="_blank">Amazon Kendra retriever</a>. The indexer runs the first time when you install it, and subsequently runs on an interval that you specify, maintaining the index to reflect any new, modified, or deleted files.</p> 
<p>The Mediasearch finder is a web search client that you use to search for content in your Amazon Q Business application. Additionally, the Mediasearch finder includes in-line embedded media players in the search result, so you can see the relevant section of the transcript, and play the corresponding section from the original media (audio files and video files in your media bucket or a YouTube video) without navigating away from the search page.</p> 
<p><img alt="" class="alignnone wp-image-80593 size-full" height="1366" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/finderexperience.png" style="margin: 10px 0px 10px 0px;" width="1958" /></p> 
<p>In the sections that follow, we discuss the following topics:</p> 
<ul> 
 <li>How to deploy the solution to your AWS account</li> 
 <li>How to use it to index and search sample media files</li> 
 <li>How to use the solution with your own media files</li> 
 <li>How the solution works</li> 
 <li>The estimated costs involved</li> 
 <li>How to monitor usage and troubleshoot problems</li> 
 <li>Options to customize and tune the solution</li> 
 <li>How to uninstall and clean up when you’re done experimenting</li> 
</ul> 
<h2>Prerequisites</h2> 
<p>Make sure you have the following:</p> 
<ul> 
 <li>An AWS account where you can launch an <a href="http://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> stack</li> 
 <li>An <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a> instance ARN that would be used by the Amazon Q Business application to provide access to users</li> 
</ul> 
<h2>Deploy the Mediasearch Q Business solution</h2> 
<p>In this section, we walk through deploying the two solution components: the indexer and the finder. We use a CloudFormation stack to deploy the necessary resources in the us-east-1 AWS Region.</p> 
<p>If you’re deploying the solution to another Region, follow the instructions in the <a href="https://github.com/aws-samples/mediasearch-with-amazon-q-transcribe?tab=readme-ov-file#build-and-publish-mediasearch" rel="noopener" target="_blank">README</a> available in the Mediasearch Q Business <a href="https://github.com/aws-samples/mediasearch-with-amazon-q-transcribe" rel="noopener" target="_blank">GitHub repository</a>.</p> 
<h3>Deploy the Mediasearch Q Business indexer component</h3> 
<p>To deploy the indexer component, complete the following steps:</p> 
<ol> 
 <li>Choose <strong>Launch Stack</strong>.<br /> <a href="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://aws-ml-blog.s3.us-east-1.amazonaws.com/artifacts/mediasearch-qbusiness/msindexer-qbusiness.yaml&amp;stackName=MediaSearch-MSINDEXER-QBUSINESS" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-80413 " height="22" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/15/launch_stack.jpg" width="121" /></a></li> 
 <li>In the <strong>Identity center ARN and Retriever selection</strong> section, for <strong>IdentityCenterInstanceArn</strong>, enter the ARN for your IAM Identity Center instance.</li> 
</ol> 
<p>You can find the ARN on the <strong>Settings</strong> page of the IAM Identity Center console. The ARN is a required field.</p> 
<p><img alt="" class="alignnone wp-image-80586 size-full" height="972" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/identity-centre-copy-arn.png" style="margin: 10px 0px 10px 0px;" width="1968" /></p> 
<ol start="3"> 
 <li>Use default values for all other parameters. We will customize these values later to suit your specific requirements.</li> 
 <li>Acknowledge that the stack might create IAM resources with custom names, then choose <strong>Create stack</strong>.</li> 
</ol> 
<p>The indexer stack takes around 10 minutes to deploy. Wait for the indexer to finish deploying before you deploy the Mediasearch Q Business finder.</p> 
<h3>Deploy the Mediasearch Q Business finder component</h3> 
<p>The Mediasearch finder uses <a href="https://aws.amazon.com/cognito/" rel="noopener" target="_blank">Amazon Cognito</a> to authenticate users to the solution. For an authenticated user to interact with an Amazon Q Business application, you must configure an IAM Identity Center <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/customermanagedapps.html" rel="noopener" target="_blank">customer managed application</a> that either supports SAML 2.0 or OAuth 2.0.</p> 
<p>In this post, we create a customer managed application that supports OAuth 2.0, a secure way for applications to communicate and share user data without exposing passwords. We use a technique called <em>trusted identity propagation</em>, which allows the Mediasearch Q Business finder app to access the Amazon Q service securely without sharing passwords between the two identity providers (Amazon Cognito and IAM Identity Center in our example).</p> 
<p>Instead of sharing passwords, trusted identity propagation uses tokens. Tokens are like digital certificates that prove who the user is and what they’re allowed to do. AWS managed applications that work with trusted identity propagation get tokens directly from IAM Identity Center. IAM Identity Center can also exchange identity tokens and access tokens from external authorization servers like Amazon Cognito. This lets an application authenticate users and obtain tokens outside of AWS (like with Amazon Cognito, Microsoft Entra ID, or Okta), exchange that token for an IAM Identity Center token, and then use the new token to request access to AWS services like Amazon Q Business.</p> 
<p>For more information, see <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/trustedidentitypropagation-using-customermanagedapps-setup.html" rel="noopener" target="_blank">Using trusted identity propagation with customer managed applications</a>.</p> 
<p>When the IAM Identity Center instance is in the same account where you are deploying the Mediasearch Q Business solution, the finder stack allows you to automatically create the IAM Identity Center customer managed application as part of the stack deployment.</p> 
<p>If you use the organization instance of IAM Identity Center enabled in your management account, then you will be deploying the Mediasearch Q Business finder stack in a different AWS account. In this case, follow the steps in the <a href="https://github.com/aws-samples/mediasearch-with-amazon-q-transcribe/tree/v0.1.0?tab=readme-ov-file#configuring-the-aws-iam-identity-centre-application" rel="noopener" target="_blank">README</a> to create an IAM Identity Center application manually.</p> 
<p>To deploy the finder component and create the IAM Identity Center customer managed application, complete the following steps:</p> 
<ol> 
 <li>Choose <strong>Launch Stack</strong>.<br /> <a href="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://aws-ml-blog.s3.us-east-1.amazonaws.com/artifacts/mediasearch-qbusiness/msfinder-qbusiness.yaml&amp;stackName=MediaSearch-MSFINDER-QBUSINESS" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-80413" height="22" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/15/launch_stack.jpg" width="120" /></a></li> 
 <li>For <strong>IdentityCenterInstanceArn</strong>, enter the ARN for the IAM Identity Center instance. This is the same value you used while deploying the indexer stack.</li> 
 <li>For <strong>CreateIdentityCenterApplication</strong>, choose <strong>Yes</strong> to create the IAM Identity Center application for the Mediasearch finder application.</li> 
 <li>Under <strong>Mediasearch Indexer parameters</strong>, enter the Amazon Q Business application ID that was created by the indexer stack. You can copy this from the <code>QBusinessApplicationId</code> output of the indexer stack.</li> 
 <li>Select the retriever type that was used to deploy the Mediasearch indexer. (If you deployed an Amazon Kendra index, then select <strong>Kendra</strong>, otherwise select <strong>Native</strong>.</li> 
 <li>If you selected <strong>Kendra</strong>, enter the Amazon Kendra index ID that was used by the indexer stack.</li> 
 <li>For <strong>MediaBucketNames</strong>, use the <code>MediaBucketsUsed</code> output from the indexer CloudFormation stack to allow the search page to access media files across <code>YTMediaBucket</code> and Mediabucket.</li> 
 <li>Acknowledge that the stack might create IAM resources with custom names, then choose <strong>Create stack</strong>.</li> 
</ol> 
<h3>Configure user access to Amazon Q Business</h3> 
<p>To access the Mediasearch Q Business solution, add a user with an appropriate subscription to the Amazon Q Business application and to the IAM Identity Center customer managed application.</p> 
<h4>Add a user to the Amazon Q Business application</h4> 
<p>To start using the Amazon Q Business application, you can add users or groups to the Amazon Q Business application from your IAM Identity Center instance. Complete the following steps to add a user to the application:</p> 
<ol> 
 <li>Access the Amazon Q Business application by choosing the link for <code>QBusinessApplication</code> in the indexer CloudFormation stack outputs.<br /> <img alt="" class="alignnone wp-image-80587 size-full" height="738" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/indexer-output-qbusinessapplication.png" style="margin: 10px 0px 10px 0px;" width="1470" /></li> 
 <li>Under <strong>Groups and users</strong>, on the <strong>Users</strong> tab, choose <strong>Manage access and subscription</strong>.<br /> <img alt="" class="alignnone wp-image-80580 size-full" height="1056" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/manage-user-subscription.png" style="margin: 10px 0px 10px 0px;" width="2220" /></li> 
 <li>Choose <strong>Add groups and users</strong>.<br /> <img alt="" class="alignnone wp-image-80581 size-full" height="664" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/manage-user-subscription-2.png" style="margin: 10px 0px 10px 0px;" width="2408" /></li> 
 <li>Choose <strong>Add existing users and groups</strong>.<br /> <img alt="" class="alignnone wp-image-80582 " height="345" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/manage-user-subscription-3.png" style="margin: 10px 0px 10px 0px;" width="647" /></li> 
 <li>Search for an existing user, choose the user, and choose <strong>Assign</strong>.<br /> <img alt="" class="size-full wp-image-80420 aligncenter" height="622" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/15/user_search.png" style="margin: 10px 0px 10px 0px;" width="644" /></li> 
 <li>Select the added user and on the <strong>Change subscription</strong> menu, choose<strong> Update subscription tier</strong>.<br /> <img alt="" class="alignnone wp-image-80583 size-full" height="630" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/manage-user-subscription-5.png" style="margin: 10px 0px 10px 0px;" width="2242" /></li> 
 <li>Select the appropriate subscription tier and choose <strong>Confirm</strong>.</li> 
</ol> 
<p>For details of each Amazon Q subscription, refer to <a href="https://aws.amazon.com/q/business/pricing/" rel="noopener" target="_blank">Amazon Q Business pricing</a>.<br /> <img alt="" class="alignnone wp-image-80589 size-full" height="968" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/manage-user-subscription-6.png" style="margin: 10px 0px 10px 0px;" width="1600" /></p> 
<h4>Assign users to the IAM Identity Center customer managed application</h4> 
<p>Now you can assign users or groups to the IAM Identity Center customer managed application. Complete the following steps to add a user:</p> 
<ol> 
 <li>From the outputs section of the finder CloudFormation stack, choose the URL for <code>IdentityCenterApplicationConsoleURLto</code> navigate to the customer managed application.<br /> <img alt="" class="alignnone wp-image-80590 " height="456" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/IdentityCenterApplicationConsoleURL-output.png" width="776" /></li> 
</ol> 
<ol start="2"> 
 <li>Choose <strong>Assign users and groups</strong>.<br /> <img alt="" class="alignnone wp-image-80588 size-full" height="737" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/add-users-idc-application.png" style="margin: 10px 0px 10px 0px;" width="1484" /></li> 
</ol> 
<ol start="3"> 
 <li>Select users and choose <strong>Assign users</strong>.<br /> <img alt="" class="alignnone wp-image-80584 size-full" height="880" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/select-assign-users-idcapplication.png" style="margin: 10px 0px 10px 0px;" width="2128" /></li> 
</ol> 
<p>This concludes the user access configuration to the Mediasearch Q Business solution.</p> 
<h2>Test with the sample media files</h2> 
<p>When the Mediasearch indexer and finder stack are deployed, the indexer should have completed processing the audio (mp3) files for the YouTube videos and sample media files (selected <a href="https://aws.amazon.com/podcasts/aws-podcast" rel="noopener" target="_blank">AWS Podcast</a> episodes and <a href="https://www.youtube.com/playlist?list=PLhr1KZpdzukfdjsOHZ-BazZt1iK1J8UUw" rel="noopener" target="_blank">AWS Knowledge Center videos</a>). You can now run your first Mediasearch query.</p> 
<ol> 
 <li>To log in to the Mediasearch finder application, choose the URL for <code>MediasearchFinderURL</code> in the stack outputs.<br /> <img alt="" class="size-full wp-image-80427 aligncenter" height="186" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/15/cfn_url.jpg" style="margin: 10px 0px 10px 0px;" width="844" /></li> 
</ol> 
<p>The Mediasearch finder application in your browser will show a splash page for Amazon Q Business.</p> 
<ol start="2"> 
 <li>Choose <strong>Get Started</strong> to access the Amazon Cognito page.<br /> <img alt="" class="alignnone wp-image-80591 size-full" height="1276" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/msqb-homepage.png" style="margin: 10px 0px 10px 0px;" width="2386" /></li> 
</ol> 
<p>To access Mediasearch Q Business, you need to log in to the application using a user ID in the <a href="https://console.aws.amazon.com/cognito/v2/idp/user-pools" rel="noopener" target="_blank">Amazon Cognito user pool</a> created by the finder stack. The email address in Amazon Cognito must match the email address for the user in IAM Identity Center. Alternatively, the Mediasearch solution allows you to create a user through the application.</p> 
<ol start="3"> 
 <li>On the <strong>Create Account</strong> tab, enter your email (which matches the email address in IAM Identity Center), followed by a password and password confirmation, and choose <strong>Create Account</strong>.</li> 
</ol> 
<p>Amazon Cognito will send an email with a confirmation code for email verification.</p> 
<ol start="4"> 
 <li>Enter this confirmation code to complete your email verification.<br /> <img alt="" class="size-full wp-image-80430 aligncenter" height="512" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/15/demo_signin.jpg" style="margin: 10px 0px 10px 0px;" width="844" /></li> 
</ol> 
<ol start="5"> 
 <li>After email verification, you will now be able to log in to the Mediasearch Q Business application.</li> 
 <li>After you’re logged in, in the <strong>Enter a prompt</strong> box, write a query, such as “What is AWS Fargate?”</li> 
</ol> 
<p>The query returns a response from Amazon Q Business based on the media (sample media files and YouTube audio sources) ingested into the index.</p> 
<p><img alt="" class="alignnone wp-image-80592 size-full" height="1362" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/finderexperience-1.png" style="margin: 10px 0px 10px 0px;" width="2032" /><br /> The response includes citations, with reference to sources. Users can verify their answer from Amazon Q Business by playing media files from their S3 buckets or YouTube starting at the time marker where the relevant information is found.</p> 
<ol start="7"> 
 <li>Use the embedded video player to play the original video inline. Observe that the media playback starts at the relevant section of the video based on the time marker.</li> 
 <li>To play the video full screen in a new browser tab, use the <strong>Full screen</strong> menu option in the player, or choose the media file hyperlink shown above the answer text.</li> 
 <li>Choose (right-click) the video file hyperlink, copy the URL, and enter it into a text editor.</li> 
</ol> 
<p>If the media is an audio file for a YouTube video, it looks something like the following:</p> 
<p><code>https://www.youtube.com/watch?v=unFVfqj9cQ8&amp;t=36.58s</code></p> 
<p>If the media file is a non-YouTube audio file that resides in <code>MediaBucket</code>, the URL looks like the following:</p> 
<p><code>https://mediasearchtest.s3.amazonaws.com/mediasamples/What_is_an_Interface_VPC_Endpoint_and_how_can_I_create_Interface_Endpoint_for_my_VPC_.mp4?AWSAccessKeyId=ASIAXMBGHMGZLSYWJHGD&amp;Expires=1625526197&amp;Signature=BYeOXOzT585ntoXLDoftkfS4dBU%3D&amp;x-amz-security-token=.... #t=253.52</code></p> 
<p>This is a <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html" rel="noopener" target="_blank">presigned S3 URL</a> that provides your browser with temporary read access to the media file referenced in the search result. Using presigned URLs means you don’t need to provide permanent public access to all of your indexed media files.</p> 
<ol start="11"> 
 <li>Experiment with additional queries, such as “How has AWS helped customers in building MLOps platform?” or “How can I use Generative AI to improve customer experience?” or try your own questions.</li> 
</ol> 
<h2>Index and search your own media files</h2> 
<p>To index media files stored in your own S3 bucket, replace the <code>MediaBucket</code> and <code>MediaFolderPrefix</code> parameters with your own bucket name and prefix when you install or update the indexer component stack, and modify the <code>MediaBucketName</code> parameter with your own bucket name when you install or update the finder component stack. Additionally, you can replace the YouTube playlist (<code>PlayListURL</code>) with your own playlist URL and update the indexer stack.</p> 
<ol> 
 <li>When creating a new MediaSearch indexer stack, you can choose to use either a native retriever or an Amazon Kendra retriever. You can make this selection using the parameter <code>RetrieverType</code>. When using the Amazon Kendra retriever, you can either let indexer stack create an Amazon Kendra index or use an existing Amazon Kendra <code>IndexId</code> to add files stored in the new location. To deploy a new indexer, follow the steps from earlier in this post, but replace the defaults to specify the media bucket name and prefix for your own media files or replace the YouTube playlist URL with your own playlist URL. Make sure that you comply with the <a href="https://www.youtube.com/static?template=terms" rel="noopener" target="_blank">YouTube Terms of Service</a>.</li> 
 <li>Alternatively, update an existing MediaSearch indexer stack to replace the previously indexed files with files from the new location or update the YouTube playlist URL or the number of videos to download from the playlist: 
  <ol> 
   <li>Select the stack on the AWS CloudFormation console, choose <strong>Update</strong>, then <strong>Use current template</strong>, then <strong>Next</strong>.</li> 
   <li>Modify the media bucket name and prefix parameter values as needed.</li> 
   <li>Modify the <strong>YouTube Playlist URL</strong> and <strong>Number of YouTube Videos</strong> values as needed.</li> 
   <li>Choose <strong>Next</strong> twice, select the acknowledgement check box, and choose <strong>Update stack</strong>.</li> 
  </ol> </li> 
 <li>Update an existing MediaSearch finder stack to change bucket names or add additional bucket names to the <code>MediaBucketNames</code></li> 
</ol> 
<p>When the MediaSearch indexer stack is successfully created or updated, the indexer automatically finds, transcribes, and indexes the media files stored in your S3 bucket. When it’s complete, you can submit queries and find answers from the audio tracks of your own audio and video files.</p> 
<p>You have the option to provide metadata for any or all of your media files. Use metadata to assign values to index attributes for sorting, filtering, and faceting your search results, or to specify access control lists to govern access to the files. Metadata files can be in the same S3 folder as your media files (default), or in a parallel folder structure specified by the optional indexer parameter <code>MetadataFolderPrefix</code>. For more information about how to create metadata files, see <a href="https://docs.aws.amazon.com/kendra/latest/dg/s3-metadata.html" rel="noopener" target="_blank">Amazon S3 document metadata</a>.</p> 
<p>You can also provide customized transcription options for any or all of your media files. This allows you to take full advantage of Amazon Transcribe features such as <a href="https://docs.aws.amazon.com/transcribe/latest/dg/how-vocabulary.html" rel="noopener" target="_blank">custom vocabularies</a>, <a href="https://docs.aws.amazon.com/transcribe/latest/dg/content-redaction.html" rel="noopener" target="_blank">automatic content redaction</a>, and <a href="https://docs.aws.amazon.com/transcribe/latest/dg/custom-language-models.html" rel="noopener" target="_blank">custom language models</a>.</p> 
<h3>How the Mediasearch solution works</h3> 
<p>Let’s take a quick look at how the solution works, as illustrated in the following diagram.</p> 
<p><img alt="" class="alignnone wp-image-80594 size-full" height="901" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/MediaSearchArchitectureYT1.png" style="margin: 10px 0px 10px 0px;" width="2361" /></p> 
<p>The Mediasearch solution has an event-driven <a href="https://aws.amazon.com/serverless/" rel="noopener" target="_blank">serverless</a> computing architecture with the following steps:</p> 
<ol> 
 <li>You provide an S3 bucket containing the audio and video files you want to index and search. This is also known as the <code>MediaBucket</code>. Leave this blank if you don’t want to index media from your <code>MediaBucket</code>.</li> 
 <li>You also provide your YouTube playlist URL and the number of videos to index from the YouTube playlist. Make sure that you comply with the <a href="https://www.youtube.com/static?template=terms" rel="noopener" target="_blank">YouTube Terms of Service</a>. The <code>YTIndexer</code> will index the latest files from the YouTube playlist. For example, if the number of videos is set to 5, then the <code>YTIndexer</code> will index the five latest videos in the playlist. Any YouTube video indexed prior is ignored from being indexed.</li> 
 <li>An <a href="http://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> function fetches the YouTube videos from the playlist as audio (mp3 files) into the <code>YTMediaBucket</code> and also creates a metadata file in the <code>MetadataFolderPrefix</code> location with metadata for the YouTube video. The YouTube <code>videoid</code> along with the related metadata are recorded in an <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a> table (<code>YTMediaDDBQueueTable</code>).</li> 
 <li><a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a> generates events on a repeating interval (every 2 hours, every 6 hours, and so on) These events invoke the Lambda function S3CrawlLambdaFunction.</li> 
 <li>An <a href="http://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> function is invoked initially when the CloudFormation stack is first deployed, and then subsequently by the scheduled events from EventBridge. The S3CrawlLambdaFunction function crawls through the <code>MediaBucket</code> and the <code>YTMediabucket</code> and starts an Amazon Q Business index (or Amazon Kendra) data source sync job. The Lambda function lists all the supported media files (FLAC, MP3, MP4, Ogg, WebM, AMR, or WAV) and associated metadata and transcribe options stored in the user provided S3 bucket.</li> 
 <li>Each new file is added to another DynamoDB tracking table and submitted to be transcribed by an Amazon Transcribe job. Any file that has been previously transcribed is submitted for transcription again only if it has been modified since it was previously transcribed, or if associated Amazon Transcribe options have been updated. The DynamoDB table is updated to reflect the transcription status and last modified timestamp of each file. Any tracked files that no longer exist in the S3 bucket are removed from the DynamoDB table and from the Amazon Q Business index (or Amazon Kendra index). If no new or updated files are discovered, the Amazon Q Business index (or Amazon Kendra) data source sync job is immediately stopped. The DynamoDB table holds a record for each media file with attributes to track transcription job names and status, and last modified timestamps.</li> 
 <li>As each Amazon Transcribe job completes, EventBridge generates a job complete event, which invokes another Lambda function (S3JobCompletionLambdaFunction).</li> 
 <li>The Lambda function processes the transcription job output, generating a modified transcription that has a time marker inserted at the start of each sentence. This modified transcription is indexed in Amazon Q Business (or Amazon Kendra), and the job status for the file is updated in the DynamoDB table. When the last file has been transcribed and indexed, the Amazon Q Business (or Amazon Kendra) data source sync job is stopped.</li> 
 <li>The index is populated and kept in sync with the transcriptions of all the media files in the S3 bucket monitored by the Mediasearch indexer component, integrated with any additional content from any other provisioned data sources. The media transcriptions are used by the Amazon Q Business application, which allows users to find content and answers to their questions.</li> 
 <li>The sample finder client application enhances users’ search experience by embedding an inline media player with each source or citation that is based on a transcribed media file. The client uses the time markers embedded in the transcript to start media playback at the relevant section of the original media file.</li> 
 <li>An Amazon Cognito user pool is used to authenticate users and is configured to exchange tokens from IAM Identity Center to support Amazon Q Business service calls.</li> 
</ol> 
<h3>Estimated costs</h3> 
<p>In addition to Amazon S3 costs associated with storing your media, the Mediasearch solution incurs usage costs from the Amazon Q, Amazon Kendra (if using an Amazon Kendra index), Amazon Transcribe, and <a href="https://aws.amazon.com/api-gateway" rel="noopener" target="_blank">Amazon API Gateway</a>. Additional minor costs are incurred by the other services mentioned after free tier allowances have been used. For more information, see the pricing pages for <a href="https://aws.amazon.com/q/business/pricing/" rel="noopener" target="_blank">Amazon Q Business</a>, <a href="https://aws.amazon.com/kendra/pricing/" rel="noopener" target="_blank">Amazon Kendra</a>, <a href="https://aws.amazon.com/transcribe/pricing/" rel="noopener" target="_blank">Amazon Transcribe</a>, <a href="https://aws.amazon.com/lambda/pricing/" rel="noopener" target="_blank">Lambda</a>, <a href="https://aws.amazon.com/dynamodb/pricing/" rel="noopener" target="_blank">DynamoDB</a>, and <a href="https://aws.amazon.com/eventbridge/pricing/" rel="noopener" target="_blank">EventBridge</a>.</p> 
<h3>Monitor and troubleshoot</h3> 
<p>To see the details of each media file transcript job, navigate to the <strong>Transcription jobs </strong>page on the Amazon Transcribe console.</p> 
<p>Each media file is transcribed only one time, unless the file is modified. Modified files are re-transcribed and re-indexed to reflect the changes.</p> 
<p>Choose any transcription job to review the transcription and examine additional job details.</p> 
<p><img alt="" class="alignnone wp-image-80595 size-full" height="776" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/transcribe.png" style="margin: 10px 0px 10px 0px;" width="2602" /></p> 
<p>You can check the status of the data source sync by navigating to the Amazon Q Business application deployed by the indexer stack (choose the link on the indexer stack outputs page for QApplication). In the data source section, choose the custom data source and view the status of the sync job.</p> 
<p><img alt="" class="alignnone wp-image-80596 size-full" height="569" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/datasource-screenshot.png" style="margin: 10px 0px 10px 0px;" width="1578" /></p> 
<p>On the DynamoDB console, choose <strong>Tables </strong>in the navigation pane. Use your MediaSearch stack name as a filter to display the MediaSearch DynamoDB tables, and examine the items showing each indexed media file and corresponding status.</p> 
<p>The table MediaSearch-Indexer-<code>YTMediaDDBQueueTable</code> has one record for each YouTube <code>videoid</code> that is downloaded as an audio (mp3) file along with the metadata for the video like author, view count, video title, and so on.</p> 
<p><img alt="" class="alignnone wp-image-80598 size-full" height="1222" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/ddbtable-ytindexer.png" style="margin: 10px 0px 10px 0px;" width="2582" /></p> 
<p>The table <code>MediaSearch-Indexer-MediaDynamoTable</code> has one record for each media file (including YouTube videos), and contains attributes with information about the file and its processing status.</p> 
<p><img alt="" class="alignnone wp-image-80597 size-full" height="1056" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/ddbtable-indexer.png" style="margin: 10px 0px 10px 0px;" width="2590" /></p> 
<p>On the <strong>Functions </strong>page of the Lambda console, use your indexer stack name as a filter to list the Lambda functions that are part of the solution:</p> 
<ul> 
 <li>The <code>YouTubeVideoIndexer</code> function indexes and downloads YouTube videos if the CloudFormation stack parameter <code>PlayListURL</code> is set to a valid YouTube playlist</li> 
 <li>The S3CrawlLambdaFunction function crawls the <code>YTMediaBucket</code> and the <code>MediaBucket</code> for media files and initiates the transcription jobs for the media files</li> 
</ul> 
<p>When the transcription job is complete, a completion event invokes the <code>S3JobCompletionLambdaFunction</code> function, which ingests the transcription into the Amazon Q Business index (or Amazon Kendra index) with any related metadata.</p> 
<p><img alt="" class="alignnone wp-image-80599 size-full" height="328" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/indexer-lambda-functions.png" style="margin: 10px 0px 10px 0px;" width="975" /></p> 
<p>Choose any of the functions to examine the function details, including environment variables, source code, and more. Choose <strong>Monitor </strong>and<strong> View logs in CloudWatch </strong>to examine the output of each function invocation and troubleshoot any issues.</p> 
<p>On the <strong>Functions </strong>page of the Lambda console, use your finder stack name as a filter to list the Lambda functions that are part of the solution:</p> 
<ul> 
 <li>The <code>BuildTriggerLambda</code> function runs the build of the finder <a href="https://aws.amazon.com/amplify/" rel="noopener" target="_blank">AWS Amplify</a> application after cloning the <a href="https://aws.amazon.com/codecommit/" rel="noopener" target="_blank">AWS CodeCommit</a> repository with the finder ReactJS code.</li> 
 <li>The <code>IDCTokenCreateLambda</code> function uses the authorization header that contains a JWT token from a successful authentication with Amazon Cognito to exchange bearer tokens from IAM Identity Center.</li> 
 <li>The <code>IDCAppCreateLambda</code> function creates an OAuth 2.0 IAM Identity Center application to exchange tokens from IAM Identity Center and a trusted token issuer for the Amazon Cognito user pool.</li> 
 <li>The <code>UserConversationLambda</code> function is called from API Gateway to <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/qbusiness/client/list_conversations.html" rel="noopener" target="_blank">list</a> or <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/qbusiness/client/delete_conversation.html" rel="noopener" target="_blank">delete</a> Amazon Q Business conversations.</li> 
 <li>The <code>UserPromptsLambda</code> function is called from API Gateway to call the <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/qbusiness/client/chat_sync.html" rel="noopener" target="_blank">chat_sync</a> API of the Amazon Q Business service.</li> 
 <li>The <code>PreSignedURLCreateLambda</code> function is called from API Gateway to create a presigned URL for S3 buckets. The presigned URL is used to play the media files residing on the <code>Mediabucket</code> that serves as the source for an Amazon Q Business response.</li> 
</ul> 
<p><img alt="" class="alignnone wp-image-80600 size-full" height="341" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/17/finder-lambda-functions.png" style="margin: 10px 0px 10px 0px;" width="1247" /></p> 
<p>Choose any of the functions to examine the function details, including environment variables, source code, and more. Choose <strong>Monitor </strong>and<strong> View logs in CloudWatch </strong>to examine the output of each function invocation and troubleshoot any issues.</p> 
<h2>Customize and enhance the solution</h2> 
<p>You can fork the MediaSearch Q Business <a href="https://github.com/aws-samples/mediasearch-with-amazon-q-transcribe" rel="noopener" target="_blank">GitHub repository</a>, enhance the code, and send us pull requests so we can incorporate and share your improvements.</p> 
<p>The following are a few suggestions for features you might want to implement:</p> 
<ul> 
 <li>Enhance the indexer stack to allow the existing Amazon Q Business application IDs to be used</li> 
 <li>Extend your search sources to include other video streaming platforms relevant to your organization</li> 
 <li>Build <a href="https://aws.amazon.com/cloudwatch" rel="noopener" target="_blank">Amazon CloudWatch</a> metrics and dashboards to improve the manageability of MediaSearch</li> 
</ul> 
<h2>Clean up</h2> 
<p>When you’re finished experimenting with this solution, clean up your resources by using the AWS CloudFormation console to delete the indexer and finder stacks that you deployed. This deletes all the resources that were created by deploying the solution.</p> 
<p>Preexisting Amazon Q Business applications or indexes or IAM Identity Center applications or trusted token issuers that were created manually aren’t deleted.</p> 
<h2>Conclusion</h2> 
<p>The combination of Amazon Q Business and Amazon Transcribe enables a scalable, cost-effective solution to surface insights from your media files. You can use the content of your media files to find accurate answers to your users’ questions, whether they’re from text documents or media files, and consume them in their native format. This solution enhances the overall experience of the <a href="https://aws.amazon.com/blogs/machine-learning/make-your-audio-and-video-files-searchable-using-amazon-transcribe-and-amazon-kendra/" rel="noopener" target="_blank">previous Mediasearch solution</a> by using the powerful generative artificial intelligence (AI) capabilities of Amazon Q Business.</p> 
<p>The sample MediaSearch Q Business solution is provided as open source—use it as a starting point for your own solution, and help us make it better by contributing back fixes and features through GitHub pull requests. For expert assistance, AWS Professional Services and other Amazon partners are here to help.</p> 
<p>We’d love to hear from you. Let us know what you think in the comments section, or use the issues forum in the MediaSearch Q Business <a href="https://github.com/aws-samples/mediasearch-with-amazon-q-transcribe" rel="noopener" target="_blank">GitHub repository</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><img alt="" class="wp-image-54252 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/04/17/Roshan-thomas.jpg" width="100" /><strong>Roshan Thomas</strong> is a Senior Solutions Architect at Amazon Web Services. He is based in Melbourne, Australia, and works closely with power and utilities customers to accelerate their journey in the cloud. He is passionate about technology and helping customers architect and build solutions on AWS.</p> 
<p style="clear: both;"><img alt="" class="wp-image-80519 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/07/15/Anup-100.png" width="100" /><strong>Anup Dutta</strong> is a Solutions Architect with AWS based in Chennai, India. In his role at AWS, Anup works closely with startups to design and build cloud-centered solutions on AWS.</p> 
<p style="clear: both;"><strong><img alt="Bob Strahan" class="size-full wp-image-21654 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2021/02/10/Bob-Strahan-p.png" width="100" />Bob Strahan</strong> is a Principal Solutions Architect in the AWS Language AI Services team.</p> 
<p style="clear: both;"><strong><img alt="Abhinav Jawadekar" class="size-full wp-image-20223 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2020/12/24/Abhinav-Jawadekar.jpg" width="100" />Abhinav Jawadekar</strong> is a Principal Solutions Architect in the Amazon Q Business service team at AWS. Abhinav works with AWS customers and partners to help them build generative AI solutions on AWS.</p>
