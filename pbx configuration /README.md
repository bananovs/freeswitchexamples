#PBX freeswitch Configuration 

## Preprocessor Variables in vars.xml

These variables are important because  configuration strings must be consistent across modules.


The following variables are set dynamically - calculated if possible by freeswitch - and are available to the config as $${variable}.  Can be checkd from fs_cli by cmd eval $${variable}

hostname
local_ip_v4
local_mask_v4
local_ip_v6
switch_serial
base_dir
recordings_dir
sound_prefix
sounds_dir
conf_dir
log_dir
run_dir
db_dir
mod_dir
htdocs_dir
script_dir
temp_dir
grammar_dir
certs_dir
storage_dir
cache_dir
core_uuid
zrtp_enabled
nat_public_addr
nat_private_addr
nat_type


**bind_server_ip**

only used for dialing. Can be an ip address, a dns name, or "auto".
This determines an ip address available on this host to bind.
If you are separating RTP and SIP traffic, you will want to have use different addresses where this variable appears.
Used by: dingaling.conf.xml
```
<X-PRE-PROCESS cmd="set" data="bind_server_ip=54.152.177.149"/>
```

**external_rtp_ip**
       Can be an one of:
           ip address: "12.34.56.78"
           a stun server lookup: "stun:stun.server.com"
           a DNS name: "host:host.server.com"
       where fs.mydomain.com is a DNS A record-useful when fs is on
       a dynamic IP address, and uses a dynamic DNS updater.
       If unspecified, the bind_server_ip value is used.
       Used by: sofia.conf.xml dingaling.conf.xml
```
<X-PRE-PROCESS cmd="set" data="external_rtp_ip=54.152.177.149"/>
```

**external_sip_ip**
      Used as the public IP address for SDP.
       Can be an one of:
           ip address: "12.34.56.78"
           a stun server lookup: "stun:stun.server.com"
           a DNS name: "host:host.server.com"
       where fs.mydomain.com is a DNS A record-useful when fs is on
       a dynamic IP address, and uses a dynamic DNS updater.
       If unspecified, the bind_server_ip value is used.
       Used by: sofia.conf.xml dingaling.conf.xml
```
<X-PRE-PROCESS cmd="set" data="external_sip_ip=54.152.177.149"/>
```

**SIP and TLS settings**
TLS versions
valid options: sslv2,sslv3,sslv23,tlsv1,tlsv1.1,tlsv1.2
default: tlsv1,tlsv1.1,tlsv1.2
```
<X-PRE-PROCESS cmd="set" data="sip_tls_version=tlsv1,tlsv1.1,tlsv1.2"/>
```
TLS cipher suite 
default ALL:!ADH:!LOW:!EXP:!MD5:@STRENGTH
The actual ciphers supported will change per platform.
openssl ciphers -v 'ALL:!ADH:!LOW:!EXP:!MD5:@STRENGTH'
Will show what is available in your verion of openssl.
```
<X-PRE-PROCESS cmd="set" data="sip_tls_ciphers=ALL:!ADH:!LOW:!EXP:!MD5:@STRENGTH"/>
```

**Internal SIP Profile variables**
```
<X-PRE-PROCESS cmd="set" data="internal_auth_calls=true"/>
<X-PRE-PROCESS cmd="set" data="internal_sip_port=5060"/>
<X-PRE-PROCESS cmd="set" data="internal_tls_port=5061"/>
<X-PRE-PROCESS cmd="set" data="internal_ssl_enable=false"/>
```

**External SIP Profile varibles**
```
  <X-PRE-PROCESS cmd="set" data="external_auth_calls=false"/>
  <X-PRE-PROCESS cmd="set" data="external_sip_port=5080"/>
  <X-PRE-PROCESS cmd="set" data="external_tls_port=5081"/>
  <X-PRE-PROCESS cmd="set" data="external_ssl_enable=false"/>
````

**Video Settings**
Setting the max bandwdith
```
  <X-PRE-PROCESS cmd="set" data="rtp_video_max_bandwidth_in=1mb"/>
  <X-PRE-PROCESS cmd="set" data="rtp_video_max_bandwidth_out=1mb"/>
