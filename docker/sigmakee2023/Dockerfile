FROM ubuntu:latest

# Install required libraries
RUN apt update
RUN apt upgrade -qy
# RUN apt install gcompat
RUN apt install bash
RUN apt install unzip
RUN apt install git -y
RUN apt install make -y
RUN apt install gcc -y
RUN apt install musl-dev -y
RUN apt install graphviz -y
RUN apt install ant -y
RUN apt install zlib1g-dev -y
RUN apt install g++ -y
RUN apt install linux-headers-generic -y
RUN apt install openjdk-8-jdk -y
RUN apt install jedit -y

# Setup home directory
RUN cd /root \
    && mkdir Programs \
    && mkdir workspace \
    && mkdir .sigmakee \
    && cd .sigmakee \
    && mkdir KBs

# Setup Bash
RUN cd /root \
    && touch .bashrc \
    && echo "alias dir='ls --color=auto --format=vertical -la'" >> .bashrc \
    && echo "export HISTSIZE=10000 HISTFILESIZE=100000" >> .bashrc \
    && echo "export SIGMA_HOME=/root/.sigmakee" >> .bashrc \
    && echo "export SIGMA_SRC=/root/workspace/sigmakee" >> .bashrc \
    && echo "export ONTOLOGYPORTAL_GIT=/root/workspace" >> .bashrc \
    && echo 'export CATALINA_OPTS="$CATALINA_OPTS -Xmx10g"' >> .bashrc \
    && echo "export CATALINA_HOME=/root/Programs/apache-tomcat-8.5.23" >> .bashrc \
    && echo "export JEDIT_HOME=/root/.jedit" >> .bashrc \
    && . /root/.bashrc

# Setup Vampire
RUN cd /root/workspace \
    && git clone https://github.com/vprover/vampire \
    && cd vampire \
    && make -j 4 vampire_rel \
    && mv vampire_rel_master* vampire

RUN apt install wget -y
# Setup Apache, E and Wordnet
RUN cd /root/Programs \
    && wget 'https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.23/bin/apache-tomcat-8.5.23.zip' \
    && wget 'http://wordnetcode.princeton.edu/3.0/WordNet-3.0.tar.gz' \
    && wget 'http://wwwlehre.dhbw-stuttgart.de/~sschulz/WORK/E_DOWNLOAD/V_2.0/E.tgz' \
    && tar -xvzf E.tgz \
    && unzip apache-tomcat-8.5.23.zip \
    && rm apache-tomcat-8.5.23.zip

RUN cd /root/Programs/apache-tomcat-8.5.23/bin \
    && chmod 777 *

RUN cd /root/Programs/apache-tomcat-8.5.23/webapps \
    && chmod 777 *

# Setup E
RUN cd /root/Programs/E \
    && ./configure \
    && make -j 4 \
    && make install

# Setup Wordnet
RUN cd /root/Programs \
    && gunzip WordNet-3.0.tar.gz \
    && tar -xvf WordNet-3.0.tar \
    && mkdir /root/.sigmakee/KBs/WordNetMappings \
    && cp WordNet-3.0/dict/* /root/.sigmakee/KBs/WordNetMappings/

# Clone ontologyportal git repos
RUN cd /root/workspace \
    && git clone -b rwu/feature/use_sumo_in_docker https://github.com/ipa-rwu/sigmakee \
    && git clone https://github.com/ontologyportal/sumo \
    && git clone https://github.com/ontologyportal/TPTP-ANTLR \
    && git clone https://github.com/ontologyportal/SigmaUtils \
    && git clone https://github.com/ontologyportal/SUMOjEdit.git

RUN update-java-alternatives --set java-1.8.0-openjdk-amd64
RUN echo "export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")" >> /root/.bashrc

# Setup sigmakee
RUN cd /root/workspace/sigmakee \
    && . /root/.bashrc \
    && ant

COPY jedit_catalog /root/.jedit/modes/catalog

RUN cd /root/workspace/SUMOjEdit \
    && . /root/.bashrc \
    && ant

# Add sumo kif knowledge-bases
RUN cp -R /root/workspace/sumo/* /root/.sigmakee/KBs
COPY kbs /root/.sigmakee/KBs

CMD cd /root && . /root/.bashrc && /root/Programs/apache-tomcat-8.5.23/bin/startup.sh && tail -f /root/Programs/apache-tomcat-8.5.23/logs/catalina.out
