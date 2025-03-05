# Exercise 1: Creating a Basic AI Agent with Azure OpenAI

### Estimated Duration: 60 Minutes

## Lab Scenario

In this exercise, you'll create a basic AI Agent powered by Azure OpenAI's GPT-4 and deploy it as an API within an hour. You'll begin by understanding how AI Agents autonomously process prompts to generate intelligent responses, then build your agent using GPT-4. Next, you'll deploy the AI Agent as an Azure Function to expose it via an API, enabling easy integration with other services. Finally, you'll test the functionality of your agent using both the Python SDK and REST API to ensure it responds accurately.

## Lab Objectives

After completing this exercise, you will:

- Understand AI Agents and how they process prompts autonomously
- Setting up the Visual Studio Code
- Deploy the AI Agent as an Azure Function with API access
- Test AI Agent Responses using REST API

## Task 1: Understand AI Agents and how they process prompts autonomously

In this read-only lab, you'll explore the fundamentals of AI Agents and understand how they autonomously process prompts to generate intelligent responses.

### What is an AI Agent?

At its core, an AI Agent is a computer program that uses artificial intelligence to interact with you in natural language. It works by taking in your words, breaking them down into smaller pieces, and then using its pre-learned knowledge to generate a response. In simple terms, it’s like having a digital assistant that can:

- **Listen to Your Input:** Capture your words whether you type or speak them.

- **Understand Your Request:** Analyze the words by breaking them down (a process called tokenization) to get the main idea.

- **Generate an Answer:** Use patterns learned from massive amounts of text to craft a response that is relevant and easy to understand.

### How Do AI Agents Process Prompts?

The journey from your input to the final answer involves several clear steps:

1. **Tokenization:** When you provide a prompt, the agent first splits your sentence into smaller parts, known as tokens. This makes it easier for the system to analyze each element of your message.

   **Example:** For the sentence "What's the weather like today?" the agent identifies key tokens like "weather" and "today."

2. **Contextual Analysis:** After tokenization, the agent examines these tokens to understand the context and intent behind your words. This step is crucial because it allows the AI to grasp the meaning, even if the phrasing is different from what it has seen before.

3. **Response Generation:** Once the meaning is clear, the AI Agent uses its trained model (like GPT-4) to predict and generate a suitable response. It pieces together the answer by considering what it has learned from a vast range of data.

4. **Delivering the Answer:** Finally, the AI Agent sends the response back to you—often in just a few seconds—making the interaction feel both natural and instantaneous.

### Real-World Use Cases of AI Agents

- **Customer Support Chatbots:** AI agents handle customer inquiries, answer FAQs, and resolve common issues 24/7, reducing wait times and improving service quality.

- **Fraud Detection in Finance:** AI agents monitor transactions in real time to identify unusual patterns and flag potential fraudulent activities, enhancing security in banking and financial services.

## Task 2: Setting up the Visual Studio Code

In this task, you will configure the visual studio code with Azure credentials and create a new project for api.

1. From the JumpVM, double click on **Visual Studio Code** icon.

1. Once the Visual Studio Code is open, on the ***Welcome*** pane, click on **Open Folder**.

   ![](./media/ex1img1.png)

