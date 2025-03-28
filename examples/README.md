# Protein Similarity Search in Snowflake with NVIDIA BioNeMo
<img width="1308" alt="313891216-73e0e27a-897e-4011-887a-73def37daf8b" src="https://github.com/user-attachments/assets/15302a5a-5274-4ea8-bb12-42dd048754c5" />

Inspired by https://github.com/aws-samples/protein-similarity-search?tab=readme-ov-file
and these others:
1.  [https://developer.nvidia.com/blog/nvidia-nim-offers-optimized-inference-microservices-for-deploying-ai-models-at-scale/](https://developer.nvidia.com/blog/nvidia-nim-offers-optimized-inference-microservices-for-deploying-ai-models-at-scale/)
2.  [https://www.snowflake.com/blog/fast-easy-secure-llm-app-development-snowflake-cortex/](https://www.snowflake.com/blog/fast-easy-secure-llm-app-development-snowflake-cortex/)


Conceptual Architecture
![1_CpRXfArKD43-M8awYa8h8A](https://github.com/user-attachments/assets/72ed2ef2-731d-4000-9cef-a5d5fdf87a50)

For this demo, we initially loaded 570K UniProtIDs with corresponding protT5 embeddings stored in a column with a VECTOR(FLOAT, 1024)) data type in a Snowflake table, called PROTEINS:
![1_hTxZaady9JsOaPllPF5DrA](https://github.com/user-attachments/assets/a152d3ed-75cc-49b6-a335-6c596abde1ae)

We chose two smaller, sample proteins in a fasta file (as shown below) as base proteins to compare to the protein structures with pre-calculated embeddings in the PROTEINS table.
![1_OqY6eA-iQHgbaemvMJ7eUw](https://github.com/user-attachments/assets/99074787-45e5-4b0d-9e38-9cb8a58cf978)

/protT5/proteins.fasta

Let’s look at the individual components to identify similar proteins:

1.  **protT5 Embedding Generation:** We deployed a Jupyter service in Snowpark Container Services and executed the code using GPU\_NV\_M, 4 NVIDIA A10G s, where we brought in the Encoder only [ProtT5-XL-UniRef50, half-precision model](https://huggingface.co/Rostlab/prot_t5_xl_half_uniref50-enc) from HuggingFace. We stored sample base proteins in /protT5/proteins.fasta location in our stage, and created embeddings from the last hidden layer of the model. We saved the embeddings into an h5 file in a Snowflake stage that is mounted to the service.
2.  **Identification of similar proteins:** Using the h5 file that is generated for the base proteins, we used Snowflake Vector Search pre-built function to identify the Top 3 matching proteins in the PROTEINS table.
3.  **BioNeMo Service Inference:** To visualize each protein, we made synchronous and asynchronous calls (based on the size of the protein) to the [BioNemo Python client](https://pypi.org/project/bionemo/) to generate PDB files for each protein structure and stored the files in a stage location. (You will need a BioNeMo API key to use the Python client.)

![1_VkBLdqAcZy6T3BhdXDa7dQ](https://github.com/user-attachments/assets/5534b03c-db91-474a-9b2d-5d6321f01f61)
Services Created in Snowpark Container Services

After all the steps are complete, we then visualize the proteins in the Streamlit UI that runs in Snowpark Container Services as shown below:
[![YouTube](http://i.ytimg.com/vi/csC7Hnps480/hqdefault.jpg)](https://www.youtube.com/watch?v=csC7Hnps480)
Streamlit UI running in Snowpark Container Services

What’s Next?
------------

Everything runs within Snowflake in this demo, with the exception of the BioNeMo service and models, where we make external API calls for model inference. We are very excited about the [NVIDIA Inference Microservices (NIM) announcement](https://developer.nvidia.com/blog/nvidia-nim-offers-optimized-inference-microservices-for-deploying-ai-models-at-scale/) at the GTC 2024 that will allow us to bring these inference calls via NIMs into Snowpark Container Services, all within Snowflake’s security perimeter.

