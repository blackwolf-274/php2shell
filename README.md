# php2shell — PHP Multipart / Session Memory-Corruption Research PoC

> **Security notice:** This source implements an exploit that attempts to achieve command execution in a PHP/Apache process. Treat it as dual-use security research code. Use it only in an isolated lab that you own or are explicitly authorized to test. Do not expose a vulnerable target or this tool to the public internet. This README documents the source and its environment assumptions; it intentionally does not provide operational instructions for executing the exploit against a target.

## Overview

The supplied Python program is a proof-of-concept for a PHP multipart-form/session handling memory-corruption condition. At a high level, its implementation:

- Constructs multipart HTTP request bodies and sends them over HTTP or HTTPS using Python sockets.
- Attempts to derive memory-layout information from PHP session-cookie data.
- Uses build-specific offsets and heap-layout assumptions to construct forged PHP/Zend data structures.
- Contains a final stage intended to reach the PHP `system` function and execute a command in the target process context.
- Can inspect a configured local proof-file path and can optionally retrieve a response path, as described by the source's command-line parser.

The program is highly dependent on the exact PHP build, Apache/PHP packaging, allocator behavior, and process layout. A successful result on one image does not establish that another build is vulnerable or compatible.

## Requirements

### Runtime

- **Python 3.10 or newer.** The source uses modern type-annotation syntax, including union types (`X | None`).
- A POSIX-like environment is recommended for the surrounding lab and helper scripts.
- Network access to an authorized, isolated test environment is required by the program's transport layer. Do not use this software against systems without explicit authorization.

### Python packages

No third-party Python packages are required. The source imports only modules from the Python standard library:

- `argparse`
- `collections`
- `os`
- `re`
- `socket`
- `ssl`
- `struct`
- `subprocess`
- `threading`
- `sys`
- `time`
- `urllib.parse`

`requirements.txt` is therefore intentionally limited to comments documenting the runtime requirement. There is nothing to install with `pip` for the supplied source file itself.

## Environment and compatibility notes

The source contains hard-coded defaults and calibration values for particular PHP/Zend memory layouts. Those values are not general-purpose offsets.

The source comments identify these assumptions:

- A lab profile associated with PHP 8.4.26, Apache 2.4.68, Debian 12, and GCC 12.
- A `bookworm` profile associated with the official `php:8.4.26-apache` image and Debian 12.
- Separate offsets and heap-grooming calibration values for the two profiles.

**Version discrepancy in the source:** one comment describes the lab build as Apache 2.4.68, while another describes the official-image profile as Apache 2.4.69. Verify the exact image and package versions used in any authorized research environment; do not assume these comments identify interchangeable builds.

Rebuilding PHP, changing compiler or linker versions, changing Apache/PHP packaging, or changing allocator behavior can invalidate the offsets and calibration values. The source also references an optional helper at `docker/offsets.py` and an optional offsets file at `out/offsets.env`; those files are not included in the supplied source excerpt and must not be assumed to exist.

## Repository contents

The supplied material contains a Python source file. The following paths are referenced by that source but may need to be supplied separately by the repository:

```
.
├── README.md
├── requirements.txt
├── <python-source-file>
├── docker/
│   └── offsets.py        # Optional helper referenced by the source
└── out/
    ├── offsets.env       # Optional locally generated offsets
    └── p1                # Default proof-file location
```

The tree above documents references made by the program; it does not assert that the optional helper or output files were present in the uploaded material.

## Installation of the Python runtime environment

Install Python 3.10 or newer using your operating system's trusted package manager or the official Python distribution. An isolated virtual environment is recommended for consistent project management:

```
python3 --version
python3 -m venv .venv
source .venv/bin/activate
```

There are no third-party dependencies to install. The virtual environment is optional for this standard-library-only source.

## Safety and handling

Because this code attempts memory corruption and command execution:

1. Keep experiments inside a disposable, isolated lab with no route to production systems.
2. Obtain explicit written authorization before testing any system that you do not own.
3. Do not run it against public hosts, shared infrastructure, or systems containing real user data.
4. Do not place real credentials, tokens, or sensitive data in test requests, logs, or proof files.
5. Snapshot lab systems before testing and destroy or restore them afterward.
6. Restrict access to the source and any resulting logs or memory-layout information.

For defensive validation, prefer reviewing the relevant PHP and application logs, checking exact package versions, reproducing behavior in a disposable environment, and applying vendor updates or mitigations rather than testing third-party systems.

## Limitations

- The source is calibrated for specific build assumptions and may fail on other builds.
- Heap-layout and address-space randomization make results environment-dependent.
- The program's proof check depends on the configured proof-file path or optional fetch behavior; absence of proof does not independently establish whether a target is vulnerable.
- The uploaded source does not include a test suite, dependency lock file, Docker Compose definition, target PHP application, or the optional `docker/offsets.py` helper. Their existence and behavior cannot be inferred from the Python source alone.

## License

No license information was included in the supplied source. Do not assume the code is open source or that redistribution is permitted. Add a license only when the copyright holder has selected and authorized one.

## Acknowledgements and references

No upstream repository, advisory identifier, CVE number, author attribution, or publication reference was included in the supplied material. Add verified references here rather than guessing at attribution or vulnerability identifiers.