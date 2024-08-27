# Using AWS Bedrock LLM GuardRails

LLM GuardRails are an important Component in LLMOps Lifecycle. 

It helps our LLM system to Mitigate Unexpected and Harmful Input in terms of prompts and instructions. It also provides an additional layer of security to avoid PII leakage.

In terms of LLM Output, it restricts LLM output if the model is hallucinating or generating something out of expectation.

## Step1. Create Guardrails in AWS bedrock Platform

![create_guardrails](/LLMGuard/images/create_guardrails.png)

## Step2. Provide initial details like name and description

![step1_provide_details](/LLMGuard/images/step1_provide_details.png)

## Step3. Configure content Filtering
This is an important configuration step to prevent harmful user input and also avoid a `Prompt Attack`

The prompt attack is something that the end user does in terms of input to override Prompt instruction or make prompt instruction meaningless

![step2_configure_content_filter](/LLMGuard/images/step2_configure_content_filter.png)

We can select different categories with a choice of level based on the need

![step2_1_configure_content_filter](/LLMGuard/images/step2_1_configure_content_filter.png)

## Step4. Add denied topics

Sometimes we want the scope of the model to be restricted to certain topics and we can configure such behaviour on this page

![step3_add_denied_topics.png](/LLMGuard/images/step3_add_denied_topics.png)

## Step5. Add Word Filters

There might be some word in terms of io of the model that needs to be filtered such behavior can be configured in this section

![step4_add_word_filters](/LLMGuard/images/step4_add_word_filters.png)

## Step6. Filter Sensitive Content Configuration

We can either `mask` or `block` PII Information in LLM Guardrail

We can also add custom Sensitive Information Regex with similar behavior

![filter_PII_information_or_sensitive_content1](/LLMGuard/images/step5_filter_PII_information_or_sensitive_content.png)
![filter_PII_information_or_sensitive_content2](/LLMGuard/images/step5_2_filter_PII_information_or_sensitive_content.png)

## Step7. Context Grounding and Relevancy

We can directly filter output based on how context is used in response generation it's called `Context Grounding`. Also based on the standard Relevancy score also we can restrict the output

![Step6_context_grounding_relavancy](/LLMGuard/images/Step6_context_grounding_relavancy.png)

## Step8. Create Guardrail and Version it
By clicking on the Create button it will create a guardrail instance for you
![step7_create](/LLMGuard/images/step7_create.png)

We will also need to create a version for using it via Bedrock python SDK
![step8_create_guardrail_version](/LLMGuard/images/step8_create_guardrail_version.png)

![step8_2_create_guardrail_version](/LLMGuard/images/step8_2_create_guardrail_version.png)

## Step9. Test The Guardrails

In the test section, we will need to select one of the available models that we have access to

![testing_select_model](/LLMGuard/images/testing_select_model.png)

We can provide prompt and reference documents and run the inference

![testing_select_model_v2](/LLMGuard/images/testing_select_model_v2.png)

We will need to click on `View Trace` to get an in-depth analysis of what guardrails are doing in the background like what things are blocking and why
![testing_select_model_v3](/LLMGuard/images/testing_select_model_v3.png)

### Incorporate Guardrails while making LLM calls using langchain

Once we do configure the guardrails, we can directly  use it while calling AWS Bedrock API in the Python SDK version as well.

It's integrated into the bedrock SDK objects. We need to pass  
```
guardrails={
    "guardrailIdentifier": "#replace this with your guardrail id", 
    "guardrailVersion": "1", "trace": True
}
```
in the object creation as an argument also we can add `BedrockAsyncCallbackHandler` for certain behaviors based on the application

```python
from typing import Any

from langchain_core.callbacks import AsyncCallbackHandler
from langchain_aws import ChatBedrock
import boto3
import os
# Initialize boto3 client with bedrock-runtime to access bedrock-related feature
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
1. A Similar Code is available in the Langchain official document but I modified and provided a working version

2. I am still testing features of AWS Guardrails so I provided my initial understanding of each configuration of LLM Guard. 


Detailed implentation is provided in [AWS_Bedrock_Guardrails.ipynb](AWS_Bedrock_Guardrails.ipynb) notebook


## Reference
* [langchain-bedrock-doc](https://python.langchain.com/v0.2/docs/integrations/llms/bedrock/)
