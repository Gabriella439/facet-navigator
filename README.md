# Semantic project navigator

Introductory blog post about this tool: [Browse code by meaning](https://haskellforall.com/2026/02/browse-code-by-meaning).

This project provides a tool that lets you browse a repository's files
by their meaning:

https://github.com/user-attachments/assets/95f04087-453d-413e-8661-5bcd7dce4062


## Setup

Currently I've only built and run this script using uv and Nix.  However, you
can feel free to submit pull requests for other installation instructions if
you've vetted them.

You will need either a CLI AI tool installed and configured for authentication
(any tool that reads from stdin and writes to stdout will work, e.g. `gemini`,
`llm`, `aichat`), or you can use a local LLM via `--local` (see below).

Note: On first run, the embedding model (~130MB) will be downloaded automatically.

### uv

You can run the script in a single command, like this:

```ShellSession
$ uvx git+https://github.com/Gabriella439/semantic-navigator ./path/to/repository
```

… or you can install the script:

```ShellSession
$ uv tool install git+https://github.com/Gabriella439/semantic-navigator

$ semantic-navigator ./path/to/repository
```

### Nix

You can run the script in a single command, like this:

```ShellSession
$ nix run github:Gabriella439/semantic-navigator -- ./path/to/repository
```

… or you can build the script and run it separately:

```ShellSession
$ nix build github:Gabriella439/semantic-navigator

$ ./result/bin/semantic-navigator ./path/to/repository
```

… or you can install the script:

```ShellSession
$ nix profile install github:Gabriella439/semantic-navigator

$ semantic-navigator ./path/to/repository
```

## Usage

Depending on the size of the project it will probably take between a few
seconds to a minute to produce a tree viewer.  You must specify which backend
to use:

```ShellSession
# Gemini CLI
$ semantic-navigator --gemini ./path/to/repository

# Simon Willison's llm with GPT-4o
$ semantic-navigator ./path/to/repository --llm -m gpt-4o

# aichat
$ semantic-navigator --aichat ./path/to/repository
```

### OpenAI

You can use OpenAI directly via `--openai`. Install the optional dependency
first:

```ShellSession
$ uv sync --extra openai
```

Then run with `--openai`:

```ShellSession
# Default model (gpt-4o-mini)
$ semantic-navigator --openai ./path/to/repository

# Custom completion model
$ semantic-navigator --openai --completion-model gpt-4o ./path/to/repository

# Use OpenAI for both labeling and embeddings
$ semantic-navigator --openai --openai-embedding-model text-embedding-3-large ./path/to/repository
```

This requires the `OPENAI_API_KEY` environment variable to be set.

### Local LLM

Instead of an external CLI tool, you can use a local GGUF model via
`llama-cpp-python`. Install the optional dependency first:

```ShellSession
$ uv sync --extra local
```

Then use `--local` with either a local file path or a Hugging Face repo ID:

```ShellSession
# Local GGUF file
$ semantic-navigator --local ./models/qwen2.5-7b-q4_k_m.gguf ./repo

# Hugging Face repo (auto-downloads, default quantization: Q4_K_M)
$ semantic-navigator --local Qwen/Qwen2.5-7B-Instruct-GGUF ./repo

# Specific quantization
$ semantic-navigator --local bartowski/Qwen2.5-7B-Instruct-GGUF --local-file "*Q8_0.gguf" ./repo

# With GPU offloading (Vulkan on AMD, CUDA on NVIDIA)
$ semantic-navigator --local Qwen/Qwen2.5-7B-Instruct-GGUF --gpu ./repo
```

When `--gpu` is used with `--local`, all model layers are offloaded to the GPU
via whichever backend `llama-cpp-python` was compiled with (Vulkan, CUDA, etc.).
The `--gpu` flag also enables DirectML for the embedding model (existing
behavior).

For small repositories (up to 20 files) you won't see any clusters and the tool
will just summarize the individual files:

![](./images/small.png)

This is a tradeoff the tool makes for ergonomic reasons: the tool avoids
subdividing clusters with 20 files or fewer.

For a a medium-sized repository you'll begin to see top-level clusters:

![](./images/medium.png)

The label for each cluster describes the files within that cluster and will
also display a file pattern if all files within the cluster begin with the same
prefix or suffix.  In the above example the "Project Prelude" doesn't display a
file pattern because there is no common prefix or suffix within the cluster,
whereas the "Condition Rendering" cluster displays a file pattern of
`*/Condition.dhall` because both files within the cluster share the same
suffix.

For an even larger repository you'll begin to see nested clusters:

![](./images/large.png)

On a somewhat modern MacBook this tool can handle up to ≈10,000 files within a
few minutes.

You can use this tool on any text documents; not just code!  For example,
here's the result when running the tool on the repository for my self-hosted
blog:

![](./images/haskellforall.png)

In other words, this tool isn't just a code indexer or project indexer; it's a
general file indexer.

## Development

If you use Nix and `direnv` this project provides a `.envrc` which
automatically provides a virtual environment with all the necessary
dependencies (both Python and non-Python dependencies).

Otherwise if you don't use `direnv` you can enter the virtual environment using:

```ShellSession
$ nix develop
```

… and you can test any of the setup commands with a local checkout by replacing
`github:Gabriella439/semantic-navigator` with `.`, like this:

```ShellSession
$ nix run . -- ./path/to/repository
```

Embeddings are generated locally using [fastembed](https://github.com/qdrant/fastembed) (ONNX-based), so no API key is needed for the embedding step.
