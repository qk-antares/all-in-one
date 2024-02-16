FROM node:18

# 设置工作目录
WORKDIR /root/workplace

# 更新apt源
RUN tee /etc/apt/sources.list <<-'EOF'
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
deb https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
EOF

# 安装必要的包
RUN <<EOF
apt-get update
apt-get upgrade -y
apt-get install -y --no-install-recommends \
    curl \
    zip \
    unzip \
    git \
    vim \
    openssh-server
EOF

# 将本地的 Go Java Maven 压缩包复制到镜像中
COPY go1.21.6.linux-amd64.tar.gz /tmp/go.tar.gz
COPY jdk-21_linux-x64_bin.tar.gz /tmp/jdk.tar.gz
COPY apache-maven-3.9.6.zip /tmp/maven.zip

# 解压并安装Go Java Maven
RUN <<EOF
tar -xzvf /tmp/go.tar.gz -C /
rm /tmp/go.tar.gz
tar -xzvf /tmp/jdk.tar.gz -C /
rm /tmp/jdk.tar.gz
unzip /tmp/maven.zip -d /
rm /tmp/maven.zip
chmod +x /apache-maven-3.9.6/bin/mvn
EOF

# 配置环境变量
ENV GOPROXY https://goproxy.cn,direct
ENV GOPATH /go
ENV JAVA_HOME /jdk-21.0.2
ENV CLASSPATH .:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib
ENV MAVEN_HOME /apache-maven-3.9.6
ENV PATH $PATH:$GOPATH/bin:$JAVA_HOME/bin:$MAVEN_HOME/bin

RUN tee /etc/profile <<-'EOF'
export GOPROXY=https://goproxy.cn,direct
export GOPATH=/go
export JAVA_HOME=/jdk-21.0.2
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib
export MAVEN_HOME=/apache-maven-3.9.6
export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin:$GOPATH/bin
EOF

# 安装 goctl protoc protoc-gen-go protoc-gen-go-grpc
RUN <<EOF
GOPROXY=https://goproxy.cn/,direct go install github.com/zeromicro/go-zero/tools/goctl@latest
curl -L -o /tmp/protoc.zip https://github.com/protocolbuffers/protobuf/releases/download/v3.19.1/protoc-3.19.1-linux-x86_64.zip
unzip -d /tmp/protoc /tmp/protoc.zip
mv /tmp/protoc/bin/protoc $GOPATH/bin
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
EOF

# 配置ssh git npm
RUN <<EOF
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
git config --global user.name "antares"
git config --global user.email "1716607668@qq.com"
npm config set registry http://registry.npmmirror.com
echo 'fs.inotify.max_user_watches=524288' >> /etc/sysctl.conf
EOF

# 配置vim 
RUN tee ~/.vimrc <<-'EOF'
set tabstop=4
syntax enable
if has('mouse')
    set mouse-=a
endif
EOF

# 清理垃圾
RUN <<EOF
apt-get clean
rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
rm /var/log/lastlog /var/log/faillog
EOF

EXPOSE 22