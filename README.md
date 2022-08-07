# hfm 🤗

**HFM** is the _Hugging Face Download (Cache) Manager_.

_⚠NOTE️⚠️: HFM is not an official Hugging Face project, and not endorsed by the HF team. I just use `transformers` a lot, and made a little CLI to make the most of my disk space._

[Hugging Face Transformers](https://huggingface.co/docs/transformers/index) is a great library, but in the course of working with many different models, your `.cache/huggingface/transformers` can fill up quickly with data. If you don't keep an eye on it (or if you, like me, don't have a lot of disk space), this folder can grow to dozens of GBs and eat up your disk space. HFM is a little command-line utility that helps keep an eye on this download cache folder, and easily remove/inspect cached Hugging Face Transformers downloads.

## How to use `hfm`

In the most basic use case, just running `hfm` (or `hfm ls`, for which `hfm` is an alias) shows a table of all the downloads Huggingface has put in `~/.cache/huggingface/transformers/`, sorted by file size.

The table shows you the download size, the first 8 characters of the download ID (used in file names), download date and time, and the URL (within `https://huggingface.co/`) from where that file that was downloaded.

```
$ hfm
548.12MB 752929ac 2021-11-20T07:05:57Z gpt2/resolve/main/pytorch_model.bin
267.84MB 8d04c767 2021-11-20T07:03:48Z distilbert-base-uncased/resolve/main/pytorch_model.bin
  1.37MB 6a79d94e 2022-04-21T23:32:37Z EleutherAI/gpt-j-6B/resolve/main/tokenizer.json
798.16KB 3138d7eb 2022-04-21T23:32:34Z EleutherAI/gpt-j-6B/resolve/main/vocab.json
456.36KB 2f9c5228 2022-04-21T23:32:36Z EleutherAI/gpt-j-6B/resolve/main/merges.txt
  4.04KB 9b1d5815 2022-04-21T23:32:40Z EleutherAI/gpt-j-6B/resolve/main/added_tokens.json
  1.35KB 42252c22 2021-12-29T19:13:56Z EleutherAI/gpt-neo-1.3B/resolve/main/config.json
    762B f985248d 2022-04-21T22:37:09Z distilgpt2/resolve/main/config.json
    200B 5fe35a59 2021-12-29T19:13:56Z EleutherAI/gpt-neo-1.3B/resolve/main/tokenizer_config.json
     90B 953b5ce4 2021-12-29T19:35:21Z EleutherAI/gpt-neo-2.7B/resolve/main/special_tokens_map.json
832.76MB total
```

To remove any specific cached download, just run `hfm rm <id>`. This command can take many IDs at once, and will let you know if any ID you gave doesn't exist, or matches more than one download. Like `git checkout`, you don't have to type the full ID -- just the first few characters will do.

```
$ hfm rm 7529 8d04 13d2 9
No download with id "2314".
More than one download matching id "9".
```

You can pass `--dry-run` or `-d` at the end to see which files will be deleted, without actually deleting them.

```
$ hfm rm 7529 --dry-run
[dry-run] rm "752929ace039baa8ef70fe21cdf9ab9445773d20e733cf693d667982e210837e.323c769945a351daa25546176f8208b3004b6f563438a7603e7932bae9025925"
[dry-run] rm "752929ace039baa8ef70fe21cdf9ab9445773d20e733cf693d667982e210837e.323c769945a351daa25546176f8208b3004b6f563438a7603e7932bae9025925.lock"
[dry-run] rm "752929ace039baa8ef70fe21cdf9ab9445773d20e733cf693d667982e210837e.323c769945a351daa25546176f8208b3004b6f563438a7603e7932bae9025925.json"
```

For composing commands with other UNIX tools, it's often nice to be able to get the full path to a downloaded file. `hfm which <id>` prints the full path to a file. With this, you can run `jq` on a downloaded JSON file, as an example:

```
$ cat $(hfm which 6a79) | jq '.added_tokens | map(.content)'
[
  "<|endoftext|>",
  "<|extratoken_1|>",
  "<|extratoken_2|>",
  "<|extratoken_3|>",
  "<|extratoken_4|>",
  "<|extratoken_5|>",
  "<|extratoken_6|>",
  "<|extratoken_7|>",
[...]
```

Of course, the output of `hfm ls` itself is made to be greppable and interoperate with standard UNIX utilities like `awk`. To see all files about the `gpt-j` model...

```
$ hfm | grep gpt-j
  1.37MB 6a79d94e 2022-04-21T23:32:37Z EleutherAI/gpt-j-6B/resolve/main/tokenizer.json
798.16KB 3138d7eb 2022-04-21T23:32:34Z EleutherAI/gpt-j-6B/resolve/main/vocab.json
456.36KB 2f9c5228 2022-04-21T23:32:36Z EleutherAI/gpt-j-6B/resolve/main/merges.txt
  4.04KB 9b1d5815 2022-04-21T23:32:40Z EleutherAI/gpt-j-6B/resolve/main/added_tokens.json
    619B ec7f9c4f 2022-04-21T23:32:31Z EleutherAI/gpt-j-6B/resolve/main/tokenizer_config.json
    357B 3cd6a981 2022-04-21T23:32:41Z EleutherAI/gpt-j-6B/resolve/main/special_tokens_map.json
```

`hfm` features two flags, `--no-total` and `--no-humanize`, that specifically make the `hfm ls` output more machine-readable.

You can read more about all the commands and flags from the help message, at `hfm help` or `hfm -h` / `hfm --help`.

## Install

If you have [Oak](https://oaklang.org) installed, you can build from source (see below). Otherwise, I provide pre-built binaries for macOS (x86 and arm64) and Linux (x86) on the [releases page](https://github.com/thesephist/hfm/releases). Just drop those into your `$PATH` and you should be good to go.

## Build and development

HFM is built with my [Oak programming language](https://oaklang.org), and I manage build tasks with a Makefile.

- `make` or `make build` builds a version of HFM at `./hfm`
- `make install` installs HFM to `/usr/local/bin`, in case that's where you like to keep your bins
- `make fmt` or `make f` formats all Oak source files tracked by Git
