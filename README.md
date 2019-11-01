# vyatta-tech-support
Pluggable infrastructure used for gathering system info for debugging and support

## Branching
Unless it is unavoidable vyatta-tech-support will not be branched. The reason
for this is that improvements/fixes to our debugging capabilities are a
"good thing" and those improvements should filter to as many future releases as
possible.

All PRs will be reviewed with the above in mind; any changes which require new
capabilities of a specific release MUST fail silently or gracefully on a release
without that capability.

## Feature specific scripts
The vyatta-tech-support infrastructure is entirely pluggable; `run-parts` is used
to run executables from the following directories and grab their outputs:

 * "brief" output - `/opt/vyatta/share/vyatta-op/functions/tech-support-brief.d`
 * "full" output - `/opt/vyatta/share/vyatta-op/functions/tech-support.d`

Tech support scripts for an individual feature should be maintained with that
feature and installed to one or both (as appropriate) of the above directories
by one of its packages.

### Script naming
`run-parts` executes scripts in a lexical sort order and this in turn determines
the ordering of tech support output.

To ensure this ordering can be easily determined script names MUST be prefixed
with a four digit number in the range 0101-9999. Features may choose a prefix in
this range which best suits them, for example to be located with similar/related
features in the output.

As a general rule core system functionality should use the lowest prefixes,
followed by core features, with any remaining features last. Number prefixes
may be re-used so long as the entire script name is unique.

Script names MUST further conform to the following, from run-parts(8):
> the names must consist entirely of ASCII upper- and lower-case letters,
> ASCII digits, ASCII underscores, and ASCII minus-hyphens.

## Generate a list of tech support scripts
The listing of existing tech support scripts can be generated from a running system:

`sudo run-parts --test /opt/vyatta/share/vyatta-op/functions/tech-support.d`

Substitute `tech-support.d` with `tech-support-brief.d` to obtain the script
listing for the brief tech support output.

## Technical documentation
### Infrastructure scripts
 * `tech-support` - handles the `show tech-support ...` operational commands
 * `tech-support-archive` - handles the `generate tech-support archive ...`
                            operational commands
 * `tech-support.functions` - provides common functionality for infrastructure
                              and feature scripts
 * `tech-support-op` - shim script between opd and
                       `tech-support`/`tech-support-archive`, used for securely
                       handling "sensitive" arguments

### Argument passing
The tech support scripts support passing of "sensitive" command arguments via
environment variables which removes the risk of those arguments being leaked
in process lists.

The supported environment variables are:

 * `VYATTA_TECH_SUPPORT_COPY_USER` - username for copying output to remote host
 * `VYATTA_TECH_SUPPORT_COPY_PASS` - remote host user password
 * `VYATTA_TECH_SUPPORT_FILE_PASS` - passphrase used for encrypting the output file

These variables are set from `tech-support-op` where the command arguments are
initially processed. They are then copied into local variables and removed from
the environment by `tech-support-archive` and `tech-support` (via sourcing of
`tech-support.functions`) before they do any work.

Since these variables are only used as a method of initially passing the
arguments into these scripts there is no need to keep them in the
environment; removing them reduces the risk of inadvertent leaking.
