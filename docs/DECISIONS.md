# Documind project decisions

## Current phase: 1 - Document Pipeline development 

1. <b>Document PDFs stored locally or on the cloud: </b> Before starting the project, I have decided to download and store 10-K filings of 3 major companies locally. It will help me in saving cloud bills and for the early stage of pipeline development I can experiment with the PDFs. On a later stage I plan to shift the document storage on the cloud, for example, AWS S3 bucket.

2. <b>Qdrant locally or cloud: </b> Running Qdrant locally with Docker will be free without any limits. I decide to use Qdrant locally as well.

3. <b>Embedding Model choice: </b> Since this project primarily deals with financial documents, I have decided to use a Sentence Transformer (SBERT) with model fine-tuned on financial domain.

4. <b>LLM Choice: </b> My intention is to use a free-tier LLM API available publicly, for e.g., Google Gemini or Groq (Llama 3.3 70B). At this stage I decide to evaluate both of these LLMs and measure the latency per agent hop. In the end, only one LLM will be kept in the project and the same will be discussed in this document.

5. <b>Evaluation Storage: </b> All the evaluation metrics calculated by RAGAS and LLM-as-a-judge must be stored in a traditional database. Hence, I decide to use PostgreSQL for the same purpose. The complete schema is explained in the [Architecture Document](architecture.md).