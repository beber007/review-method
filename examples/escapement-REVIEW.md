<!-- An example of REVIEW.md filled in, from the template of this plugin. It is the
REVIEW.md of Escapement, a small real-time operating system for microcontrollers
(https://github.com/beber007/escapement). There, a build produces a firmware image: the
binary written to the board. Issue numbers link to that repository. -->

# Review instructions

The reviews of this repository follow the plugin
[`review-method`](https://github.com/beber007/review-method): how many runs of
`/code-review`, how each finding is checked and tagged, when the reviews stop.
`CLAUDE.md` brings this file in, so that `/code-review` reads it. The first four
sections are for each review. The last three are for the plugin.

## What Important means here

Important means a wrong behaviour, or a false statement. Here, that is above all:

- A comment, or an option off by default, changes the firmware image it should leave
  the same, byte for byte.
- A build drops what it is asked for, an option, a kernel or a scheduler, and builds
  another image without a word ([#84](https://github.com/beber007/escapement/issues/84), [#86](https://github.com/beber007/escapement/issues/86), [#92](https://github.com/beber007/escapement/issues/92)).
- The compiler moves or merges a store that an interrupt, or the other core of a
  dual-core chip, relies on (`CLAUDE.md`, "Pitfalls met here").
- A fix is made to one kernel or one port, and not to those that share its code.
- A text lies: a figure not checked against the code, a measurement without its date,
  a claim the code does not keep.
- A tool or a CI job now refuses, or skips without a word, what it ran before.

Style, naming and rewording are Nit at most.

## Cap the nits

Report at most five Nits per review.

## Do not report

- What `tools/check_encoding.sh` already checks: each tracked file is valid UTF-8.

## Always check

- Each option that one example's Makefile passes to the compiler: do the other
  examples that build the same kernel pass it too, or refuse it ([#84](https://github.com/beber007/escapement/issues/84))?
- Each new value an option takes: is a value the code does not know refused ([#92](https://github.com/beber007/escapement/issues/92))?
- Each change to one kernel or one port: do the others that share this code need it?
- Each figure in a text: is it in the code, or in a measurement with its date?

## How deep, by risk

- **Deep**, three runs: the kernels, the ports to each chip, the drivers. The mutants
  of the kernel lines changed too (`CLAUDE.md`, "Rules that do not change").
- **Standard**, one run: anything else, such as the tools, the CI, the examples, the
  Makefiles, the tests on the host and the tests under the Renode emulator.
- **Documentation alone**: one run, then one run on its fixes (`CLAUDE.md`, "Rules
  that do not change").

## Commands a review may run

- `"Bash(make:*)"`: build the examples, and run the tests on the host.
- `"Bash(sh tools/check_options.sh)"`: check the options of the Makefiles.
- `"Bash(arm-none-eabi-objcopy:*)"`, `"Bash(cmp:*)"`: compare two images.

No tool that reaches the bench: a run of endurance may hold its probes.

## Checks before the reviews

- The list of `CLAUDE.md`, "Before a commit": the tests on the host, the models, the
  static analysis, and every variant of every example built.
- `sh tools/check_options.sh`.
- A change meant to leave an image alone: `arm-none-eabi-objcopy -O binary` of each
  image, compared with that of `main`.
