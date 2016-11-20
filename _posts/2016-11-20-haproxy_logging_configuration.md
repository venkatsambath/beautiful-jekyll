---
layout: post
title: Configuring HAProxy Loadbalancer
---

#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local0          # writes event logs
    log         127.0.0.1 local1 notice   # writes service logs

    #chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /tmp/haproxy

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    option                  httplog
    option                  dontlognull
    log			    global
    #option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    #timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000


# enable admin stats at :8000/haproxy?stats
listen admin
	bind *:8000
	stats enable



#
# This sets up the admin page for HA Proxy at port 25002.
#
listen stats
    bind :25002
    balance
    mode http
    stats enable
    stats auth username:password

# This is the setup for Impala. Impala client connect to load_balancer_host:25003 ssl crt /etc/cdep-ssl-conf/CA_STANDARD/cert_key.pem.
# HAProxy will balance connections among the list of servers listed below.
# The list of Impalad is listening at port 21000 ssl ca-file /etc/cdep-ssl-conf/CA_STANDARD/truststore.pem for beeswax (impala-shell) or original ODBC driver.
# For JDBC or ODBC version 2.x driver, use port 21050 ssl ca-file /etc/cdep-ssl-conf/CA_STANDARD/truststore.pem instead of 21000 ssl ca-file /etc/cdep-ssl-conf/CA_STANDARD/truststore.pem.
listen impala
    bind :25003 ssl crt /etc/cdep-ssl-conf/CA_STANDARD/cert_key.pem
    mode tcp
    option tcplog
    balance leastconn

    server impalad0 node1:21000 ssl ca-file /etc/cdep-ssl-conf/CA_STANDARD/truststore.pem
    server impalad1 node2:21000 ssl ca-file /etc/cdep-ssl-conf/CA_STANDARD/truststore.pem
    server impalad2 node3:21000 ssl ca-file /etc/cdep-ssl-conf/CA_STANDARD/truststore.pem

# Setup for Hue or other JDBC-enabled applications.
# In particular, Hue requires sticky sessions.
# The application connects to load_balancer_host:21051 ssl crt /etc/cdep-ssl-conf/CA_STANDARD/cert_key.pem, and HAProxy balances
# connections to the associated hosts, where Impala listens for JDBC
# requests on port 21050 ssl ca-file /etc/cdep-ssl-conf/CA_STANDARD/truststore.pem.
listen impalajdbc
    bind :21051 ssl crt /etc/cdep-ssl-conf/CA_STANDARD/cert_key.pem
    mode tcp
    option tcplog
    balance leastconn
    stick match src
    stick-table type ip size 200k expire 30m

    server impalad0 node1:21050 ssl ca-file /etc/cdep-ssl-conf/CA_STANDARD/truststore.pem
    server impalad1 node2:21050 ssl ca-file /etc/cdep-ssl-conf/CA_STANDARD/truststore.pem
    server impalad2 node3:21050 ssl ca-file /etc/cdep-ssl-conf/CA_STANDARD/truststore.pem


==========================================================================================

cat haproxy.conf 
$ModLoad imudp
$UDPServerRun 514 
$template Haproxy,"%msg%n"
local0.=info -/var/log/haproxy.log;Haproxy
local0.notice -/var/log/haproxy-status.log;Haproxy
### keep logs in localhost ##
local0.* ~ 

