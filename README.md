# Quarantine Formats

Documentation and parsers for different anti-virus quarantine formats.

So far, I analyzed quarantine file formats of seven different AV software solutions.

* Avira
* Windows Defender
* Malwarebytes
* Symantec Endpoint Protection
* G Data
* Sophos Antivirus
* Kaspersky for Windows Server

The documentation can be found in `./docs`.

For binary file formats that require parsing, Kaitai Struct parser definition files can be found in `./formats`.

## License

The content of this repository is licensed under Creative Commons CC BY-SA 4.0.

* License Description: https://creativecommons.org/licenses/by-sa/4.0/
* Complete legal text: https://creativecommons.org/licenses/by-sa/4.0/legalcode

Author: Florian Bausch, ERNW Research GmbH, https://ernw-research.de/

## Improvements / Errors

If you find any errors or something to improve, please open an issue or create a pull request.

## Remarks

If you want to make the parser definition files to work with Python, you need to use the Kaitai Struct runtime file from the current Github master branch (https://github.com/kaitai-io/kaitai_struct). The parser definition files make use of enums that are not yet supported in the most recent stable release (0.8).

