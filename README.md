# 🦜️🌐 WebLangChain
Using QianWen LLM via Tavily API to fetch webpage content and pass it to the LLM for generation.

## ✅ HuggingFaceEmbeddings
Load local models with HuggingFaceEmbeddings to save on invocation costs and circumvent network access restrictions.

## ✅ Custom Retriever Demo
YouRetriever demonstrates an example of retrieving information from multiple web pages, storing it in a vector database, and serving as a retriever.

## ✅ Running locally

By default, WebLangChain uses [Tavily](https://tavily.com) to fetch content from webpages. You can get an API key from [by signing up](https://tavily.com/).
If you'd like to add or swap in different base retrievers (e.g. if you want to use your own data source), you can update the `get_retriever()` method in `main.py`.

1. Install backend dependencies: `poetry install`.
2. Make sure to set your environment variables to configure the application:
```
export OPENAI_API_KEY=
export TAVILY_API_KEY=

# for Anthropic
# remove models from code if unused
ANTHROPIC_API_KEY=

# if you'd like to use the You.com retriever
export YDC_API_KEY=

# if you'd like to use the Google retriever
export GOOGLE_CSE_ID=
export GOOGLE_API_KEY=

# if you'd like to use the Kay.ai retriever
export KAY_API_KEY=

# for tracing
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_ENDPOINT="https://api.smith.langchain.com"
export LANGCHAIN_API_KEY=
export LANGCHAIN_PROJECT=
```

Under the hood, the chain is converted to a FastAPI server with various endpoints via [LangServe](https://github.com/langchain-ai/langserve).
This also includes a playground that you can use to interactively swap and configure various pieces of the chain.
You can find it running at `http://localhost:8080/chat/playground`.

## ⚙️ How it works

The general retrieval flow looks like this:

1. Pull in raw content related to the user's initial query using a retriever that wraps Tavily's Search API.
    - For subsequent conversation turns, we also rephrase the original query into a "standalone query" free of references to previous chat history.
2. Because the size of the raw documents usually exceed the maximum context window size of the model, we perform additional [contextual compression steps](https://python.langchain.com/docs/modules/data_connection/retrievers/contextual_compression/) to filter what we pass to the model.
    - First, we split retrieved documents using a [text splitter](https://python.langchain.com/docs/modules/data_connection/document_transformers/).
    - Then we use an [embeddings filter](https://python.langchain.com/docs/modules/data_connection/retrievers/contextual_compression/#embeddingsfilter) to remove any chunks that do not meet a similarity threshold with the initial query.
3. The retrieved context, the chat history, and the original question are passed to the LLM as context for the final generation.

Here's a LangSmith trace illustrating the above:

https://smith.langchain.com/public/f4493d9c-218b-404a-a890-31c15c56fff3/r

It's built using:

- [Tavily](https://tavily.com) as a retriever
- [LangChain](https://github.com/langchain-ai/langchain/) for orchestration
- [LangServe](https://github.com/langchain-ai/langserve) to directly expose LangChain runnables as endpoints
- [FastAPI](https://fastapi.tiangolo.com/)
- [Chroma](https://github.com/chroma-core/chroma) as embedding database

