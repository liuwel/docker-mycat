FROM java:8-jre
MAINTAINER <liuwel liuwel@live.com>
LABEL Description="使用mycat做mysql数据库的读写分离"
ENV mycat-version Mycat-server-1.6.5-release-20180122220033-linux.tar.gz
USER root
COPY ./Mycat-server-1.6.5-release-20180122220033-linux.tar.gz /
RUN tar -zxf /Mycat-server-1.6.5-release-20180122220033-linux.tar.gz
ENV MYCAT_HOME=/mycat
ENV PATH=$PATH:$MYCAT_HOME/bin
WORKDIR $MYCAT_HOME/bin
RUN chmod u+x ./mycat
EXPOSE 8066 9066
CMD ["./mycat","console"]
