# SDG Library Design

## Objective

Library called `instructlab-sdg` that can be called per seed example that includes question and answer pairs, and context for grounded skills.

## Structure of the SDG Library

We propose the following structure for the SDG library. There will be config files that contain all the prompt templates for the pipelines.

```markdown
- src/instructlab/sdg/
    - configs/ 
        - gen_q.yaml 
        - gen_a.yaml
        - ...
    - init.py 
    - block.py 
    - llmblock.py 
    - pipeline.py 
    - sdg.py
```

![example API interface](../images/sdg-api-interface.png)

## CLI

The CLI client uses the instructlab SDG library and provides it a run configuration with input parameters. The following represents a sample of what Library usage could look like.

```python
# cli_driver.py

from sdg import SDG
from run_config import SynthDataFlow
from pipeline import Pipeline
import yaml

client = openai_client(endpoint)
model = "model-version"

synth_skills_flow = SynthDataFlow(client, model).get_flow()
skills_pipe = Pipeline(synth_skills_flow)

cli_sdg = SDG([synth_skills_flow])  # run config has all the variables like num_samples, pipelinesteps etc
generated_samples = cli_sdg.generate()
```

As an initial integration milestone, we will modify the `generate_data` function in `instructlab.sdg.generate_data` to make use of the updated SDG API. This is the function the `ilab` CLI already uses, so modifying this implementation will allow us to get the updated SDG API in place without disrupting the CLI integration.

CLI integration will require additional changes later to allow passing in customizations to the SDG pipeline, but we will treat that as a follow-up implementation milestone.

The run configuration includes the necessary parameters for executing the SDG code library. It specifies the templates required for running the SDG code, the prompt template, and the default model system prompt template.

* `num_samples` is the number of synthetic samples that you wish to generate per seed example.
* `num_procs` is the number of parallel processes that you want to run
* `max_retry` is the maximum number of non-greedy retries you want to make if the `num_samples` is not reached. The number of samples in the generated output will be the samples achieved until `max_retry` is reached.
* Pipeline steps contains the steps that you want to invoke in the SDG pipeline and the prompt configurations per step. The variable names of the blocks can be anything and the prompt configurations must be compatible with the teacher model.
* `max_new_tokens` is the maximum number of tokens we want to generate. In other words, the size of the output sequence, not including the tokens in the prompt.
* `model` is the name of the served up teacher model we would want to use to generate the synthetic data.
* `model_prompt`: the default model prompt for the model.
* `client` points to an OpenAI client used to interface with the model. Example of a client:
  
  ```python
  client = OpenAI(
      api_key=openai_api_key,
      base_url=openai_api_base,
  )
  ```

```python
# run_config.py
class Flow(ABC):
    def __init__(self, client, model_id) -> None:
        self.client = client
        self.model_id = model_id
    
    @abstractmethod
    def get_flow(self) -> list:
        pass


class SynthDataFlow(Flow):
    def get_flow(self) -> list:
        return [
            {
                'block_type': LLMBlock,
                'block_config': {
                    'block_name': "gen_q",
                    'config_path': "configs/gen_q.yaml",
                    'client': self.client,
                    'model_id': self.model_id,
                    'model_prompt': '<s> [INST] {prompt} [/INST]',
                    'output_cols': ['question'],
                    'batch_kwargs': {
                        'num_procs': 8,
                        'num_samples': 30,
                        'batched': True,
                    },
                    'max_retry' : 5,
                    'max_new_tokens': 10000
```
