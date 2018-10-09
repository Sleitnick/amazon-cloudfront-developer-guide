# Tutorial: Creating a Simple Lambda@Edge Function<a name="lambda-edge-how-it-works-tutorial"></a>

This tutorial shows you how to get started with Lambda@Edge by helping you create and add a sample Node\.js function that runs in CloudFront\. The example that we walk through adds HTTP security headers to a response, which can improve security and privacy for a website\. \(That said, you don’t need a website for this walkthrough; we simply add security headers to a response when CloudFront retrieves a file\.\) 

This example illustrates, step by step, how you create and configure a Lambda@Edge function\. You will follow similar steps and choose from the same options for your own Lambda@Edge solution\.

**Topics**
+ [Step 1: Sign Up for an AWS Account](#lambda-edge-how-it-works-tutorial-AWS)
+ [Step 2: Create a CloudFront Distribution](#lambda-edge-how-it-works-tutorial-cloudfront)
+ [Step 3: Create Your Function](#lambda-edge-how-it-works-tutorial-create-function)
+ [Step 4: Add a CloudFront Trigger to Run the Function](#lambda-edge-how-it-works-tutorial-add-trigger)
+ [Step 5: Verify that the Function Runs](#lambda-edge-how-it-works-tutorial-verify)
+ [Step 6: Troubleshoot Issues](#lambda-edge-how-it-works-tutorial-troubleshoot)
+ [Step 7: Clean Up Your Example Resources](#lambda-edge-how-it-works-tutorial-cleanup-resources)
+ [Resources for Learning More](#lambda-edge-how-it-works-tutorial-resources)

## Step 1: Sign Up for an AWS Account<a name="lambda-edge-how-it-works-tutorial-AWS"></a>

If you haven't already done so, sign up for Amazon Web Services at https://aws\.amazon\.com/\. Choose **Sign Up Now** and enter the required information\. 

## Step 2: Create a CloudFront Distribution<a name="lambda-edge-how-it-works-tutorial-cloudfront"></a>

Before you create the example Lambda@Edge function, you must have a CloudFront environment to work with that includes an origin to serve content from\.

**Are you new to CloudFront?** CloudFront delivers content through a worldwide network of edge locations\. When you set up a Lambda function with CloudFront, the function can customize content closer to viewers, improving performance\. If you’re not familiar with CloudFront, take a few minutes before you complete the tutorial to [read a short overview](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/introduction.html) and [learn a bit about how CloudFront caches and serves content](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/HowCloudFrontWorks.html)\.

For this example, you create a CloudFront distribution that is set up with an Amazon S3 bucket, the origin for your CloudFront distribution\. If you already have an environment to use, you can skip this step\.<a name="lambda-edge-how-it-works-tutorial-cf-proc"></a>

**To create a CloudFront distribution with an Amazon S3 origin**

1. Create an Amazon S3 bucket with a file or two, such as image files, for sample content\. You can follow the steps in [Upload your content to Amazon S3](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/GettingStarted.html#GettingStartedUploadContent)\. Make sure that you set permissions to grant public read access to the objects in your bucket\.

1. Create a CloudFront distribution and add your S3 bucket as an origin, by following the steps in [Create a CloudFront web distribution](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/GettingStarted.html#GettingStartedCreateDistribution)\. If you already have a distribution, you can add the bucket as an origin for that distribution instead\.
**Tip**  
Make a note of your distribution ID\. Later in this tutorial when you add a CloudFront trigger for your function, you must choose the ID for your distribution in a drop\-down list—for example, E653W22221KDDL\.

## Step 3: Create Your Function<a name="lambda-edge-how-it-works-tutorial-create-function"></a>

In this step, you create a Lambda function, starting with a blueprint template that's provided in the Lambda Console\. The function adds code to update security headers in your CloudFront distribution\. 

**Are you new to Lambda or Lambda@Edge?** Lambda@Edge lets you use CloudFront triggers to invoke a Lambda function\. When you associate a CloudFront distribution with a Lambda function, CloudFront [intercepts requests and responses](https://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html) at CloudFront edge locations and runs the function\. Lambda functions can improve security or customize information close to your viewers, to improve performance\. In this tutorial, the function that we create updates the security headers in a CloudFront response\.

There are several steps to take when you create a Lambda function\. In this tutorial, you use a blueprint template as the basis for your function, and then update the function with code that sets the security headers\. Finally, you add and deploy a CloudFront trigger to run the function\.<a name="lambda-edge-how-it-works-tutorial-create-function-blueprint-proc"></a>

**To create a Lambda function**

1. Sign in to the AWS Management Console and open the AWS Lambda console at [https://console\.aws\.amazon\.com/lambda/](https://console.aws.amazon.com/lambda/)\. 
**Important**  
Make sure that you’re in the US\-East\-1 \(N\. Virginia\) Region\. You must be in this region to create Lambda@Edge functions\.

1. Choose **Create function**\.

1. On the **Create function** page, choose **Blueprints**, and then filter for the CloudFront blueprints\. Type `cloudfront` in the search field, and the press **Return**\. The keyword `cloudfront` is shown, and all the blueprints that are tagged for CloudFront are listed\.  
![\[Filter for CloudFront blueprints in the Lambda console.\]](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/images/lambda-modify-response-headers.png)

1. Choose the **cloudfront\-modify\-response\-header** blueprint to use as the template for your function\.

1. Enter the information about your function:  
**Name**  
Type a name for your function\.  
**Role**  
Choose how to set the permissions for your function\. It's easiest to get started by using the basic Lambda@Edge permissions policy template, so for this option, choose **Create new role from template\(s\)**\.   
**Role name**  
Type a name for the role that will be created from the policy template, which you choose next, under **Policy templates**\.  
**Policy templates**  
 The policy template **Basic Edge Lambda permissions** is automatically added for you, because you chose a CloudFront blueprint as the basis for your function\. This policy template adds execution role permissions that allow CloudFront to run your Lambda function for you in CloudFront locations around the world\. For more information, see [Setting IAM Permissions and Roles for Lambda@Edge](lambda-edge-permissions.md)\.

1. Choose **Create function**\. Lambda creates the function, and on the next page, you see a Congratulations\! success message box\.  
![\[Code editing and configuration page for a function in the Lambda console.\]](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/images/lambda-after-choose-create-function.png)

   On the **Configuration** tab, there are three main areas: A diagram with boxes for your function by name \(in our example, UpdateSecurityHeaders\), any triggers for the function, and logging information\. There's also section on the bottom that changes to allow different options, depending on which box you choose in the main area—for example, the function name, a trigger, or Amazon CloudWatch Logs\. 

   In addition, on the left, there's list of trigger types that you can choose from\. Because you chose a CloudFront blueprint when you created your function, a CloudFront trigger was automatically added for the function\.

   Now you can update the function code to modify security headers, and then, in the next step, configure and deploy the CloudFront trigger to run the code\.  
![\[Lambda console page where you can add code or triggers for a function.\]](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/images/lambda-function-update-page-full-with-cf.png)

1. Choose the box with your function name, and then in the **Function code** section, replace the template code with a function that modifies security headers that are returned from your origin\. For example, you could use code similar to the following:

   ```
   'use strict';
   exports.handler = (event, context, callback) => {
       
       //Get contents of response
       const response = event.Records[0].cf.response;
       const headers = response.headers;
   
   //Set new headers 
    headers['strict-transport-security'] = [{key: 'Strict-Transport-Security', value: 'max-age= 63072000 
    ; includeSubdomains; preload'}]; 
    headers['content-security-policy'] = [{key: 'Content-Security-Policy', value: "default-src 'none'; img-src 'self'; script-src 'self'; style-src 'self'; object-src 'none'"}]; 
    headers['x-content-type-options'] = [{key: 'X-Content-Type-Options', value: 'nosniff'}]; 
    headers['x-frame-options'] = [{key: 'X-Frame-Options', value: 'DENY'}]; 
    headers['x-xss-protection'] = [{key: 'X-XSS-Protection', value: '1; mode=block'}]; 
    headers['referrer-policy'] = [{key: 'Referrer-Policy', value: 'same-origin'}]; 
       
       //Return modified response
       callback(null, response);
   };
   ```

1. Choose **Save** to save your updated code\.

## Step 4: Add a CloudFront Trigger to Run the Function<a name="lambda-edge-how-it-works-tutorial-add-trigger"></a>

Now that you have a Lambda function to update security headers, configure the CloudFront trigger to run your function to add the headers in any response that CloudFront receives from the origin for your distribution\. <a name="lambda-edge-how-it-works-tutorial-add-trigger-proc"></a>

**To configure the CloudFront trigger for your function**

1. Choose the CloudFront box to highlight it\.

1. Below, in the **Configure triggers** section, choose **Deploy to Lambda@Edge** to open a page where you can define the trigger, and then deploy it\.  
![\[Lambda console page where you have the option to go configure and deploy a CloudFront trigger.\]](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/images/lambda-function-configure-triggers.png)

1. On the **Deploy to Lambda@Edge** page, under **Configure CloudFront trigger**, enter the following information:  
**Distribution**  
The CloudFront distribution ID to associate with your function\. In the drop\-down list, choose the distribution ID\.  
**Cache behavior**  
The cache behavior to use with the trigger\. For this example, leave the value set to \*, which applies your distribution's default cache behavior to all requests\. For more information, see [Cache Behavior Settings](distribution-web-values-specify.md#DownloadDistValuesCacheBehavior) in the [Values That You Specify When You Create or Update a Distribution](distribution-web-values-specify.md) topic\.  
**CloudFront event**  
The trigger that specifies when your function will run\. We want the security headers function to run whenever CloudFront returns a response from the origin\. So in the drop\-down list, choose **Origin response**\. For more information, see [Adding Triggers for a Lambda@Edge Function](lambda-edge-add-triggers.md)\.   
**\(Optional\) Include body**  
Select this check box if you want to access the request body in your function, for viewer request or origin request events\. For more information, see [Accessing the Request Body by Choosing the Include Body Option](lambda-include-body-access.md)\.   
![\[Lambda console page where you can add code or triggers for a function.\]](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/images/lambda-function-configure-trigger-dialog.png)

1. Under **Confirm deploy to Lambda@Edge**, select the check box to acknowledge that the trigger will be deployed and run your function in all AWS Regions\.

1. Choose **Deploy** to add the trigger and replicate the function to AWS Regions worldwide\.

1. Wait for the function to replicate\. This typically takes a few minutes but can take up to 15 minutes\.

    You can check to see if replication is finished by going to the CloudFront console and viewing your distribution:
   + Go to the CloudFront console at https://console\.aws\.amazon\.com/cloudfront/\.

   Check for the distribution status to change from **In Progress** back to **Deployed**, which means that your function has been replicated\. Then follow the steps in the next section to verify that the function works\.

## Step 5: Verify that the Function Runs<a name="lambda-edge-how-it-works-tutorial-verify"></a>

Now that you've created your Lambda function and configured a trigger to run it for a CloudFront distribution, check to make sure that the function is accomplishing what you expect it to\. In this example, we check the HTTP headers that are returned from the origin, to make sure that the security headers are added\.<a name="lambda-edge-how-it-works-tutorial-verify-proc"></a>

**To verify that your Lambda@Edge function adds security headers**

1. In a browser, type the URL for a file in your S3 bucket\. For example, you might use a URL similar to `http://d111111abcdef8.cloudfront.net/image.jpg`\.

   For more information about the CloudFront domain name to use in the file URL, see [Customizing the URL Format for Files in CloudFront](LinkFormat.md)\.

1. Open your browser’s Web Developer toolbar\. For example, in your browser window in Chrome, open the context \(right\-click\) menu, and then choose **Inspect**\.

1. Choose the **Network** tab\.

1. Reload the page to view your image, and then choose an HTTP request on the left pane\. You will see the HTTP headers displayed in a separate pane\.

1. Look through the list of HTTP headers to verify that the expected security headers are included in the list\. For example, you might see headers similar to those shown in the following screenshot:  
![\[HTTP headers list with the expected security headers highlighted.\]](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/images/lambda-at-edge-security-headers-list.png)

If the security headers are included in your headers list, great\! You've successfully created your first Lambda@Edge function\. If, on the other hand, CloudFront returns errors or there are other issues, continue to the next step to troubleshoot the issues\.

## Step 6: Troubleshoot Issues<a name="lambda-edge-how-it-works-tutorial-troubleshoot"></a>

If CloudFront returns errors or doesn't add the security headers as expected, you can investigate your function’s execution by looking at CloudWatch Logs\. Be sure to use the logs stored in the AWS location that is closest to the location where the function is executed\.

For example, if you view the file from London, try changing the Region in the CloudWatch console to EU \(London\)\.<a name="lambda-edge-how-it-works-tutorial-cloudwatch-proc"></a>

**To examine CloudWatch logs for your Lambda@Edge function**

1. Sign in to the AWS Management Console and open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. Change **Region** to the region where the function is executing; that is, the location that is shown when you view the file in your browser\.

1. In the left pane, choose **Logs** to view the logs for your distribution\. 

For more information, see [Monitoring CloudFront Activity Using CloudWatch](monitoring-using-cloudwatch.md)\.

## Step 7: Clean Up Your Example Resources<a name="lambda-edge-how-it-works-tutorial-cleanup-resources"></a>

If you created an S3 bucket and CloudFront distribution just for this tutorial, as a learning exercise, make sure to delete the AWS resources that you allocated so that you no longer accrue charges\. After you delete your AWS resources, any content that you added is no longer available\.

**Tasks**
+ [ Delete the S3 Bucket](#lambda-edge-how-it-works-tutorial-delete-bucket) 
+ [ Delete the CloudFront Distribution](#lambda-edge-how-it-works-tutorial-delete-distribution)

### Delete the S3 Bucket<a name="lambda-edge-how-it-works-tutorial-delete-bucket"></a>

Before you delete your S3 bucket, make sure that logging is disabled for the bucket\. Otherwise, AWS continues to write logs to your bucket as you delete it\.<a name="lambda-edge-how-it-works-tutorial-delete-bucket-proc"></a>

**To disable logging for a bucket**

1. Open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. Select your bucket, and then choose **Properties**\.

1. From **Properties**, choose **Logging**\.

1. Clear the **Enabled** check box\.

1. Choose **Save**\.

Now, you can delete your bucket\. For more information, see [How Do I Delete an S3 Bucket?](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/delete-bucket.html) in the *Amazon Simple Storage Service Console User Guide*\.

### Delete the CloudFront Distribution<a name="lambda-edge-how-it-works-tutorial-delete-distribution"></a>

Before you delete a CloudFront distribution, you must disable it\. A disabled distribution is no longer functional and does not accrue charges\. You can enable a disabled distribution at any time\. After you delete a disabled distribution, it's no longer available\.<a name="lambda-edge-how-it-works-tutorial-delete-distribution-proc"></a>

**To disable and delete a CloudFront distribution**

1. Open the CloudFront console at [ https://console\.aws\.amazon\.com/cloudfront/](https://console.aws.amazon.com/cloudfront/)\.

1. Select the distribution that you want to disable, and then choose **Disable**\.

1. When prompted for confirmation, choose **Yes, Disable**\.

1. Select the disabled distribution, and then choose **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

## Resources for Learning More<a name="lambda-edge-how-it-works-tutorial-resources"></a>

Now that you have a basic idea of how Lambda@Edge functions work, learn more by reading the following:
+ [Lambda@Edge Example Functions](lambda-examples.md)
+ [ Lambda@Edge Design Best Practices](https://aws.amazon.com/blogs/networking-and-content-delivery/lambdaedge-design-best-practices/)
+ [ Reducing Latency and Shifting Compute to the Edge with Lambda@Edge](https://aws.amazon.com/blogs/networking-and-content-delivery/reducing-latency-and-shifting-compute-to-the-edge-with-lambdaedge/)