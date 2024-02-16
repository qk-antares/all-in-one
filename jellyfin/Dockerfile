FROM jellyfin/jellyfin:latest

RUN <<EOF
apt update
apt install -y \
    jellyfin-ffmpeg5 \
    fonts-noto-cjk-extra
EOF

# 清理垃圾
RUN <<EOF
apt-get clean
rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
rm /var/log/lastlog /var/log/faillog
EOF