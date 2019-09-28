# Get Started

**Welcome to Tenable.io API! ** Tenable.io is the world’s first Cyber Exposure platform, giving you complete visibility into your network and helping you to manage and measure your modern attack surface. All the capabilities of Tenable.io Vulnerability Management are available in the Tenable.io API, a robust platform for users of all experience levels. The platform is designed to support and visualize elastic IT assets, such as containers and web apps. Tenable offers pre-built integrations and allows developers to build new integrations quickly in order to improve their vulnerability management program.

![Tenable.io Architecture](https://files.readme.io/5887470-tenable_arch_diagrampng.png
 "Tenable.io Architecture")

Using the API, you can seamlessly integrate Tenable.io into your cybersecurity infrastructure, for example:
- Automate asset data import into Tenable.io.
- Import third-party scan data.
- Export scan results from Tenable.io into a workflow management system for remediation.

You can get started with Tenable.io API using our API Explorer. You can also use utilities like cURL or Postman to gather data and get additional details that may not be readily available in the API Explorer. Once you become familiar with the API, we provide libraries and SDKs to facilitate development.

## Set up API access
To set up access to Tenable.io API:
* Verify that you have a valid user account with appropriate permissions by logging into [Tenable.io](https://cloud.tenable.com).
* Generate the API keys for the account. For more information, see [Generate an API Key](https://docs.tenable.com/cloud/Content/Settings/GenerateAPIKey.htm) in the Tenable.io Vulnerability Management User Guide.


## Use the API Explorer
Our [Tenable.io API Explorer](ref:authentication) (based on <a href="https://swagger.io/docs/specification/about/" target="_blank">OpenAPI 3 specification</a>) provides complete reference documentation for all available Tenable.io API endpoints. It also allows you to try most of the API calls out of the box. You can run the calls against your Tenable.io environment.

To try a Tenable.io API call using the the API Explorer, simply navigate to the endpoint, enter the API keys and the input parameters for the call, and click **Try It**. Here is an example of using the API Explorer to list the assets in your environment:

![API Explorer API Key](https://files.readme.io/bd91eae-ExplorerAssetsResults.png "API Explorer API Key")

You can use the API Explorer for API reference information (for example, request parameters and response schemas), and also copy the generated code samples in the language of your choice. We currently provide samples in Python, cURL, Node, Ruby, JavaScript, Objective-C, Java, PHP, C#, Swift,  and Go.

## Use any REST client
The API Explorer can help you build a sufficient foundation so that you can then perform more complex requests with other tools such as cURL or Postman. As usual, authentication is necessary to make the requests for data work. Use your own API key.

Here is an example of how to upload a scan file:

```curl
curl -H \"X-APIKeys: accessKey=<access_key>;secretKey=<secret_key>\" -F \"Filedata=@~/nessus.db\" -X POST https://cloud.tenable.com/file/upload?no_enc=1
```

Once you have uploaded the file, import the scan:

```curl
curl -H \"X-APIKeys: accessKey=<access_key>;secretKey=<secret_key>\" -d '{\"file\":\"nessus.db\",\"folder_id\":<int_folder_id>,\"password\":\"SuperSecretP@ssw0rd\"}'  -X POST https://cloud.tenable.com/scans/import
```
Now launch the scan:

```curl
curl -H \"X-APIKeys: accessKey=<access_key>;secretKey=<secret_key>\" -X POST https://cloud.tenable.com/scans/<int_scan_id>/launch
```

These cURL requests can be easily modified for use in other REST clients or scripting languages.

## Use Tenable.io SDKs and tools
The optimal way to develop with Tenable.io REST APIs is to use one of our SDKs or client libraries. We recommend that you use these libraries both for testing and in production because they provide standard interfaces that handle authentication and request construction for you.

  * [Tenable.io Java SDK](https://github.com/tenable/Tenable.io-SDK-for-Java) -  The SDK provides a complete set of Java client interfaces for the Tenable.io REST services.
  * [Tenable.io Python SDK](https://github.com/tenable/Tenable.io-SDK-for-Python) - Use the Python SDK if you plan to develop only against the Tenable.io platform and are not integrating any other Tenable products. It is written in a highly object-oriented way and covers all documented API endpoints in Tenable.io.
  * [pyTenable Library](https://github.com/tenable/pyTenable) - pyTenable is a great choice if you intend to develop against additional Tenable products such as Security Center. pyTenable is a lightweight library that allows you to interact with the Tenable.io APIs in a pythonic, unambiguous way.

The GitHub repositories for these libraries provide detailed explanations for getting started.

## Get help: Tenable developer information ecosystem
* Before using the APIs, we recommend that you familiarize yourself with the [user documentation](https://docs.tenable.com/TenableIO.htm). There is a strong correlation between the business logic of Tenable.io UI and the API.
* Use the Guides section of this site to find detailed step-by-step instructions for specific use cases, for example, importing asset data and exporting scan data.
* Use the Expert Articles section for expert advice from Tenable developers, for example, about using our Python API tools.
* Use the API Explorer to try the API calls and find reference information: Endpoint URLs, HTTP methods, input parameters, response schemas, etc. We also provide client request code samples in multiple languages to help your get started.
* Use [Tenable.sc API documentation](https://docs.tenable.com/sccv/api/index.html) to find reference information for the latest version of Tenable.sc REST API.
* Use our [Github project](https://github.com/tenable) to get the code and documentation for Tenable.io SDKs and tools, as well extensibility points for other Tenable products (for example, NASL for Nessus plugin development).
* If you are unable to find the information that you are looking for, you may be able to get help in the [Tenable Community](https://community.tenable.com/s/topic/0TOf2000000HPDKGA4?tabset-a32df=2) portal. We encourage you to join our Community and participate in discussions.