1. Navigate to `C:\LabFiles\agent-api\` and click on **Select Folder**.

   ![](./media/ex1img2.png)

1. Now from the **Visual Studio Code** pane left menu, select **Azure** icon and click on **Sign in to Azure**.

   ![](./media/ex1img3.png)

1. In the pop up window, click om **Allow**.

   ![](./media/ex1img4.png)

1. Use the following credentials to sign in:

- **Useremail:** **<inject key="AzureAdUserEmail"></inject>**

   ![](./media/ex1img5.png)

- **Password:** **<inject key="AzureAdUserEmail"></inject>**

   ![](./media/ex1img6.png)

1. Once the authentication is completed, select **View** from the top menu and click on **Command Palette**.

   ![](./media/ex1img7.png)

1. In the command palette, search and select **Azure Functions: Create New Project**.

   ![](./media/ex1img8.png)

1. Now in the next step, select **agent-api** folder, where the project will be initiated.

   ![](./media/ex1img9.png)

1. In the **Select a language** pane, choose **Python**.

   ![](./media/ex1img10.png)

1. In the next pane, select **Model V2**.

   ![](./media/ex1img11.png)

1. In the next pane, select **HTTP trigger** as the template to create the function.

   ![](./media/ex1img12.png)

1. In the input box, provide the function name as `agentapi`.

   ![](./media/ex1img13.png)

1. Once the function is created you will be able to see the starter codefiles.

1. In the **Visual Studio Code** pane, select **function_app.py** file and replace he existing code by this:

   ```
   import azure.functions as func
   import logging
   import os
   import requests

   app = func.FunctionApp(http_auth_level=func.AuthLevel.ANONYMOUS)

   @app.route(route="agent", methods=["GET", "POST"])
   def agent(req: func.HttpRequest) -> func.HttpResponse:
      logging.info("Processing request for GPT‑4 agent.")

      # Try to get the 'message' parameter from the query string.
      user_message = req.params.get("message")
      
      # If not provided via GET, check if it's provided in the JSON body (for POST)
      if not user_message and req.method == "POST":
         try:
               req_body = req.get_json()
         except ValueError:
               return func.HttpResponse("Invalid JSON payload.", status_code=400)
         else:
               user_message = req_body.get("message")
      
      if not user_message:
         return func.HttpResponse(
               "Please provide a 'message' parameter (in the query string or request body).",
               status_code=400
         )
      
      # Retrieve your API key from the environment variables.
      api_key = os.environ.get("OPENAI_API_KEY")
      if not api_key:
         return func.HttpResponse("API key not configured.", status_code=500)
      
      # Define the Azure OpenAI endpoint URL.
      url = ("<OPENAI ENDPOINT>")
      
      headers = {
         "Content-Type": "application/json",
         "api-key": api_key
      }
      
      payload = {
         "messages": [
               {"role": "system", "content": "You are a helpful assistant."},
               {"role": "user", "content": user_message}
         ]
      }
      
      # Post the request to the OpenAI endpoint.
      response = requests.post(url, headers=headers, json=payload)
      
      if response.status_code != 200:
         logging.error(f"Error calling OpenAI API: {response.status_code} - {response.text}")
         return func.HttpResponse("Error calling OpenAI API.", status_code=response.status_code)
      
      result = response.json()
      # Extract the assistant's reply from the API response.
      assistant_reply = result.get("choices", [{}])[0].get("message", {}).get("content", "No response")
      
      return func.HttpResponse(assistant_reply, status_code=200)
   ```

   ![](./media/ex1img14.png)

1. Once updated use **CTRL + S** to save the updated file.

1. Select **requirement.txt** file and add **requests** package to it as shown.

   ![](./media/ex1img16.png)

1. Save the file once updated.

1. As you have updated the files, navigate to the Azure Portal, which you have already opened previously.

1. Once you are in the Azure Portal, scroll down and select **Resource groups** under Navigate.

   ![](./media/ex1img17.png)

1. From the list select **agent-<inject key="DeploymentID" enableCopy="false"/>** resource group.

   ![](./media/ex1img18.png)

1. From the resource list, select **openai-<inject key="DeploymentID" enableCopy="false"/>**.

   ![](./media/ex1img19.png)

1. Once you are in the Azure OpenAI overview page, click on **Go to Azure Foundry portal**. You will be navigated to Azure AI Foundry Portal.

   ![](./media/ex1img20.png)

1. In the AI Foundry Portal, select **Deployments (1)** from the left menu and click on **gpt-40** model that is predeployed.

   ![](./media/ex1img21.png)

1. From the **Details** pane, copy **Target URI**, **Key** value and note these values down safely in a notepad.

   ![](./media/ex1img22.png)

1. Navigate back to the resource group page, from the resource list, select the storage account. The name starts as **agent** with some suffix.

   ![](./media/ex1img23.png)

1. From overview pane, select **Access Keys (1)**, click on **Show (2)** and copy **Connection string (3)** value.

   ![](./media/ex1img24.png)

1. From the resource list, select **agent-function-<inject key="DeploymentID" enableCopy="false"/>** function app.

   ![](./media/ex1img30.png)

1. From the left menu, select **Environment variables (1)** and click on **+ Add (2)**. 

   ![](./media/ex1img33.png)

1. In **Add/Edit application setting** pane, add **Name** as `OPENAI_API_KEY` and **Value** with the **API Key** value that you copied from AI Foundry Portal and click on **Apply**.

   ![](./media/ex1img34.png)

1. Again click on **Apply** to save the environment variable.

   ![](./media/ex1img35.png)

1. Now, navigate back to **Visual Studio Code** and open **function_app.py** file.

1. In the **function_app.py** file, update the **\<OPENAI ENDPOINT>**  value with the actual **Target URI** value that you have copied earlier and save the file.

   ![](./media/ex1img26.png)

### Task 3: Deploy the AI Agent as an Azure Function with API access

In this task, we will deploy our locally developed AI Agent function from Visual Studio Code to an Azure Function App, thereby making it accessible via an API endpoint. You will learn how to prepare your project for deployment, connect Visual Studio Code to your Azure account, publish your function to the cloud, and verify the deployment in the Azure Portal.

1. In the **Visual Studio Code**, select **View** from the top menu and click on **Command Palette**.

   ![](./media/ex1img7.png)

1. In the command palette, search and select **Azure Functions: Deploy to Function App**.

   ![](./media/ex1img27.png)

1. In the next pane, select function app **agent-function-<inject key="DeploymentID" enableCopy="false"/>**.

   ![](./media/ex1img28.png)

1. In the pop up window, click on **Deploy**.

   ![](./media/ex1img29.png)

1. Wait for sometime, you will see a success message once the deployment succeeds.

### Task 4: Test AI Agent Responses using REST API

In this task, we will test the deployed AI Agent API using a REST API testing tool Hoppscotch(Postwoman). You will configure the tool with your Azure Function App endpoint, set the necessary HTTP headers and payload, and send a test request to verify that the agent processes the input correctly and returns an appropriate response. This task ensures that your API is functioning as expected and provides a baseline for further integration and troubleshooting.

1. As you have successfully deployed agent, now its time to test it.

1. Navigate to [Hoppscotch](https://hoppscotch.io) to test your API. Hoppscotch is a free, open-source API development tool that simplifies testing, debugging, and documenting APIs.

1. Once you are in the Hoppscotch portal, the UI will look similar to this.

   ![](./media/ex1img39.png)

1. Now paste this cURL in the input box. Now a Import cURL pane will open, review the URL and click on **Import**.

   ![](./media/ex1img36.png)
   
   > In this request, you are doing a **POST** request to the API which is running in the Function App with the message `tell me a joke`. This will query the GPT model with the message and gets back the response.

1. Once imported, the portal look like this. Click on **Send** to send the request.

   ![](./media/ex1img37.png)

1. Now you can see the response from the Agent API which is running in the Function App.

   ![](./media/ex1img38.png)

## Summary

In this exercise, you have explored AI Agents, configured Visual Studio Code with Azure credentials to create an API project, deployed your AI Agent function to an Azure Function App, and tested the API using Hoppscotch.






