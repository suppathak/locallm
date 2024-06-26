# Code Generation Application

This demo provides a simple recipe to help developers start building out their own custom LLM enabled code generation applications. It consists of two main components; the Model Service and the AI Application.

There are a few options today for local Model Serving, but this recipe will use [`llama-cpp-python`](https://github.com/abetlen/llama-cpp-python) and their OpenAI compatible Model Service. There is a Containerfile provided that can be used to build this Model Service within the repo, [`model_servers/llamacpp_python/base/Containerfile`](/model_servers/llamacpp_python/base/Containerfile).

Our AI Application will connect to our Model Service via it's OpenAI compatible API. In this example we rely on [Langchain's](https://python.langchain.com/docs/get_started/introduction) python package to simplify communication with our Model Service and we use [Streamlit](https://streamlit.io/) for our UI layer. Below please see an example of the code generation application.               


![](/assets/codegen_ui.png) 


# Build the Application

In order to build this application we will need a model, a Model Service and an AI Application.  

* [Download a model](#download-a-model)
* [Build the Model Service](#build-the-model-service)
* [Deploy the Model Service](#deploy-the-model-service)
* [Build the AI Application](#build-the-ai-application)
* [Deploy the AI Application](#deploy-the-ai-application)
* [Interact with the AI Application](#interact-with-the-ai-application)

### Download a model

If you are just getting started, we recommend using a [Mistral-7B](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.1) based model that's been fine-tuned for code generation. Mistral-7B is a well performant mid-sized model with an apache-2.0 license. In order to use it with our Model Service we need it converted and quantized into the [GGUF format](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md). There are a number of ways to get a GGUF version of Mistral-7B based models, but the simplest is to download a pre-converted one from [huggingface.co](https://huggingface.co). For this demo we recommend using https://huggingface.co/TheBloke/Mistral-7B-Code-16K-qlora-GGUF.  

There are a number of options for quantization level, but we recommend `Q4_K_M`. 

The recommended model can be downloaded using the code snippet below:

```bash
cd models
wget https://huggingface.co/TheBloke/Mistral-7B-Code-16K-qlora-GGUF/resolve/main/mistral-7b-code-16k-qlora.Q4_K_M.gguf
cd ../
```

_A full list of supported open models is forthcoming._  


### Build the Model Service

The complete instructions for building and deploying the Model Service can be found in the [the llamacpp_python model-service document](../model_servers/llamacpp_python/README.md).

The Model Service can be built from the root directory with the below code snippet:

```bash
cd model_servers/llamacpp_python
podman build -t llamacppserver -f base/Containerfile .
```


### Deploy the Model Service

The complete instructions for building and deploying the Model Service can be found in the [the llamacpp_python model-service document](../model_servers/llamacpp_python/README.md).

The local Model Service relies on a volume mount to the localhost to access the model files. You can start your local Model Service using the following podman command:  
```
podman run --rm -it \
        -p 8001:8001 \
        -v Local/path/to/locallm/models:/locallm/models \
        -e MODEL_PATH=models/<model-filename> \
        -e HOST=0.0.0.0 \
        -e PORT=8001 \
        llamacppserver
```

### Build the AI Application

Now that the Model Service is running we want to build and deploy our AI Application. Use the provided Containerfile to build the AI Application image from the `code-generation/` directory.

Run:

```bash
cd code-generation
podman build -t codegen . -f builds/Containerfile   
```
### Deploy the AI Application

Make sure the Model Service is up and running before starting this container image. When starting the AI Application container image we need to direct it to the correct `MODEL_SERVICE_ENDPOINT`. This could be any appropriately hosted Model Service (running locally or in the cloud) using an OpenAI compatible API. In our case the Model Service is running inside the podman machine so we need to provide it with the appropriate address `10.88.0.1`. The following podman command can be used to run your AI Application:  

```bash
podman run --rm -it -p 8501:8501 -e MODEL_SERVICE_ENDPOINT=http://10.88.0.1:8001/v1 codegen   
```

### Interact with the AI Application

Everything should now be up an running with the chat application available at [`http://localhost:8501`](http://localhost:8501). By using this recipe and getting this starting point established, users should now have an easier time customizing and building their own LLM enabled code generation applications.

_Note: Future recipes will demonstrate integration between locally hosted LLM's and developer productivity tools like VSCode._
