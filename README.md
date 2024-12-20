# Tandem Series LLM Training - Ongoing Research
License: CC-BY-SA

Part of an ongoing independant research project, whereby two LLMs are trained to operate in tandem, one feeding into the other in series.

I am working on creating and testing a step-by-step approach to fine-tuning a "composite" large language model using two models working in series. This could be seen as similar to a merge where all the layers of one model occur before all the layers of the second model, but allowing for observability at the in-between space, segmented training processes, and even the effective merging of different architectures. Perhaps a future version may even be able to effectively merge and train a composite of vastly different architectures such as Transformers and Mamba models, into functionally one single model.  The in-between space, where one model feeds into the other, can been viewed and thus may help stack processing abilities and allow a further window into interpretability. This may be seen as similar to the ensemble methods in machine learning, or ensemble of LLMs, however it is distinct in that it chains the LLMs together in series and thereby only looks at the output of a single LLM in the end (other than for interpretability and tweaking the interactions.)

For the first approach, "Single Dynamic", I have one static and one dynamic LLM, whereby one feeds into the next resulting in the desired result as well as an intermediary in-between point. For the second approach, "Dual Dynamic", I've modified the implementation to allow alternating training between the two models. This creates an adversarial-like training dynamic where each model takes turns learning to work with the other.

**Single Dynamic**

The problem involves fine-tuning a large language model (LLM) using a novel approach with two models:

1. **Static Model (a)**: This model remains unchanged during the fine-tuning process.
2. **Dynamic Model (b)**: This model is fine-tuned to learn how to communicate effectively with the static model (a).

The goal is to train model (b) to generate responses that, when fed into model (a), produce the desired output. This creates an "in-between space" where model (b) learns to adapt its output to elicit the correct response from model (a).

Step-by-Step Approach

To fine-tune model (b) using this approach, follow these steps:

1. **Prepare the training data**: Collect a dataset of input prompts and corresponding desired outputs. This dataset will be used to fine-tune model (b).
2. **Initialize model (b)**: Start with a pre-trained language model (e.g., BERT, RoBERTa, or a smaller variant of GPT-2 or GPT-J) as model (b).
3. **Freeze model (a)**: Ensure that model (a) remains unchanged throughout the fine-tuning process.
4. **Define the fine-tuning objective**: The objective is to minimize the difference between the output of model (a) and the desired output in the training data, given the input prompt and the response generated by model (b).
5. **Fine-tuning loop**:
	* Feed the input prompt from the training data to model (b).
	* Generate a response using model (b).
	* Feed the response from model (b) to model (a).
	* Compare the output of model (a) with the desired output in the training data.
	* Calculate the loss between the two outputs.
	* Backpropagate the loss to update the weights of model (b).
6. **Repeat the fine-tuning loop**: Iterate through the training data, updating model (b) after each iteration.
7. **Evaluate and refine**: Periodically evaluate the performance of model (b) on a validation set. Refine the fine-tuning process as needed.

Single Dynamic Diagram
```markdown
+---------------+
|  Input Prompt  |
+---------------+
       |
       |
       v
+---------------+
|  Model (b)    |
|  (Dynamic)     |
+---------------+
       |
       |
       v
+---------------+
|  Model (a)    |
|  (Static)      |
+---------------+
       |
       |
       v
+---------------+
|  Desired Output|
+---------------+
       |
       |
       v
+---------------+
|  Loss Calculation|
+---------------+
       |
       |
       v
+---------------+
|  Update Model (b)|
+---------------+
```

I realize that I may have oversimplified the fine-tuning objective. In practice, the objective function may need to be more complex to account for the nuances of the in-between space between models (a) and (b). Additionally, the choice of model (b) and its initialization may significantly impact the fine-tuning process. May require experimenting with different models and hyperparameters to find the optimal configuration long term.

**Dual Dynamic**

* Added alternating training mechanism where models switch roles periodically
* Introduced TrainingMode enum to track which model is being trained
* Separated optimizers for each model
* Added configuration dataclass for better parameter management
* Modified loss computation to handle both models
* Added proper logging for both models' metrics
* Implemented proper gradient computation based on active model
* Added switch_frequency parameter to control how often models alternate
* Enhanced evaluation to track both models' performance
* Added proper model state management (train/eval modes)

Considerations in Dual Dynamic version:
* Gradient Flow:
	- Only the active model's gradients are computed
	- Frozen model's outputs are detached from computation graph
	- Gradient accumulation handles larger effective batch sizes

* Model Switching:
	- Happens at regular intervals during training
	- Maintains separate optimizers for each model
	- Each model learns to work with the other's current state

* Loss Computation:
	- Cross-entropy loss measures prediction accuracy
	- Separate losses tracked for each model
	- Losses guide the learning process

* Memory Management:
	- Gradients cleared after each optimization step
	- Tensors moved to correct device
	- Memory freed when no longer needed

* Logging and Monitoring:
	- Regular logging of metrics
	- Validation performed periodically
	- Progress tracked through Weights & Biases

Process of Dual Dynamic:

1. The training dataset is loaded into a `DataLoader` with a specified batch size.
2. The `AlternatingModelTrainer` class takes the `DataLoader` as input and trains two models, `Model A` and `Model B`, in an alternating manner.
3. `Model A` is a static model, while `Model B` is a dynamic model that is being fine-tuned.
4. The forward pass is performed on `Model B` first, followed by the forward pass on `Model A`.
5. The loss is computed for both models using the output of `Model A` and the target response.
6. The backward pass is performed on `Model B` to update its parameters.
7. The optimizer is used to update the parameters of `Model B`.
8. The models are switched every N batches, where N is a hyperparameter.
9. The process is repeated until convergence.

Dual Dynamic Diagram
```markdown
                                      +---------------+
                                      |  Training    |
                                      |  Dataset     |
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  DataLoader  |
                                      |  (batch_size)  |
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  Alternating  |
                                      |  Model Trainer|
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  Model A     |
                                      |  (static)     |
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  Model B     |
                                      |  (dynamic)    |
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  Forward Pass|
                                      |  (Model B)    |
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  Forward Pass|
                                      |  (Model A)    |
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  Compute Loss|
                                      |  (Model A & B) |
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  Backward Pass|
                                      |  (Model B)    |
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  Optimizer   |
                                      |  (Model B)    |
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  Switch Models|
                                      |  (every N batches)|
                                      +---------------+
                                             |
                                             |
                                             v
                                      +---------------+
                                      |  Repeat       |
                                      |  (until convergence)|
                                      +---------------+
```

# Note on why keeping the training logic and adapting inference is advantageous:

1. Training Stability
* Training directly with logits provides more granular learning signals
* Using .generate() during training would introduce non-differentiable sampling
* The current training approach allows proper backpropagation through both models
2. Inference Flexibility
.generate() provides important inference features like:
* Temperature control
* Top-p/nucleus sampling
* Max length constraints
* Beam search
These help produce better/more diverse outputs at inference time
3. Best of Both Worlds
* Training: Precise optimization with direct logit supervision
* Inference: Rich generation features for better real-world use
* No need to compromise either use case
So adapting inference to match training would remove valuable generation capabilities, while keeping training focused on exact logit matching maintains optimal learning while allowing flexible inference strategies.
