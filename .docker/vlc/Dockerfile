ARG BASE_IMAGE=m1k1o/neko:base
FROM $BASE_IMAGE

#
# install vlc and yt-dlp
RUN set -eux; \
    echo "deb http://deb.debian.org/debian bullseye-backports main" >> /etc/apt/sources.list; \
    apt-get update; \
    apt-get install -y --no-install-recommends openbox vlc yt-dlp; \
    #
    # clean up
    apt-get clean -y; \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/*

# install vlc yt-dlp plugin
# https://www.remlab.net/vlc-plugin-ytdl/
COPY ytdlp/libytdl_plugin.so /usr/lib/x86_64-linux-gnu/vlc/plugins/stream_filter/libytdl_plugin.so
COPY ytdlp/ytdl-extract.py /usr/share/vlc/ytdl-extract.py

RUN chmod 0775 /usr/lib/x86_64-linux-gnu/vlc/plugins/stream_filter/libytdl_plugin.so; \
    chmod 0775 /usr/share/vlc/ytdl-extract.py; \
    rm /usr/lib/x86_64-linux-gnu/vlc/lua/playlist/youtube.luac

ENV VLC_MEDIA="/media"

#
# copy configuation files
COPY supervisord.conf /etc/neko/supervisord/vlc.conf
COPY openbox.xml /etc/neko/openbox.xml
