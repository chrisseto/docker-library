FROM openjdk:8

# Configuration variables.
ENV JIRA_HOME     /var/atlassian/jira
ENV JIRA_INSTALL  /opt/atlassian/jira
ENV JIRA_VERSION  7.5.1

# explicitly set user/group IDs
RUN groupadd -r jira --gid=999 && useradd -r -g jira --uid=999 jira

RUN set -ex; \
	if ! command -v gpg > /dev/null; then \
		apt-get update; \
		apt-get install -y --no-install-recommends \
			gnupg2 \
			dirmngr \
		; \
		rm -rf /var/lib/apt/lists/*; \
	fi

# Grab gosu for easy step-down from root
ENV GOSU_VERSION 1.10
RUN set -x \
    && apt-get update \
    && apt-get install -y curl \
    && gpg --keyserver pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && curl -o /usr/local/bin/gosu -SL "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
  	&& curl -o /usr/local/bin/gosu.asc -SL "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
  	&& gpg --verify /usr/local/bin/gosu.asc \
  	&& rm /usr/local/bin/gosu.asc \
  	&& chmod +x /usr/local/bin/gosu \
    && gosu nobody true \
    && apt-get clean \
    && apt-get autoremove -y curl \
    && rm -rf /var/lib/apt/lists/*

# Install Atlassian JIRA and helper tools and setup initial home
# directory structure.
RUN set -x \
    && echo "deb http://ftp.debian.org/debian jessie-backports main" > /etc/apt/sources.list.d/jessie-backports.list \
    && apt-get update --quiet \
    && apt-get install --quiet --yes --no-install-recommends curl ca-certificates xmlstarlet \
    && apt-get install --quiet --yes --no-install-recommends -t jessie-backports libtcnative-1 \
    && apt-get clean

RUN set -x \
    && mkdir -p                "${JIRA_HOME}" \
    && mkdir -p                "${JIRA_HOME}/caches/indexes" \
    && chmod -R 700            "${JIRA_HOME}" \
    && chown -R jira:jira      "${JIRA_HOME}" \
    && mkdir -p                "${JIRA_INSTALL}/conf/Catalina"

RUN set -x \
    && curl -Ls                "https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-${JIRA_VERSION}.tar.gz" | tar -xz --directory "${JIRA_INSTALL}" --strip-components=1 --no-same-owner \
    && curl -Ls                "https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.38.tar.gz" | tar -xz --directory "${JIRA_INSTALL}/lib" --strip-components=1 --no-same-owner "mysql-connector-java-5.1.38/mysql-connector-java-5.1.38-bin.jar" \
    && rm -f                   "${JIRA_INSTALL}/lib/postgresql-9.1-903.jdbc4-atlassian-hosted.jar" \
    && curl -Ls                "https://jdbc.postgresql.org/download/postgresql-9.4.1212.jar" -o "${JIRA_INSTALL}/lib/postgresql-9.4.1212.jar" \
    && chmod -R 700            "${JIRA_INSTALL}/conf" \
    && chmod -R 700            "${JIRA_INSTALL}/logs" \
    && chmod -R 700            "${JIRA_INSTALL}/temp" \
    && chmod -R 700            "${JIRA_INSTALL}/work" \
    && chown -R jira:jira      "${JIRA_INSTALL}/conf" \
    && chown -R jira:jira      "${JIRA_INSTALL}/logs" \
    && chown -R jira:jira      "${JIRA_INSTALL}/temp" \
    && chown -R jira:jira      "${JIRA_INSTALL}/work" \
    && sed --in-place          "s/java version/openjdk version/g" "${JIRA_INSTALL}/bin/check-java.sh" \
    && echo -e                 "\njira.home=$JIRA_HOME" >> "${JIRA_INSTALL}/atlassian-jira/WEB-INF/classes/jira-application.properties" \
    && touch -d "@0"           "${JIRA_INSTALL}/conf/server.xml"

# Expose default HTTP connector port.
EXPOSE 8080

# Set volume mount points for installation and home directory. Changes to the
# home directory needs to be persisted as well as parts of the installation
# directory due to eg. logs.
VOLUME /var/atlassian/jira

# Set the default working directory as the home directory.
WORKDIR /var/atlassian/jira

COPY entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]

# Run Atlassian JIRA as a foreground process by default.
CMD ["/opt/atlassian/jira/bin/catalina.sh", "run"]
