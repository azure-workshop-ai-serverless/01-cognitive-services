# Application step 01 - Azure Function and SignalR real-time application.


1. Signal IR and Azure Functions. Steps below following official MS tutorial with important details
	https://docs.microsoft.com/en-us/azure/azure-signalr/signalr-quickstart-azure-functions-csharp

2. Proceed with a demo chat application from this tutorial clone tutorial repository.
	https://github.com/Azure-Samples/signalr-service-quickstart-serverless-chat.git
	6. Azure functions application located here
	/src/chat/csharp

3. The local web application to edit is located here.
	/docs/demo/chat-v2

4. Copy SignalR primary connection string from notepad 

5. In Visual Studio, in Solution Explorer, rename local.settings.sample.json to local.settings.json.

6. In local.settings.json, paste the SignalR connection string into the value of the AzureSignalRConnectionString setting. Save the file.

7. Open Functions.cs. There are two HTTP triggered functions in this function app:
		a. GetSignalRInfo - Uses the SignalRConnectionInfo input binding to generate and return valid connection information.
		b. SendMessage - Receives a chat message in the request body and uses the SignalR output binding to broadcast the message to all connected client applications.

8. Visual Studio: In the Debug menu, select Start debugging to run the application.

9. You can deploy a function app to Azure via publishing profile if you want to.

10. Let's connect local and test remote applications.

11. There is a sample single page web application hosted in GitHub for your convenience. Open your browser to https://azure-samples.github.io/signalr-service-quickstart-serverless-chat/demo/chat-v2/.

12. When prompted for the function app base URL, enter http://localhost:7071.

13. Enter a username when prompted.

14. The web application calls the GetSignalRInfo function in the function app to retrieve the connection information to connect to Azure SignalR Service. When the connection is complete, the chat message input box appears.

15. Type a message and press enter. The application sends the message to the SendMessage function in the Azure Function app, which then uses the SignalR output binding to broadcast the message to all connected clients. If everything is working correctly, the message should appear in the application.

16. Open another instance of the web application in a different browser window. You will see that any messages sent will appear in all instances of the application.
    
Be aware of the cors issues and HTTP vs. HTTPS usage. My suggestion is to deploy the function to the Web app and setup CORS exceptions in the configuration.
