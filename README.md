# reg_hunter
Blueteam operational triage registry hunting/forensic tool.

Thank you to https://twitter.com/Hexacorn and https://twitter.com/SBousseaden for their open research. Many of the explicit registry keys and values defined in this tool came from their graciously shared hard work.

Thanks to https://github.com/lilopkins and https://github.com/gentoo90 for the Lnk and Registry Rust crates.

Output is in JSON line delimited.

If you just want the tool, download the reg_hunter_x32.exe and/or reg_hunter_x64.exe binary. Note that you'll want to run the 64 bit binary on 64 bit OS so that it will not be partially blinded by Windows WOW64 redirection.

Registry key "last_write_time" is included in Registry JSON logs.

The "tags" field is an array populated by any hunts that are a positive match.

I needed a selfcontained tool, as when I'm triaging an event, the less files I have to push to a remote device the better. Adding in new hunts and recompiling is simple as well. I also wanted a tool that was not dependent on having a minimum .Net version installed.

NOTE: The "parent_data_type" field specifies the "data_type" that caused the generation of this data type. E.g. If a Lnk file was found in a registry value, this will generate a "ShellLink" data_type with a parent_data_type of "Registry". Then a data_type of "File" with a parent_data_type of "ShellLink" will be generated if the file that the Lnk file points to is found/exists. I.e. Registry --> ShellLink --> File

A file/lnk's meta data will only be collected once no matter how many times it is referenced in registry values.

```
Reg Hunter
    Author: Brian Kellogg
    License: MIT
    Many thanks: @Hexacorn and @SBousseaden
    Disclaimer:
        This tool comes with no warranty or support.
        If anyone chooses to use it, you accept all responsibility and liability.

Usage:
    reg_hunter --help
    reg_hunter [options]
    reg_hunter --all [-bcefimnorsuwyz] [--ip <ip> --port <port>] [--limit]
    reg_hunter -a [-bn] [--regex <regex> [--path | --name | --value]]

Options:
    Registry context (one required):
        -a, --all                   Examine all the registry; HKLM and HKU
        -x, --explicit              Examine only more often forensically interesting keys and values
                                        This option will always report out all
                                        value names and values unless values are empty/null

    Hunts:
                                        Tag: MzHeader
        -c, --shell                 Find command shells (cmd.exe, powershell.exe, ...)
                                        Tag: Shell
        -e, --encoding              Find possible encoded values
                                        Tag: Encoding
        -f, --file                  Find files referenced in registry values
                                        and collect lnk/file metadata. If a lnk file is found,
                                        metadata on both the lnk and file it points to will be
                                        reported.
                                        Tag: File
        -i, --ip                    Search for IPv4 addresses
                                        Tag: IPv4
        -m, --email                 Find email addresses
                                        Tag: Email
        -n, --null                  Hunt for null prefixed value names
                                        Tag: NullPrefixedName
        -o, --obfuscation           Find obfuscated values
                                        Tag: Obfuscation
        -r, --script                Find script files
                                        Tag: Script
        -s, --shellcode             Find possible shellcode
                                        Tag: Shellcode
        -u, --unc                   Find possible UNC paths
                                        Tag: UNC
        -w, --url                   Find URLs
                                        Tag: URL
        -y, --everything            Run ALL the hunts
        -z, --suspicious            Find various suspicious substrings
                                        e.g. iex, invoke-expression, etc.
                                        Tag: Suspicious

    Time window:
        This option will compare the specified date window to the registry last_write_time
        and only output logs where the last_write_time falls within that window.
        Window start is inclusive, window end is exclusive.
        REMEMBER: key last_write_time can be timestomped.
        --start <UTC_start_time>        Start of time window: [default: 0000-01-01T00:00:00]
                                        format: YYYY-MM-DDTHH:MM:SS
        --end <UTC_end_time>            End of time window: [default: 9999-12-31T23:59:59]
                                        format: YYYY-MM-DDTHH:MM:SS

    Custom hunt (regex required):
        -q, --regex <regex>         Custom regex [default: $^]
                                        Does not support look aheads/behinds/...
                                        Uses Rust regex crate (case insensitive and multiline)
                                        Any match will add 'Custom' to the tags field
                                        Tag: Custom
        -k, --path                  Search reg key path
        -t, --name                  Search value name
        -v, --value                 Search reg value

    Network output:
        -d, --destination <ip>      IP address to send output to [default: NONE]
        -p, --port <port>           Destination port to send output to [default: 80]

    Misc:
        -h, --help                  Show this screen
        -l, --limit                 Try to minimize CPU use as much as possible

Note:
    If not run as an administrator some telemetry cannot be harvested.

    The output is mostly meant to be fed into some hunting backend. But,
    there are some built in hunts; --null, --binary, ...

    Depending on the options used, considerable output can be generated.

    To capture output remotely, start a netcat listener on your port of choice.
    Use the -k option with netcat to prevent netcat from closing after a TCP connection is closed.

    Files larger than 256MB will not be hashed.
```


Example JSON log:
```
{
   "parent_data_type":"",
   "data_type":"Registry",
   "timestamp":"2020-11-18T02:26:05.144",
   "device_name":"DESKTOP-NDPUZZM4",
   "device_domain":"DESKTOP-NDPUZZM4",
   "device_type":"Windows 10",
   "registry_hive":"HKEY_USERS",
   "registry_key":"Volatile",
   "registry_value_name":"MsaDevice",
   "registry_type":"REG_SZ",
   "registry_value":"t=GwAWAbuEBAAUPrSa9Xbh1D0J93uIPuLO4a+WXwAOZgAAEBTGT0K0Z4Yb1yQ+kp9BEdHgANLuAcfHOSjYFFBzGrBrLhP7Tn42DVLHomaP99kfluqc6pesVhV/Pwr486/KC0rhecROAWOfhfLOeIzcCP3ac+7Gd39nLfE3i0XBqwixziztwygu+xEFSlxrHSRLu0Rl1YWZ4rasrpcX+r43oj6PLzuVWtCkwq+mcFMKhjdC9394PnyoO4hh0oPxt9Gk3JZN784wc6D3AKMT8nntlvzhsBpN+nedTBBTzDmqDh3KiZCgGQTghwy/qXV4/wIg/2Hu1XXbe2f1EbymQeQ1+flMSoIzD15JRNDXITeFWljFcGwE=&p=",
   "last_write_time":"2020-11-16T12:36:21.928",
   "tags":[
      "Obfuscation",
      "Encoding"
   ]
}
```
