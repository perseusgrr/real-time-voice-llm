<p align="center">
  <picture>
    <img alt="LLM" src="https://zfmrfvimiaqahezndsse.supabase.co/storage/v1/object/public/images/custom/Introducing%20Ultravox%20Wide.jpg">
  </picture>
</p>

<h3 align="center">
A fast multimodal LLM designed for real-time voice interactions
</h3>

# About

This is a new kind of multimodal LLM that can understand text as well as human speech, without the need for a separate Audio Speech Recognition (ASR) stage. Building on research like [AudioLM](https://arxiv.org/abs/2209.03143), [SeamlessM4T](https://ai.meta.com/blog/seamless-m4t/), [Gazelle](https://tincans.ai/slm), [SpeechGPT](https://github.com/0nutation/SpeechGPT/tree/main/speechgpt), and others, this is able to extend any open-weight LLM with a multimodal projector that converts audio directly into the high-dimensional space used by LLM. We've trained versions on Llama 3, Mistral, and Gemma. this direct coupling allows this to respond much more quickly than systems that combine separate ASR and LLM components. In the future this will also allow this to natively understand the paralinguistic cues of timing and emotion that are omnipresent in human speech.

This currently takes in audio and emits streaming text. As we evolve the model, we'll train it to be able to emit a stream of speech tokens that can then be converted directly into raw audio by an appropriate unit vocoder.

Our default model is built on top of Llama 3.3 70B. We also have an 8B variant available on Hugging Face.

This can be trained against any open-weight model. See below for more details on training.


## Environment Setup (Mac)

Install the basic tools:

- [`Homebrew`](https://brew.sh) is a package manager for MacOS that also mostly works for Linux. If you're running Debian or Ubuntu Linux, you can alternatively get by with apt.
- [`Just`](https://just.systems/man/en/) simplifies our shell workflows. It frequently functions as our interface to all the other tools.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew update
brew install just
```

It's recommended to use pyenv for managing environments due to the use of Poetry:

```bash
brew install xz
brew install pyenv
pyenv init
pyenv install 3.11
pyenv global 3.11

# Optional
pyenv shell 3.11
```

>**Note**: Use of conda is NOT recommended with Poetry

After creating a virtual environment, install required packages using `just` and `poetry`:

```bash
just install
```

If you plan to use augmentations (optional), you may also want to install system packages necessary for augmentations. You can do that with `just install-augs-system`.

We're using Poetry to manage the Python virtual environment. You can observe your environment with `poetry env info`.

## Training

Currently, we keep both the LLM and the audio encoder frozen and only train the adapter/projector. Training Ultraox v0.4 took 2-3 hours on 8xH100 GPUs for 14K training steps.

### Use-Cases for Training this

Why would you want to (re-) train this? Here are a few scenarios:

1. You want to use a different LLM or audio encoder backbone.

   a. In this case you need to re-train the adapter. You can use `example_config.yaml`, which contains our config for our latest release, and you should be able to simply change the base LLM or encoder by specifying `--text-model <hf-model-id-for-llm>` and/or `--audio-model <hf-model-id-for-encoder>`.

2. You want to improve the knowledge of the model

    a. We suggest to either use RAG on the fly (no training needed), or fine-tune the LLM backbone instead. Fine-tuning the LLM backbone does not require re-training this (i.e., the existing adapter will work).

3. You want to use your own audio data, for example to add support for a new language.

   a. First step, prepare your dataset: at bare minimum, the samples should have an `audio` and a text `continuation` field.

   b. Take a look at [`ds_tool.py`](<repo>/tools/ds_tool/ds_tool.py) and [`continuation.jinja`](<repo>/tools/ds_tool/continuation.jinja) as well as [our variant of Common Voice](https://huggingface.co/datasets/fixie-ai/common_voice_17_0/viewer/fr) that was created using `ds_tool` to add the `continuation` field.

   c. Add your dataset to the dataset mix in `example_config.yaml` and train.

There's no one-size fits all. If you need help you can find us on our Discord server [here](https://discord.gg/Qw6KHxv8YB).

### Running evaluations

For inference or evaluations, you can use:

```bash
just eval --config_path <repo>/evaluation/configs/eval_config.yaml
```

where `eval_config.yaml` is a config file that specifies the model, datasets, and configurations to use for inference or evaluation. If your dataset is not already defined in llm, you need to create a config file for your dataset in `<repo>/data/configs/` (with the appropriate `eval_config` field to specify evaluation metrics and arguments), and register it in `<repo>/data/registry.py`. Please refer to examples in `<repo>/data/configs/`.

## Misc

The [Justfile](Justfile) is a good resource for finding popular commands. Here are a few:

```bash
just update    # update dependencies
just format    # run formatting (black, isort, autoflake)
just test      # run tests
just python    # activate venv and run python
```
