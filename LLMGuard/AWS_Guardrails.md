# Using AWS Bedrock LLM GuardRails

LLM GuardRails are important Component in LLMOps Lifecycle. 

It help our LLM system to Mitigate Unexpected and Harmful Input interms of prompt and instructions. It Also provide additional layer of security to avoid PII leakeage.

In terms of LLM Output, it restricts LLM output if model is halucinating or generating something out of expectation.

## Step1. Create Guardrails in AWS bedrock Platform

![create_guardrails](images\create_guardrails.png)

## Step2. Provide initial details like name and description

![step1_provide_details](images\step1_provide_details.png)

## Step3. Configure content Filtering
This is an important configuaration step to prevent harmful user input and also avoid `Prompt Attack`

Prompt attack is something that enduser do interms of input to override Prompt instruction or making prompt instruction meaningless

![step2_configure_content_filter](images\step2_configure_content_filter.png)

We can select different categories with choice of level based on the need

![step2_1_configure_content_filter](images\step2_1_configure_content_filter.png)

## Step4. Add denied topics

Sometimes we want the scope of model to be restricted in certain topics and we can configure such behaviour in this page

![step3_add_denied_topics.png](images\step3_add_denied_topics.png)

## Step5. Add Word Filters

There might be some word interms of io of model that need to be filtered such behavior can be configured in this section

![step4_add_word_filters](images\step4_add_word_filters.png)

## Step6. Filter Sensitive Content Configuration

We can either `mask` or `block` PII Information in LLM Guardrail

We can also add custom Sensitive Information Regex with similar behaviour

![filter_PII_information_or_sensitive_content1](images\step5_filter_PII_information_or_sensitive_content.png)
![filter_PII_information_or_sensitive_content2](images\step5_2_filter_PII_information_or_sensitive_content.png)

## Step7. Context Grounding and Relavancy

We can directly filter output based on how context is used in response generation its called `Context Grounding`. Also based on standard Relavancy score also we can restrict the output

![Step6_context_grounding_relavancy](images\Step6_context_grounding_relavancy.png)

## Step8. Create Guardrail and Version it
By clicking on Create button it will create guardrail instance for you
![step7_create](images\step7_create.png)

We will also need to create version for using it via Bedrock python sdk
![step8_create_guardrail_version](images\step8_create_guardrail_version.png)

![step8_2_create_guardrail_version](images\step8_2_create_guardrail_version.png)

## Step9. Test The Guardrails

In the test section we will need to select one of the available model that we have access to

![testing_select_model](images\testing_select_model.png)

We can provide prompt and reference document and run the inference

![testing_select_model_v2](images\testing_select_model_v2.png)

We will need to click on `View Trace` to get indepth analysis on what guardrails is doing in background like what things its blocking and why
![testing_select_model_v3](images\testing_select_model_v3.png)

### Incorporate Guardrails while making LLM calls using langchain

Once we do configure the guardrails, we can directly  use it while calling AWS Bedrock API in python sdk version as well.

Its intigrated in the bedrock sdk objects. We just need to pass  
```
guardrails={
    "guardrailIdentifier": "#replace this with your guardrail id", 
    "guardrailVersion": "1", "trace": True
}
```
in the object creation as argument also we can add `BedrockAsyncCallbackHandler` for certain behaviour based on application

```python
from typing import Any

from langchain_core.callbacks import AsyncCallbackHandler
from langchain_aws import ChatBedrock
import boto3
import os
# Initialize boto3 client with bedrock-runtime to access bedrock related feature
bedrock_client = boto3.client(
    'bedrock-runtime',
    region_name="us-east-1",
    aws_access_key_id=os.environ.get('AWS_ACCESS_KEY_ID'),  # Retrieve from environment variables
    aws_secret_access_key=os.environ.get('AWS_SECRET_ACCESS_KEY'),
    aws_session_token=os.environ.get('AWS_SESSION_TOKEN', None)  # Optional session token
)

class BedrockAsyncCallbackHandler(AsyncCallbackHandler):
    # Async callback handler that can be used to handle callbacks from langchain.

    async def on_llm_error(self, error: BaseException, **kwargs: Any) -> Any:
        reason = kwargs.get("reason")
        if reason == "GUARDRAIL_INTERVENED":
            print(f"Guardrails: {kwargs}")


# Guardrails for Amazon Bedrock with trace
llm = ChatBedrock(
    # credentials_profile_name="bedrock-admin",
    model_id="mistral.mistral-7b-instruct-v0:2", 
    model_kwargs=dict(temperature=0),
    guardrails={"guardrailIdentifier": "#replace this with your guardrail id", "guardrailVersion": "1", "trace": True},
    callbacks=[BedrockAsyncCallbackHandler()],
    client=bedrock_client,
)
```

Note: 
1. Similar Code is available in Langchain offcial document but i modified and provided working version

2. I am still testing features of AWS Guardrails so i provided my initial understanding of each configuration of LLM Guard. 


Detailed implentation is provided in [AWS_Bedrock_Guardrails.ipynb](AWS_Bedrock_Guardrails.ipynb) notebook


## Reference
* [langchain-bedrock-doc](https://python.langchain.com/v0.2/docs/integrations/llms/bedrock/)