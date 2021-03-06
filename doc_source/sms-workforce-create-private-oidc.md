# Create a Private Workforce \(OIDC IdP\)<a name="sms-workforce-create-private-oidc"></a>

Create a private workforce using an OpenID Connect \(OIDC\) Identity Provider \(IdP\) when you want to authenticate and manage workers using your own identity provider\. Use this page to learn how to configure your IdP to communicate with Amazon SageMaker Ground Truth \(Ground Truth\) or Amazon Augmented AI \(Amazon A2I\) and to learn how to create a workforce using your own IdP\. 

To create a workforce using an OIDC IdP, your IdP must support *groups* because Ground Truth and Amazon A2I use one or more groups that you specify to create work teams\. You use work teams to specify workers for your labeling jobs and human review tasks\. Because groups are not a [standard claim](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims), your IdP may have a different naming convention for a group of users \(workers\)\. Therefore, you must identify one or more user groups to which a worker belongs using the custom claim `sagemaker:groups` that is sent to Ground Truth or Amazon A2I from your IdP\. To learn more, see [Send Required and Optional Claims to Ground Truth and Amazon A2I](#sms-workforce-create-private-oidc-configure-idp)\.

You create an OIDC IdP workforce using the SageMaker API operation [https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateWorkforce.html](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateWorkforce.html)\. Once you create a private workforce, that workforce and all work teams and workers associated with it are available to use for all Ground Truth labeling job tasks and Amazon A2I human review workflows tasks\. To learn more, see [Create an OIDC IdP Workforce](#sms-workforce-create-private-oidc-createworkforce)\.

## Configure your OIDC IdP<a name="sms-workforce-create-private-oidc-configure-url"></a>

How you configure your OIDC IdP depends on the IdP you use, and your business requirements\. 

When you configure your IdP, you must to specify a callback or redirect URI\. After Ground Truth or Amazon A2I authenticates a worker, this URI will redirect the worker to the worker portal where the workers can access labeling or human review tasks\. To create a worker portal URL, you need to create a workforce with your OIDC IdP details using the [https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateWorkforce.html](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateWorkforce.html) API operation\. Specifically, you must configure your OIDC IdP with required custom sagemaker claims \(see the next section for more details\)\. Therefore, it is recommended that you configure your OIDC with a place\-holder redirect URI, and then update the URI after you create the workforce\. See [Create an OIDC IdP Workforce](#sms-workforce-create-private-oidc-createworkforce) to learn how to create a workforce using this API\. 

You can view your worker portal URL in the SageMaker Ground Truth console, or using the SageMaker API operation, `DescribeWorkforce`\. The worker portal URL is in the [https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_Workforce.html#sagemaker-Type-Workforce-SubDomain](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_Workforce.html#sagemaker-Type-Workforce-SubDomain) parameter in the response\.

**Important**  
Make sure you add the workforce subdomain to your OIDC IdP allow list\. 

**To view your worker portal URL after creating a private workforce \(Console\):**

1. Open the SageMaker console at [https://console\.aws\.amazon\.com/sagemaker/](https://console.aws.amazon.com/sagemaker/)\. 

1. In the navigation pane, choose **Labeling workforces**\. 

1. Select the **Private** tab\.

1. In **Private workforce summary** you will see **Labeling portal sign\-in URL**\. This is your worker portal URL\.

**To view your worker portal URL after creating a private workforce \(API\):**

When you create a private workforce using `[CreateWorkforce](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateWorkforce.html)`, you specify a `WorkforceName`\. Use this name to call [https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_DescribeWorkforce.html](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_DescribeWorkforce.html)\. The following table includes examples of requests using the AWS CLI and AWS SDK for Python \(Boto3\)\. 

------
#### [ SDK for Python \(Boto3\) ]

```
response = client.describe_workforce(WorkforceName='string')
print(f'The workforce subdomain is: {response['SubDomain']}')
```

------
#### [ AWS CLI ]

```
$ C:\>  describe-workforce --workforce-name 'string'
```

------

## Send Required and Optional Claims to Ground Truth and Amazon A2I<a name="sms-workforce-create-private-oidc-configure-idp"></a>

When you use your own IdP, Ground Truth and Amazon A2I use your `Issuer`, `ClientId`, and `ClientSecret` to authenticate workers by obtaining an authentication CODE from your `AuthorizationEndpoint`\. 

Ground Truth and Amazon A2I will use this CODE to obtain a custom claim from either your IdP's `TokenEndpoint` or `UserInfoEndpoint`\. You can either configure `TokenEndpoint` to return a JSON web token \(JWT\) or `UserInfoEndpoint` to return a JSON object\. The JWT or JSON object must contain required and optional claims that you specify\. A [https://openid.net/specs/openid-connect-core-1_0.html#Terminology](https://openid.net/specs/openid-connect-core-1_0.html#Terminology) is a key\-value pair that contains information about a worker or metadata about the OIDC service\. The following table lists the claims that must be included, and that can optionally be included in the JWT or JSON object that your IdP returns\. 

**Note**  
Some of the parameters in the following table can be specified using a `:` or a `-`\. For example, you can specify the groups a worker belongs to using `sagemaker:groups` or `sagemaker-groups` in your claim\. 


|  Name  | Required | Accepted Format and Values | Description | Example | 
| --- | --- | --- | --- | --- | 
|  `sagemaker:groups` or `sagemaker-groups`  |  Yes  |  **Data type**: If a worker belongs to a single group, identify the group using a string\. If a worker belongs to multiple groups, use a list of up to 10 strings\.  **Allowable characters**: Regex: \[\\p\{L\}\\p\{M\}\\p\{S\}\\p\{N\}\\p\{P\}\]\+ **Quotas**: 10 groups per worker 63 characters per group name  |  Assigns a worker to one or more groups\. Groups are used to map the worker into work teams\.   |  Example of worker that belongs to a single group: `"work_team1"` Example of a worker that belongs to more than one groups: `["work_team1", "work_team2"]`   | 
|  `sagemaker:sub` or `sagemaker-sub`  |  Yes  |  **Data type**: String  |  This is mandatory to track a worker identity inside the Ground Truth platform for auditing and to identify tasks worked on by that worker\.  For ADFS: Customers must use the Primary Security Identifier \(SID\)\.   |  `"111011101-123456789-3687056437-1111"`  | 
|  `sagemaker:client_id` or `sagemaker-client_id`  |  Yes  |  **Data type**: String **Allowable characters**: Regex: \[\\w\+\-\]\+ **Quotes**: 128 characters   |  A client ID\. All tokens must be issued for this client ID\.   |  `"00b600bb-1f00-05d0-bd00-00be00fbd0e0"`  | 
|  `sagemaker:name` or `sagemaker-name`  |  Yes  |  **Data type**: String  |  The worker name to be displayed in the worker portal\.  |  `"Jane Doe"`  | 
|  `email`  |  No  |  **Data type**: String  |  The worker email\. Ground Truth uses this email to notify workers that they have been invited to work on labeling tasks\. Ground Truth will also use this email to notify your workers when labeling tasks become available if you set up an Amazon SNS topic for a work team that this worker is on\.  |  `"example-email@domain.com"`  | 
|  `email_verified`  |  No  |  **Data type**: Bool **Accepted Values:** `True`, `False`  |  Indicates if the user email was verified or not\.   |  `True`  | 

The following an example of the JSON object syntax your `UserInfoEndpoint` can return\. 

```
{
    "sub":"122",
    "exp":"10000",
    "sagemaker-groups":["group1","group2"]
    "sagamaker-name":"name",
    "sagemkaer-sub":"122",
    "sagemaker-client_id":"123456"
}
```

Ground Truth or Amazon A2I compares the groups listed in `sagemaker:groups` or `sagemaker-groups` to verify that your worker belongs to the work team specified in the labeling job or human review task\. After the work team has been verified, labeling or human review tasks are sent to that worker\. 

## Create an OIDC IdP Workforce<a name="sms-workforce-create-private-oidc-createworkforce"></a>

You can create a workforce using the SageMaker API operation `CreateWorkforce` and associated language\-specific SDKs\. Specify a `WorkforceName` and information about your OIDC IDP in the parameter `OidcConfig`\. The following shows an example of the request\. See [https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateWorkforce.html](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateWorkforce.html) to learn more about each parameter in this request\.

```
CreateWorkforceRequest: {
    #required fields
    WorkforceName: "example-oidc-workforce",
    OidcConfig: { 
        ClientId: "clientId",
        ClientSecret: "secret",
        Issuer: "https://example-oidc-idp.com/adfs",
        AuthorizationEndpoint: "https://example-oidc-idp.com/adfs/oauth2/authorize",
        TokenEndpoint: "https://example-oidc-idp.com/adfs/oauth2/token",
        UserInfoEndpoint: "https://example-oidc-idp.com/adfs/oauth2/userInfo",
        LogoutEndpoint: "https://example-oidc-idp.com/adfs/oauth2/log-out",
        JwksUri: "https://example-oidc-idp.com/adfs/discovery/keys"
    },
    SourceIpConfig: {
        Cidrs: ["string", "string"]
    }
}
```

## Next Steps<a name="sms-workforce-create-private-oidc-next-steps"></a>

After you create your OIDC IdP, make sure you add the workforce subdomain to your OIDC IdP allow list\.

Once you've created a private workforce using your IdP, you can create work teams using your IdP groups\. To learn more, see [Manage a Private Workforce \(OIDC IdP\)](sms-workforce-manage-private-oidc.md)\. 

You can restrict worker access to tasks to specific IP addresses, and update or delete your workforce using the SageMaker API\. To learn more, see [Manage Private Workforce Using the Amazon SageMaker API](sms-workforce-management-private-api.md)\.