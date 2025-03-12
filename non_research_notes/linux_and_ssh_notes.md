# Linux Notes

## System Information/Status

### RAM
#### Total used/available
To get the current amount of RAM used and available (note **"available"** instead of **"free"**!),
you apparently can do `free -h`
(the `-h` makes things "human-readable"; you can do -m for megabytes instead, -k for kilobytes, etc.).

The "available" column says how much you can use for starting a new application. The "free" column might be a _lot_ smaller; why is that?
Why can you start a new application that uses more RAM than what is "free"?

According to the apparently well-known site [https://www.linuxatemyram.com/](https://www.linuxatemyram.com/), this is because of disk caching to make things faster.
So if your apps need more RAM, the kernel will just use less for disk caching, and this is apparently "immediate".

Now, the "total" value may also be less than might be expected; from some reading online, it seems this is because the kernel, graphics card, etc. will reserve some.

Apparently, one can see the amount used by the kernel by running `cat /proc/meminfo | grep Slab`.

Worth noting: `top` also shows cumulative amounts, with the same naming/terminology above.

### RAM usage by application
Probably the easiest is via `top`, though it also shows the cumulative amounts as well (and updates over time, which might be a benefit over using `free`).

To sort by memory usage, run `top -o %MEM`. One can apparently also use `<` and `>` inside top to change which column to sort by, but I'm finding that the
column used for sorting is not necessarily highlighted, so using `-o` might be easier. Maybe `htop` is nicer for this?

There are apparently more options detailed in [https://stackoverflow.com/questions/131303/how-can-i-measure-the-actual-memory-usage-of-an-application-or-process](https://stackoverflow.com/questions/131303/how-can-i-measure-the-actual-memory-usage-of-an-application-or-process)
Perhaps I should give this a read someday, though for now, I'll just note the link.

# SSH Notes
## Running things when logged off
Probably best to use `screen` for this.

Most of the above come from Red Hat's tutorial at [https://www.redhat.com/en/blog/tips-using-screen](https://www.redhat.com/en/blog/tips-using-screen).

- Start: Just type `screen`.
    - To give it a name, use `screen -S "my screen name"
- Leave screen without stopping it: Ctrl+A, then D.
- List all current screen sessions: `screen -ls`
- Rejoin after leaving (if single session): `screen -x`
    - If there are multiple to pick from, need to put "enough" of the identifier (as seen in `screen -ls`) as required to be unique, e.g., `screen -x 257`.
    - If we named it earlier, we can use the name now: `screen -x "my screen name"
- Split screen: Ctrl+A and then `|` (that is, the pipe character).
    - After splitting screens, can use Ctrl+A then Tab to switch.
        - I guess there's no prompt yet after this? If so, do Ctrl+A then C.
    - Can repeat to get more (e.g., 3) side-by-side split screens.

To fully quit a session (not just leave with it still running), you can run `exit` inside the session.
If you can't do that for whatever reason, you can try `screen -XS <session-id> quit`.