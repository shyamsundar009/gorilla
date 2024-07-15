# Berkeley Function Calling Leaderboard (BFCL)

💡 Read more in our [Gorilla OpenFunctions Leaderboard Blog](https://gorilla.cs.berkeley.edu/blogs/8_berkeley_function_calling_leaderboard.html)

🦍 Berkeley Function Calling Leaderboard live [Berkeley Function Calling Leaderboard](https://gorilla.cs.berkeley.edu/leaderboard.html#leaderboard)

🦍 Berkeley Function Calling Leaderboard on Hugginface [Berkeley Function Calling Leaderboard Huggingface](https://huggingface.co/spaces/gorilla-llm/berkeley-function-calling-leaderboard)

## Introduction
We introduce the Berkeley Function Leaderboard (BFCL), the **first comprehensive and executable function call evaluation dedicated to assessing Large Language Models' (LLMs) ability to invoke functions**. Unlike previous function call evaluations, BFCL accounts for various forms of function calls, diverse function calling scenarios, and their executability. Additionally, we release Gorilla-Openfunctions-v2, the most advanced open-source model to date capable of handling multiple languages, parallel function calls, and multiple function calls simultaneously. A unique debugging feature of this model is its ability to output an "Error Message" when the provided function does not suit your task.

Read more about the technical details and interesting insights in our blog post!

![image](./architecture_diagram.png)

### Install Dependencies

```bash
conda create -n BFCL python=3.10
conda activate BFCL
pip install -r requirements.txt # Inside gorilla/berkeley-function-call-leaderboard
pip install vllm # If you have vLLM supported GPU(s) and want to run our evaluation data against self-hosted OSS models.
```

## Prepare Evaluation Dataset

To download the evaluation dataset from huggingface, from the current directory `./berkeley-function-call-leaderboard`, run the following command:

```bash
huggingface-cli download gorilla-llm/Berkeley-Function-Calling-Leaderboard --local-dir ./data --repo-type dataset
```

This will download our dataset to `data` repository. 

## Evaluation Dataset

The evaluation datasets are now stored in the `./data` folder. The possible answers are stored in the `./data/possible_answer` folder. 

## Execution Evaluation Data Post-processing 
Input your API keys into `function_credential_config.json`, so that the original placeholder values in questions, params, and answers will be cleaned. 

To run the executable test categories, there are 4 API keys to include:

1. RAPID-API Key: https://rapidapi.com/hub

    * Yahoo Finance: https://rapidapi.com/sparior/api/yahoo-finance15
    * Real Time Amazon Data : https://rapidapi.com/letscrape-6bRBa3QguO5/api/real-time-amazon-data
    * Urban Dictionary: https://rapidapi.com/community/api/urban-dictionary
    * Covid 19: https://rapidapi.com/api-sports/api/covid-193
    * Time zone by Location: https://rapidapi.com/BertoldVdb/api/timezone-by-location

    All the Rapid APIs we use have free tier usage. As a result, you need to subscribe to those API providers in order to have the executable test environment setup but it will be free of charge!

2. Exchange Rate API:https://www.exchangerate-api.com
3. OMDB API: http://www.omdbapi.com/apikey.aspx
4. Geocode API: https://geocode.maps.co/

The `apply_function_credential_config.py` takes an input and optionally an output file. If the output file is not given as an argument, it will overwrite your original file with the data reset.

```bash
python apply_function_credential_config.py --input-file data/gorilla_openfunctions_v1_test_rest.json
```

Then, use `eval_data_compilation.py` to compile all files by

```bash
python eval_data_compilation.py
```
## Berkeley Function-Calling Leaderboard Statistics


Make sure models API keys are included in your environment variables.

```bash
export OPENAI_API_KEY=sk-XXXXXX
export MISTRAL_API_KEY=XXXXXX
export FIRE_WORKS_API_KEY=XXXXXX
export ANTHROPIC_API_KEY=XXXXXX
export COHERE_API_KEY=XXXXXX
export NVIDIA_API_KEY=nvapi-XXXXXX
```

To generate leaderboard statistics, there are two steps:

1. LLM Inference of the evaluation data from specific models

```bash
python openfunctions_evaluation.py --model MODEL_NAME --test-category TEST_CATEGORY
```
For TEST_CATEGORY, we have `executable_simple`, `executable_parallel_function`, `executable_multiple_function`, `executable_parallel_multiple_function`, `simple`, `relevance`, `parallel_function`, `multiple_function`, `parallel_multiple_function`, `java`, `javascript`, `rest`, `sql`, `chatable`.

If you want to run all evaluations at the same time, you can use `all` as the test category.

Running proprietary models like GPTs, Claude, Mistral-X will require an API-Key which can be supplied in `openfunctions_evaluation.py`.

If decided to run OSS model, openfunction evaluation uses vllm and therefore requires GPU for hosting and inferencing. If you have questions or concerns about evaluating OSS models, please reach out to us in our [discord channel](https://discord.gg/grXXvj9Whz).




## Checking the Evaluation Results

### Running the Checker

Navigate to the `gorilla/berkeley-function-call-leaderboard/eval_checker` directory and run the `eval_runner.py` script with the desired parameters. The basic syntax is as follows:

```bash
python eval_runner.py --model MODEL_NAME --test-category {TEST_CATEGORY,all,ast,executable,python,non-python}
```

- `MODEL_NAME`: Optional. The name of the model you wish to evaluate. This parameter can accept multiple model names separated by spaces. Eg, `--model gorilla-openfunctions-v2 gpt-4-0125-preview`.
    - If no model name is provided, the script will run the checker on all models exist in the `result` folder. This path can be changed by modifying the `INPUT_PATH` variable in the `eval_runner.py` script.
- `TEST_CATEGORY`: Optional. The category of tests to run. You can specify multiple categories separated by spaces. Available options include:
    - `all`: Run all test categories.
    - `ast`: Abstract Syntax Tree tests.
    - `executable`: Executable code evaluation tests.
    - `python`: Tests specific to Python code.
    - `non-python`: Tests for code in languages other than Python, such as Java and JavaScript.
    - Individual test categories:
        - `simple`: Simple function calls.
        - `parallel_function`: Multiple function calls in parallel.
        - `multiple_function`: Multiple function calls in sequence.
        - `parallel_multiple_function`: Multiple function calls in parallel and in sequence.
        - `executable_simple`: Executable function calls.
        - `executable_parallel_function`: Executable multiple function calls in parallel.
        - `executable_multiple_function`: Executable multiple function calls in sequence.
        - `executable_parallel_multiple_function`: Executable multiple function calls in parallel and in sequence.
        - `java`: Java function calls.
        - `javascript`: JavaScript function calls.
        - `rest`: REST API function calls.
        - `relevance`: Function calls with irrelevant function documentation.
    - If no test category is provided, the script will run all available test categories.
> If you want to run the `all` or `executable` or `python` category, make sure to register your REST API keys in `function_credential_config.json`. This is because Gorilla Openfunctions Leaderboard wants to test model's generated output on real world API! 

> If you do not wish to provide API keys for REST API testing, set `test-category` to `ast` or any non-executable category.

> By setting the `--api-sanity-check` flag, or `-c` for short, if the test categories include `executable`, the evaluation process will perform the REST API sanity check first to ensure that all the API endpoints involved during the execution evaluation process are working properly. If any of them are not behaving as expected, we will flag those in the console and continue execution.

### Example Usage

If you want to run all tests for the `gorilla-openfunctions-v2` model, you can use the following command:

```bash
python eval_runner.py --model gorilla-openfunctions-v2

```

If you want to evaluate all offline tests (do not require RapidAPI keys) for OpenAI GPT-3.5, you can use the following command:

```bash
python eval_runner.py --model gpt-3.5-turbo-0125 --test-category ast
```

If you want to run `rest` tests for all GPT models, you can use the following command:

```bash
python eval_runner.py --model gpt-3.5-turbo-0125 gpt-4-0613 gpt-4-1106-preview gpt-4-0125-preview --test-category rest
```

If you want to run `rest` and `javascript` tests for all GPT models and `gorilla-openfunctions-v2`, you can use the following command:

```bash
python eval_runner.py --model gorilla-openfunctions-v2 gpt-3.5-turbo-0125 gpt-4-0613 gpt-4-1106-preview gpt-4-0125-preview --test-category rest javascript
```

### Model-Specific Optimization

Some companies have proposed some optimization strategies in their models' handler, which we (BFCL) think is unfair to other models, as those optimizations are not generalizable to all models. Therefore, we have disabled those optimizations during the evaluation process by default. You can enable those optimizations by setting the `USE_{COMPANY}_OPTIMIZATION` flag to `True` in the `model_handler/constants.py` file.


## Models Available
Below is *a table of models we support* to run our leaderboard evaluation against. If supported function calling (FC), we will follow its function calling format provided by official documentation. Otherwise, we will construct a system message to prompt the model to generate function calls in the right format.
|Model | Type |
|---|---|
|gorilla-openfunctions-v2 | Function Calling|
|claude-3-{opus-20240229,sonnet-20240229,haiku-20240307}-FC | Function Calling |
|claude-3-{opus-20240229,sonnet-20240229,haiku-20240307} | Prompt |
|claude-3-5-sonnet-20240620-FC | Function Calling |
|claude-3-5-sonnet-20240620 | Prompt |
|claude-{2.1,instant-1.2}| Prompt|
|command-r-plus-FC | Function Calling|
|command-r-plus | Prompt|
|databrick-dbrx-instruct | Prompt|
|deepseek-ai/deepseek-coder-6.7b-instruct 💻| Prompt|
|firefunction-{v1,v2}-FC | Function Calling|
|gemini-1.0-pro | Function Calling|
|gemini-1.5-pro-preview-{0409,0514} | Function Calling|
|gemini-1.5-flash-preview-0514 | Function Calling|
|glaiveai/glaive-function-calling-v1 💻| Function Calling|
|gpt-3.5-turbo-0125-FC| Function Calling|
|gpt-3.5-turbo-0125| Prompt|
|gpt-4-{0613,1106-preview,0125-preview,turbo-2024-04-09}-FC| Function Calling|
|gpt-4-{0613,1106-preview,0125-preview,turbo-2024-04-09}| Prompt|
|gpt-4o-2024-05-13-FC | Function Calling|
|gpt-4o-2024-05-13| Prompt|
|google/gemma-7b-it 💻| Prompt|
|meetkai/functionary-{small,medium}-v2.4-FC| Function Calling|
|meetkai/functionary-small-v2.2-FC| Function Calling|
|meta-llama/Meta-Llama-3-{8B,70B}-Instruct | Prompt|
|mistral-large-2402-FC-{Any,Auto} | Function Calling|
|mistral-large-2402 | Prompt|
|mistral-medium-2312 | Prompt|
|mistral-small-2402-FC-{Any,Auto} | Function Calling|
|mistral-small-2402 | Prompt|
|mistral-tiny-2312 | Prompt|
|Nexusflow-Raven-v2 | Function Calling|
|NousResearch/Hermes-2-Pro-Mistral-7B 💻| Function Calling|
|snowflake/arctic | Prompt|
|nvidia/nemotron-4-340b-instruct| Prompt|
|THUDM/glm-4-9b-chat 💻| Function Calling|

Here {MODEL} 💻 means the model needs to be hosted locally and called by vllm, {MODEL} means the models that are called API calls. For models with a trailing `-FC`, it means that the model supports function-calling feature. You can check out the table summarizing feature supports among different models [here](https://gorilla.cs.berkeley.edu/blogs/8_berkeley_function_calling_leaderboard.html#prompt).


For model names with {.}, it means that the model has multiple versions. For example, we provide evaluation on three versions of GPT-4: `gpt-4-0125-preview`, `gpt-4-1106-preview`, and `gpt-4-0613`.

For Mistral large and small models, we provide evaluation on both of their `Any` and `Auto` settings. More information about this can be found [here](https://docs.mistral.ai/guides/function-calling/).


For inferencing `Gemini-1.0-pro`, you need to fill in `model_handler/gemini_handler.py` with your GCP project ID that has access to Vertex AI endpoint.
For inferencing `Databrick-DBRX-instruct`, you need to create a Databrick Azure workspace and setup an endpoint for inference.

## Contributing

To add a new model to the Function Calling Leaderboard, here are a few things you need to do:

1. Take a look at the `model_handler/handler.py`. This is the base handler object which all handlers are inherited from. Also, free feel to take a look at the existing model handers; very likely you can re-use some of the existing code if the new model outputs in a similar format.
2. Create your handler and define the following functions 
    1. `__init__`: on initialization, you need to create a `self.client` object if you have an existing endpoint(e.g. `self.client = OpenAI()`) or follow `model_handler/oss_handler.py` for starting a vLLM serving.
    2. `inference`: inference function takes in prompt, functions, as well as optional programming language parameters. It will make call to the endpoint, compile result in the desired format, as well as logging the token number and latency
    3. `decode_ast`: decode_ast will convert the response from raw output in the format of `[{func1:{param1:val1,...}},{func2:{param2:val2,...}}]` This format will be used to check for exact matching the parameters.
    4. `decode_execute`: deocde_execute will convert the response from raw output in the format of `"[func1(param1=val1),func2(param2=val2)]"`
3. Modify `model_handler/handler_map.py`. This mapping contains the key as the exact model name and value as the handler object of the specific model.
4. Modify `eval_checker/eval_runner_helper.py`:
    - Update the `MODEL_METADATA_MAPPING` with the model display name, URL, license and company information. The key should be the same as the one in `model_handler/handler_map.py`.
    - If your model is price-based, you should update the `INPUT_PRICE_PER_MILLION_TOKEN` and `OUTPUT_PRICE_PER_MILLION_TOKEN`. - If your model doesn't have a cost, you should add it to the `NO_COST_MODELS` list.
    - If your model is open-source and is hosted locally, the `OSS_LATENCY` list needs to be updated with the latency for the whole batch of data generation. This information will affect the cost calculation.
5. Raise a [Pull Request](https://github.com/ShishirPatil/gorilla/pulls) with your new Model Handler. We will run the model handler if an endpoint is established. If self-hosting is required and the model size is large, we might not be able to accommodate model hosting therefore an OpenAI compatible endpoint for evaluation is desired. 
6. Feel Free to join [Gorilla Discord](https://discord.gg/grXXvj9Whz) `#leaderboard` and reach out to us for any questions or concerns about adding new models. We are happy to help you!

All the leaderboard statistics, and data used to train the models are released under Apache 2.0.
Gorilla is an open source effort from UC Berkeley and we welcome contributors. 
Please email us your comments, criticisms, and questions. More information about the project can be found at [https://gorilla.cs.berkeley.edu/](https://gorilla.cs.berkeley.edu/)
