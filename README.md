# DnD Dice Roller

Dice roller for DnD, and other purposes. Capable of rolling multiple die, showing the results, and applying modifiers and rules (advantage / disadvantage, etc.).

The script was made in bash to get a deeper understanding of the bash scripting language.

This script has 2 modes of operation:

- Interactive
	- This mode will continuously prompt for a new roll command
- Regular
	- This mode will run the provided roll command and exit

The following features are available:

- Regular dice rolling, and quantity
- Custom die support
- Recursive roll commands for more complex use-cases
- Percentile die format (`d%`)
- Multiply modifier
- Skill modifier (`+` / `-`)
- Keeping or dropping the lowest / highest rolls

## Usage

Run the `./dndroll` command directly from the folder to get an interactive session.

Run the `./dndroll <roll_command>` command directly from the folder to run in regular mode for scripts or just for singular rolls.

To see the help manual, run the command with the `-h` or `--help` flag (`./dndroll -h`)

## Install

This script is not intended as an installation script, but you may move it into a folder in your path so you can run it from any folder.