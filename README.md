# remote_syslog Ruby daemon & sender

Lightweight Ruby daemon to tail one or more log files and transmit UDP syslog 
messages to a remote syslog host (centralized log aggregation).

remote_syslog generates UDP packets itself instead of depending on a system 
syslog daemon, so its configuration doesn't affect system-wide 
logging - syslog is just the transport.

Uses:

* collecting logs from servers & daemons which don't natively support syslog
* when reconfiguring the system logger is less convenient than a 
purpose-built daemon (e.g., automated app deployments)
* aggregating files not generated by daemons (e.g., package manager logs)

The library can also be used to generate one-off log messages from Ruby code.

Tested with the hosted log management service [Papertrail] and should work for 
transmitting to any syslog server.


## Installation

Install the gem, which includes a binary called "remote_syslog":

    gem install remote_syslog

Optionally, create a log_files.yml with the log file paths to read and the 
host/port to log to (see examples/log_files.yml.example). These can also be 
specified as arguments to the remote_syslog daemon. More below.


## Usage

    $ remote_syslog -h
    Usage:   remote_syslog [options] <path to add'l log 1> .. <path to add'l log n>

    Example: remote_syslog -c configs/logs.yml -p 12345 /var/log/mysqld.log

    Options:
        -c, --configfile PATH            Path to config (/etc/log_files.yml)
        -d, --dest-host HOSTNAME         Destination syslog hostname or IP (logs.papertrailapp.com)
        -D, --no-detach                  Don't daemonize and detach from the terminal
        -f, --facility FACILITY          Facility (user)
        -p, --dest-port PORT             Destination syslog port (514)
        -P, --pid-dir DIRECTORY          Directory to write .pid file in (/var/run/)
        -s, --severity SEVERITY          Severity (notice)
        -h, --help                       Show this message
    

## Example

Daemonize, collecting from files mentioned in ./config/logs.yml as well as
/var/log/mysqld.log:
    remote_syslog -c configs/logs.yml -p 12345 /var/log/mysqld.log

Stay attached to the terminal, look for and use /etc/log_files.yml if it exists, 
write PID to /tmp/remote_syslog.pid, and send with facility local0:
    remote_syslog -d a.server.com -f local0 -P /tmp /var/log/mysqld.log

remote_syslog will daemonize by default. A sample init file is in the gem as 
remote_syslog.init.d. You may be able to:
    cp examples/remote_syslog.init.d /etc/init.d/remote_syslog


## Configuration

By default, the gem looks for a configuration in /etc/log_files.yml.

The gem comes with a sample config.  Optionally:

    cp examples/log_files.yml.example /etc/log_files.yml
    
log_files.yml has filenames to log from (as an array) and hostname and port 
to log to (as a hash).  Only 1 destination server is supported; the command-line
argument wins.  Filenames given on the command line are additive to those 
in the config file.

    files: [/var/log/httpd/access_log, /var/log/httpd/error_log, /var/log/mysqld.log, /var/run/mysqld/mysqld-slow.log]
    destination:
      host: logs.papertrailapp.com
      port: 12345

## Contribute

Bug report:

1. See whether the issue has already been reported:
   http://github.com/papertrail/remote_syslog/issues/
2. If you don't find one, create an issue with a repro case.

Enhancement or fix:

1. Fork the project:
   http://github.com/papertrail/remote_syslog
2. Make your changes with tests.
3. Commit the changes without changing the Rakefile or other files unrelated 
to your enhancement.
4. Send a pull request.

[Papertrail]: http://papertrailapp.com/