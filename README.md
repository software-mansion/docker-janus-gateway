# Janus Gateway - Docker Image

This repository contains a scripts necessary to build a Docker image of the
[Janus Gateway](https://janus.conf.meetecho.com/).

It aims to create a high-quality image that allows to run the gateway. By high
quality we mean in particular:

- being able to fully configure Janus Gateway over environment variables,
- mitigating issues that might arise if other dependentn services such as RabbitMQ
  are not ready upon Janus boot time,
- being able to easily build custom plugins on top of this image,
- being able to monitor the gateway.

# Status

This is work in progress.

# Versioning scheme

The image is named `swmansion/janus-gateway:JANUS_VERSION-REVISION`, where revision is
being incremented from 0.

# Building

```
docker build -t swmansion/janus-gateway:0.10.9-2 .
```

# Usage

## Sample

```
docker run --rm \
 -e GATEWAY_IP=192.168.0.123 \
 -e STUN_SERVER=stun.l.google.com \
 -e STUN_PORT=19302 \
 -e WEBSOCKETS_ENABLED=true \
 -e RTP_PORT_RANGE=10000-10099 \
 -p 8188:8188 \
 -p 10000-10099:10000-10099/udp \
 -ti swmansion/janus-gateway:0.10.9-2
```

(replace 192.168.0.123 with your IP)

## Environment variables

Some of the environment variable names or default values were altered compared
to the original configuration files in order to provide more consistent usage.

### Gateway itself

#### General

- SERVER_NAME - String that identifies this particular Janus instance (default=MyJanusInstance)

#### Recordings

- RECORDINGS_TMP_EXT - if set, adds an extension to temporary recording files (e.g. for "tmp" unfinished recordings are saved with ".mjr.tmp" extension)
- RECORDINGS_DIR - a directory that should be available for Janus to write recordings. WARNING - the directory has to be provided each time you create the room ("rec_dir" property in "create" request)!

#### Authentication

- ADMIN_SECRET - String that all Janus requests must contain to be accepted/authorized by the admin/monitor (default=janusoverlord).

#### Logging

- DEBUG_LEVEL - Log level, 0-7 (default=4),
- DEBUG_COLORS - Whether colors should be disabled in the log (default=true),
- DEBUG_LOCKS - Whether to enable debugging of locks (very verbose!, default=false)

### Media-related

- MIN_NACK_QUEUE - the minimum size of the NACK queue (in ms, defaults to 200ms) for retransmissions no matter the RTT
- RTP_PORT_RANGE - the range of ports to use for RTP and RTCP (default=10000-20000)
- DTLS_MTU - the starting MTU for DTLS (1200 by default, it adapts automatically)
- NO_MEDIA_TIMER - how much time, in seconds, should pass with no media (audio or video) being received before Janus notifies you about this (default=1s, 0 disables these events entirely)
- SLOWLINK_THRESHOLD - how many lost packets should trigger a 'slowlink' event to users (default=4)
- TWCC_PERIOD - how often, in milliseconds, to send the Transport Wide Congestion Control feedback information back to senders, if negotiated (default=200ms)

### NAT

- GATEWAY_IP - IP address to use, set it to the IP of the Docker host or if you're using 1:1 NAT (e.g. Amazon EC2) to the public IP of the machine.
- STUN_SERVER - STUN server address to use
- STUN_PORT - STUN server port to use
- NICE_DEBUG - debug ICE (default=false)
- FULL_TRICKLE - whether Janus should do Full ICE Trickle (default=true)
- ICE_LITE - whether Janus should enable ICE-Lite (default=false)
- ICE_TCP - whether Janus should enable ICE-TCP (default=false)
- ICE_ENFORCE_LIST - choose which interfaces should be explicitly used by the gateway for the purpose of ICE candidates gathering (e.g. "eth0,192.168.0.1"), empty by default
- ICE_IGNORE_LIST - choose which interfaces should be ignored by the gateway for the purpose of ICE candidates gathering (e.g. "eth0,192.168.0.1"), (default=vmnet)

### Loaded libraries

- DISABLED_PLUGINS - disables plugins, e.g. "libjanus_voicemail.so,libjanus_recordplay.so"
- DISABLED_TRANSPORTS - disables transport libraries (default="libjanus_pfunix.so")
- DISABLED_LOGGERS - disables logging plugins, e.g. "libjanus_jsonlog.so" (default = "")
- DISABLED_EVENT_HANDLERS - disabled event handler libs, e.g. "libjanus_sampleevh.so"

### RabbitMQ

See [janus.transport.rabbitmq.jcfg.sample](https://github.com/meetecho/janus-gateway/blob/v0.10.9/conf/janus.transport.rabbitmq.jcfg.sample)
in Janus Gateway's source code for detailed explanation of the parameters.

- RABBITMQ_ENABLED - Whether the support must be enabled (default=false),
- RABBITMQ_EVENTS - Whether to notify event handlers about transport events (default=true),
- RABBITMQ_JSON - Whether the JSON messages should be indented (default), plain (no indentation) or compact (no indentation and no spaces),
- RABBITMQ_HOST - The address of the RabbitMQ server,
- RABBITMQ_PORT - The port of the RabbitMQ server (5672 by default)
- RABBITMQ_USERNAME - Username to use to authenticate, if needed
- RABBITMQ_PASSWORD - Password to use to authenticate, if needed
- RABBITMQ_VHOST - Virtual host to specify when logging in, if needed
- RABBITMQ_TO_JANUS - Name of the queue for incoming messages (default=to-janus)
- RABBITMQ_FROM_JANUS - Name of the queue for outgoing messages (default=from-janus)
- RABBITMQ_EXCHANGE - Exchange for outgoing messages, using default if not provided (default=wembrane)
- RABBITMQ_EXCHANGE_TYPE - Exchange for outgoing messages, using default if not provided (default=direct)
- RABBITMQ_SSL_ENABLED - Whether ssl support must be enabled
- RABBITMQ_SSL_VERIFY_PEER - Whether peer verification must be enabled
- RABBITMQ_SSL_VERIFY_HOSTNAME - Whether hostname verification must be enabled
- RABBITMQ_SSL_CACERT - Path to cacert.pem
- RABBITMQ_SSL_CERT - Path to cert.pem
- RABBITMQ_SSL_KEY - Path to key.pem
- RABBITMQ_ADMIN_ENABLED - Whether the support must be enabled
- RABBITMQ_TO_JANUS_ADMIN - Name of the queue for incoming messages
- RABBITMQ_FROM_JANUS_ADMIN - Name of the queue for outgoing messages

- RABBITMQ_EVENTHANDLER_ENABLED - Whether events should be delivered over RabbitMQ (default=false)
- RABBITMQ_EVENTHANDLER_EVENTS - Comma separated list of the events mask you're interested in. Valid values are none, sessions, handles, jsep, webrtc, media, plugins, transports, core, external and all. By default we subscribe to everything (all)
- RABBITMQ_EVENTHANDLER_GROUPING - Whether events should be sent individually , or if it's ok to group them. The default is 'true' to limit the number of messages
- RABBITMQ_EVENTHANDLER_JSON - Whether the JSON messages should be indented (default), plain (no indentation) or compact (no indentation and no spaces
- RABBITMQ_EVENTHANDLER_HOST - The address of the RabbitMQ server,
- RABBITMQ_EVENTHANDLER_PORT - The port of the RabbitMQ server (5672 by default)
- RABBITMQ_EVENTHANDLER_USERNAME - Username to use to authenticate, if needed
- RABBITMQ_EVENTHANDLER_PASSWORD - Password to use to authenticate, if needed
- RABBITMQ_EVENTHANDLER_VHOST - Virtual host to specify when logging in, if needed
- RABBITMQ_EVENTHANDLER_ROUTE_KEY - Name of the queue for incoming messages (default=from-janus-event)
- RABBITMQ_EVENTHANDLER_EXCHANGE - Exchange for outgoing messages, using default if not provided (default=wembrane)
- RABBITMQ_EVENTHANDLER_EXCHANGE_TYPE - Exchange type for outgoing messages, using default if not provided (default=direct)
- RABBITMQ_EVENTHANDLER_SSL_ENABLED - Whether ssl support must be enabled
- RABBITMQ_EVENTHANDLER_SSL_VERIFY_PEER - Whether peer verification must be enabled
- RABBITMQ_EVENTHANDLER_SSL_VERIFY_HOSTNAME - Whether hostname verification must be enabled
- RABBITMQ_EVENTHANDLER_SSL_CACERT - Path to cacert.pem
- RABBITMQ_EVENTHANDLER_SSL_CERT - Path to cert.pem
- RABBITMQ_EVENTHANDLER_SSL_KEY - Path to key.pem

### GELF

- GELF_EVENTHANDLER_ENABLED - Whether events should be delivered over GELF (default=false)
- GELF_EVENTHANDLER_EVENTS - Comma separated list of the events mask you're interested in. Valid values are none, sessions, handles, jsep, webrtc, media, plugins, transports, core, external and all. By default we subscribe to everything (all)
- GELF_EVENTHANDLER_BACKEND - Address of the Graylog server
- GELF_EVENTHANDLER_PORT - Port of the Graylog server
- GELF_EVENTHANDLER_PROTOCOL - tcp or udp transport type (tcp by default, although udp is recommended)
- GELF_EVENTHANDLER_MAX_MESSAGE_LEN - Note that we add 12 bytes of headers + standard UDP headers (8 bytes), when calculating packet size based on MTU (1024 by default)
- GELF_EVENTHANDLER_COMPRESS - Optionally, only for UDP transport, JSON messages can be compressed using zlib (true by default)
- GELF_EVENTHANDLER_COMPRESSION - In case of compression, you can specify the compression factor, where 1 is the fastest (low compression), and 9 gives the best compression

### WebSockets

See [janus.transport.websockets.jcfg.sample](https://github.com/meetecho/janus-gateway/blob/v0.10.9/conf/janus.transport.websockets.jcfg.sample)
in Janus Gateway's source code for detailed explanation of the parameters.

- WEBSOCKETS_ENABLED - Whether to enable the WebSockets API (default=false)
- WEBSOCKETS_EVENTS - Whether to notify event handlers about transport events (default=true)
- WEBSOCKETS_JSON - Whether the JSON messages should be indented (default), plain (no indentation) or compact (no indentation and no spaces),
- WEBSOCKETS_PINGPONG_TRIGGER - After how many seconds of idle, a PING should be sent
- WEBSOCKETS_PINGPONG_TIMEOUT - After how many seconds of not getting a PONG, a timeout should be detected
- WEBSOCKETS_PORT - WebSockets server port
- WEBSOCKETS_INTERFACE - Whether we should bind this server to a specific interface only (FIXME: currently has no effect)
- WEBSOCKETS_IP - Whether we should bind this server to a specific IP address only (FIXME: currently has no effect)
- WEBSOCKETS_SSL_ENABLED - Whether to enable secure WebSockets
- WEBSOCKETS_SSL_PORT - WebSockets server secure port, if enabled
- WEBSOCKETS_SSL_INTERFACE - Whether we should bind this server to a specific interface only (FIXME: currently has no effect)
- WEBSOCKETS_SSL_IP - Whether we should bind this server to a specific interface only (FIXME: currently has no effect)
- WEBSOCKETS_ACL - Only allow requests coming from this comma separated list of addresses (FIXME: currently has no effect)
- WEBSOCKETS_LOGGING - libwebsockets debugging level as a comma separated list of things to debug, supported values: err, warn, notice, info, debug, parser, header, ext, client, latency, user, count (plus 'none' and 'all')
- WEBSOCKETS_ADMIN_ENABLED - Whether to enable the Admin API WebSockets API (default=false)
- WEBSOCKETS_ADMIN_PORT - Admin API WebSockets server port, if enabled
- WEBSOCKETS_ADMIN_INTERFACE - Whether we should bind this server to a specific interface only (FIXME: currently has no effect)
- WEBSOCKETS_ADMIN_IP - Whether we should bind this server to a specific IP address only (FIXME: currently has no effect)
- WEBSOCKETS_ADMIN_SSL_ENABLED - Whether to enable the Admin API secure WebSockets
- WEBSOCKETS_ADMIN_SSL_PORT - Admin API WebSockets server secure port, if enabled
- WEBSOCKETS_ADMIN_SSL_INTERFACE - Whether we should bind this server to a specific interface only (FIXME: currently has no effect)
- WEBSOCKETS_ADMIN_SSL_IP - Whether we should bind this server to a specific IP address only (FIXME: currently has no effect)
- WEBSOCKETS_ADMIN_ACL - Only allow requests coming from this comma separated list of addresses (FIXME: currently has no effect)
- WEBSOCKETS_SSL_CERT_PEM - If SSL is enabled, path to certificate's PEM file
- WEBSOCKETS_SSL_CERT_KEY - If SSL is enabled, path to certificate's key file
- WEBSOCKETS_SSL_CERT_PWD - If SSL is enabled, certificate's passphrase

### Plugins

#### VideoRoom

- PLUGIN_VIDEOROOM_ADMIN_KEY - If set, rooms can be created via API only if this key is provided in the request (default=supersecret)
- PLUGIN_VIDEOROOM_EVENTS - Whether events should be sent to event handlers (default=true)
- PLUGIN_VIDEOROOM_LOCK_RTP_FORWARD - Whether the admin_key above should be enforced for RTP forwarding requests too (default=true)
- PLUGIN_VIDEOROOM_STRING_IDS - By default, integers are used as a unique ID for both rooms and participants. In case you want to use strings instead (e.g., a UUID), set string_ids to true. (default=false)
- PLUGIN_VIDEOROOM_EXTRA - An extra config that will be appended to the config file, with rooms config

```
#room-<unique room ID>: {
# description = This is my awesome room
# is_private = true|false (whether this room should be in the public list, default=true)
# secret = <optional password needed for manipulating (e.g. destroying) the room>
# pin = <optional password needed for joining the room>
# require_pvtid = true|false (whether subscriptions are required to provide a valid
#			a valid private_id to associate with a publisher, default=false)
# publishers = <max number of concurrent senders> (e.g., 6 for a video
#              conference or 1 for a webinar)
# bitrate = <max video bitrate for senders> (e.g., 128000)
# bitrate_cap = true|false (whether the above cap should act as a hard limit to
#			dynamic bitrate changes by publishers; default=false, publishers can go beyond that)
# fir_freq = <send a FIR to publishers every fir_freq seconds> (0=disable)
# audiocodec = opus|g722|pcmu|pcma|isac32|isac16 (audio codec(s) to force on publishers, default=opus
#			can be a comma separated list in order of preference, e.g., opus,pcmu)
# videocodec = vp8|vp9|h264 (video codec(s) to force on publishers, default=vp8
#			can be a comma separated list in order of preference, e.g., vp9,vp8,h264)
# opus_fec = true|false (whether inband FEC must be negotiated; only works for Opus, default=false)
# video_svc = true|false (whether SVC support must be enabled; only works for VP9, default=false)
# audiolevel_ext = true|false (whether the ssrc-audio-level RTP extension must
#		be negotiated/used or not for new publishers, default=true)
# audiolevel_event = true|false (whether to emit event to other users or not, default=false)
# audio_active_packets = 100 (number of packets with audio level, default=100, 2 seconds)
# audio_level_average = 25 (average value of audio level, 127=muted, 0='too loud', default=25)
# videoorient_ext = true|false (whether the video-orientation RTP extension must
#		be negotiated/used or not for new publishers, default=true)
# playoutdelay_ext = true|false (whether the playout-delay RTP extension must
#		be negotiated/used or not for new publishers, default=true)
# transport_wide_cc_ext = true|false (whether the transport wide CC RTP extension must be
#		negotiated/used or not for new publishers, default=true)
# record = true|false (whether this room should be recorded, default=false)
# rec_dir = <folder where recordings should be stored, when enabled>
# notify_joining = true|false (optional, whether to notify all participants when a new
#               participant joins the room. The Videoroom plugin by design only notifies
#               new feeds (publishers), and enabling this may result extra notification
#               traffic. This flag is particularly useful when enabled with require_pvtid
#               for admin to manage listening only participants. default=false)
#}
```

## Copyright and License

Copyright 2020, [Software Mansion](https://swmansion.com/?utm_source=git&utm_medium=readme&utm_campaign=docker-janus)

[![Software Mansion](https://logo.swmansion.com/logo?color=white&variant=desktop&width=200&tag=docker-janus-github)](https://swmansion.com/?utm_source=git&utm_medium=readme&utm_campaign=docker-janus)

Please note that underlying software contained in the image might have
different licenses, most importantly Janus Gateway is
[licensed under GPLv3](https://janus.conf.meetecho.com/docs/COPYING.html).

The code located in this repository is licensed under the [Apache License, Version 2.0](LICENSE).