```
WebRTC Video 
Suppress CNG for WebRTC Audio
```
<X-PRE-PROCESS cmd="set" data="suppress_cng=true"/>
```
Enable liberal DTMF
```
<X-PRE-PROCESS cmd="set" data="rtp_liberal_dtmf=true"/>
```  
Stock Video Avatars
```
<X-PRE-PROCESS cmd="set" data="video_mute_png=$${images_dir}/default-mute.png"/>
<X-PRE-PROCESS cmd="set" data="video_no_avatar_png=$${images_dir}/default-avatar.png"/>
```


outbound_caller_id and outbound_caller_name
       The caller ID telephone number we should use when calling out.
       Used by: conference.conf.xml and user directory for default
       outbound callerid name and number.

  <X-PRE-PROCESS cmd="set" data="outbound_caller_name=FreeSWITCH"/>
  <X-PRE-PROCESS cmd="set" data="outbound_caller_id=0000000000"/>


## sip profile 

Inject delay between dtmf digits on send to help some slow interpreters (also per channel with rtp_digit_delay var
```
 <param name="rtp-digit-delay" value="40"/>
```
When calls are in no media this will bring them back to media when you press the hold button.
```
<param name="media-option" value="resume-media-on-hold"/>
```

allow a call after an attended transfer go back to
        bypass media after an attended transfer.

        <param name="media-option" value="bypass-media-after-att-xfer"/>

         set to "_undef_" to remove the User-Agent header

         <param name="user-agent-string" value="FreeSWITCH Rocks!"/>

### watchdogs 

 Sometimes, in extremely rare edge cases, the Sofia SIP stack may stop
        responding. These options allow you to enable and control a watchdog
        on the Sofia SIP stack so that if it stops responding for the
        specified number of milliseconds, it will cause FreeSWITCH to crash
        immediately. This is useful if you run in an HA environment and
        need to ensure automated recovery from such a condition. Note that if
        your server is idle a lot, the watchdog may fire due to not receiving
        any SIP messages. Thus, if you expect your system to be idle, you
        should leave the watchdog disabled. It can be toggled on and off
        through the FreeSWITCH CLI either on an individual profile basis or
        globally for all profiles. So, if you run in an HA environment with a
        master and slave, you should use the CLI to make sure the watchdog is
        only enabled on the master.
        If such crash occurs, FreeSWITCH will dump core if allowed. The
        stacktrace will include function watchdog_triggered_abort().
    <param name="watchdog-enabled" value="no"/>
    <param name="watchdog-step-timeout" value="30000"/>
    <param name="watchdog-event-timeout" value="30000"/>

### TLS

    <!-- TLS: disabled by default, set to "true" to enable -->
    <param name="tls" value="$${internal_ssl_enable}"/>
    <!-- Set to true to not bind on the normal sip-port but only on the TLS port -->
    <param name="tls-only" value="false"/>
    <!-- additional bind parameters for TLS -->
    <param name="tls-bind-params" value="transport=tls"/>
    <!-- Port to listen on for TLS requests. (5061 will be used if unspecified) -->
    <param name="tls-sip-port" value="$${internal_tls_port}"/>
    <!-- Location of the agent.pem and cafile.pem ssl certificates (needed for TLS server) -->
    <!--<param name="tls-cert-dir" value=""/>-->
    <!-- Optionally set the passphrase password used by openSSL to encrypt/decrypt TLS private key files -->
    <param name="tls-passphrase" value=""/>
    <!-- Verify the date on TLS certificates -->
    <param name="tls-verify-date" value="true"/>
    <!-- TLS verify policy, when registering/inviting gateways with other servers (outbound) or handling inbound registration/invite requests how should we verify their certificate -->
    <!-- set to 'in' to only verify incoming connections, 'out' to only verify outgoing connections, 'all' to verify all connections, also 'subjects_in', 'subjects_out' and 'subjects_all' for subject validation. Multiple policies can be split with a '|' pipe -->
    <param name="tls-verify-policy" value="none"/>
    <!-- Certificate max verify depth to use for validating peer TLS certificates when the verify policy is not none -->
    <param name="tls-verify-depth" value="2"/>
    <!-- If the tls-verify-policy is set to subjects_all or subjects_in this sets which subjects are allowed, multiple subjects can be split with a '|' pipe -->
    <param name="tls-verify-in-subjects" value=""/>
    <!-- TLS version default: tlsv1,tlsv1.1,tlsv1.2 -->
    <param name="tls-version" value="$${sip_tls_version}"/>

    <!-- TLS ciphers default: ALL:!ADH:!LOW:!EXP:!MD5:@STRENGTH  -->
    <param name="tls-ciphers" value="$${sip_tls_ciphers}"/>

## directory.xml

When freeswitch gets a register packet it looks for the user in the directory based on the from or to domain in the packet depending on how your sofia profile is configured.  

Out of the box the default domain will be the IP address of the machine running FreeSWITCH.  This IP can be found by typing "sofia status" at the CLI.  

You will register your phones to the IP and not the hostname by default.
    If you wish to register using the domain please open vars.xml in the root conf
    directory and set the default domain to the hostname you desire.  Then you would
    use the domain name in the client instead of the IP address to register
    with FreeSWITCH.

## debugging support 

Issues 

Error Creating SIP UA for profile: external-ipv6 (sip:mod_sofia@[fe80::15:a8ff:fec0:13bc]:5080;transport=udp,tcp) ATTEMPT 2 (RETRY IN 5 SEC)
Error Creating SIP UA for profile: external (sip:mod_sofia@3.218.248.139:5080;transport=udp,tcp) ATTEMPT 2 (RETRY IN 5 SEC)
Error Creating SIP UA for profile: internal-ipv6 (sip:mod_sofia@[fe80::15:a8ff:fec0:13bc]:5060;transport=udp,tcp) ATTEMPT 2 (RETRY IN 5 SEC)
Error Creating SIP UA for profile: external-ipv6 (sip:mod_sofia@[fe80::15:a8ff:fec0:13bc]:5080;transport=udp,tcp) ATTEMPT 3 (RETRY IN 5 SEC)
Error Creating SIP UA for profile: external-ipv6 (sip:mod_sofia@[fe80::15:a8ff:fec0:13bc]:5080;transport=udp,tcp)
The likely causes for this are:
1) Another application is already listening on the specified address.
2) The IP the profile is attempting to bind to is not local to this system.
2019-08-01 07:53:35.454366 [ERR] sofia.c:3228 Error Creating SIP UA for profile: external (sip:mod_sofia@3.218.248.139:5080;transport=udp,tcp) ATTEMPT 3 (RETRY IN 5 SEC)
2019-08-01 07:53:35.454366 [ERR] sofia.c:3238 Error Creating SIP UA for profile: external (sip:mod_sofia@3.218.248.139:5080;transport=udp,tcp)
The likely causes for this are:
1) Another application is already listening on the specified address.
2) The IP the profile is attempting to bind to is not local to this system.
2019-08-01 07:53:35.454366 [ERR] sofia.c:3228 Error Creating SIP UA for profile: internal-ipv6 (sip:mod_sofia@[fe80::15:a8ff:fec0:13bc]:5060;transport=udp,tcp) ATTEMPT 3 (RETRY IN 5 SEC)
2019-08-01 07:53:35.454366 [ERR] sofia.c:3238 Error Creating SIP UA for profile: internal-ipv6 (sip:mod_sofia@[fe80::15:a8ff:fec0:13bc]:5060;transport=udp,tcp)
The likely causes for this are:
1) Another application is already listening on the specified address.
2) The IP the profile is attempting to bind to is not local to this system.

