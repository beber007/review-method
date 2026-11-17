<!-- An example of REVIEW.md filled in, from the template of this plugin. It is drawn
from the rules of Escapement, a small real-time operating system for microcontrollers
(https://github.com/beber007/escapement). There, a build produces a firmware image: the
binary written to the board. Issue numbers link to that repository. -->

# The reviews before a merge

The reviews of this repository follow the method of the plugin `review-method`. This
file holds what is local.

## What failure looks like here

- A comment, or an option off by default, changes the firmware image it should leave
  the same, byte for byte.
- A build drops what it is asked for, an option, a kernel or a scheduler, and builds
  another image without a word
  ([#84](https://github.com/beber007/escapement/issues/84),
  [#86](https://github.com/beber007/escapement/issues/86)).
- The compiler moves or merges a store that an interrupt, or the other core of a
  dual-core chip, relies on.
- A fix is made to one kernel or one port, and not to those that share its code.
- A text lies: a figure not checked against the code, a measurement without its date,
  a claim the code does not keep.
- A tool or a CI job now refuses, or skips without a word, what it ran before.

## How deep, by risk

- **Deep**, three reviewers: the kernels, the ports to each chip, the drivers.
- **Standard**, one reviewer: anything else, such as the tools, the CI, the examples,
  the Makefiles, the tests on the host and the tests under the Renode emulator.
- **Documentation alone**: one full review, then one review of its fixes.

## What to skip

- Nothing so far. The files a build writes are not tracked.

## Checks that come before the reviews

- The tests of the scheduler on the host: `make -C test/host run`.
- `make` in each directory `Escapement/CORTEX-Mx/*/Examples/*`: every example of every
  chip still builds.
- `sh tools/check_encoding.sh` and `sh tools/check_options.sh`.
- A change meant to leave an image alone: `arm-none-eabi-objcopy -O binary` of each
  image, compared with that of `main`.
