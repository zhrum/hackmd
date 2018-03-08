FROM node:boron 
ENV DEBIAN_FRONTEND noninteractive 
ENV DOCKERIZE_VERSION v0.3.0 
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ wheezy-pgdg main" > /etc/apt/sources.list.d/pgdg.list && \
	wget -O - http://apt.postgresql.org/pub/repos/apt/ACCC4CF8.asc | apt-key add - && \
	apt-get update && \
	apt-get install -y git postgresql-client-9.6 build-essential && \
	wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
	tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
	rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

# source
RUN mkdir /home/hackmd
WORKDIR /home/hackmd
RUN git clone https://github.com/hackmdio/hackmd.git /home/hackmd
# npm, deps 
RUN npm install
# add config 
ADD config.json /home/hackmd/config.json
ADD .sequelizerc /home/hackmd/.sequelizerc
ADD docker-entrypoint.sh /home/hackmd/docker-entrypoint.sh
RUN chmod +x /home/hackmd/docker-entrypoint.sh
# build front-end bundle
RUN npm run build
# remove dev dependencies 
RUN npm prune --production
# remove build dependencies 
RUN apt-get remove -y --auto-remove build-essential && \
	apt-get clean && apt-get purge && rm -r /var/lib/apt/lists/*
# add user
RUN useradd hackmd --uid 3333 -g 0 --home /home/hackmd
RUN chown -R hackmd /home/hackmd 
RUN chmod -R g+rwX /home/hackmd
USER hackmd
EXPOSE 3000
CMD ["/bin/bash", "-c", "dockerize -wait tcp://hackmdPostgres:5432 -timeout 30s /home/hackmd/docker-entrypoint.sh"]
