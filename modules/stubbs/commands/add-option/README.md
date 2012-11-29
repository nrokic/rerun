Use *stubbs:add-option* to add an option to a command.

Basic usage
-----------

After creating a command with 
[stubbs:add-command](../add-command/index.html),
use stubbs:add-option to define a command option. 
The add-option command takes several arguments that specify
the various option properties.

Below the `--jumps` option is added to the command freddy:dance.

    rerun stubbs:add-option --option jumps --description "jump #num times" --module freddy --command dance --required true --export false --default 3

You will see output similar to:

    Created option: /Users/alexh/.rerun/modules/freddy/options/jumps/metadata
    Updated command metadata:  /Users/alexh/.rerun/modules/freddy/commands/dance/metadata

Besides the `options/jumps/metadata` file, the `add-option` 
command also generates an option parsing function contained
in the modules function library.

The dance command's `script` calls this option parsing function 
to read the command line arguments and set shell variables
accessible to the command script.

Users will now be able to specify a "--jumps" option
to the `freddy:dance` command:

    $ rerun freddy
    freddy:
     dance: tell freddy to dance
        --jumps <>: "jump #num times"

Use the [stubbs:edit](../edit/index.html) command to 
open the generated `script` file in the text 
editor referenced via the `$EDITOR` environment variable. Eg,

    rerun stubbs: edit --module freddy --command dance 

See the [stubbs:rm-option](../rm-option) command to remove an option.
If a command option is not supplied by the user, 
the options parser script (generated by add-option) can set a default value.

Add an option that has a default
--------------------------------

Call the add-option command again using 
its `--default <>` parameter to set a default value.

Here the "--jumps" option is set to use a default value, "1":

    rerun stubbs:add-option \
      --option jumps -description "jump #num times" \
      --module freddy --command dance \
      --default 1

The add-option will update the metadata for the 
"jumps" option.

List the freddy commands to see "dance" and the new 
"--jumps" option:

    $ rerun freddy
    dance: "tell freddy to dance"
        --jumps <1>

We see the default value "1" printed.

Internal details
----------------

You might be interested in the options.sh script that's 
created behind the scenes. Below, the "dance" command's 
options.sh script is shown.
The meat of the script is the while loop and case statement. 
In the body of the case statement, you can see a case 
for the "--jumps" option and the `JUMPS` variable that will 
be set to the value of the "--jumps" argument.

    # generated by add-option
    # Tue Sep 13 20:11:52 PDT 2011

    rerun_options_parse() {
     
        while [ "$#" -gt 0 ]; do
            OPT="$1"
            case "$OPT" in
                --jumps) rerun_option_check $# ; JUMPS=$2 ; shift ;;
                # help option
                -?)
                    rerun_option_usage
                    exit 2
                    ;;
                # end of options, just arguments left
                *)
                  break
            esac
            shift
        done
     
        # Set defaultable options.
        [ -z "$JUMPS" ] && JUMPS="/tmp/mock"
        # Check required options are set
        [ -z "$JUMPS" ] && { echo >&2 "missing required option: --basedir" ; return 2 ; }
        #
        return 0
    }

Below the while loop, you can see a test for the `JUMPS`
variable (check for empty string). A statement like this 
is added for options that declare DEFAULT metadata.

Separating options processing into the options.sh script, 
away from the command implementation logic in script, 
keeps the options parsing clutter out of your way.