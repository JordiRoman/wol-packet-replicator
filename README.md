# wol-packet-replicator

A very simple python script Cloned from <https://github.com/jaime29010/wol-packet-replicator>

This is a easy way of using Wake on Lan from outside the network _when your router does not support forwarding ports_ to the broadcast address.

It also allows you to enforce the SecureOn if your NIC does not support it.

## Usage

* Run directly

  ```bash
  python replicator.py 
  ```

* Run as sysV service

    ```bash
    $ cat /etc/init.d/wol-replicator
    #!/bin/bash
    #
    # wol-packet-replicator         Start up myservice
    #
    # chkconfig: 2345 55 25
    # description: WoL packet replicator
    #
    # processname: myservice

    # Source function library
    . /etc/init.d/functions

    #the service name, a python script
    SNAME=replicator.py

    #the full path and name of the daemon program
    #Warning: The name of executable file must be identical with service name
    PROG=/usr/bin/$SNAME


    # start function
    start() {
        #check the daemon status first
        if [ -f /var/lock/subsys/$SNAME ]
        then
            echo "$SNAME is already started!"
            exit 0;
        else
            echo $"Starting $SNAME ..."
            python $PROG &
            [ $? -eq 0 ] && touch /var/lock/subsys/$SNAME
            echo $"$SNAME started."
            exit 0;
        fi
    }

    #stop function
    stop() {
        echo "Stopping $SNAME ..."
        pid=`ps -ef | grep '[p]ython $PROG' | awk '{ print $2 }'`
        [ "$pid"X != "X" ] && kill $pid
        rm -rf /var/lock/subsys/$SNAME
    }

    case "$1" in
    start)
    start
    ;;
    stop)
    stop
    ;;
    reload|restart)
    stop
    start
    ;;
    status)
    pid=`ps -ef | grep '[p]ython $PROG' | awk '{ print $2 }'`
    if [ "$pid"X = "X" ]; then
        echo "$SNAME is stoped."
    else
        echo "$SNAME (pid $pid) is running..."
    fi
    ;;
    *)
    echo $"\nUsage: $0 {start|stop|restart|status}"
    exit 1
    esac
    ```

    ```data
    update-rc.d wol-replicator defaults

    /etc/init.d/wol-replicator start

    /etc/init.d/wol-replicator status

    /etc/init.d/wol-replicator stop
    ```

* run as SystemD service

    ```bash
    $ cat /etc/systemd/system/wol-replicator.service
    [Unit]
    Description=wol-replicator
    After=multi-user.target
    
    [Service]
    Type=simple
    Restart=always
    ExecStart=/usr/bin/python /usr/bin/replicator.py
    
    [Install]
    WantedBy=multi-user.target
    ```

    ```bash
    sudo systemctl daemon-reload

    sudo systemctl enable wol-replicator.service

    sudo systemctl start wol-replicator.service
    ```

## Usage Options

./replicator.py [-h] [-i ip] [-p port] [-t ip] [-r port] [-s password] [-z password] [-f mac_and_passwd_json_file] [-l log file]

   -h show this help.
   -i binding ip. Default 0.0.0.0.
   -p binding port. Default 5009.
   -t target ip. Default 255.255.255.255.
   -r target port. Default 9.
   -f json file with mac's and passwords. View format down.
   -s password for use on forward packet. Default ''.
   -z password to test on received packets. Default ''.

The json file for wol with secureOn have the format:

```json
[
  { "ethernet": "112233445566", "password": "aabbccddeeff",
  { "ethernet": "aa2233445566", "password": "11bbccddeeff"  
]
```

Environment Variables:
   LOGLEVEL=[DEBUG|INFO|WARNING|ERROR|CRITICAL] default=INFO

Examples:
     ./replicator.py -h
     ./replicator.py -i 192.168.1.100
     ./replicator.py -p 8000
     ./replicator.py -t 192.168.1.100
     ./replicator.py -r 8000
     ./replicator.py -s 112233445566
     ./replicator.py -f /etc/wol/mac_and_pass.json
     ./replicator.py -z aabbccddeeff
     ./replicator.py -l /var/log/wol_replicator.log
     LOGLEVEL=DEBUG ./replicator.py

## Daisy Chain use

We can use the daemon as a daisy chain. A server configured for sending with _secureOn_ and a second server for receive wol packets with _secureOn_.

```mermaid
graph LR
  WS[WoL Sender]

  WS-->RSS

  RSS[Wol Repeater \nsend with SecureOn]

    RSS === RSR[SecureOn Enabled]

  RSR[Wol Repeater\nreceive with SecureOn]
    
    RSR --> MA

  MA[Final Machines]

```

This is a great way of making sure only you can wake up your devices even if your NIC does not support SecureOn.
